<template>
  <div class="panel col-6">
    <h1>GitHub Theme Studio</h1>
    <div class="scroll">
      <label for="link">GitHub Link</label>
      <input type="text" class="form-control" id="link" v-model="link" placeholder="https://github.com/username/repository_name">
      <a class="btn btn-primary form-control" v-on:click="getGithubInfo">Fetch Info</a>
      <label>Themes</label>
      <div class="row themes">
        <a v-on:click="setTheme(1)" v-bind:class="{active: theme == 1, theme: true}">1</a>
        <a v-on:click="setTheme(2)" v-bind:class="{active: theme == 2, theme: true}">2</a>
        <a v-on:click="setTheme(3)" v-bind:class="{active: theme == 3, theme: true}">3</a>
        <a v-on:click="setTheme(4)" v-bind:class="{active: theme == 4, theme: true}">4</a>
        <a v-on:click="setTheme(5)" v-bind:class="{active: theme == 5, theme: true}">5</a>
      </div>
      <label>Project Image</label>
      <br>
      <input id="logo-img" type="file" v-on:change="imageUploaded('logoImg')" ref="logoImgInput">
      <label for="logo-img" class="img-label">
        <img src="https://source.unsplash.com/256x256/?logo,icon,nature" class="logo-img" ref="logoImg">
      </label>
      <br>
      <a class="remove" v-on:click="removeImage('logoImg', $event)">Remove</a>
      <br>
      <label>Background Image</label>
      <br>
      <input id="background-img" type="file" v-on:change="imageUploaded('backgroundImg')" ref="backgroundImgInput">
      <label for="background-img">
        <img src="https://source.unsplash.com/1600x900/?abstract" class="background-img" ref="backgroundImg">
      </label>
      <br>
      <a class="remove" v-on:click="removeImage('backgroundImg', $event)">Remove</a>
      <br>
      <label>Background Shift</label>
      <input type="range" min="1" max="100" value="100" class="slider form-control" v-model="backgroundShift" v-on:change="setTheme(theme)">
      <label>Title Size</label>
      <input type="range" min="50" max="100" value="90" class="slider form-control" v-model="titleSize" v-on:change="setTheme(theme)">
      <label>Description Shift</label>
      <input type="range" min="-150" max="150" value="0" class="slider form-control" v-model="descriptionShift" v-on:change="setTheme(theme)">
      <label>Logo Size</label>
      <input type="range" min="50" max="150" value="100" class="slider form-control" v-model="logoSize" v-on:change="setTheme(theme)">
      <label>Logo Shift</label>
      <input type="range" min="-100" max="100" value="0" class="slider form-control" v-model="logoShift" v-on:change="setTheme(theme)">
      <label for="color">Color</label>
      <input id="color" type="color" class="form-control color" v-model="color" v-bind:style="{'background-color': color}" v-on:change="setTheme(theme)">
    </div>
    <a class="save-button">To save image, right click on the preview</a>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      link: "",
      repo: {},
      backgroundShift: 75,
      descriptionShift: 0,
      color: this.randomColor(),
      theme: 0,
      titleSize: 90,
      logoSize: 100.0,
      logoShift: 0.0,
    };
  },
  methods: {
    randomColor: function() {
      let hsl = function hslToHex(h, s, l) {
        l /= 100;
        const a = s * Math.min(l, 1 - l) / 100;
        const f = n => {
          const k = (n + h / 30) % 12;
          const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
          return Math.round(255 * color).toString(16).padStart(2, '0');   // convert to Hex and prefix "0" if needed
        };
        return `#${f(0)}${f(8)}${f(4)}`;
      };
      return hsl((Math.random() * 100), 100, 50);
    },
    removeImage: function(name) {
      this.$refs[name].src = "";
    },
    imageUploaded: function(name) {
      var files = this.$refs[name + "Input"];

      if (files.files && files.files[0]) {
        var img = this.$refs[name];
        img.onload = () => {
          URL.revokeObjectURL(img.src);
          this.setTheme(this.theme);
        }

        img.src = URL.createObjectURL(files.files[0]);
      }
    },
    setTheme: function(n) {
      if (n != 1) { return; }
      if (this.repo.created_at == null) { alert("Enter github link first"); return; }
      this.theme = n;
      this.eventHub.$emit('setTheme', {
        theme: n,
        data: this,
        logoImg: this.$refs.logoImg,
        backgroundImg: this.$refs.backgroundImg
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
      request.setRequestHeader("Accept", "application/vnd.github.mercy-preview+json");

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
      var link = document.createElement('a');
      link.download = 'filename.png';
      link.href = document.getElementById('canvas').toDataURL()
      link.click();
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
  min-height: 150px;
  min-width: 250px;
  border-radius: 10px;
  background-color: var(--grey);
  cursor: pointer;
}
.logo-img {
  max-width: 70%;
  max-height: 150px;
  min-height: 50px;
  min-width: 50px;
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
.theme:first-child {
  cursor: pointer !important;
}
.themes .theme {
  cursor: not-allowed;
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
input[type=file] {
  display: none;
}

/* Image Uploading */
.remove {
  padding: 6px 16px 4px 16px;
  margin-left: 10px;
  cursor: pointer;
  color: black;
  text-decoration: none;
  margin-top: -20px;
}
.remove:hover {
  background-color: var(--grey);
  border-radius: 10px;
}

.img-label {
  position: relative;
}

.save-button {
  color: var(--dark-grey);
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
