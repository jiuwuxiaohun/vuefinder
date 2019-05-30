<template>
    <div>
        <file-icon v-if="listview==true || item['type']!=='file-image'"
                   :icon="item.type"
                   data-selectable
        >
    <span>
      {{ listview ? item.basename : title_shorten(item.basename) }}
    </span>

            <span
                    v-show="listview"
                    class="vuefinder-file-size is-hidden-mobile"
            >
      {{ item.type != 'folder' ? fileSizeIEC(item.size) : '' }}
    </span>

            <span
                    v-show="listview"
                    class="vuefinder-file-time is-hidden-mobile"
            >
      {{ time(item.timestamp) }}
    </span>
        </file-icon>

        <div v-if="listview==false && item['type']==='file-image'">
            <div class="jw-img-box">
                <div class="jw-img-file-icon" >
                    <!--<img class="jw-div-bg-img" :src="url+'?q=read&path='+encodeURIComponent(item['path'])"/>-->
                    <img class="jw-div-bg-img" :src="fileBaseUrl+item['path']"/>
                </div>

                <div class="jw-item-file-name">
                    <span>
                      {{ listview ? item.basename : title_shorten(item.basename) }}
                    </span>
                </div>
            </div>

        </div>
    </div>
</template>

<script>
  import FileIcon from './FileIcon.vue'
  import format from 'date-fns/format'
  import filesize from '../mixins/filesize'

  export default {
    name: 'Item',
    components: { FileIcon },
    mixins: [filesize],
    props: {
      url: {
        type: String,
        required: true
      },
      fileBaseUrl:{
        type: String,
        required: true
      },
      item: {
        type: Object,
        required: true
      },
      listview: {
        type: Boolean,
        required: true
      }
    },
    methods: {
      title_shorten(title) {
        return title.replace(/((?=([\w\W]{0,14}))([\w\W]{8,})([\w\W]{8,}))/, '$2..$4')
      },
      time(time) {
        return format(new Date(time * 1000), 'dd MMMyy HH:mm')
      }
    }
  }
</script>

<style lang="scss" scoped>
    .vuefinder-file-size {
        width: 66px;
    }

    .vuefinder-file-time {
        width: 120px;
    }

    /*-------------*/

    .jw-img-box {
        width: 150px;
        height: 200px;
        margin: 4px;
        display: flex;
        flex-direction: column;
        overflow: hidden;
        border: 1px solid #cccccc;
        border-radius: 4px;
    }
    .jw-div-bg-img{
        height: 100%;
        background-repeat:no-repeat;
        background-position: center center;
        background-image:url('../image/img_loading.png');
    }

    .jw-img-file-icon {
        height: 80%;
        display: flex;
        margin-top: 5px;
        align-self: center;
        justify-content: flex-start;
        overflow: hidden;
    }


    .jw-item-file-name {
        display: flex;
        margin-top: 5px;
        font-size: 12px;
        line-height: 24px;
        align-items: flex-start;
        flex: 1;
        justify-content: flex-start;

        span:first-child {
            flex: 1;
            //align-self: stretch;
        }

        span {
            padding-left: 5px;
            padding-right: 5px;
            border-radius: 5px;
            text-align: center;
            word-break: break-all;
            border: 1px dashed transparent;
        }
    }

</style>
