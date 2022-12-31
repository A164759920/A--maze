<template>
  <div id="app">
    <HelloWorld></HelloWorld>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style lang="scss">
* {
  margin: 0px;
  padding: 0px;
}

#app,
html,
body {
  height: 100%;
}
</style>
