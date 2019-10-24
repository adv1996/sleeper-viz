<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';

  import scores from '../data/collated_stats.json';
  import snapshot from '../data/snapshot_output.json';

  export default {
    data() {
      return {
        height: 700,
        width: 800,
        margin: {top: 40, right: 20, bottom: 20, left: 20}
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
        
        // get the data averaged out

        //IDEA: timelapse of all week scores
        let snapshot_data = _.groupBy(snapshot["players"], 'owner_id')
        let positions = Object.keys(scores["1"]["scores"][0])

        // TODO programatically figure out colors
        let colors = ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c','#fdbf6f']
        let colorPos = {
          'QB-0': colors[0],
          'RB-0': colors[1],
          'RB-1': colors[1],
          'WR-0': colors[2],
          'WR-1': colors[2],
          'TE-0': colors[3],
          'FLEX-0': colors[4],
          'K-0': colors[5],
          'DEF-0': colors[6]
        }
        let weeks = scores["1"]["scores"].length
        let maxPos = 0
        let averagedData = _.map(scores, (d) => {
          let output = {}
          let players = []
          for (let pos in positions) {
            let score = Math.round(_.meanBy(d["scores"], positions[pos]) * 100) / 100
            if (score > maxPos) {
              maxPos = score
            }
            players.push({
              "position": positions[pos],
              "score": score
            })
          }
          output["players"] = players
          output["wins"] = snapshot_data[d["owner_id"]][0]["wins"]
          output["display_name"] = d["display_name"]
          return output
        })
        
        // y scale
        let winScale = d3.scaleBand()
          .domain(d3.range(0, weeks + 1, 1))
          .range([this.height, 0])

        let winGroups = _.groupBy(averagedData, 'wins')
        // need to preprocess each team 
        let winKeys = Object.keys(winGroups)
        let maxRow = 0
        _.forEach(winKeys, (d) => {
          let total = winGroups[d].length
          if (total > maxRow) {
            maxRow = total
          }
          _.forEach(winGroups[d], (i, index) => {
            i["placeX"] = index
            i["total"] = total
          })
        })

        let averagePositions = []
        for (let pos in positions) {
          let position = positions[pos]
          let averagePosition = {}
          averagePosition["score"] = Math.round(_.meanBy(averagedData, (d) => {
            let posGroup = _.groupBy(d["players"], "position")
            return posGroup[position][0]['score']
          }) * 100) / 100
          averagePosition["position"] = position
          averagePositions.push(averagePosition)
        }

        let boxRange = (width - 70) / (maxRow + (maxRow - 1) /2 ) 
        let xScale = d3.scaleLinear()
          .domain([0, maxPos])
          .range([0, boxRange])
        let winRange = _.map(Object.keys(winGroups), d => parseInt(d))
        winRange = d3.range(d3.min(winRange), d3.max(winRange) + 1, 1)

        let yHeight = 480 / winRange.length                       
        let yScale = d3.scaleBand()
          .domain(positions)
          .range([0, yHeight])

        width = width - this.margin.left - this.margin.right
        let spacing = boxRange / 2
        let pyramids = g.selectAll('pyramids')
          .data(averagedData)
          .enter()
          .append('g')
          .attr('transform', (d, i) => {
            // width 
            let start = (width/2 - boxRange/2) - ((d.total - 1) * spacing)
            return "translate(" + (start + 2 * spacing * d.placeX) + "," + winScale(d["wins"]) + ")"
          })
      
        
        let centeredBars = pyramids.selectAll('bars')
          .data((d) => {
            return d["players"]
          })
          .enter()
          .append('rect')
          .attr('x', d => (boxRange - xScale(d['score'])) / 2)
          .attr('y', d => yScale(d['position']))
          .attr('width', (d) => {
            return xScale(d['score'])
          })
          .attr('height', yHeight / 10) // need to figure out optimal height
          .attr('fill', (d) => {
            return colorPos[d['position']]
          })
        
        let names = pyramids.selectAll('labels')
          .data((d) => {
            return [d["display_name"]]
          })
          .enter()
          .append('text')
          .attr('x', boxRange / 2)
          .attr('y', yHeight + 10)
          .text((d) => d)
          .attr('fill', '#000000')
          .style('font-size', '10px')
          .style('text-anchor', 'middle')

        // find average structure and use that as a legend

        let legendYScale = d3.scaleBand()
          .domain(positions)
          .range([0, yHeight])


        let legend = g.append('g')
          .attr('transform', "translate(" + (this.width - 200) + "," + 0 + ")")
        let legendBars = legend.selectAll('legend')
          .data(averagePositions)
          .enter()
          .append('rect')
          .attr('x', d => (boxRange - xScale(d['score'])) / 2)
          .attr('y', d => legendYScale(d['position']))
          .attr('width', (d) => {
            return xScale(d['score'])
          })
          .attr('height', yHeight / 10) // need to figure out optimal height
          .attr('fill', (d) => {
            return colorPos[d['position']]
          })
          // .attr('stroke', 'black')
        
        let labelPositions = legend.selectAll('legendLabels')
          .data(averagePositions)
          .enter()
          .append('text')
          .attr('x', d =>  (boxRange - xScale(d['score'])) / 2 - 10)
          .attr('y', d => legendYScale(d['position']) + 6)
          .text((d) => {
            return d['position']
          })
          .attr('fill', '#000000')
          .style('font-size', '7px')
          .style('text-anchor', 'end')
        
        let labelScores = legend.selectAll('legendScores')
          .data(averagePositions)
          .enter()
          .append('text')
          .attr('x', d => (boxRange - xScale(d['score'])) / 2 + xScale(d['score']) + 5)
          .attr('y', d => legendYScale(d['position']) + 6)
          .text((d) => {
            return d['score']
          })
          .attr('fill', '#000000')
          .style('font-size', '7px')
          .style('text-anchor', 'start')
        
        legend.append('text')
          .attr('x', boxRange / 2)
          .attr('y', -10)
          .text('Average Position Score')
          .style('text-anchor', 'middle')
        
        let winPlacement = boxRange / 2
        g.append('text')
          .attr('x', winPlacement)
          .attr('y', 0)
          .text('# of Wins')
          .style('text-anchor', 'middle')
        
        g.selectAll('wins')
          .data(winRange)
          .enter()
          .append('text')
          .attr('x', winPlacement)
          .attr('y', (d) => {
            return winScale(d) + 50
          })
          .text((d) => d)
          .style('text-anchor', 'middle')
        
        // title
        g.append('text')
          .attr('x', (width - this.margin.left) / 2)
          .attr('y', -20)
          .text('Fantasy Football Roster Snapshot')
          .style('text-anchor', 'middle')
          .style('font-size', '20px')
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>