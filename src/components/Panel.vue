<template>
  <div class="panel col-6">
    <h1>GitHub Theme Creator</h1>
    <div class="scroll">
      <label for="link">GitHub Link</label>
      <input type="text" class="form-control" id="link" v-model="link" placeholder="https://github.com/darkzek/RustCraft">
      <a class="btn btn-primary form-control" v-on:click="getGithubInfo">Fetch Info</a>
      <div class="row themes">
        <a v-on:click="setTheme(1)" v-bind:class="{active: theme == 1, theme: true}">1</a>
        <a v-on:click="setTheme(2)" v-bind:class="{active: theme == 2, theme: true}">2</a>
        <a v-on:click="setTheme(3)" v-bind:class="{active: theme == 3, theme: true}">3</a>
        <a v-on:click="setTheme(4)" v-bind:class="{active: theme == 4, theme: true}">4</a>
        <a v-on:click="setTheme(5)" v-bind:class="{active: theme == 5, theme: true}">5</a>
      </div>
      <label>Project Image</label>
      <br>
      <img src="@/assets/logo.png" class="logo-img" ref="background">
      <br>
      <label>Background Image</label>
      <br>
      <img src="@/assets/unknown.png" class="background-img" ref="background">
      <br>
      <label>Background Shift</label>
      <input type="range" min="1" max="100" value="100" class="slider form-control" v-model="backgroundShift" v-on:change="setTheme(theme)">
      <label>Title Size</label>
      <input type="range" min="50" max="100" value="90" class="slider form-control" v-model="titleSize" v-on:change="setTheme(theme)">
      <label for="color">Color</label>
      <input id="color" type="color" class="form-control color" v-model="color" v-bind:style="{'background-color': color}" v-on:change="setTheme(theme)">
    </div>
    <a v-on:click="save" class="btn btn-secondary form-control">Save</a>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      link: "https://github.com/darkzek/rustcraft",
      repo: {},
      backgroundShift: 100,
      color: "#b1333a",
      theme: 0,
      titleSize: 90
    };
  },
  methods: {
    setTheme: function(n) {
      if (this.repo == {}) { alert("Enter github link first"); }
      this.theme = n;
      this.eventHub.$emit('setTheme', {
        theme: n,
        data: this,
        background: this.$refs.background
      });
    },
    getGithubInfo: function() {

      // Get useful part of url
      let searchQuery = "github.com/";
      if (!this.link.includes(searchQuery)) {
        alert("Invalid URL");
        return;
      }
      let cullIndex = this.link.indexOf(searchQuery) + searchQuery.length;
      let address = this.link.substr(cullIndex);

      // Remove tailing slash
      if (address.endsWith("/")) {
        address = address.substr(0, address.length - 1);
      }

      var request = new XMLHttpRequest();
      request.open('GET', 'https://api.github.com/repos/' + address, true);

      request.onload = function() {
        if (request.status >= 200 && request.status < 400) {
          // Success!
          this.repo = JSON.parse(request.responseText);
          this.theme = 1;
          this.setTheme(this.theme);
        } else {
          alert("Invalid url");
        }
      }.bind(this);

      request.onerror = function() {
          alert("Connection error to GitHub");
      };

      request.send();
    },
    save: function() {
      let downloadLink = document.createElement('a');
      downloadLink.setAttribute('download', 'Github_Social.png');

      let canvas = document.getElementById('canvas');
      canvas.toBlob(function(blob) {
      let url = URL.createObjectURL(blob);
      downloadLink.setAttribute('href', url);
      downloadLink.click();
    });
    }
  }
}
</script>

<style scoped>
.panel {
  background-color: var(--light-grey);
  padding: 40px 30px 30px 140px;
  position: sticky;
  max-height: 100vh;
  max-width: 600px;
}
img {
  border-radius: 10px;
  background-color: var(--grey);
  cursor: pointer;
}
.background-img {
  max-width: 100%;
  max-height: 200px;
  min-height: 200px;
  min-width: 50%;
  border-radius: 10px;
  background-color: var(--grey);
  cursor: pointer;
}
.logo-img {
  max-width: 70%;
  max-height: 150px;
  min-height: 50px;
  min-width: 20%;
  border-radius: 10px;
  background-color: var(--grey);
  cursor: pointer;
}
.panel *:not(label) {
  margin-bottom: 20px;
}
.themes {
  display: flex;
  justify-content: center;
}
.themes .theme {
  border-radius: 100%;
  padding: 5px 12px 5px 12px;
  margin-left: 5px;
  margin-right: 5px;
  background-color: var(--grey);
  color: black;
  cursor: pointer;
  text-decoration: none;
  transition: ease-in 0.1s all;
}
.active {
  background-color: var(--primary) !important;
  color: white !important;
}
.btn-primary {
  color: white;
  background-color: var(--primary);
  border-color: var(--primary);
}
.scroll {
  overflow: scroll;
  height: calc(100% - 125px);
  overflow-x: hidden;
}
.color {
  cursor: pointer;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
