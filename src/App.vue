<template>
  <div id="app">
    <div class="cont_editor">
      <Editor :numEditor="0"/>
    </div>
    <div class="cont_editor">
      <Editor :numEditor="1"/>
    </div>
    <div class="cont_editor">
      <Editor :numEditor="2"/>
    </div>
  </div>
</template>

<script>
import Editor from './components/Editor.vue'

export default {
  name: 'App',
  components: {
    Editor
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
  margin-top: 60px;
}

  .cont_editor {
    margin-bottom: 50px;
  }
</style>
