<template>
  <div id="app">
    <h3>AWESOME GALLERY</h3>
    <div class="content container">
      <input type="file" class="form-control" ref="file" v-on:change="uploadImage($event.target.files)" accept="image/*">
      <top-progress ref="topProgress"></top-progress>
      <div class="has-error">
        <ul v-if="errors && errors.length">
            <li v-for="error in errors">
              {{ error }}
            </li>
         </ul>
      </div>
    </div>
    <br>
    <div class="row">
      <div class="col-xs-4" v-for="image in images" v-on:click="openModal(image)">
        <img :src="image.url" :key="image.public_id" style="width: 100%; height: 270px; margin-top: 20px;">
      </div>
      <sweet-modal modal-theme="dark" overlay-theme="dark" ref="modal">
        <img :src="url" alt="" width="560" height="360">
      </sweet-modal>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import axios from 'axios';
import topProgress from 'vue-top-progress';
import { SweetModal, SweetModalTab } from 'sweet-modal-vue';

export default {
  name: 'app',
  components: {
    topProgress,
    SweetModal,
    SweetModalTab
  },
  data () {
    return {
      file: '',
      errors: [],
      images: [],
      cloudinary: {
        cloud_name: "nguyentd1501", 
          api_key: "364785325217511", 
          upload_preset: "inrzetlv"
      },
      url: ''
    }
  },
  created: function() {
    axios.get('https://res.cloudinary.com/'+this.cloudinary.cloud_name+'/image/list/vue-gallery.json')
    .then(response => {
        console.log(response.data);
        var data = response.data.resources;
        this.images = data.map(t => {
          return {
            public_id: t.public_id,
            url: `http://res.cloudinary.com/${this.cloudinary.cloud_name}/image/upload/v${t.version}/${t.public_id}.jpg`
          }
        })
      })
      .catch(e => {
          this.errors.push(e)
      })
  },
  
  
  methods: {
    uploadImage: function(file) {
      var Extension = this.$refs.file.value.substring(this.$refs.file.value.lastIndexOf('.') + 1).toLowerCase();
      if (Extension == "gif" || Extension == "png" || Extension == "bmp" || Extension == "jpeg" || Extension == "jpg") {
        this.$refs.topProgress.start()
        const formData = new FormData();
        formData.append('file', file[0]);
        formData.append('upload_preset', this.cloudinary.upload_preset);
        formData.append('tags', 'vue-gallery');
        axios.post('https://api.cloudinary.com/v1_1/'+this.cloudinary.cloud_name+'/upload', formData)
          .then(response => {
            this.$refs.topProgress.done();
            var data = response.data;
            this.images.unshift({public_id: data.public_id, url: data.url});
            this.errors = [];
            this.$refs.file.value = null;
          })
          .catch(e => {
            this.$refs.topProgress.fail();
            this.errors.push(e)
          })
      } else {
        this.errors.push('Only image is allowed!')
      }
    },
    openModal(image) {
      this.$refs.modal.open();
      this.url = image.url;
    }
  },
}
</script>

<style>
#app{
  font-family: 'Montserrat', sans-serif;
}
h3{
  text-align: center;
  padding: 30px;
}
.has-error {
  color: red;
}
</style>
