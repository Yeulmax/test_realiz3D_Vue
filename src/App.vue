<template>
  <div id="app">
      <group-type-list />
  </div>
</template>

<script>
import GroupTypeList from './components/GroupTypeList.vue'

export default {
  name: 'App',
  components: {
    GroupTypeList
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
