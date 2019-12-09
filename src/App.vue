<template>
  <div id="app">
    <dwt/>
  </div>
</template>

<script>
import dwt from "./components/dwt.vue";

export default {
  name: "app",
  components: {
    dwt
  },
};
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
