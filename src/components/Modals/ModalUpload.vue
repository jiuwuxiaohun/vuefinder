<template>
  <modal @close="$emit('close')">
    <div class="vuefinder-modal-header">
      上传吗?选择的文件
    </div>
    <div class="vuefinder-modal-body">
      <div
              v-for="({file, percentage, done, failed}, key) in files"
              :key="key"
              :class="{ 'vuefinder-upload-succeed': done, 'vuefinder-upload-failed' : failed }"
              class="vuefinder-file-listing"
      >
        <div class="vuefinder-uploaded-name">{{ file.name }}</div>
        <div class="vuefinder-uploaded-size">{{ fileSizeIEC(file.size) }}</div>
        <div class="vuefinder-uploaded-extra">
          <span v-if="percentage != 0">{{ percentage }}%</span>
          <button
                  v-else
                  class="remove-file"
                  @click="removeFile( key )">移除
          </button>
        </div>
      </div>

      <div class="vuefinder-uploader">
        <input
                id="files"
                ref="files"
                type="file"
                :accept="accept"
                multiple
                @change="handleFilesUpload">
      </div>
    </div>
    <div
            v-if="errors.length > 0 "
            class="vuefinder-upload-errors"
    >
      <div
              class="vuefinder-upload-clear-errors"
              @click="errors = []"
      >X
      </div>
      <div
              v-for="(error, key) in errors"
              :key="key"
      >
        {{ error }}
      </div>
    </div>

    <div class="vuefinder-modal-footer">
      <button
              class="vuefinder-button"
              @click="addFiles"
      >添加文件
      </button>
      <button
              :disabled="uploadableFiles.length==0"
              class="vuefinder-button"
              @click="submitFiles"
      ><b>上传! 现有({{ uploadableFiles.length }}个任务在队列里)</b>
      </button>
      <button
              class="vuefinder-button"
              @click="$emit('close')"
      >取消
      </button>
    </div>
  </modal>
</template>

<script>

  import Modal from './Modal'
  import axios from 'axios'
  import filesize from '../../mixins/filesize'

  export default {
    name: 'ModalUpload',
    components: { modal: Modal },
    mixins: [filesize],
    props: {
      url: {
        type: String,
        required: true
      },
      directory: {
        type: String,
        required: true
      },
      accept:{
        type:String,
        default:'image/gif,image/jpeg,image/jpg,image/png'
      }
    },
    data() {
      return {
        files: [],
        errors: []
      }
    },
    computed: {
      uploadableFiles: function() {
        return this.files.filter(({ done, failed }) => {
          return done == false && failed == false
        })
      }
    },
    methods: {
      addFiles() {
        this.$refs.files.click()
      },
      handleFilesUpload() {
        let uploadedFiles = this.$refs.files.files
        //let maximumFileSize = Math.pow(2, 20) * 1.5; // 1.5MB
        /*
          Adds the uploaded file to the files array
        */
        for (var i = 0; i < uploadedFiles.length; i++) {
          // let size = (uploadedFiles[i].size/Math.pow(2, 20)).toFixed(1)

          // if (uploadedFiles[i].size > maximumFileSize ){
          //     this.errors.push(uploadedFiles[i].name + "(" + size +"MB). The file size exceeds the allowable limit.(1.5MB)");
          //     return;
          // }

          let elementPos = this.files.map(({ file }) => {
            return file.name
          }).indexOf(uploadedFiles[i].name)
          if (elementPos) {
            this.files.push({
              file: uploadedFiles[i],
              percentage: 0,
              done: false,
              failed: false
            })
          }
        }
      },
      /*
         Submits files to the server
       */
      submitFiles() {
        this.errors = []
        let promises = []

        this.files.forEach(({ file, done, failed }, index) => {
          if (done == true || failed == true) {
            return
          }

          let formData = new FormData()
          formData.append('file', file)
          formData.append('path', this.directory)

          promises.push(
            axios.post(this.url + '?q=upload',
              formData,
              {
                headers: {
                  'Content-Type': 'multipart/form-data'
                },
                //上传组件的进度功能,将可能被mock.js所影响,正式环境请去掉mock.js.
                onUploadProgress: progressEvent => this.$set(this.files[index], 'percentage', parseInt(Math.round((progressEvent.loaded * 100) / progressEvent.total))),
              }
            ).then(() => {
              this.$set(this.files[index], 'done', true)
              //this.$root.$emit("vuefinder-item-uploaded");
            }).catch((e) => {
              //console.log('axios catch',e)

              this.$set(this.files[index], 'failed', true)
              if (e.response.data.message) {
                this.errors.push(e.response.data.message)
              } else {
                this.errors.push(e.message)
              }

            })
          )
        })

        axios.all(promises)
          .then(axios.spread(() => {
            this.$root.$emit('vuefinder-item-uploaded')
          }))
      },
      removeFile(key) {
        this.files.splice(key, 1)
      }
    }
  }
</script>

<style lang="scss">
  input[type="file"] {
    position: absolute;
    top: -500px;
  }

  .vuefinder-file-listing {
    display: flex;
    font-size: 13px;
    padding: 3px;

    .vuefinder-uploaded-name {
      width: 0;
      flex: 1;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }
    .vuefinder-uploaded-size {
      margin-left: 10px;
    }
    .vuefinder-uploaded-extra {
      margin-left: 10px;
      padding: 1px;
    }
  }

  .vuefinder-upload-succeed {
    color: darkgreen;
    .vuefinder-uploaded-extra {
      background-color: lightgreen;
      border: 1px solid #80bb80;
    }
  }

  .vuefinder-upload-failed {
    color: darkred;
    .vuefinder-uploaded-extra {
      background-color: #ffedef;
      border: 1px solid darksalmon;
    }
  }

  .vuefinder-upload-errors {
    position: relative;
    border: 1px solid lightcoral;
    background-color: #ffedef;
    color: darkred;
    font-size: 12px;
    margin: 5px;
    padding: 2px;
  }

  .vuefinder-upload-clear-errors {
    position: absolute;
    top: 1px;
    right: 4px;
    cursor: pointer;
  }
</style>
