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
  import Scores from '../data/week4_output.json';

  export default {
    data() {
      return {
        height: 600,
        width: 400,
        margin: {top: 30, right: 20, bottom: 20, left: 20},
        colors: ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c']
      }
    },
    mounted () {
      this.initGraph();
    },
    methods: {
      initGraph() {
        let height = this.height
        let width = this.width
        let positions = ["QB", "RB", "WR", "TE", "FLEX", "K"]

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
        svg.append('text')
          .attr('x', 38)
          .attr('y', 15)
          .text('Week 4 Scoring Breakdown')
          .style('text-anchor', 'start')

        // combine positons and calculate sub totals
        // TODO move this processing to the backend
        let data = _.map(Scores["players"], (d) => {
          return {
            "QB": d["scores"]["QB-0"] > 0 ? d["scores"]["QB-0"] : 0,
            "RB": d["scores"]["RB-0"] + d["scores"]["RB-1"],
            "WR": d["scores"]["WR-0"] + d["scores"]["WR-1"],
            "TE": d["scores"]["TE-0"],
            "FLEX": d["scores"]["FLEX-0"],
            "K": d["scores"]["K-0"],
            "avatar": d["avatar"],
            "points": d["points"],
            "sub_points": d["scores"]["QB-0"] + d["scores"]["RB-0"] + d["scores"]["RB-1"] + d["scores"]["WR-0"] + d["scores"]["WR-1"] + d["scores"]["TE-0"] + d["scores"]["FLEX-0"] + d["scores"]["K-0"]
          }
        })

        data = _.orderBy(data, ['sub_points'], ['desc'])

        let xMax = _.maxBy(data, 'sub_points')['sub_points']

        // hardcoded range for length of bars
        // TODO make responsive
        let rightRange = 200

        // Scale to transpose bars over a certain range (30 - rightRange)
        let xScale = d3.scaleLinear()
          .domain([0, xMax])
          .range([30, rightRange])

        // https://github.com/d3/d3-shape/blob/v1.3.5/README.md#stack
        // D3 stack will combine all positions as if they were one bar
        // the results will dictate the starting point and ending point for each position
        // ex. [{"QB": 5, "WR": 10}]
        // roughly something like this stack output [[0,5],[5, 15]]
        let stack = d3.stack()
          .keys(positions)
          .order(d3.stackOrderNone)
          .offset(d3.stackOffsetNone)
        
        // handle the actual stacking of our data
        let series = stack(data)

        // creates a g (group) for each position
        // here we specify the color of each position (left to right)
        let gauges = g.selectAll('gauges')
          .data(series)
          .enter().append('g')
          .style('fill', (d, i) => {
            return this.colors[i]
          })

        // creates a rectangle for each pairing in the stack
        let bars = gauges.selectAll('gauge')
          .data((d) => {
            return d
          })
          .enter().append('rect')
          .attr('x', (d) => {
            return xScale(d[0])
          })
          .attr('y', (d, i) => {
            // hardcoded
            return i * 25
          })
          .attr('width', (d) => {
            return xScale(d[1]) - xScale(d[0])
          })
          .attr('height', 10)
        
        // fetches the avatar images on the left of the group
        g.selectAll('avatars')
          .data(data)
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            return 'https://sleepercdn.com/avatars/thumbs/' + d["avatar"]
          })
          .attr('width', 17)
          .attr('height', 17)
          .attr('x', (d) => 0)
          .attr('y', (d, i) => i * 25 - 5)

        // Legend 

        // create a rectangle for each color
        g.append('g')
          .attr('class', 'legends')
          .selectAll('bars')
          .data(this.colors)
          .enter().append('rect')
            .attr('x', (d, i) => {
              return i * 40
            })
            .attr('y', 300)
            .attr('width', 40)
            .attr('height', 10)
            .attr('fill', (d) => {return d})
            // .attr('stroke', 'black')

        // create a label for each position will coincide with color
        g.append('g')
          .attr('class', 'labels')
          .selectAll('label')
          .data(this.colors)
          .enter().append('text')
            .attr('x', (d, i) => {
              return i * 40 + 20
            })
            .attr('y', 320)
            .text((d,i) => positions[i])
            .style('font-size', '10px')
            .style('text-anchor', 'middle')
        
        // append the sub_points the right of each gauge (top-bottom)
        g.append('g')
          .attr('class', 'points')
          .selectAll('points')
          .data(data)
          .enter().append('text')
            .attr('x', rightRange + 15)
            .attr('y', (d, i) => i * 25 + 8)
            .text((d) => d["sub_points"].toFixed(0))
            .style('font-size', '10px')
            .style('text-anchor', 'start')
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>