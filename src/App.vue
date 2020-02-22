<template>
  <div id="app">
    <OLMap />
  </div>
</template>

<script>
import OLMap from "./components/OLMap.vue";

export default {
  name: "App",
  components: {
    OLMap
  }
};
</script>

<style>
html,
body {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  width: 100%;
  height: 100%;
}
</style>
