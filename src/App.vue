<template>

<div id="" >

<nav class="navbar is-transparent">
  <div class="navbar-brand">
    <a class="navbar-item" :href="cleanUrl">
      <img src="./assets/logo.png" alt="TweetWeave" width="112" height="28">
    </a>
    <div class="navbar-burger burger" data-target="navbarExampleTransparentExample">
      <span></span>
      <span></span>
      <span></span>
    </div>
  </div>

  <div id="navbarExampleTransparentExample" class="navbar-menu">
   
    <div class="navbar-end">
      <div class="navbar-item">
        <div class="field is-grouped">
         
          <p class="control">
            <a class="button is-primary" target="_blank" href="https://tokens.arweave.org" >
            
              <span>Get Free AR Tokens</span>
            </a>
          </p>
        </div>
      </div>
    </div>
  </div>
</nav>

    <section class="hero">
      <div class="hero-body">
        <div class="container">
          <div class="columns" id="app">
            <div class="column is-three-fifths is-offset-one-fifth">
             

              <section v-if="arImgViewTxn.status == 200" >
                    <img   v-if="arImgView" alt="AR image" :src="arImg">

                    <br/><br/>
                    <a :href="cleanUrl">
                      <b-button size="is-medium" type="is-primary" outlined>Create new snapshot
                      </b-button>
                    </a>

              </section>

              <section v-if="arImgView">
                <b-notification :active.sync="arImgViewTxn.status != 200" aria-close-label="notification">
                  Transaction <a :href="'https://viewblock.io/arweave/tx/'+this.arViewTxnId">{{this.arViewTxnId}}</a> is not confirmed yet in Arweave. Please refresh in a few minutes.</br>
                </b-notification>
              </section>

              <section v-if="jwk == null && !arImgView">
                <b-field>
                    <b-upload v-model="keyFile"
                    @input="loadJwk()"
                        drag-drop>
                        <section class="section">
                            <div class="content has-text-centered">
                                <p>
                                    <b-icon
                                        icon="upload"
                                        size="is-large">
                                    </b-icon>
                                </p>
                                <p>Drop your key file to login</p>
                            </div>
                        </section>
                    </b-upload>
                </b-field>
              </section>
            
          <section v-if="jwk"> 

             <!-- <h1 class="title">
                Primary title
              </h1> -->
              <h2 class="subtitle">
                Enter tweet url and create snapshot for permaweb
              </h2>


             <b-field label="">
               <b-input v-model="tweet" placeholder="https://twitter.com/ArweaveTeam/status/1129386133249511425" required></b-input>
             </b-field>

             <b-button rounded @click="sumbitTweet()" >Snapshot</b-button>

                <b-loading :is-full-page="isFullPage" :active.sync="isLoading" :can-cancel="false">
                    <b-icon
                        pack="fas"
                        icon="sync-alt"
                        size="is-large"
                        custom-class="fa-spin">
                    </b-icon>
                </b-loading> 

          </section>

          <br/><br/>

          <section v-if="tweetImg">
            <figure class="image  is-inline-block">
                    <img class="" :src="tweetImg"/>
            </figure>
          </section>    

          <br/> 

          <button v-if="tweetImg" class="button is-medium is-primary" @click="confirmSave">
            Save to Permaweb
          </button>

       </div>
      </div>     

          
        </div>
      </div>
    </section>


    <div class="container"> 

      <div class="columns">
        <div class="column is-three-fifths is-offset-one-fifth">
          <section  v-if="Object.keys(txnData).length">
                <hr>
                <b-table  :data="txnData" :columns="txnColumns"></b-table>

          </section>
        </div>
      </div>       
  
    </div>


  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

import vue2Dropzone from 'vue2-dropzone'
import 'vue2-dropzone/dist/vue2Dropzone.min.css'
import Arweave from 'arweave/web';
import base64url from "base64-url";

const arweave = Arweave.init({
  host: 'arweave.net', 
  port: 1984, 
  protocol: 'https'
  });



