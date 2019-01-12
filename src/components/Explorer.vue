<template>
  <transition-group
    :class="{ 'list': listview }"
    class="vuefinder-explorer"
    name="vuefinder-items"
    tag="div"
  >
    <file-icon
      v-show="! isRoot"
      key="keyBackButton"
      icon="angle-left"
      @click.native="$emit('back')">
      <span>返回上一级</span>
    </file-icon>

    <slot />

    <file-icon key="keyLoading" v-show="loading" icon="spinner" spin>
      <span>数据加载中...</span>
    </file-icon>

    <div
      v-if="! $slots.default"
      key="empty-list"
      class="vuefinder-empty-list">
      <span>这里没有任何文件或文件夹!</span>
    </div>

  </transition-group>

</template>

<script>
import FileIcon from './FileIcon.vue';

export default {
    name: 'Explorer',
    components: { FileIcon },
    props:{
        isRoot: {
            type: Boolean,
            required: true
        },
        listview: {
            type: Boolean,
            required: true
        },
        loading: {
          type: Boolean,
          default:false
        }
    },
};
</script>

<style lang="scss" scoped>
.vuefinder-explorer {
  display: flex;
  box-sizing: content-box;
  padding: 10px;
  margin: 0;
  flex-wrap: wrap;
  min-height: 95px;
  max-height: 400px;
  list-style: none;
  justify-content: flex-start;
  align-content: flex-start;
  overflow-y: auto;

  &.list {
    display: block;
  }
}

.vuefinder-empty-list {
  margin: auto;
  padding: 10px;
  position: absolute;
  text-align: center;
  left: 0;
  right: 0;
  bottom: 0;
  width: 50%;
  max-width: 350px;
  height: 50px;
}

#{"/deep/"} .vuefinder-items-enter-active {
  transition: opacity 0.5s;
}

#{"/deep/"} .vuefinder-items-enter, #{"/deep/"} .vuefinder-items-leave-to /* .list-leave-active below version 2.1.8 */
 {
  opacity: 0;
}
</style>
