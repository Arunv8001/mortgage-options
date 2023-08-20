<template>
  <div class="container">
    <Form />
  </div>
</template>
<script>
import Form from './components/Form.vue'

export default {
  name: 'App',
  components: {
    Form
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  background: #eee;
  min-height: 100vh;
}

body {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.container {
  margin: 0 auto;
  max-width: 960px;
  padding: 1rem;
}
</style>
