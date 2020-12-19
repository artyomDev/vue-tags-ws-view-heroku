<template>
  <Tags msg="Welcome to Artem's Valiu UI Test !"/>
</template>

<script>
import Tags from './components/Tags.vue'

export default {
  name: 'App',
  components: {
    Tags
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
  margin-top: 100px;
}
</style>
