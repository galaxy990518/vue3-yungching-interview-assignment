<template>
  <div id="app">
    <Toolbar />
    <router-view></router-view>
  </div>
</template>

<script setup>
import Toolbar from './components/Toolbar.vue'
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
}

#app {
  font-family: Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
}
</style>
