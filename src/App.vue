<template>
  <div>
    <md-toolbar>
      <h1 class="md-title" style="flex: 1">{{project_name}}</h1>
      <md-button class="md-icon-button" @click.native="toggleRightSidenav">
          <md-icon>settings</md-icon>
      </md-button>
    </md-toolbar>
    <md-button @click.native="fetchData">request</md-button>
    <md-sidenav class="md-right" ref="rightSidenav" @open="open('Right')" @close="close('Right')">
      <md-toolbar>
        <div class="md-toolbar-container">
          <h3 class="md-title">Sidenav content</h3>
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
      </form>
    </md-sidenav>
  </div>
</template>

<script>
const remote = require('electron').remote
export default {
  name: 'app',
  data () {
    return {
      project_name: 'Desktop GitLab Snippets',
      domain: 'gitlab.com',
      private_token: 'private_token'
    }
  },
  methods: {
    fetchData() {
      remote.require("./main").fetchData(this.domain, this.private_token);
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
