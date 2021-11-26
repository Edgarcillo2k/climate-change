<template>
    <b-container>
        <b-form-select v-model="yearSelected" :options="years" @input="changeYear" required></b-form-select>
        <svg id="svg" :viewBox="`0 0 ${width} ${height}`">
        </svg>
    </b-container>
</template>

<script lang="ts">
import Vue from 'vue';
import * as d3 from 'd3v4';
import { interpolateRdYlGn} from 'd3-scale-chromatic';
import { select } from 'd3';
import d3Tip from "d3-tip";
import * as topojson from "topojson-client";

export default Vue.extend({
    name: 'WorldMap',
    data(){
        return {
            width: window.innerWidth,
            height: window.innerHeight,
            projection: undefined,
            data: undefined,
            svg: undefined as any,
            path: undefined as any,
            tip: undefined as any,
            years: [],
            yearSelected: 1990
        }
    },
    methods: {
        changeYear(){
            this.svg.selectAll('path')
                .style("fill", (d) => {
                    if(d.data)
                        return interpolateRdYlGn(1-d.data[this.yearSelected].color);
                    return "#FFFFFF"; 
                })
        },
        ready(error: any, data: any, emissions: any) {
            const emissionsById = {};

            emissions.forEach((d) => {
                const newObj = {};
                newObj.emissions = +d['Total GHG emissions excluding LUCF (CAIT)'];
                newObj.color = +d.color;
                if(!emissionsById[d.Code])
                    emissionsById[d.Code] = {};
                emissionsById[d.Code][d.Year] = newObj;
            });
            data.features.forEach((d) => {
                if(emissionsById[d.id])
                    d.data = emissionsById[d.id]
            });

            const tip = this.tip;

            this.svg.append("g")
                .attr("class", "countries")
                .selectAll("path")
                .data(data.features)
                .enter().append("path")
                .attr("d", this.path)
                .style("fill", (d) => {
                    if(d.data)
                        return interpolateRdYlGn(1-d.data[this.yearSelected].color);
                    return "#FFFFFF"; 
                })
                .style('stroke', 'white')
                .style('stroke-width', 1.5)
                .style("opacity",0.8)
                // tooltips
                .style("stroke","white")
                .style('stroke-width', 0.3)
                .on('mouseover',function(event, d){
                    tip.show(d,this);

                    d3.select(this)
                        .style("opacity", 1)
                        .style("stroke","white")
                        .style("stroke-width",3);
                    })
                .on('mouseout', function(event, d){
                    tip.hide(d);

                    select(this)
                        .style("opacity", 0.8)
                        .style("stroke","white")
                        .style("stroke-width",0.3);
                });

            this.svg.append("path")
                .datum(topojson.mesh(data.features, function(a, b) { return a.id !== b.id; }))
                // .datum(topojson.mesh(data.features, function(a, b) { return a !== b; }))
                .attr("class", "names")
                .attr("d", this.path);
            }
    },
    mounted(){
        for(let i = 1990; i <= 2016; i++){
            this.years.push(i);
        }
        const format = d3.format(",");
        this.tip = d3Tip()
            .attr('class', 'd3-tip')
            .offset([-10, 0])
            .html((d) => {
                return "<strong>Country: </strong><span class='details'>" + d.properties.name + "<br></span>" + "<strong>Emissions: </strong><span class='details'>" + format(d.data[this.yearSelected].emissions) +"</span>";
            });
        const margin = {top: 0, right: 0, bottom: 0, left: 0},
            width = 960 - margin.left - margin.right,
            height = 500 - margin.top - margin.bottom;

        this.path = d3.geoPath();

        this.svg = select("#svg")
            .append('g')
            .attr('class', 'map');

        this.projection = d3.geoMercator()
            .scale(250)
            .center([0,20])
            .translate([this.width / 2, this.height / 1.5]);

        this.path = d3.geoPath().projection(this.projection);

        this.svg.call(this.tip);

        d3.queue()
            .defer(d3.json, "https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson")
            .defer(d3.csv, "https://raw.githubusercontent.com/Edgarcillo2k/climate-change/master/src/assets/total_ghg_emissions_per_year.csv?token=AKWLPXE2KQ32EG2UZ7F7ZH3BVJL2G")
            .await(this.ready);
        
    }
})
</script>
<style>
    .names {
        fill: none;
        stroke: #fff;
        stroke-linejoin: round;
    }

    /* Tooltip CSS */
    .d3-tip {
        line-height: 1.5;
        font-weight: 400;
        font-family:"avenir next", Arial, sans-serif;
        padding: 6px;
        background: rgba(0, 0, 0, 0.6);
        color: #FFA500;
        border-radius: 1px;
        pointer-events: none;
    }

    /* Creates a small triangle extender for the tooltip */
    .d3-tip:after {      
      box-sizing: border-box;
      display: inline;
      font-size: 8px;
      width: 100%;
      line-height: 1.5;
      color: rgba(0, 0, 0, 0.6);
      position: absolute;
      pointer-events: none;
      
    }

    /* Northward tooltips */
    .d3-tip.n:after {
      content: "\25BC";
      margin: -1px 0 0 0;
      top: 100%;
      left: 0;
      text-align: center;
    }

    /* Eastward tooltips */
    .d3-tip.e:after {
      content: "\25C0";
      margin: -4px 0 0 0;
      top: 50%;
      left: -8px;
    }

    /* Southward tooltips */
    .d3-tip.s:after {
      content: "\25B2";
      margin: 0 0 1px 0;
      top: -8px;
      left: 0;
      text-align: center;
    }

    /* Westward tooltips */
    .d3-tip.w:after {
      content: "\25B6";
      margin: -4px 0 0 -1px;
      top: 50%;
      left: 100%;
    }

/*    text{
      pointer-events:none;
    }*/

    .details{
      color:white;
    }
</style>