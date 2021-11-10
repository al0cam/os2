<template>
  <div style="align-content: center; justify-content: center">
    <b-button style="align-content: center" @click="whatever">
      Button
    </b-button>
    <b-button style="align-content: center" @click="whatever2">
      Button2
    </b-button>
    <v-file-input
        ref="inputForFile"
        v-model="file1"
        truncate-length="15"
    ></v-file-input>
<!--    <v-text-field v-model="file1.content"></v-text-field>-->

  </div>
</template>

<script>
// import glstools from 'glstools'
// import crypto from 'crypto'
// const GlsTools = require('glstools');
const Crypto = require('crypto-js');

export default {
  name: "MainPage",
  data(){
    return{
      file1: null,
      file1Content: null
    }
  },
  methods:{
    whatever(){
      console.log(this.file1)
      const reader = new FileReader()
      reader.readAsText(this.file1, 'UTF-8')
      reader.onload = (item) => {this.file1Content = item.target.result; console.log(item.target.result)}
    },
    whatever2(){
      console.log(this.file1Content);
    },
  }
}

console.log(Crypto.SHA1('diego').toString());


</script>


<style scoped>

</style>