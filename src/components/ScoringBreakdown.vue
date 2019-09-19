<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import Scores from '../data/output_week2.json';

  export default {
    data() {
      return {
        height: 600,
        width: 400,
        margin: {top: 30, right: 20, bottom: 20, left: 20},
        colors: ['#8dd3c7','#ffffb3','#bebada','#fb8072','#80b1d3','#fdb462']
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
        let svg = d3.select('#scoring')
          .attr('width', width)
          .attr('height', height)
        
        const g = svg.append('g')
          .attr('transform', "translate(" + this.margin.left + "," + this.margin.top + ")")
          .attr('class', 'main_group')
        
        svg.append('text')
          .attr('x', 38)
          .attr('y', 15)
          .text('Week 2 Scoring Breakdown')
          .style('text-anchor', 'start')
        // needed to just get the values of json without the keys
        let data = _.map(Scores["players"], (d) => {
          return {
            "QB": d["scores"]["QB"],
            "RB": d["scores"]["RB1"] + d["scores"]["RB2"],
            "WR": d["scores"]["WR1"] + d["scores"]["WR2"],
            "TE": d["scores"]["TE"],
            "FLEX": d["scores"]["FLEX"],
            "K": d["scores"]["K"],
            "avatar": d["avatar"],
            "points": d["points"],
            "sub_points": d["scores"]["QB"] + d["scores"]["RB1"] + d["scores"]["RB2"] + d["scores"]["WR1"] + d["scores"]["WR2"] + d["scores"]["TE"] + d["scores"]["FLEX"] + d["scores"]["K"]
          }
        })

        data = _.orderBy(data, ['sub_points'], ['desc'])
        let sums = _.map(data, (d) => {
          let total = 0
          _.forEach(positions, (p) => {
            total = total + d[p]
          })
          return total
        })
        let xMax = _.max(sums)
        let rightRange = 200
        let xScale = d3.scaleLinear()
          .domain([0, xMax])
          .range([30, rightRange])
        // groups for each linear gauge
        let stack = d3.stack()
          .keys(positions)
          .order(d3.stackOrderNone)
          .offset(d3.stackOffsetNone)
        
        let series = stack(data)

        let gauges = g.selectAll('gauges')
          .data(series)
          .enter().append('g')
          .style('fill', (d, i) => {
            return this.colors[i]
          })

        let bars = gauges.selectAll('gauge')
          .data((d) => {
            return d
          })
          .enter().append('rect')
          .attr('x', (d) => {
            console.log(d[0])
            return xScale(d[0])
          })
          .attr('y', (d, i) => {
            // hardcoded
            return i * 25
          })
          .attr('width', (d) => xScale(d[1]) - xScale(d[0]))
          .attr('height', 10)
        
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
        // legend colors
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
        
        g.append('g')
          .attr('class', 'points')
          .selectAll('points')
          .data(data)
          .enter().append('text')
            .attr('x', rightRange + 15)
            .attr('y', (d, i) => i * 25 + 8)
            .text((d) => d["sub_points"].toFixed(0))
            .style('font-size', '10px')
            .style('text-anchor', 'middle')
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>