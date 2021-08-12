<template>
  <div id="app">
    <editor/>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Editor from "./components/Editor.vue";

export default Vue.extend({
  name: "App",
  components: {
    Editor
  },
});
</script>

<style lang="scss">
body {
  font-size: 62.5%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 12px;
  margin-right: 12px;
  background: whitesmoke;
}
</style>
