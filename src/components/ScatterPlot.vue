<template>
    <b-container>
        <h2 class="mt-5 mb-5">Dispertion diagram for the precipitation per country</h2>
        <b-form-select id="yearList" v-model="yearSelected" :options="countries" @input="changeYear" required></b-form-select>

        <svg id="scatter" :viewBox="`0 0 ${width} ${height}`">
        </svg>
    </b-container>
</template>

<script lang="ts">
import Vue from 'vue';
import * as d3 from 'd3v4';
import d3Tip from "d3-tip";
import { select } from 'd3';
export default Vue.extend({
    name: 'WorldMap',
    data(){
        return {
            width: window.innerWidth,
            height: window.innerHeight,
            projection: undefined,
            svg: undefined as any,
            path: undefined as any,
            countries: [],
            yearSelected: undefined,
            countryScales: {},
            x: undefined as any,
            y: undefined as any,
            tip: undefined as any
        }
    },
    methods: {
        changeYear(){
            this.y.domain([0, this.countryScales[this.yearSelected].yMax])
                .range([ this.height/2, 0]);
            this.x.domain([0, this.countryScales[this.yearSelected].xMax])
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
                }).attr("r", (d) => {
                    if(d[1][this.yearSelected])
                        return Math.pow(d[1][this.yearSelected].size + 1 , 2.75);
                    return 0;
                });
        },
        ready(error: any, scatterData: any) {
            const scatterDataById = {};

            scatterData.forEach((d) => {
                const newObj = {};
                newObj.emissions = +d['Annual CO2 emissions (per capita)'];
                newObj.precipitation = +d['Average monthly precipitation'];
                newObj.population = +d['Total population estimates, 1970-2100 (IIASA (2015))']
                newObj.year = +d['Year'];
                newObj.size = +d['size'];
                if(!scatterDataById[d.Year])
                    scatterDataById[d.Year] = {};
                if(!this.countryScales[d.Code])
                    this.countryScales[d.Code] = {xMax: 0, yMax: 0, xMin: Number.MAX_SAFE_INTEGER, yMin: Number.MAX_SAFE_INTEGER, 
                                                  pMax: 0, pMin: Number.MAX_SAFE_INTEGER};
                
                scatterDataById[d.Year][d.Code] = newObj;
                const countryScales = this.countryScales[d.Code];
                if(newObj.emissions < countryScales.yMin)
                    countryScales.yMin = newObj.emissions;
                if(newObj.emissions > countryScales.yMax)
                    countryScales.yMax = newObj.emissions;
                if(newObj.population < countryScales.xMin)
                    countryScales.xMin = newObj.population;
                if(newObj.population > countryScales.xMax)
                    countryScales.xMax = newObj.population;
                if(newObj.size < countryScales.pMax)
                    countryScales.pMax = newObj.size;
                if(newObj.size > countryScales.pMin)
                    countryScales.pMin = newObj.size;
            });
            this.countries = Object.keys(this.countryScales);
            this.yearSelected = this.countries[0];

            const tip = this.tip;

            // Add X axis
            this.x = d3.scaleLinear()
                .domain([0, this.countryScales[this.yearSelected].xMax])
                .range([ 0, this.width/2 ]);

            this.svg.append("g")
                .attr("transform", "translate(0," + this.height/2 + ")")
                .attr("class","x-axis")
                .call(d3.axisBottom(this.x));

            

            // Add Y axis
            this.y = d3.scaleLinear()
                .domain([0, this.countryScales[this.yearSelected].yMax])
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
                }).attr("r", (d) => {
                    if(d[1][this.yearSelected])
                        return Math.pow(d[1][this.yearSelected].size + 1 , 2.75);
                    return 0;
                }).style("fill", (d) => { return '#007EC6'} )
                .on('mouseover',function(d){
                    tip.show(d,this);

                    d3.select(this)
                        .style("opacity", 1)
                        .style("stroke","white")
                        .style("stroke-width",3);
                    })
                .on('mouseout', function(d){
                    tip.hide(d);

                    select(this)
                        .style("opacity", 0.8)
                        .style("stroke","white")
                        .style("stroke-width",0.3);
                });

                

                this.svg.append("path")
                .datum(Object.entries(scatterDataById))
                // .datum(topojson.mesh(data.features, function(a, b) { return a !== b; }))
                .attr("class", "names")
                .attr("d", this.path);
        }
    },
    mounted(){
        const format = d3.format(",");

        this.tip = d3Tip()
            .attr('class', 'd3-tip')
            .offset([-10, 0])
            .html((d) => {
                return "<strong>Population: </strong><span class='details'>" + Math.floor(d[1][this.yearSelected].population) + "<br></span>" + 
                "<strong>Emissions: </strong><span class='details'>" + d[1][this.yearSelected].emissions +"<br></span>" +
                "<strong>Precipitation: </strong><span class='details'>" + d[1][this.yearSelected].precipitation +"<br></span>" + 
                "<strong>Year: </strong><span class='details'>" + d[1][this.yearSelected].year +"</span>";
            });

        const margin = {top: 10, right: 30, bottom: 30, left: 60},
            width = 460 - margin.left - margin.right,
            height = 400 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        this.svg = d3.select("#scatter")
            .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

        this.svg.append("text")
                    .attr("class", "yaxis_label")
                    .attr("text-anchor", "middle")  // this makes it easy to centre the text as the transform is applied to the anchor
                    .attr("transform", "translate("+ (-50) +","+(this.height/4)+")rotate(-90)")  // text is drawn off the screen top left, move down and out and rotate
                    .text("CO2 Emissions");
        this.svg.call(this.tip);
        this.svg.append("text")
                    .attr("class", "yaxis_label")
                    .attr("text-anchor", "middle")  // this makes it easy to centre the text as the transform is applied to the anchor
                    .attr("transform", "translate("+ (this.width/4) +","+(this.height/2 + 50)+")")  // text is drawn off the screen top left, move down and out and rotate
                    .text("Population");
        this.svg.call(this.tip);

        d3.queue()
            .defer(d3.csv, "https://raw.githubusercontent.com/Edgarcillo2k/climate-change/master/src/assets/scatter_plot_data.csv?token=AKWLPXEQ6RAZDOOF3DT2DT3BVOZJY")
            .await(this.ready);        
    }
})
</script>
<style>

#yearList {
    padding: 10px;
    margin: 10px;
    width: 10%;
    background: #FFFFFF;
    box-shadow: 0px 4px 14px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
}

</style>