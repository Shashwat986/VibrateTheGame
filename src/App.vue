<template>
  <div id="app" class="is-relative">
    <router-view/>
    <ad></ad>
  </div>
</template>

<script>
import Ad from './components/Ad.vue';
export default {
  components: {
    ad: Ad
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
