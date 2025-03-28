<template>
  <div id="app">
    <ARPatientUI />
  </div>
</template>

<script>
import ARPatientUI from './components/ARPatientUI.vue'

export default {
  name: 'App',
  components: {
    ARPatientUI
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background-color: #072a34;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  box-sizing: border-box;
}
</style>
