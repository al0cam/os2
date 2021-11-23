<template>
  <div class="bodyAlign">
    <div class="keyAlign">
      <div>
        <v-btn @click="generateAsymmetricKeys"
        >Generate asymmetric keys</v-btn>
      </div>
      <div>
        <v-btn @click="generateSymmetricKey"
        >Generate symmetric key </v-btn>
      </div>
    </div>
    <div>
      <v-file-input
          ref="inputForFile"
          v-model="textFile.fileName"
          truncate-length="15"
          @change="readFromFile(textFile)"
          placeholder="Insert text file"
      ></v-file-input>
      <v-textarea
          placeholder="Text"
          v-model="textFile.fileContent"
      ></v-textarea>
      <v-btn @click="saveToFile(textFile)">
        Save text to file
      </v-btn>
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
        <v-btn @click="saveToFile(privateKey)">
          Save key to file
        </v-btn>
      </div>
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="publicKey.publicKeyFile"
            truncate-length="15"
            @change="readFromFile(publicKey)"
            placeholder="Insert public key file"
        ></v-file-input>
        <v-textarea
            placeholder="Public key"
            v-model="publicKey.publicKeyPem"
        ></v-textarea>
        <v-btn @click="saveToFile(publicKey)">
          Save key to file
        </v-btn>
      </div>
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="secretKey.secretKeyFile"
            truncate-length="15"
            @change="readFromFile(secretKey)"
            placeholder="Insert secret key file"
        ></v-file-input>
        <v-textarea
            placeholder="Secret key"
            v-model="secretKey.secretKeyString"
        ></v-textarea>
        <v-btn @click="saveToFile(secretKey)">
          Save key to file
        </v-btn>
      </div>
    </div>
    <v-snackbar
        v-model="snackbar"
        :multi-line="true"
        timeout="2500"
    >
      {{ this.snackbarText }}
      <template v-slot:action="{ attrs }">
        <v-btn
            color="blue"
            text
            v-bind="attrs"
            @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>

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
      snackbar: false,
      snackbarText: null,
      keyPair: null,
      textFile:{
        fileName: null,
        fileContent: null
      },
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
      if (passedFile[keys[0]] != null)
      {
        reader.readAsText(passedFile[keys[0]], 'UTF-8')
        reader.onload = (item) => {
          passedFile[keys[1]] = item.target.result.toString();
          if((keys[0]).includes('private'))
            this.translatePemToKeys(this.privateKey)
          else if((keys[0]).includes('public'))
            this.translatePemToKeys(this.publicKey)
        }
      }
      else
      {
        this.snackbarText = "No file to read from!"
        this.snackbar = true
      }
    },
    saveToFile(key){
      var keys = Object.keys(key)
      if (key[keys[1]] != null)
      {
        var blob = new Blob([key[keys[1]]], {type: 'text/plain;charset=utf-8'});
        fileSaver.saveAs(blob, keys[0]+'.txt')
      }
      else
      {
        this.snackbarText = "Field is empty, nothing to save!"
        this.snackbar = true
      }
    },
    async generateAsymmetricKeys() {
      this.keyPair = await openCrypto.getRSAKeyPair();
      await this.translateKeysToPem();
    },
    async generateSymmetricKey() {
      this.secretKey.secretKeyObject = await openCrypto.getSharedKey();
      this.secretKey.secretKeyString = (await crypto.subtle.exportKey('jwk', this.secretKey.secretKeyObject)).k;
    },
    async translateKeysToPem() {
      this.privateKey.privateKeyPem = await openCrypto.cryptoPrivateToPem(this.keyPair['privateKey'])
      this.publicKey.publicKeyPem = await openCrypto.cryptoPublicToPem(this.keyPair['publicKey'])
    },
    async translatePemToKeys(key){
      var keys = Object.keys(key)
      if((keys[0]).includes('private'))
      {
        this.privateKey.privateKeyObject = await openCrypto.pemPrivateToCrypto(this.privateKey.privateKeyPem,
            { name: 'RSA-OAEP',
              hash: 'SHA-512',
              usages: ['decrypt', 'unwrapKey'],
              isExtractable: true })
      }
      else if((keys[0]).includes('public'))
      {
        this.publicKey.publicKeyObject = await openCrypto.pemPublicToCrypto(this.publicKey.publicKeyPem.toString(),
            { name: 'RSA-OAEP',
              hash: 'SHA-512',
              usages: ['encrypt', 'wrapKey'],
              isExtractable: true })
        console.log(this.publicKey.publicKeyObject)
      }
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