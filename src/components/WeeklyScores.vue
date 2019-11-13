<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';

  // outputted files from backend script for week 2
  import scores from '../data/weeklyScores_output.json';
  import teams from '../data/snapshot_output.json';
  export default {
    data() {
      return {
        height: 625,
        width: 750,
        margin: {top: 50, right: 50, bottom: 50, left: 50},
      }
    },
    mounted () {
      this.initGraph();
    },
    methods: {
      initGraph() {
        let height = this.height
        let width = this.width
        // sets svg with appropriate height and width
        // TODO should be responsive and mobile friendly
        let svg = d3.select('#scoring')
          .attr('width', width)
          .attr('height', height)
        
        // Group inside svg for the main visualization
        const g = svg.append('g')
          .attr('transform', "translate(" + this.margin.left + "," + this.margin.top + ")")
          .attr('class', 'main_group')
        
        g.append('text')
          .attr('x', width/2 - 40) //this is hardcoded need to change
          .attr('y', -this.margin.top / 2)
          .text('Range of Weekly Scores Thru 9 Weeks')
          .style('text-anchor', 'middle')
        
        let data = scores["games"]
        let maxScore = d3.max(data, d => d.points)
        let minScore = d3.min(data, d => d.points)

        
        let players = _.orderBy(_.map(teams["players"], (d) => {
          return {
            "owner_id": d["owner_id"],
            "roster_id": d["roster_id"],
            "avatar": d["avatar"],
            "fpts": d["fpts"]
          }
        }), ['fpts'], ['asc'])

        let xScale = d3.scaleLinear()
          .domain([minScore, maxScore])
          .range([0, width - this.margin.left - this.margin.right])
        
        let yScale = d3.scaleBand()
          .domain(_.map(players, d => d["roster_id"]))
          .range([height - this.margin.top - this.margin.bottom, 0]);
      
        // d3 force simulation to beeswarm
        let radius = 5
        let playerSimulation = d3.forceSimulation(data)
          .force("collide", d3.forceCollide([radius]))
          .force("x", d3.forceX((d) => {
            return xScale(d['points'])
          }).strength([1]))
          .force("y", d3.forceY((d) => {
            return yScale(d['roster_id'])
          }).strength([1]))
          .on('tick', playerTick) //multiple ticks can correspond to different simulations

        let gameNodes = g.selectAll('games')
          .data(data)
          .enter()
          .append('circle')
            .attr('r', radius)
            .attr('fill', d => d['outcome'] === "win" ? '#67a9cf' : '#ef8a62')
            .attr('stroke', 'black')

        function playerTick() {
          gameNodes
            .attr('cx', (d) => d.x)
            .attr('cy', (d) => d.y)
        }

        g.selectAll('avatars')
          .data(players)
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            return 'https://sleepercdn.com/avatars/thumbs/' + d["avatar"]
          })
          .attr('width', 20)
          .attr('height', 20)
          .attr('x', (d) => -this.margin.left)
          .attr('y', d => yScale(d['roster_id']) - 10)


        let xAxis = d3.axisBottom()
          .scale(xScale)
          
        svg.append('g')
          .attr('transform', "translate(" + this.margin.left + "," + (this.height - (this.margin.bottom * 1.5)) + ")")
          .call(xAxis)
        
        // need a legend
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>