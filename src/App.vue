<template>
  <div id="app">
      <data-visualisation />
  </div>
</template>

<script>
import DataVisualisation from './components/DataVisualisation.vue'

export default {
  name: 'App',
  components: {
    DataVisualisation
  }
}

</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  height: 100%;
}

#app {

}

</style>
