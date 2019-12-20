<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';

  import scores from '../data/data.json';

  export default {
    data() {
      return {
        height: 900,
        width: 700,
        margin: {top: 40, right: 20, bottom: 10, left: 20},
        averagedPositionData: [],
        positions: [],
        maxPostionScore: 0
      }
    },
    mounted () {
      this.initData()
      this.initGraph()
    },
    methods: {
      initData() {
        this.positions = scores["settings"]["starters"]

        this.averagedPositionData = _.map(scores["players"], (d) => {
          let output = {}
          let players = []
          for (let pos in this.positions) {
            console.log(pos, this.positions)
            let averagePos = _.map(d["scores"], (week) => {
              console.log(this.positions[pos])
              return week["players"][this.positions[pos]]
            })
            let score = Math.round(d3.mean(averagePos) * 100) / 100
            if (score > this.maxPostionScore) { // might need to separate this out
              this.maxPostionScore = score
            }
            players.push({
              "position": this.positions[pos],
              "score": score
            })
          }
          output["players"] = players
          output["wins"] = d["wins"]
          output["display_name"] = d["display_name"]
          return output
        })

        // move color logic here, flexible for x number of positions
      },
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

        // TODO programatically figure out colors
        let colors = ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c','#fdbf6f', '#ff7f00']
        let colorPos = {
          'QB-0': colors[0],
          'RB-0': colors[1],
          'RB-1': colors[1],
          'RB-2': colors[1],
          'WR-0': colors[2],
          'WR-1': colors[2],
          'WR-2': colors[2],
          'TE-0': colors[3],
          'FLEX-0': colors[4],
          'FLEX-1': colors[4],
          'K-0': colors[5],
          'DEF-0': colors[6],
          'SUPER_FLEX-0': colors[7]
        }
        let weeks = scores["players"]["1"]["scores"].length // is there a better way to get total weeks?

        // arrange teams by wins
        let winGroups = _.groupBy(this.averagedPositionData, 'wins')
        let winKeys = Object.keys(winGroups)
        let maxWins = parseInt(d3.max(winKeys))

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

        // average legend
        // this logic shoudld be separated and generic enough for max and min legends
        let averagePositions = []
        for (let pos in this.positions) {
          let position = this.positions[pos]
          let averagePosition = {}
          averagePosition["score"] = Math.round(_.meanBy(this.averagedPositionData, (d) => {
            let posGroup = _.groupBy(d["players"], "position")
            return posGroup[position][0]['score']
          }) * 100) / 100
          averagePosition["position"] = position
          averagePositions.push(averagePosition)
        }

        // let boxRange = (width - 70) / (maxRow + (maxRow - 1) /2 ) //hardcoded, need more programmatic way of defining
        let boxRange = 125
        let xScale = d3.scaleLinear()
          .domain([0, this.maxPostionScore])
          .range([0, boxRange])

        let winExtent = d3.extent(Object.keys(winGroups), d => parseInt(d))
        let winRange = d3.range(winExtent[0], winExtent[1] + 1, 1)

        let winScale = d3.scaleBand()
          .domain(winRange)
          .range([height - 100, 0])
          .paddingInner(0.3)

        let yHeight = winScale.bandwidth() - 17// need to determine more programmatic way for yheight                
        let yScale = d3.scaleBand()
          .paddingInner(0.2)
          .domain(this.positions)
          .range([0, yHeight])

        width = width - this.margin.left - this.margin.right //better to create separate width variable
        let spacing = boxRange / 2 + 40
        // group of pyramids
        let pyramids = g.selectAll('pyramids')
          .data(this.averagedPositionData)
          .enter()
          .append('g')
          .attr('transform', (d, i) => {
            let start = ((width/2 - boxRange/2) - ((d.total - 1) * spacing)) + 2 * spacing * d.placeX
            return "translate(" + start + "," + winScale(d["wins"]) + ")"
          })
        spacing = boxRange / 2
        // bars for each group
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
          .attr('height', yScale.bandwidth())
          .attr('fill', (d) => {
            return colorPos[d['position']]
          })
        
        let names = pyramids.selectAll('labels')
          .data((d, i) => {
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
        // this needs to be relocated to a separate function
        let legend = g.append('g')
          .attr('transform', "translate(" + (this.width - 175) + "," + 0 + ")") //200 this is hardcoded
        let legendBars = legend.selectAll('legend')
          .data(averagePositions)
          .enter()
          .append('rect')
          .attr('x', d => (boxRange - xScale(d['score'])) / 2)
          .attr('y', d => yScale(d['position']))
          .attr('width', (d) => {
            return xScale(d['score'])
          })
          .attr('height', yScale.bandwidth())
          .attr('fill', (d) => {
            return colorPos[d['position']]
          })
        
        let labelPositions = legend.selectAll('legendLabels')
          .data(averagePositions)
          .enter()
          .append('text')
          .attr('x', d =>  (boxRange - xScale(d['score'])) / 2 - 10)
          .attr('y', d => yScale(d['position']) + 6)
          .text((d) => {
            return d['position']
          })
          .attr('class', 'legendLabels')
        
        let labelScores = legend.selectAll('legendScores')
          .data(averagePositions)
          .enter()
          .append('text')
          .attr('x', d => (boxRange - xScale(d['score'])) / 2 + xScale(d['score']) + 5)
          .attr('y', d => yScale(d['position']) + 6)
          .text((d) => {
            return d['score']
          })
          .attr('class', 'legendScores')
        
        legend.append('text')
          .attr('x', spacing)
          .attr('y', -10)
          .text('Average Position Score')
          .attr('class', 'label')
        
        g.append('text')
          .attr('x', 14)
          .attr('y', 0)
          .text('# of Wins')
          .attr('class', 'label')
          
        
        g.selectAll('wins')
          .data(winRange)
          .enter()
          .append('text')
          .attr('transform', (d, i) => {
            return "translate(" + (14) + "," + (winScale(d) + spacing / 2) + ")"
          })
          .text((d) => d)
          .attr('class', 'winLabels')
        
        // title
        g.append('text')
          .attr('x', (width - this.margin.left) / 2)
          .attr('y', -20)
          .text('Fantasy Football Roster Snapshot') // incorporate actual league name
          .attr('class', 'headline')
      }
    },
  }
</script>

<style>
.legendScores {
  font-size: 7px;
  text-anchor: start;
}
.legendLabels {
  font-size: 7px;
  text-anchor: end;
}
.label {
  text-anchor: middle;
}
.headline {
  text-anchor: middle;
  font-size: 20px;
}
.winLabels  {
  text-anchor: middle;
  font-size: 18px;
}
</style>