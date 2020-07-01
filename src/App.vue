<template>
  <div id="app">
    <div class="app-container">
      <Content />
    </div>
  </div>
</template>

<script>
  import Content from './components/Content.vue'


  export default {
    name: 'App',
    components: {
      Content
    }
  }
</script>

<style>
  * {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body {
    margin: 60px 20%;
    background-color: #325a83;
  }
</style>