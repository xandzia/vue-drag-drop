<template>
  <div id="file-drag-drop">
    <form ref="fileform">
      <span class="drop-files">Drop the files here!</span>
    </form>

    <div v-for="(file, key) in files" class="file-listing">
      <img class="preview" v-bind:ref="'preview'+parseInt( key )"/>
      {{ file.name }}
      <span class="remove-container">
        <a class="remove" v-on:click="removeFile( key )">X</a>
      </span>
    </div>

    <a class="submit-button" v-on:click="submitFiles()" v-show="files.length > 0">Submit</a>
  </div>
</template>

<script>
    export default {

        data() {
            return {
                dragAndDropCapable: false,
                files: []
            }
        },

        mounted() {

            this.dragAndDropCapable = this.determineDragAndDropCapable();

            if (this.dragAndDropCapable) {
                ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach(function(evt) {

                    this.$refs.fileform.addEventListener(evt, function(e) {
                        e.preventDefault();
                        e.stopPropagation();
                    }.bind(this), false);
                }.bind(this));

                this.$refs.fileform.addEventListener('drop', function(e) {
                    for (let i = 0; i < e.dataTransfer.files.length; i++) {
                        this.files.push(e.dataTransfer.files[i]);
                    }
                }.bind(this));
            }
        },

        methods: {

            determineDragAndDropCapable() {

                var div = document.createElement('div');

                return (('draggable' in div) ||
                        ('ondragstart' in div && 'ondrop' in div)) &&
                    'FormData' in window &&
                    'FileReader' in window;
            },

            submitFiles() {

                let formData = new FormData();

                for (var i = 0; i < this.files.length; i++) {
                    let file = this.files[i];

                    formData.append('files[' + i + ']', file);
                }

                axios.post('/file-drag-drop',
                        formData, {
                            headers: {
                                'Content-Type': 'multipart/form-data'
                            },
                            onUploadProgress: function(progressEvent) {
                                this.uploadPercentage = parseInt(Math.round((progressEvent.loaded * 100) / progressEvent.total));
                            }.bind(this)
                        }
                    ).then(function() {
                        console.log('SUCCESS!!');
                    })
                    .catch(function() {
                        console.log('FAILURE!!');
                    });
            },

            removeFile(key) {
                this.files.splice(key, 1);
            }
        }
    }
</script>

<style scoped>
    form {
        display: block;
        height: 400px;
        width: 400px;
        background: #ccc;
        margin: auto;
        margin-top: 40px;
        text-align: center;
        line-height: 400px;
        border-radius: 4px;
    }

    div.file-listing {
        width: 400px;
        margin: auto;
        padding: 10px;
        border-bottom: 1px solid #ddd;
    }

    div.file-listing img {
        height: 100px;
    }

    div.remove-container {
        text-align: center;
    }

    span.remove-container a {
        color: red;
        cursor: pointer;
    }

    a.submit-button {
        display: block;
        margin: auto;
        text-align: center;
        width: 200px;
        padding: 10px;
        text-transform: uppercase;
        background-color: #CCC;
        color: white;
        font-weight: bold;
        margin-top: 20px;
    }
</style>