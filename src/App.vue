<template>
  <div id="app">
    <div class="row">
        <Panel/>
        <div class="col">
          <Preview/>
        </div>
    </div>
    <div id="nomobile">
      <a>No Mobile Support.</a>
    </div>
  </div>
</template>

<script>
import Preview from './components/Preview.vue'
import Panel from './components/Panel.vue'

export default {
  name: 'App',
  components: {
    Panel,
    Preview
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Open+Sans&display=swap');
@import url('https://fonts.googleapis.com/css2?family=PT+Sans+Caption:wght@700&display=swap');
:root {
    --primary: #008fee;
    --primary-dark: #1181c2;
    --secondary: #ebd619;
    --secondary-dark: #c4b31a;

    --dark-grey: #424242;
    --grey: rgb(223, 223, 223);
    --light-grey: rgb(247, 247, 247);
}

body {
  overflow: hidden;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

@media only screen and (max-width: 1260px) {
  #app .row {
    display: none;
  }
  #nomobile a {
    position: absolute;
    left: 0px;
    right: 0px;
    top: 0px;
    bottom: 0px;
    margin: auto;
  }
}
</style>
