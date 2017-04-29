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
    <md-layout md-gutter v-if="checkSnippetHasData()">
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
          <md-input v-model="domain"></md-input>
        </md-input-container>
        <md-input-container  md-has-password>
          <label>Private Token</label>
          <md-input type="password" v-model="private_token"></md-input>
        </md-input-container>
        <p>---</p>
        <md-input-container>
          <label>ProxyUrl</label>
          <md-input v-model="proxy_url"></md-input>
        </md-input-container>
        <md-input-container  md-has-password>
          <label>UserName</label>
          <md-input v-model="use_name"></md-input>
        </md-input-container>
        <md-input-container>
          <label>PassWord</label>
          <md-input type="password" v-model="password"></md-input>
        </md-input-container>
      </form>
    </md-sidenav>
  </div>
</template>

<script>
const remote = require('electron').remote
const clipboard = require('electron').clipboard
const utf8 = require('utf-8')
export default {
  name: 'app',
  data () {
    return {
      project_name: 'Desktop GitLab Snippets',
      domain: 'gitlab.com',
      private_token: 'private_token',
      proxy_url: '',
      snippets: [],
      snippet: {file_name: "", contents: ""}
    }
  },
  created() {
    this.fetchData(true)
  },
  methods: {
    checkSnippetHasData() {
        return (this.snippet.content != "" && this.snippet.file_name != "")
    },
    fetchData(initSnnipet=false) {
      this.private_token = 'bMEfjjQHhSLm4ohZzP6y'
      var request = remote.require("./main").fetchData(this.domain, this.private_token, this.proxy_url)
      request.on('response', (response) => {
        console.log(`STATUS: ${response.statusCode}`)
        console.log(`HEADERS: ${JSON.stringify(response.headers)}`)
        response.on('data', (chunk) => {
          this.snippets = JSON.parse(chunk)
          if (initSnnipet) {
             this.fetchSnippet(this.snippets[0])
          }
        })
        response.on('end', () => {
          console.log('No more data in response.')
        })
      })
      request.end()
    },
    fetchSnippet(snippet) {
      this.private_token = 'bMEfjjQHhSLm4ohZzP6y'
      var request = remote.require("./main").fetchSnippet(this.domain, this.private_token, snippet.raw_url, this.proxy_url)
      request.on('response', (response) => {
        console.log(`STATUS: ${response.statusCode}`)
        console.log(`HEADERS: ${JSON.stringify(response.headers)}`)
        response.on('data', (chunk) => {
          this.snippet = {file_name: snippet.file_name, contents: utf8.getStringFromBytes(chunk)}
        })
        response.on('end', () => {
          console.log('No more data in response.')
        })
      })
      request.end()
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
