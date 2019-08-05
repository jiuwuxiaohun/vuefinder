<template>
  <div
    :class="theme"
    class="vuefinder"
    @contextmenu.prevent>

    <tool-bar
      :back="back"
      :selected-items="getSelectedItems()"
      :listview.sync="listview"
      @showMenu="showMenu" @selectBack="selectBack"/>

    <breadcrumb-header
      :root="data.root"
      :dirname="data.dirname"
      :loading="loading"
      @openFolder="openFolder"/>

    <listview-sortbar
      v-show="listview"
      :sort="sort"
      @select="sortItems"/>

    <explorer
      ref="explorer"
      :listview="listview"
      :is-root="data.dirname == data.root"
      :loading="loading"
      @contextmenu.native="showContextMenu($event)"
      @back="openFolder(data.parent)"
      @mousedown.native.alt="selectable.stop()"
      @mousedown.native="selectable.start()"
    >
      <explorer-item
        v-draggable
        v-dropzone
        :url="url"
        :fileBaseUrl="fileBaseUrl"
        v-for="item in sortedFiles"
        ref="files"
        :key="item.path"
        :item="item"
        :listview="listview"
        :draggable="isSelected(item) ? true : false"
        :loading="loading"
        @dblclick.native.stop.prevent="open(item)"
        @contextmenu.native="addContextItems(item)"
        @mouseover.native="hoverText = item.basename"
        @mouseleave.native="hoverText = ''"/>
    </explorer>

    <div class="vuefinder-footer">
      <span>
        {{ hoverText }}
      </span>
      <span class="vuefinder-status-message">
        <!--vuefinder v0.2-->
        文件管理器
      </span>
    </div>

    <context-menu
      v-show="context.active"
      ref="context"
      :context="context"
      @close="hideContextMenu()"/>

    <component
      v-if="modal.active"
      :is="'modal-'+ modal.type"
      :directory="data.dirname"
      :url="url"
      :uploadurl="uploadurl"
      :fileBaseUrl="fileBaseUrl"
      :accept="accept"
      :data="modal.item"
      @close="modal.active = false"
      @error="msgBox"
      @refresh="openFolder"/>

    <drag-image
      ref="dragImage"
      :count="getSelectedItems().length"/>

  </div>
</template>

<script>
import axios from 'axios';
import DragSelect from 'dragselect';

// FontAwesome icons
import { library } from '@fortawesome/fontawesome-svg-core';
import * as IconPack from './utilities/icons';
library.add(...Object.values(IconPack));

import ContextMenu from './components/ContextMenu.vue';
import ToolBar from './components/ToolBar.vue';
import Explorer from './components/Explorer.vue';
import ExplorerItem from './components/ExplorerItem.vue';
import BreadcrumbHeader from './components/BreadcrumbHeader.vue';
import ListviewSortbar from './components/ListviewSortbar.vue';
import DragImage from './components/DragImage.vue';
import * as Modals from './utilities/modals';// Modal Components



