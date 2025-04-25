<template>
  <div id="app">
    <Game2048 />
  </div>
</template>

<script>
import Game2048 from './components/Game2048.vue'

export default {
  name: 'App',
  components: {
    Game2048
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0;
  padding: 0;
  height: 100vh;
  background-color: #faf8ef;
}

body {
  margin: 0;
  padding: 0;
}
</style> 