<template>
  <b-container fluid class="w-100 h-100" id="app">
    <b-row>
      <world-map/>
    </b-row>
    <b-row>
      <b-col>
        <scatter-plot></scatter-plot>
      </b-col>
    </b-row>
  </b-container>
</template>

<script lang="ts">
import Vue from 'vue';
import WorldMap from './components/WorldMap.vue';
import ScatterPlot from './components/ScatterPlot.vue';

export default Vue.extend({
  name: 'App',
  components: {
    WorldMap,
    ScatterPlot
  }
});
</script>

<style>
</style>