export default {
    name: 'Vuefinder',
    components: {
        ToolBar,
        ContextMenu,
        ListviewSortbar,
        BreadcrumbHeader,
        Explorer,
        ExplorerItem,
        DragImage,
        ...Modals
    },
    directives: {
        draggable: {
            bind: function(el, binding, vnode, oldVnode ) {
                el.addEventListener('dragstart', function(e)   {
                    if (! e.altKey) {
                        e.preventDefault();
                        return false;
                    }
                    let img = vnode.context.$refs.dragImage.$el;
                    e.dataTransfer.setDragImage(img,0,15);
                    e.dataTransfer.effectAllowed = 'all';
                    e.dataTransfer.dropEffect = 'copy';
                    e.dataTransfer.setData('data', JSON.stringify(vnode.context.getSelectedItems()));
                });
                el.addEventListener('dragover', function(e)   {
                    if (vnode.context.isSelected(vnode.componentInstance.item) || vnode.componentInstance.item.type !== 'folder') {
                        e.dataTransfer.dropEffect = 'none';
                        e.dataTransfer.effectAllowed = 'none';
                    }
                });
                el.addEventListener('dragend', function(e)   {

                });
            },
            unbind:function(el) {
                //return el.removeEventListener('dragstart');
            }
        },
        dropzone: {
            bind: function(el, binding, vnode, oldVnode){
                el.addEventListener('dragenter', function(e){
                    e.preventDefault();
                });
                el.addEventListener('dragover', function(e){
                    e.preventDefault();

                });
                el.addEventListener('drop', function(e){
                    e.preventDefault();

                    vnode.context.selectable.start();
                    console.log(e.dataTransfer.getData('data') );
                    console.log(vnode.context.getSelectedItems() );

                });
            },
            unbind:function(el) {
                //return  el.removeEventListener('drop');
            }
        }
    },
    props: {
        url: {
            type: String,
            required: true
        },
        httpHeaders:{
          type: Object,
          default:{}
        },
        uploadurl: {
            type: String,
            default: ''
        },
        fileBaseUrl: {
            type: String,
            required: true
        },
        path: {
            type: String,
            default: '.'
        },
        theme: {
            type: String,
            default: 'light'
        },
        back: {
            type: Boolean,
            default: false
        },
        accept: {
            type: String,
            default:'image/gif,image/jpeg,image/jpg,image/png'
        },
    },
    data() {
        return {
            loading: false,
            listview: false,
            hoverText: '',
            selectedItems: [],
            data: {dirname: '.', root: '.'},
            modal: {active: false, type: ''},
            sort: {active: false, column: '', order: ''},
            context: {active: false, positions: {}, items: []}
        };
    },
    computed: {
        sortedFiles: function () {
            let files = this.data.files,
                column = this.sort.column,
                order = this.sort.order == 'asc' ? 1 : -1;

            const compare = (a,b) => {
                if (a < b) return -1;
                if (a > b) return 1;
                return 0;
            };

            if (this.sort.active) {
                files = files.slice().sort((a, b) => {
                    return compare(a[column], b[column]) * order;
                });
            }

            return files;
        }
    },
    mounted() {
      console.log('fileBaseUrl',this.fileBaseUrl);
        this.selectable = new DragSelect({
            area: this.$refs.explorer.$el,
            customStyles: true,
            selectorClass: 'node-selector',
            selectedClass: 'node-selected',
            callback: elements => {
                this.selectedItems = elements;
                this.$root.$emit('vuefinder-items-selected', elements);
            }
        });

        this.path = this.path.replace(/^\/+|\/+$/i, '');
        this.fetchIndex(this.url, this.path);

        this.$root.$on('vuefinder-item-uploaded', () => {
            this.openFolder(this.data.dirname);
        });

    },
    created(){
      // 当axios注入无效的时候,是因为使用的axios版本和第三方npm包的版本不一样!!
      if (Object.keys(this.httpHeaders).length > 0){
        console.log('vueFinder第三方包 注入httpHeaders')
        axios.interceptors.request.use(
          config => {
            config.headers=this.httpHeaders
            return config
          },
          err => {
            return Promise.reject(err)
          })
      }

    },
    methods: {

        getComponentbyNode(element) {
            return this.$refs.files.find(a => a.$el == element);
        },

        getSelectedComponents(){
            return this.selectedItems.map(element => this.getComponentbyNode(element));
        },

        getSelectedItems () {
            return this.getSelectedComponents().map(a => {
              return a.item;
            });
        },

        getNodeElements() {
            return this.$refs.files.map(a => a.$el);
        },

        fetchIndex(url, path = null) {
            this.loading = true;
            axios(url, {
                params: {
                    q: 'index',
                    path: path
                }
            })
                .then(response => {
                    this.data = response.data;
                    this.loading = false;
                    this.$nextTick(() => {
                        this.selectable.selectables = this.getNodeElements();
                    });
                })
                .catch(error => {
                    this.msgBox(error.message, 'error');
                });
        },

        sortItems(column) {
            let sort = (active, column, order) => {
                return {active, column, order};
            };

            if (this.sort.active && this.sort.column == column) {
                this.sort = sort(this.sort.order == 'asc', column, 'desc');
            } else {
                this.sort = sort(true, column, 'asc');
            }
        },

        msgBox(message, type = 'error') {
            this.showMenu('message', {message: message, type: type});
        },

        showContextMenu(e) {
            this.context.active = true;
            this.context.items.push({
                title: '新建文件夹',
                icon: 'folder-plus',
                action: () => this.showMenu('new-folder')
            });

            let area = this.$el.getBoundingClientRect(),
                left = e.pageX - area.left - window.scrollX,
                top = e.pageY - area.top - window.scrollY;

            this.$nextTick(()=>{
                let menuHeight = this.$refs.context.$el.offsetHeight+18,
                    menuWidth = this.$refs.context.$el.offsetWidth;

                left = area.right - e.pageX + window.scrollX  < menuWidth ? left-menuWidth : left;
                top = area.bottom - e.pageY +  window.scrollY < menuHeight ? top-menuHeight : top;

                this.context.positions = {
                    left: left + 'px',
                    top: top + 'px'
                };
            });

        },

        hideContextMenu() {
            this.context.items = [];
            this.context.active = false;
        },

        addContextItems(item) {

            //console.log('item',item);

            if ( this.back===true && this.getSelectedItems().length <= 1 && item.type!=='folder') {
              this.context.items.push({
                title: '选中(' + item.basename + '),返回之前操作',
                icon: 'folder-open',
                action: () =>  this.selectBack([item])
              });
            }

            this.context.items.push({
                title: '打开',
                icon: 'folder-open',
                action: () => this.open(item)
            });

            if (this.isSelected(item) && this.getSelectedItems().length > 1) {
                this.context.items.push({
                    title: '删除 (' + this.getSelectedItems().length + ' items)',
                    icon: 'times-circle',
                    action: () => this.showMenu('delete', this.getSelectedItems())
                });
                return;
            }

            this.context.items.push({
                title: '重命名',
                icon: 'edit',
                action: () => this.showMenu('rename', [item])
            });
            this.context.items.push({
                title: '预览',
                icon: 'eye',
                action: () => this.showMenu('preview', [item])
            });
            this.context.items.push({
                title: '删除',
                icon: 'times-circle',
                action: () => this.showMenu('delete', [item])
            });
        },

        isSelected(item) {
            return this.getSelectedItems().includes(item);
        },

        open(item) {
            this.selectable.clearSelection();
            if (item.type == 'folder') {
                this.$root.$emit('vuefinder-folder-clicked');
                this.selectedItems=[];
                this.fetchIndex(this.url, item.path);
            } else {
                this.$root.$emit('vuefinder-item-clicked');
                this.showMenu('preview', [item]);
            }
        },

        openFolder(folder) {
            this.open({path: folder, type: 'folder'});
        },

        showMenu(type, item = false) {
            this.modal.item = item || this.getSelectedItems();
            this.modal.type = type;
            this.modal.active = true;
        },

        selectBack(item = false){
          let arg_item = item || this.getSelectedItems();
          console.log('arg_item',arg_item);
          this.$emit('selectBack', arg_item)
        }
    }
};
</script>
<style lang="scss" scoped>
    .vuefinder {
        font-family: Helvetica, sans-serif;
        color: rgb(74, 74, 74);
        letter-spacing: 1px;
        position: relative;
        border-radius: 4px 4px 0 0;
        -webkit-font-smoothing: antialiased;
        user-select: none;
        -moz-user-select: none;
        -webkit-user-select: none;
        -ms-user-select: none;
    }

    .vuefinder-footer {
        padding: 5px;
        font-size: 12px;
        min-height: 28px;
        line-height: 18px;
        align-items: center;
        display: flex;
        .vuefinder-status-message {
            text-align: right;
            flex: 1;
        }
    }

    #{"/deep/"} .node-selector {
        pointer-events: none;
        display: none;
        opacity: 0.3;
    }

    @media screen and (max-width: 768px) {
        #{"/deep/"} .is-hidden-mobile {
            display: none !important;
        }
    }

    #{"/deep/"} .vuefinder-input {
        padding: 8px;
        font-size: 14px;
        font-weight: 200;
        &:focus {
            outline: 0;
        }
    }

    #{"/deep/"} .vuefinder-button {
        letter-spacing: 1.1px;
        font-size: 13px;
        font-weight: 200;
        padding: 0.4rem 0.8em;
        cursor: pointer;
        color: #2d4e5c;
        border: 1px solid #8db3c1;
        background-color: transparent;
        &:active {
            position: relative;
            top: 1px;
        }
        &:focus {
            outline: 0;
        }
        &:hover {
            opacity: 0.9;
        }
        &[disabled] {
            opacity: 0.4;
            cursor: not-allowed;
        }
        &[disabled]:active {
            position: relative;
            top: 0px;
        }
    }

    @import "./styles/theme.scss";
    @include vuefinder-theme(light, #2e5e8b, #fbfcfd, darken(#fbfcfd, 5));
    @include vuefinder-theme(dark, #ababab, #313131, lighten(#313131, 10));
    @include vuefinder-theme(mithril, #374d63, #c3d7eb, darken(#c3d7eb, 5));
    @include vuefinder-theme(night, #dee9f8, #2e3463, lighten(#2e3463, 5));
    @include vuefinder-theme(ember, #e8d8be, #31323a, lighten(#31323a, 5));
    @include vuefinder-theme(earthsong, #8faf6f, #fafafa, darken(#fafafa, 5));

</style>
