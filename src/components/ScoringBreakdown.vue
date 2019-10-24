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
  import Scores from '../data/week6_output.json';

  export default {
    data() {
      return {
        height: 600,
        width: 750,
        margin: {top: 30, right: 50, bottom: 100, left: 30},
        colors: ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c'],
        transitionFlag: false,
        disData: []
      }
    },
    mounted () {
      this.initGraph();
    },
    methods: {
      initGraph() {
        let height = this.height
        let width = this.width
        // need to make more generic, retrieve from settings?
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
          .text('Week 6 Positional Scoring Breakdown')
          .style('text-anchor', 'start')

        // combine positons and calculate sub totals
        // TODO move this processing to the backend
        // need to handle negative values for all positions not just QB
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

        this.disData = this.disassembleStack(data, positions)
        let xMax = _.maxBy(data, 'sub_points')['sub_points']

        // hardcoded range for length of bars
        // TODO make responsive
        let rightRange = this.width - this.margin.left - this.margin.right

        // Scale to transpose bars over a certain range (30 - rightRange)
        let xScale = d3.scaleLinear()
          .domain([0, xMax])
          .range([this.margin.left, rightRange])

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
        let yScale = d3.scaleBand()
          .domain([0,1,2,3,4,5,6,7,8,9,10,11]) // this is hardcoded
          .range([0, this.height - this.margin.top - this.margin.bottom])

        // only change is the xscale
        let bars = gauges.selectAll('gauge')
          .data((d) => {
            // get the key here and pass it to the data
            for (let j in d) {
              if (Array.isArray(d[j])) {
                d[j].push(d.key)
              }
            }
            return d
          })
          .enter().append('rect')
          .attr('x', (d, i) => {
            // get the key and determine what scale to use
            // return this.disData[d[2]].scale(0)
            return xScale(d[0])
          })
          .attr('y', (d, i) => {
            // hardcoded
            return yScale(i)
          })
          .attr('width', (d) => {
            return xScale(d[1]) - xScale(d[0])
            // return this.disData[d[2]].scale(d[1]) - this.disData[d[2]].scale(d[0])
          })
          .attr('height', 10)
          .attr('class', 'gauge')
        
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
          .attr('y', (d, i) => yScale(i) - 5)

        // Legend 
        // todo legend placement is hardcoded
        // create a rectangle for each color
        let legend = g.append('g')
          .on('click', (d) => {
            console.log('clicked')
            this.transitionFlag = !this.transitionFlag
            this.disassemble(xScale, series)
          })

        legend.append('g')
          .attr('class', 'legends')
          .selectAll('bars')
          .data(this.colors)
          .enter().append('rect')
            .attr('x', (d, i) => {
              return i * 40 + this.margin.left
            })
            .attr('y', this.height - this.margin.bottom - this.margin.top)
            .attr('width', 40)
            .attr('height', 10)
            .attr('fill', (d) => {return d})
            // .attr('stroke', 'black')

        // create a label for each position will coincide with color
        legend.append('g')
          .attr('class', 'labels')
          .selectAll('label')
          .data(this.colors)
          .enter().append('text')
            .attr('x', (d, i) => {
              return i * 40 + this.margin.left + 20
            })
            .attr('y', this.height - this.margin.bottom - this.margin.top / 2 + 5)
            .text((d,i) => positions[i])
            .style('font-size', '10px')
            .style('text-anchor', 'middle')
        
        // append the sub_points the right of each gauge (top-bottom)
        g.append('g')
          .attr('class', 'points')
          .selectAll('points')
          .data(data)
          .enter().append('text')
            .attr('x', (d, i) => xScale(series[positions.length - 1][i][1]) + 4)
            .attr('y', (d, i) => yScale(i) + 9)
            .text((d) => d["sub_points"].toFixed(0))
            .style('font-size', '10px')
            .style('text-anchor', 'start')
            .attr('class', 'point')
      },
      disassembleStack(data, positions) {
        // each position has its own scale
        // Q. How to determine the size of each pos group
        // for each scale what is the width
        // domain width = cum(max_pos)\
        let total = 0
        for (let m in positions) {
          let xMax = _.maxBy(data, positions[m])[positions[m]]
          total = total + xMax
        }
        let marginWidth = this.width - this.margin.left - this.margin.right
        let mainScale = d3.scaleLinear()
          .domain([0, total])
          .range([this.margin.left, marginWidth])

        let maxPositions = []
        let offset = 0
        for (let pos in positions) {
          let position = positions[pos]
          let maxPos = _.maxBy(data, position)[position]
          maxPositions.push({
            "position": position,
            "max": maxPos,
            "scale": d3.scaleLinear()
              .domain([0, maxPos + 20]) //20 is hardcoded but represents the margin between bars
              .range([mainScale(offset), mainScale(maxPos) + mainScale(offset)])
          })
          offset = offset + maxPos
        }
        return _.keyBy(maxPositions, 'position');
      },
      // might need to move this into its own watch function
      disassemble(xScale, series) {
        let bars = d3.selectAll('.gauge')
          .transition()
          .duration(3000)
          .attr('x', (d, i) => {
            // get the key and determine what scale to use
            return this.transitionFlag ? this.disData[d[2]].scale(0) : xScale(d[0])
          })
          .attr('width', (d) => {
            return this.transitionFlag ? this.disData[d[2]].scale(d[1]) - this.disData[d[2]].scale(d[0]) : xScale(d[1]) - xScale(d[0])
          })
        
        let points = d3.selectAll('.point')
          .transition()
          .duration(3000)
          .attr('x', (d, i) => {
            return this.transitionFlag ? this.disData["K"].scale(this.disData["K"].max) + 10 : xScale(series[5][i][1]) + 4 //5 is hardcoded length of positions
          })
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>