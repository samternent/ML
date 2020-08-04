<template>
  <div id="app">
    <capture/>
  </div>
</template>

<script>
import Capture from './components/Capture';

export default {
  name: 'App',
  components: {
    Capture,
  }
}
</script>

<style>
#app {
  background-color: #fff;
  display: flex;
  height: 100vh;
  width: 100vw;
}
</style>
