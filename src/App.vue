<template>
  <div>
    <md-toolbar>
      <h1 class="md-title" style="flex: 1">{{project_name}}</h1>
      <md-button class="md-icon-button" @click.native="fetchData">
        <md-icon>refresh</md-icon>
      </md-button>
      <md-button class="md-icon-button" @click.native="toggleRightSidenav">
        <md-icon>settings</md-icon>
      </md-button>
    </md-toolbar>
    <md-layout md-ugger v-if="!checkPrivateTokenHasData()">
        <md-card>
          <md-card-header>
            初期設定
          </md-card-header>
          <md-card-content>
            <ol>
              <li>右上の歯車マークを押す</li>
              <li>利用したいgitlabのドメインを入力</li>
              <li>利用したいドメインのgitlabから発行されているPrivateTokenを入力</li>
              <li>SaveConfigrationsを押し保存する</li>
              <li>保存後、{{project_name}}を始めるボタンを押す</li>
            </ol>
          </md-card-content>
          <md-card-actions>
            <md-button @click.native="fetchData">
              {{project_name}} を始める
           </md-button>
          </md-card-actions>
        </md-card>
    </md-layout>
    </md-layout>
    <md-layout md-gutter v-else-if="checkSnippetHasData()">
      <md-layout md-column md-flex="25">
        <md-list v-for="s in snippets" >
          <md-list-item @click.native="fetchSnippet(s)">
            <h4>{{s.title}}</h4>
            <md-divider></md-divider>
          </md-list-item>
        </md-list>
      </md-layout>
      <md-layout md-column md-gutter>
        <md-card>
          <md-card-header>
            <div class="md-title">{{snippet.file_name}}</div>
          </md-card-header>
          <md-card-content>
            <pre>{{snippet.contents}}</pre>
          </md-card-content>
          <md-card-actions>
            <md-button class="md-icon-button" @click.native="toClipboard()">
              <md-icon>content_copy</md-icon>
            </md-button>
          </md-card-actions>
        </md-card>
      </md-layout>
    </md-layout>
    <md-layout v-else>
      <md-spinner :md-size="150" md-indeterminate></md-spinner>
    </md-layout>
    <md-sidenav class="md-right" ref="rightSidenav" @open="open('Right')" @close="close('Right')">
      <md-toolbar>
        <div class="md-toolbar-container">
          <h3 class="md-title">Configrations</h3>
        </div>
      </md-toolbar>
      <form novalidate @submit.stop.prevent="submit">
        <md-input-container>
          <label>Domain</label>
          <md-input v-model="inputted_domain"></md-input>
        </md-input-container>
        <md-input-container  md-has-password>
          <label>Private Token</label>
          <md-input type="password" v-model="inputted_private_token"></md-input>
        </md-input-container>
        <p>---</p>
        <md-input-container>
          <label>ProxyDomain</label>
          <md-input v-model="inputted_proxy_domain"></md-input>
        </md-input-container>
        <md-input-container>
          <label>ProxyPort</label>
          <md-input v-model="inputted_proxy_port"></md-input>
        </md-input-container>
        <md-input-container>
          <label>UserName</label>
          <md-input v-model="inputted_use_name"></md-input>
        </md-input-container>
        <md-input-container md-has-password>
          <label>PassWord</label>
          <md-input type="password" v-model="inputted_password"></md-input>
        </md-input-container>
        <md-button class="md-primary" @click.native="saveConfigrations()">Save Configrations</md-button>
      </form>
    </md-sidenav>
  </div>
</template>

<script>
const remote = require('electron').remote
const clipboard = require('electron').clipboard
const utf8 = require('utf-8')
const localStorage = window.localStorage;
export default {
  name: 'app',
  data () {
    return {
      project_name: 'Desktop GitLab Snippets',
      domain: 'gitlab.com',
      inputted_domain: '',
      private_token: 'private_token',
      inputted_private_token: 'private_token',
      proxy_url: '',
      inputted_proxy_domain: '',
      proxy_domain: '',
      inputted_proxy_port: '',
      proxy_port: '',
      inputted_proxy_user: '',
      proxy_user: '',
      inputted_proxy_password: '',
      proxy_password: '',
      snippets: [],
      snippet: {file_name: "", contents: ""}
    }
  },
  created() {
    this.renewConfigrations()
    this.inputted_domain = this.domain
    this.inputted_private_token = this.private_token
    this.inputted_proxy_domain = this.proxy_domainp
    this.inputted_proxy_port = this.proxy_port
    this.inputted_proxy_user = this.proxy_user
    this.inputted_proxy_password = this.proxy_password
    this.fetchData(true)
  },
  methods: {
    renewConfigrations() {
      this.domain = localStorage.getItem("domain") || "gitlab.com"
      this.private_token = localStorage.getItem("private_token") || ""
      this.proxy_domain = localStorage.getItem("proxy_domain") || ""
      this.proxy_port = localStorage.getItem("proxy_port") || ""
      this.proxy_user = localStorage.getItem("proxy_user") || ""
      this.proxy_password = localStorage.getItem("proxy_password") || ""
    },
    saveConfigrations() {
      localStorage.setItem("domain", this.inputted_domain)
      localStorage.setItem("private_token", this.inputted_private_token)
      localStorage.setItem("proxy_domain", this.inputted_proxy_domain)
      localStorage.setItem("proxy_port", this.inputted_proxy_user)
      localStorage.setItem("proxy_password", this.inputted_proxy_password)
      this.renewConfigrations()
    },
    checkPrivateTokenHasData() {
      return (this.private_token != '')
    },
    checkSnippetHasData() {
      return (this.snippet.content != "" && this.snippet.file_name != "")
    },
    generateConfWithProxy() {
        if (this.proxy_domain == "") {
            return {}
        }
        conf = {}
        conf["host"] =  this.proxy_domain
        if (this.proxy_port == "") {
           conf["port"] == 80
        } else {
           conf["port"] = this.proxy_port
        }
        if (this.proxy_user == "" || this.proxy_password == "") {
           return conf
        }
        conf["auth"]["username"] = this.proxy_user
        conf["auth"]["password"] = this.proxy_password
    },
    fetchData(initSnnipet=false) {
      let client = remote.require("./main").getClient()
      let conf = this.generateConfWithProxy()
      conf["baseURL"] = `https://${this.domain}`
      conf["headers"] = {"PRIVATE-TOKEN": this.private_token}
      client.get("/api/v4/snippets", conf)
      .then( response => {
        this.snippets = response.data
        if (initSnnipet) {
           this.fetchSnippet(this.snippets[0])
        }
      }).catch( error => {
        console.log(error)
      })
    },
    fetchSnippet(snippet) {
      let client = remote.require("./main").getClient()
      let conf = this.generateConfWithProxy()
      conf["baseURL"] = snippet.raw_url
      conf["headers"] = {"PRIVATE-TOKEN": this.private_token}
      client.get("", conf)
      .then( response => {
        this.snippet = {file_name: snippet.file_name, contents: response.data}
      }).catch( error => {
        console.log(error)
      })
    },
    toClipboard() {
      clipboard.writeText(this.snippet.contents)
    },
    toggleRightSidenav() {
      this.$refs.rightSidenav.toggle();
    },
    closeRightSidenav() {
      this.$refs.rightSidenav.close();
    },
    open(ref) {
      console.log('Opened: ' + ref);
    },
    close(ref) {
      console.log('Closed: ' + ref);
    }
  }
}
</script>
<style>
/* see https://github.com/marcosmoura/vue-material/issues/51 */
body {
  overflow-x: hidden;
}
</style>
