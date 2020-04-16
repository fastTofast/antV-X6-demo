<template>
  <div id="app">
    <editor-com></editor-com>
  </div>
</template>

<script>
import EditorCom from './components/EditorCom.vue'

export default {
  name: 'App',
  components: {
    EditorCom
  }
}
</script>

<style lang="scss">
html,
body {
  margin: 0;
  border: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
