<template>
  <div id="app">
    <Index msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import Index from './page/index.vue'

export default {
  name: 'App',
  components: {
    Index
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
