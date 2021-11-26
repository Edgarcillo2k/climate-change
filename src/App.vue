<template>
  <div id="app">
    <world-map/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';
import WorldMap from './components/WorldMap.vue';

export default Vue.extend({
  name: 'App',
  components: {
    WorldMap
  }
});
</script>

<style>
</style>
