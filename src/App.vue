<template>
  <b-container fluid class="w-100 h-100" id="app">
    <h1 class="text-center">Estimation of global temperature change</h1>
    <h5 class="text-center mb-5">Estimation of the rate of increase in temperature over the years 
      according to the emissions produced by the population and those projected for the future.</h5>
    <b-row>
      <world-map/>
    </b-row>
    <b-row>
      <b-col cols="6">
        <scatter-plot></scatter-plot>
      </b-col>
      <b-col cols="6">
        <line-chart></line-chart>
      </b-col>
    </b-row>
  </b-container>
</template>

<script lang="ts">
import Vue from 'vue';
import WorldMap from './components/WorldMap.vue';
import ScatterPlot from './components/ScatterPlot.vue';
import LineChart from './components/LineChart.vue';

export default Vue.extend({
  name: 'App',
  components: {
    WorldMap,
    ScatterPlot,
    LineChart
  }
});
</script>

<style>
</style>