export default {
  name: 'app',
   data: function () {
    return {
      keyFile: null,
      jwk: null,
      tweet: '',
      tweetImg: null,
      arImg: null,
      arImgView: false,
      arImgViewTxn: false,
      arViewTxnId: false,
      isFullPage: false,
      isLoading:false,
      wallet_adddr: null,
      wallet_balance: null,
      txnStatus: '',
      txnColumns: [   
                    {
                        field: 'txn_id',
                        label: 'Txn Hash',
                        renderHtml: true
                    },
                    {
                        field: 'url',
                        label: 'Permaweb Url',
                        renderHtml: true
                    },
                    
                ],
      txnData: [ 
      ]
    }
    
  },
  created :function () { 
    

    const urlParams = new URLSearchParams(window.location.search);
    const txn = urlParams.get('txn');

   

    if(txn != '' && txn != null)   {
      this.arImgView = true
      this.arViewTxnId = txn
       arweave.transactions.getStatus(txn).then(status => {
         this.arImgViewTxn = status
        console.log(status);
        // 200
       })

       this.$axios.get('https://arweave.net/'+txn, {
        // timeout: 4000
        })
        .then(res => { 

          this.arImg = res.data
              
        }).catch(err => { 
          
            console.log(err)
        })

      
    }  
     

  },

   computed: {
    cleanUrl: function () {
      return [location.protocol, '//', location.host, location.pathname].join('');
    }
  },
  methods: {

    loadTxns() {

      arweave.wallets.jwkToAddress(this.jwk).then(async (address) =>   {

        const tx_ids = await arweave.arql({
              op: "and",
              expr1: {
                op: "equals",
                expr1: "from",
                expr2: address
              },
              expr2: {
                op: "equals",
                  expr1: "type",
                  expr2: "tweet-weave"
                
              }
        })

        tx_ids.forEach(tx_id => {
          this.txnData.unshift({txn_id: '<a href="https://viewblock.io/arweave/tx/'+ tx_id +'"  target="_blank"> '+tx_id+'</a>',
                         url: '<a href="'+ window.location.href+'?txn='+ tx_id +'"  target="_blank"> Snapshot</a>'
                        })
          
        });

      })  

    },

    confirmSave() {

       this.$dialog.confirm({
            message: 'Are you sure about this?',
            onConfirm: () => {
              this.submitArTxn()
            } 
        })

    },

    sumbitTweet() {

      if(this.tweet == '')
        return

      this.isLoading = true
      this.tweetImg = null
     
      let url =  'https://www.googleapis.com/pagespeedonline/v1/runPagespeed?screenshot=true&strategy=mobile&url='
      
      url = url + this.tweet.trim()


       this.$axios.get(url, {
        })
        .then(res => { 

          let base64 =  base64url.unescape(res.data.screenshot.data)
          // console.log(base64)
          this.tweetImg = 'data:image/jpeg;base64,'+base64
        
          this.isLoading = false         

                
        }).catch(err => { 
          this.isLoading = false
            console.log(err)
        })



    },

     async submitArTxn() { 

      let transaction =  await arweave.createTransaction({
          data: this.tweetImg
      }, this.jwk)

      transaction.addTag('Content-Type', 'image/jpeg');
      transaction.addTag('type', 'tweet-weave');
      await arweave.transactions.sign(transaction, this.jwk)

      await arweave.wallets.jwkToAddress(this.jwk).then((address) => {
           arweave.wallets.getBalance(address).then((balance) => {
          console.log(balance)
          this.wallet_balance = balance
          })
      })

      if(transaction.reward < this.wallet_balance )  {

        this.$toast.open({
                    duration: 5000,
                    message: 'Balance is too low for this transaction, please topup or use a different wallet',
                    position: 'is-bottom',
                    type: 'is-danger'
        })
        return;
      }

      const response = await arweave.transactions.post(transaction)
      console.log(response)

      this.txnData.unshift({txn_id: transaction.id,
                         fee: transaction.reward,
                         url: '<a href="'+ window.location.href+'?txn='+ transaction.id +'"  target="_blank"> Snapshot</a>'
                        })

      this.$toast.open({
                      duration: 5000,
                      message: 'Transaction sent.',
                      position: 'is-bottom',
                      type: 'is-success'
      })                  

    },

    loadJwk() {

      let reader = new FileReader();
      let jwk;
      let $this = this
      reader.addEventListener("loadend",  function(event) { 
        try {
          jwk = JSON.parse(event.target.result)
        }catch (e) {
          $this.$toast.open({
                      duration: 5000,
                      message: 'Please Use a valid key file',
                      position: 'is-bottom',
                      type: 'is-danger'
          })
          return false;
        }  

        $this.jwk = jwk

        $this.loadTxns()


      })
      reader.readAsText(this.keyFile)

    }

  },
  components: {
    HelloWorld,
    vueDropzone: vue2Dropzone

  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
