<template>
    <b-container>
        <b-row>
            <h3 class="ml-3">Line chart of climate change</h3>
        </b-row>
        <b-row class="h-100">
            <h2 class="mt-5 mb-5"></h2>
            <div id="linecharter"></div>
        </b-row>
    </b-container>
</template>

<script lang="ts">

/*
Contributors: Script taken from https://www.d3-graph-gallery.com/graph/line_brushZoom.html
Thanks to Yan Holtz for sharing the code and examples with us.
*/

import Vue from 'vue';
import * as d3 from 'd3v4';

export default Vue.extend({
    name: 'LineChart',
    data(){
        return {
            width: window.innerWidth,
            height: window.innerHeight,
            svg: undefined as any,
            path: undefined as any,
            x: undefined as any,
            y: undefined as any,
            extent: undefined as any,
            idleTimeout: undefined as any,
            clip: undefined as any,
            xAxis: undefined as any,
            yAxis: undefined as any,
            line: undefined as any,
            brush: undefined as any
        }
    },
    methods: {

        //This method allows to create a little "sleep" when updating the chart.        
        idled(){
            this.idleTimeout = null;
            return this.idleTimeout;
        },

        updateChart() {

            
                // What are the selected boundaries?
                this.extent = d3.event.selection;

                // If no selection, back to initial coordinate. Otherwise, update X axis domain
                if(!this.extent){
                    if (!this.idleTimeout) return this.idleTimeout = setTimeout(this.idled, 350); // This allows to wait a little bit
                    this.x.domain([ 4,8])
                }else{
                    this.x.domain([ this.x.invert(this.extent[0]), this.x.invert(this.extent[1]) ])
                    this.line.select(".brush").call(this.brush.move, null) // This remove the grey brush area as soon as the selection has been done
                }

                // Update axis and line position
                this.xAxis.transition().duration(1000).call(d3.axisBottom(this.x));
                
                this.line.select('.line')
                .transition()
                .duration(1000)
                .attr("d", d3.line()
                    .x((d) => { return this.x(d.day); })
                    .y((d) => { return this.y(d.temperature); })
                )  
        },

        ready(error: any, temperatureData: any) {
            
            const temperatureDataById = {}

            temperatureData.forEach((d) => {
                const newObj = {};
                newObj.entity = d.Entity;
                newObj.day = d3.timeParse("%Y-%m-%d")(d.Day.split(" ")[0]);
                newObj.temperature = d.temperature_anomaly;
                if(!temperatureDataById[d.Entity])
                    temperatureDataById[d.Entity] = [];
                temperatureDataById[d.Entity].push(newObj);
            });

            // Add X axis --> it is a date format
            this.x = d3.scaleTime()
            .domain(d3.extent(temperatureData, function(d) { return d3.timeParse("%Y-%m-%d")(d.Day.split(" ")[0]); }))
            .range([ 0, this.width/2 ]);
            
            this.xAxis = this.svg.append("g")
            .attr("transform", "translate(0," + this.height/2 + ")")
            .call(d3.axisBottom(this.x));

            // Add Y axis
            this.y = d3.scaleLinear()
            .domain([0, d3.max(temperatureData, function(d) { 
                return d.temperature_anomaly; })])
            .range([ this.height/2, 0 ]);
            
            this.yAxis = this.svg.append("g")
            .call(d3.axisLeft(this.y));

            // Add a clipPath: everything out of this area won't be drawn.
            this.clip = this.svg.append("defs").append("svg:clipPath")
                .attr("id", "clip")
                .append("svg:rect")
                .attr("width", this.width/2 )
                .attr("height", this.height/2 )
                .attr("x", 0)
                .attr("y", 0);

            // group the data: I want to draw one line per group
            var sumstat = d3.nest() // nest function allows to group the calculation per level of a factor
                            .key(function(d) { return d.Entity;})
                            .entries(temperatureData);
            
            // color palette
            var res = sumstat.map(function(d){ return d.key }) // list of group names
            
            var color = d3.scaleOrdinal()
            .domain(res)
            .range(['#377eb8','#ffff33','#f781bf','#e41a1c','#4daf4a','#984ea3','#ff7f00','#a65628','#999999'])

            const regions = Object.keys(temperatureDataById);

            //Here all the line creation to add as many lines as entities we have

            // Add brushing
            this.brush = d3.brushX()                   // Add the brush feature using the d3.brush function
                .extent( [ [0,0], [this.width/2, this.height/2] ] )  // initialise the brush area: start at 0,0 and finishes at width,height: it means I select the whole graph area
                .on("end", this.updateChart)               // Each time the brush selection changes, trigger the 'updateChart' function
        
            
            
            // Create the line variable: where both the line and the brush take place
            this.line = this.svg.append('g')
            .attr("clip-path", "url(#clip)")

            // Add the line
            this.line.append("path")
            .datum(temperatureDataById[regions[2]])
            .attr("class", "line")  // I add the class line to be able to modify this line later on.
            .attr("fill", "none")
            .attr("stroke", function(d){ return color(regions[2]) })
            .attr("stroke-width", 1.5)
            .attr("d", d3.line()
                .x((d) => { return this.x(d.day); })
                .y((d) => { return this.y(d.temperature); }) 
                )

            // Add the brushing
            this.line.append("g")
                .attr("class", "brush")
                .call(this.brush);

            // If user double click, reinitialize the chart 
            this.svg.on("dblclick", () => {
                this.x.domain(d3.extent(temperatureDataById[regions[2]], function(d) { return d.day; }));
                
                this.xAxis.transition().call(d3.axisBottom(this.x));
                
                this.line.select('.line')
                .transition()
                .attr("d", d3.line()
                    .x((d) => { return this.x(d.day); })
                    .y((d) => { return this.y(d.temperature); })
                )

                
            });
           
        }
    },
    mounted(){
        // set the dimensions and margins of the graph
        var margin = {top: 10, right: 30, bottom: 30, left: 60}

        // append the svg object to the body of the page
        this.svg = d3.select("#linecharter")
        .append("svg")
            .attr("width", this.width/2 + margin.left + margin.right)
            .attr("height", this.height/2 + margin.top + margin.bottom)
        .append("g")
            .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

        //Read the data
        d3.queue()
            .defer(d3.csv, "https://raw.githubusercontent.com/Edgarcillo2k/climate-change/master/src/assets/climate_change.csv?token=AJKTOBRQAWPV5QS5WO4HJI3BVZCUI")
            .await(this.ready);
        }  
    
})
</script>
<style>

    .comboBox {
        padding: 10px;
        margin: 10px;
        width: 10%;
        background: #FFFFFF;
        box-shadow: 0px 4px 14px rgba(0, 0, 0, 0.1);
        border-radius: 8px;
    }

</style>