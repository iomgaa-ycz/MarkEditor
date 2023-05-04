<template>
  <div id="app">
    <ProseMirrorEditor />
  </div>
</template>

<script>
import ProseMirrorEditor from './components/ProseMirrorEditor.vue';

export default {
  name: 'App',
  components: {
    ProseMirrorEditor,
  },
};
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

.prosemirror-editor {
  border: 1px solid gray;
  padding: 8px;
  min-height: 300px;
}
</style>

