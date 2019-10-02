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
  import scores from '../data/snapshot_output.json';

  export default {
    data() {
      return {
        height: 450,
        width: 500,
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
        
        // Text for Title
        // TODO need to get automatic week # from output file
        
        let data = scores["players"]
        let extentFor = d3.extent(data, (d) => d.fpts)
        let extentAgainst = d3.extent(data, (d) => {
          console.log(d)
          return d.fpts_against
        })
        
        let localMin = d3.min([extentFor[0], extentAgainst[0]])
        let localMax = d3.max([extentFor[1], extentAgainst[1]])
        let xScale = d3.scaleLinear()
          .domain([localMin * .95, localMax * 1.05])
          .range([2, this.width - this.margin.right - this.margin.left])
        
        let yScale = d3.scaleLinear()
          .domain([localMin * .95, localMax * 1.05])
          .range([this.height - this.margin.top - this.margin.bottom, 2])
                // average baselines points for
        let averageFor = d3.mean(data, (d) => d.fpts)
        g.append('line')
          .attr('x1', xScale(averageFor))
          .attr('y1', yScale(localMin * .95))
          .attr('x2', xScale(averageFor))
          .attr('y2', yScale(localMax * 1.05))
          .style('stroke', 'black')
        
        let averageAgainst = d3.mean(data, (d) => d.fpts_against)
        g.append('line')
          .attr('x1', xScale(localMin * .95))
          .attr('y1', yScale(averageAgainst))
          .attr('x2', xScale(localMax * 1.05))
          .attr('y2', yScale(averageAgainst))
          .style('stroke', 'black')
        
        g.append('text')
          .attr('x', xScale(averageFor))
          .attr('y', -10)
          .text('Points For / Against - Week 4')
          .style('text-anchor', 'middle')

        g.selectAll('avatars')
          .data(data)
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            return 'https://sleepercdn.com/avatars/thumbs/' + d["avatar"]
          })
          .attr('width', 20)
          .attr('height', 20)
          .attr('x', (d) => xScale(d.fpts) - 10)
          .attr('y', (d, i) => yScale(d.fpts_against) - 10)
          // need to offset height and width to constants instead of hardcoded values

        let xAxis = d3.axisBottom()
          .scale(xScale)
        
        let yAxis = d3.axisLeft()
          .scale(yScale)
        
        g.append('g')
          .call(yAxis)
        
        svg.append('text')
          .attr('x', this.height / -2)
          .attr('y', 15)
          .text('Points Against')
          .style('text-anchor', 'middle')
          .attr('transform', "rotate(-90)")

        g.append('g')
          .attr('transform', "translate(" + 0 + "," + (this.height - this.margin.top - this.margin.bottom) + ")")
          .call(xAxis)
        
        svg.append('text')
          .attr('x', this.width / 2)
          .attr('y', this.height - 2)
          .text('Points For')
          .style('text-anchor', 'middle')
      

      }
    },
  }
</script>

<style lang="scss" scoped>

</style>