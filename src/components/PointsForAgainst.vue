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
        height: 600,
        width: 750,
        margin: {top: 50, right: 50, bottom: 50, left: 95},
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
          .attr('transform', "translate(" + (this.margin.left) + "," + this.margin.top + ")")
          .attr('class', 'main_group')
        
        // Text for Title
        // TODO need to get automatic week # from output file
        
        let data = scores["players"]
        let extentFor = d3.extent(data, (d) => d.fpts)
        let extentAgainst = d3.extent(data, (d) => {
          return d.fpts_against
        })
        
        let localMin = d3.min([extentFor[0], extentAgainst[0]])
        let localMax = d3.max([extentFor[1], extentAgainst[1]])
        let xScale = d3.scaleLinear()
          .domain([localMin, localMax])
          .range([2, this.width - this.margin.right - this.margin.left])
        
        // separate mins and maxes for each axis, wasted space
        let yScale = d3.scaleLinear()
          .domain([localMin, localMax])
          .range([this.height - this.margin.top - this.margin.bottom, 2])
                // average baselines points for
        let averageFor = d3.mean(data, (d) => d.fpts)
        g.append('line')
          .attr('x1', xScale(averageFor))
          .attr('y1', yScale(localMin))
          .attr('x2', xScale(averageFor))
          .attr('y2', yScale(localMax))
          .attr('class', 'averageLines')
        
        let averageAgainst = d3.mean(data, (d) => d.fpts_against)

        g.append('line')
          .attr('x1', xScale(localMin))
          .attr('y1', yScale(averageAgainst))
          .attr('x2', xScale(localMax))
          .attr('y2', yScale(averageAgainst))
          .attr('class', 'averageLines')

        let avatarDimension = 24
        g.selectAll('avatars')
          .data(data)
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            return 'https://sleepercdn.com/avatars/thumbs/' + d["avatar"]
            // return d["avatar"]
          })
          .attr('width', avatarDimension)
          .attr('height', avatarDimension)
          .attr('x', (d) => xScale(d.fpts) - avatarDimension/2)
          .attr('y', (d, i) => yScale(d.fpts_against) - avatarDimension/2)

        let xAxis = d3.axisBottom()
          .scale(xScale)
          .tickSizeInner([0])
          .tickSizeOuter([0])
        
        let yAxis = d3.axisLeft()
          .scale(yScale)
          .tickSizeInner([0])
          .tickSizeOuter([0])
        
        g.append('g')
          .attr('transform', "translate(" + -10 + "," + 0 + ")")
          .call(yAxis)
        
        // todo change the text into one and word wrap
        svg.append('text')
          .attr('x', 3)
          .attr('y', this.height / 2)
          .text('Points')
          .attr('class', 'labels')
        
        svg.append('text')
          .attr('x', 3)
          .attr('y', this.height / 2 + 23)
          .text('Against')
          .attr('class', 'labels')

        g.append('g')
          .attr('transform', "translate(" + 0 + "," + (this.height - this.margin.top - this.margin.bottom) + ")")
          .call(xAxis)
        
        // todo create different label for horizontal labels
        svg.append('text')
          .attr('x', xScale(averageFor) + this.margin.left - 35)
          .attr('y', this.height - 14)
          .text('Points For')
          .attr('class', 'labels')
        
        svg.append('text')
          .attr('x', xScale(averageFor) + this.margin.left - 27)
          .attr('y', this.margin.top - 5)
          .text('Week 5')
          .attr('class', 'labels')

        d3.selectAll(".domain").remove();

      }
    },
  }
</script>

<style>
.averageLines {
  stroke: #121212;
  stroke-dasharray: 6,6; 
}
#scoring > g > g > .tick > text {
  /* color: white; */
}
.labels {
  /* fill: white; */
  text-anchor: 'start'
}
#scoring {
  /* background-color: #121212 */
}
</style>