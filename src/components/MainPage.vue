<template>
  <div class="bodyAlign">
    <div class="keyAlign">
      <div>
        <b-button style="align-content: center" @click="readFromFile">
          Read from file
        </b-button>
      </div>
      <div>
        <b-button style="align-content: center" @click="writeContentOfFile">
          Print file1
        </b-button>
      </div>
      <div>
        <v-btn @click="generateAsymmetricKeys"
        >Generate asymmetric keys</v-btn>
      </div>
      <div>
        <v-btn @click="generateSymmetricKey"
        >Generate symmetric key </v-btn>
      </div>
      <div>
        <v-btn @click="generateSymmetricKey"
        >Save keys to file</v-btn>
      </div>
    </div>
    <div class="keyAlign">
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="privateKey.privateKeyFile"
            truncate-length="15"
            @change="readFromFile(privateKey)"
            placeholder="Insert private key file"
        ></v-file-input>
        <v-textarea
            placeholder="Private key"
            v-model="privateKey.privateKeyPem"
        ></v-textarea>
        <v-btn @click="saveKeyToFile(privateKey)">
          Save key to file
        </v-btn>
      </div>
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="publicKey.publicKeyFile"
            truncate-length="15"
            placeholder="Insert public key file"
        ></v-file-input>
        <v-textarea
            placeholder="Public key"
            v-model="publicKey.publicKeyPem"
        ></v-textarea>

        <v-btn @click="saveKeyToFile(publicKey)">
          Save key to file
        </v-btn>
      </div>
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="secretKey.secretKeyFile"
            truncate-length="15"
            placeholder="Insert secret key file"
        ></v-file-input>
        <v-textarea
            placeholder="Secret key"
            v-model="secretKey.secretKeyString"
        ></v-textarea>
        <v-btn @click="saveKeyToFile(secretKey)">
          Save key to file
        </v-btn>
      </div>
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
const reader = new FileReader()
// const forge = require('node-forge');
// forge.options.usePureJavaScript = true;

export default {
  name: "MainPage",
  data(){
    return{
      textFile: null,
      textFileContent: null,
      keyPair: null,
      privateKey:{
        privateKeyFile: null,
        privateKeyPem: null,
        privateKeyObject: null
      },
      publicKey:{
        publicKeyFile: null,
        publicKeyPem: null,
        publicKeyObject: null
      },
      secretKey:{
        secretKeyFile: null,
        secretKeyString: null,
        secretKeyObject: null
      },
    }
  },
  mounted() {

  },
  methods: {
    readFromFile(passedFile) {
      var keys = Object.keys(passedFile)
      console.log(keys)
      if (passedFile[keys[0]] != null)
      {
        reader.readAsText(passedFile[keys[0]], 'UTF-8')
        reader.onload = (item) => {
          passedFile[keys[1]] = item.target.result;
        }
      }
    },
    writeContentOfFile() {
      console.log(this.file1Content);
    },
    saveKeyToFile(key){
      var keys = Object.keys(key)
      if (key[keys[1]] != null)
      {
        var blob = new Blob([key[keys[1]]], {type: 'text/plain;charset=utf-8'});
        console.log(keys)
        fileSaver.saveAs(blob, keys[0]+'.txt')
      }
      else
      {

      }
    },
    async generateAsymmetricKeys() {
      this.keyPair = await openCrypto.getRSAKeyPair();
      this.translateKeysToPem();
    },
    async generateSymmetricKey() {
      this.secretKey.secretKeyObject = await openCrypto.getSharedKey();
      this.secretKey.secretKeyString = (await crypto.subtle.exportKey('jwk', this.secretKey.secretKeyObject)).k;
    },
    async translateKeysToPem() {
      this.privateKey.privateKeyPem = await openCrypto.cryptoPrivateToPem(this.keyPair['privateKey'])
      this.publicKey.publicKeyPem = await openCrypto.cryptoPublicToPem(this.keyPair['publicKey'])
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