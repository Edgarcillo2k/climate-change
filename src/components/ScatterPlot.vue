<template>
    <b-container>
        <b-form-select v-model="yearSelected" :options="years" @input="changeYear" required></b-form-select>
        <svg id="scatter" :viewBox="`0 0 ${width} ${height}`">
        </svg>
    </b-container>
</template>

<script lang="ts">
import Vue from 'vue';
import * as d3 from 'd3v4';
export default Vue.extend({
    name: 'WorldMap',
    data(){
        return {
            width: window.innerWidth,
            height: window.innerHeight,
            projection: undefined,
            svg: undefined as any,
            path: undefined as any,
            years: [],
            yearSelected: 1949,
            yearsScales: {},
            x: undefined as any,
            y: undefined as any
        }
    },
    methods: {
        changeYear(){
            this.y.domain([0, this.yearsScales[this.yearSelected].yMax])
                .range([ this.height/2, 0]);
            this.x.domain([0, this.yearsScales[this.yearSelected].xMax])
                .range([ 0, this.width/2 ]);
            this.svg.select('.x-axis').transition().duration(1500).call(d3.axisBottom(this.x));
            this.svg.select('.y-axis').transition().duration(1500).call(d3.axisLeft(this.y));
            // Add dots
            this.svg.selectAll("circle").transition().duration(1500)
                .attr("cx", (d) => {
                    if(d[1][this.yearSelected])
                        return this.x(d[1][this.yearSelected].population); 
                }).attr("cy", (d) => {
                    if(d[1][this.yearSelected])
                        return this.y(d[1][this.yearSelected].emissions); 
                }).attr("r", 5);
        },
        ready(error: any, scatterData: any) {
            const scatterDataById = {};

            scatterData.forEach((d) => {
                const newObj = {};
                newObj.emissions = +d['Annual CO2 emissions (per capita)'];
                newObj.precipitation = +d['Average monthly precipitation'];
                newObj.population = +d['Total population estimates, 1970-2100 (IIASA (2015))']
                if(!scatterDataById[d.Code]){
                    scatterDataById[d.Code] = {};
                }
                if(!this.yearsScales[d.Year])
                    this.yearsScales[d.Year] = {xMax: 0, yMax: 0, xMin: Number.MAX_SAFE_INTEGER, yMin: Number.MAX_SAFE_INTEGER};
                scatterDataById[d.Code][d.Year] = newObj;
                const yearScales = this.yearsScales[d.Year];
                if(newObj.emissions < yearScales.yMin)
                    yearScales.yMin = newObj.emissions;
                if(newObj.emissions > yearScales.yMax)
                    yearScales.yMax = newObj.emissions;
                if(newObj.population < yearScales.xMin)
                    yearScales.xMin = newObj.population;
                if(newObj.population > yearScales.xMax)
                    yearScales.xMax = newObj.population;
            });

            // Add X axis
            this.x = d3.scaleLinear()
                .domain([0, this.yearsScales[this.yearSelected].xMax])
                .range([ 0, this.width/2 ]);

            this.svg.append("g")
                .attr("transform", "translate(0," + this.height/2 + ")")
                .attr("class","x-axis")
                .call(d3.axisBottom(this.x));

            // Add Y axis
            this.y = d3.scaleLinear()
                .domain([0, this.yearsScales[this.yearSelected].yMax])
                .range([ this.height/2, 0]);

            this.svg.append("g")
                .attr("class","y-axis")
                .call(d3.axisLeft(this.y));

            // Add dots
            this.svg.append('g')
                .selectAll("dot")
                .data(Object.entries(scatterDataById))
                .enter()
                .append("circle")
                .attr("cx", (d) => {
                    if(d[1][this.yearSelected])
                        return this.x(d[1][this.yearSelected].population); 
                }).attr("cy", (d) => {
                    if(d[1][this.yearSelected])
                        return this.y(d[1][this.yearSelected].emissions); 
                }).attr("r", 5)
                .style("fill", (d) => { return '#007EC6'} );
        }
    },
    mounted(){
        for(let i = 1949; i <= 2014; i++){
            this.years.push(i);
        }
        const format = d3.format(",");

        const margin = {top: 10, right: 30, bottom: 30, left: 60},
            width = 460 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        this.svg = d3.select("#scatter")
            .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

        d3.queue()
            .defer(d3.csv, "https://raw.githubusercontent.com/Edgarcillo2k/climate-change/master/src/assets/scatter_plot_data.csv?token=AKWLPXEQ6RAZDOOF3DT2DT3BVOZJY")
            .await(this.ready);        
    }
})
</script>
<style>

</style>