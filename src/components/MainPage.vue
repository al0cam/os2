<template>
  <div class="bodyAlign" >
    <div class="keyAlign generateButtons">
      <div>
        <v-btn @click="generateAsymmetricKeys"
        >Generate asymmetric keys</v-btn>
      </div>
      <div>
        <v-btn @click="generateSymmetricKey"
        >Generate symmetric key </v-btn>
      </div>
      <div>
        <v-btn @click="getHash(textFile.fileContent)"
        >Generate hash</v-btn>
      </div>
    </div>
    <div class="keyAlign encryptButtons">
      <div>
        <v-btn @click="sign"
        >Sign</v-btn>
      </div>
      <div>
        <v-btn @click="encryptText(publicKey)"
        >Asymmetric encrypt</v-btn>
      </div>
      <div>
        <v-btn @click="encryptText(secretKey)"
        >Symmetric encrypt</v-btn>
      </div>
    </div>
    <div class="keyAlign decryptButtons">
      <div>
        <v-btn @click="verifySignature"
        >Verify signature</v-btn>
      </div>
      <div>
        <v-btn @click="decryptText(privateKey)"
        >Asymmetric decrypt</v-btn>
      </div>
      <div>
        <v-btn @click="decryptText(secretKey)"
        >Symmetric decrypt</v-btn>
      </div>
    </div>
    <div class="keyAlign">
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
      <div>
        <v-file-input
            ref="inputForFile"
            v-model="encryptedTextFile.fileName"
            truncate-length="15"
            @change="readFromFile(encryptedTextFile)"
            placeholder="Insert encrypted text file"
        ></v-file-input>
        <v-textarea
            placeholder="Encrypted text"
            v-model="encryptedTextFile.fileContent"
        ></v-textarea>
        <v-btn @click="saveToFile(encryptedTextFile)">
          Save encrypted text to file
        </v-btn>
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
const cryptoed = require('crypto')

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
      encryptedTextFile:{
        fileName: null,
        fileContent: null
      },
      privateKey:{
        privateKeyFile: null,
        privateKeyPem: null,
        privateKeyObject: null,
        privateSignKey: null
      },
      publicKey:{
        publicKeyFile: null,
        publicKeyPem: null,
        publicKeyObject: null,
        publicSignKey: null
      },
      secretKey:{
        secretKeyFile: null,
        secretKeyString: null,
        secretKeyObject: null
      },
    }
  },
  methods: {
    invokeSnackBar(text)
    {
      this.snackbarText = text
      this.snackbar = true
    },
    //File I/O START
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
          else if((keys[0]).includes('secret'))
            this.translateSecretKey(this.secretKey)
        }
      }
      else
        this.invokeSnackBar("No file to read from!")
    },
    saveToFile(key){
      var keys = Object.keys(key)
      if (key[keys[1]] != null)
      {
        var blob = new Blob([key[keys[1]]], {type: 'text/plain;charset=utf-8'});
        fileSaver.saveAs(blob, keys[0]+'.txt')
      }
      else
        this.invokeSnackBar("Field is empty, nothing to save!")
    },
    //File I/O END
    //
    async generateAsymmetricKeys() {
      this.keyPair = await openCrypto.getRSAKeyPair();
      this.publicKey.publicKeyObject = this.keyPair['publicKey']
      this.privateKey.privateKeyObject = this.keyPair['privateKey']
      await this.translateKeysToPem();
    },
    async generateSymmetricKey() {
      this.secretKey.secretKeyObject = await openCrypto.getSharedKey();
      var raw = await crypto.subtle.exportKey('raw', this.secretKey.secretKeyObject)
      this.secretKey.secretKeyString = openCrypto.arrayBufferToHexString(raw)
    },
    async translateKeysToPem() {
      this.privateKey.privateKeyPem = await openCrypto.cryptoPrivateToPem(this.keyPair['privateKey'])
      this.publicKey.publicKeyPem = await openCrypto.cryptoPublicToPem(this.keyPair['publicKey'])
      this.publicKey.publicSignKey = await openCrypto.pemPublicToCrypto(this.publicKey.publicKeyPem,
          {name: 'RSA-PSS', hash: 'SHA-512' , usages: ['verify'], isExtractable: true})
      this.privateKey.privateSignKey = await openCrypto.pemPrivateToCrypto(this.privateKey.privateKeyPem,
          {name: 'RSA-PSS', hash: 'SHA-512' , usages: ['sign'], isExtractable: true})
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
      }
    },
    async translateSecretKey() {
      //TODO add error test for key import and wrong key size
      if (this.secretKey.secretKeyString != null) {
        var text = openCrypto.hexStringToArrayBuffer(this.secretKey.secretKeyString)
        var key = await crypto.subtle.importKey("raw",
            text,
            {name: "AES-GCM"},
            true,
            ["encrypt", "decrypt"])
        this.secretKey.secretKeyObject = key
      } else
        this.invokeSnackBar("No secret key to translate!")
    },
    async encryptText(key) {
      var keys = Object.keys(key)
      if(key[keys[2]] != null)
      {
        var arrayBufferText, encryptedText
        if((keys[2]).includes('public'))
        {
          arrayBufferText = openCrypto.stringToArrayBuffer(this.textFile.fileContent)
          encryptedText = await crypto.subtle.encrypt({name: 'RSA-OAEP'}, this.publicKey.publicKeyObject, arrayBufferText)
          this.encryptedTextFile.fileContent = openCrypto.arrayBufferToBase64(encryptedText)
        }
        else if((keys[2]).includes('secret'))
        {
          arrayBufferText = openCrypto.stringToArrayBuffer(this.textFile.fileContent)
          encryptedText = await openCrypto.encrypt(this.secretKey.secretKeyObject, arrayBufferText)
          this.encryptedTextFile.fileContent = encryptedText
        }
      }
      else
        this.invokeSnackBar( "No key to encrypt with!")
    },
    async decryptText(key) {
      var keys = Object.keys(key)
      if(key[keys[2]] != null)
      {
        var arrayBufferText, decryptedText
        if((keys[2]).includes('private'))
        {
          arrayBufferText = openCrypto.base64ToArrayBuffer(this.encryptedTextFile.fileContent)
          decryptedText = await crypto.subtle.decrypt({name: 'RSA-OAEP'}, this.privateKey.privateKeyObject, arrayBufferText)
          this.textFile.fileContent = openCrypto.arrayBufferToString(decryptedText)
        }
        else if((keys[2]).includes('secret'))
        {
          decryptedText = await openCrypto.decrypt(this.secretKey.secretKeyObject, this.encryptedTextFile.fileContent, {cypher: "AES-GCM"})
          this.textFile.fileContent = openCrypto.arrayBufferToString(decryptedText)
        }
      }
      else
        this.invokeSnackBar("No key to decrypt with!")
    },
    async sign()
    {
      if(this.textFile.fileContent != null && this.privateKey.privateSignKey != null)
      {
        var arrayBufferText = openCrypto.stringToArrayBuffer(this.textFile.fileContent)
        var encryptedText = await crypto.subtle.sign({name: 'RSA-PSS', saltLength: 0}, this.privateKey.privateSignKey, arrayBufferText)
        this.encryptedTextFile.fileContent = openCrypto.arrayBufferToBase64(encryptedText)
      }
      else if(this.textFile.fileContent == null)
        this.invokeSnackBar("Nothing to sign!")
      else
        this.invokeSnackBar("No private key to sign with!")
    },
    async verifySignature(){
      if(this.encryptedTextFile.fileContent != null && this.textFile.fileContent != null)
      {
        var arrayBufferEncryptedText = openCrypto.base64ToArrayBuffer(this.encryptedTextFile.fileContent)
        var arrayBufferText = openCrypto.stringToArrayBuffer(this.textFile.fileContent)
        var good = await crypto.subtle.verify({name: 'RSA-PSS', saltLength: 0},
            this.publicKey.publicSignKey, arrayBufferEncryptedText, arrayBufferText)
        if (good)
          this.invokeSnackBar("The signature is good!")
        else
          this.invokeSnackBar("The signature is bad!")
      }
      else
        this.invokeSnackBar("Both text fields are required for verification!")
    },
    getHash(text)
    {
      if(text != null)
      {
        var hash = cryptoed.createHash('sha256')
        hash.update(text)
        this.encryptedTextFile.fileContent = hash.digest('hex')
      }
      else
        this.invokeSnackBar("There is no text to hash!")
    }

  }
}


</script>

<style scoped>
.bodyAlign
{
  display: flex;
  flex-direction: column;
  align-content: center;
  /*align-items: center;*/
}
.bodyAlign div
{
  color: #9ea05f;
  padding: 0.5%;
}
.keyAlign
{
  display: flex;
  flex-direction: row;
  padding: 0.5%;
  align-self: center;
}

.generateButtons button
{
  color: white !important;
  background: cadetblue !important;
}
.encryptButtons button
{
  color: white !important;
  background: #9ea05f !important;
}
.decryptButtons button
{
  color: white !important;
  background: #a05f9e !important;
}
</style>