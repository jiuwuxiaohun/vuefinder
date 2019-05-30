<template>
  <div class="vuefinder-file-previewer">

    <div v-if="type=='default'">no preview</div>

    <img v-if="type=='file-image'" :src="fileBaseUrl+path" />
    <div v-if="type=='file-alt'" class='vuefinder-text-preview'>{{ content }}</div>

    <slot />
  </div>
</template>

<script>
  export default {
    name: 'FilePreviewer',
    props:{
      url: {
        type: String,
        required: true
      },
      fileBaseUrl: {
        type: String,
        required: true
      },
      path: {
        type: String,
        required: true
      },
      type: {
        type: String,
        required: true
      }
    },
    data () {
      return {

      };
    }
  };
</script>

<style lang="scss" scoped>
  .vuefinder-file-previewer {
    display: flex;
    margin-bottom: 1em;
    max-height: 200px;

    #{"/deep/"} .vuefinder-text-preview {
      background-color: whitesmoke;
      color: #4a4a4a;
      font-size: 0.875em;
      overflow-x: auto;
      max-width: 800px;
      padding: 1.1em 1.2rem;
      white-space: pre-line;
      word-wrap: break-word;
      border: 1px solid lightgray;
      user-select: text;
      width: 100%;
    }

    #{"/deep/"} img,
    embed,
    object,
    audio,
    video {
      max-width: 100%;
      max-height: 180px;
    }

    #{"/deep/"} img {
      max-width: 100%;
      max-height: 100%;
      width: 100%;
      height: auto;
      object-fit: contain;
    }
  }
</style>
