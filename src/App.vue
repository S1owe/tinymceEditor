<template>
  <div id="app">
    <div class="cont_editor">
      <Editor :idEditor="0"
              :typeEditor="1"
              :editorContent="editorFirst"
              @onBlur="updateEditorContent"
      />
    </div>
    <div class="cont_editor">
      <Editor :idEditor="1"
              :typeEditor="2"
              :editorContent="'<p>Hi, I`m fine</p>'"
      />
    </div>
    <div class="cont_editor">
      <Editor :idEditor="2"
              :typeEditor="0"
      />
    </div>
    <div class="cont_editor">
      <Editor :idEditor="3"
              :typeEditor="1"
      />
    </div>
  </div>
</template>

<script>
import Editor from './components/Editor.vue'

export default {
  name: 'App',
  data() {
    return {
      editorFirst: '<p class="pad">Hello new world! I`m front-end developer</p>'
    }
  },

  watch: {

  },

  methods: {
    updateEditorContent(data) {
      this.editorFirst = data
    }
  },

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
