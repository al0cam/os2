<template>
  <div class="bodyAlign">
    <div>
      <b-button style="align-content: center" @click="whatever" class="">
        Read from file
      </b-button>
    </div>
    <div>
      <b-button style="align-content: center" @click="whatever2">
        Print file1
      </b-button>
    </div>
    <div>
      <v-btn @click="whatever3"
      >Save blob</v-btn>
    </div>
    <div>
      <v-btn @click="whatever4"
      >Generate RSA key</v-btn>
    </div>
    <div>
      <v-btn @click="whatever5"
      >Print RSA key</v-btn>
    </div>
    <div>
      <v-btn @click="whatever6"
      >Print RSA key 2</v-btn>
    </div>
    <v-file-input
        ref="inputForFile"
        v-model="file1"
        truncate-length="15"
    ></v-file-input>
    <div class="keyAlign">
      <v-textarea
          placeholder="Private key"
          v-model="privateKey"
      ></v-textarea>
      <v-textarea
          placeholder="Public key"
          v-model="privateKey"
      ></v-textarea>
      <v-textarea
          placeholder="Secret key"
          v-model="secretKey"
      ></v-textarea>
    </div>
<!--    <v-text-field v-model="file1.content"></v-text-field>-->

  </div>
</template>

<script>
// import glstools from 'glstools'
// import crypto from 'crypto'
// const GlsTools = require('glstools');
// const Crypto = require('crypto-js');
import fileSaver from 'file-saver';
import OpenCrypto from 'opencrypto';
const openCrypto = new OpenCrypto();
// const forge = require('node-forge');
// forge.options.usePureJavaScript = true;

export default {
  name: "MainPage",
  data(){
    return{
      file1: null,
      file1Content: null,
      keyPair: null,
      keyPair2: null,
      privateKey: null,
      publicKey: null,
      secretKey: null
    }
  },
  methods: {
    whatever() {
      console.log(this.file1)
      const reader = new FileReader()

      reader.readAsText(this.file1, 'UTF-8')
      reader.onload = (item) => {
        this.file1Content = item.target.result;
        console.log(item.target.result)
      }
    },
    whatever2() {
      console.log(this.file1Content);
    },
    whatever3() {
      console.log('sugma')
      var blob = new Blob([this.file1Content], {type: 'text/plain;charset=utf-8'});
      fileSaver.saveAs(blob, 'nice.txt')
    },
    async whatever4() {
      this.keyPair = await crypto.subtle.generateKey(
          {
            name: "RSA-OAEP",
            modulusLength: 2048,
            publicExponent: new Uint8Array([1, 0, 1]),
            hash: "SHA-256"
          },
          true,
          ["encrypt", "decrypt"]
      );

      this.keyPair2 = await openCrypto.getRSAKeyPair();
    },
    async whatever5() {
      console.log(this.keyPair['publicKey']);
      var pubKey = await crypto.subtle.exportKey("jwk", this.keyPair['publicKey'])
      console.log(pubKey)
      var Key = await crypto.subtle.exportKey("jwk", this.keyPair['privateKey'])
      console.log(Key)


    },
    async whatever6() {
      this.privateKey = await openCrypto.cryptoPrivateToPem(this.keyPair2.privateKey)
      this.publicKey = await openCrypto.cryptoPublicToPem(this.keyPair2['publicKey'])
    }
  }
}

// console.log(Crypto.SHA1('diego').toString());


</script>


<style scoped>

.bodyAlign
{
  display: flex;
  flex-direction: column;
  align-content: center;
}
.bodyAlign div
{
  padding: 0.5%;

}
.keyAlign
{
  display: flex;
  flex-direction: row;
  padding: 0.5%;
}


</style>