<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';

  import scores from '../data/weeklyScores_output.json';
  import teams from '../data/snapshot_output.json';
  export default {
    data() {
      return {
        height: 600,
        width: 300,
        margin: {top: 10, right: 10, bottom: 10, left: 20},
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

        players = _.keyBy(players, 'roster_id')

        let yScale = d3.scaleLinear()
          .domain([maxScore, 0])
          .range([0, height/12])

        let rosterGroups = _.groupBy(data, 'roster_id')
        
        let teamScale = d3.scaleBand()
          .domain(Object.keys(rosterGroups))
          .range([height, 0]);

        let sampleData = _.orderBy(rosterGroups["5"], ['week'], ['asc']) // hardcoded replace with first roster team
        // debugger
        let weeks = _.map(sampleData, d => d.week)
        let padding = 0.4
        let xScale = d3.scaleBand()
          .domain(Object.keys(rosterGroups))
          .range([0, width - this.margin.right])
          .padding([padding])
        
        let sparkbars = g.selectAll('bars')
          .data(Object.keys(rosterGroups))
          .enter()
          .append('g')
          .attr('transform', d => "translate(" + 0 + "," + teamScale(d) + ")")
        
        console.log(Object.keys(rosterGroups))
        console.log(rosterGroups)
        console.log(players)
        let teamBars = sparkbars.selectAll('teamBars')
          .data(d => rosterGroups[d])
          .enter()
          .append('rect')
          .attr('x', (d) => {
            // console.log(xScale(d.week), yScale(d.points), d.week, d.points)
            return xScale(d.week)
          })
          .attr('y', d => yScale(d.points))
          .attr('width', xScale.bandwidth())
          .attr('height', d => height/12 - this.margin.top - yScale(d.points)) // - this.margin.top because thats how much the group has been moved down
          .attr('fill', d => d.outcome === "win" ? "#0571b0" : "#ca0020")
        // need a legend
        let avatars = sparkbars.selectAll('avatars')
          .data(d => [players[d]])
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            console.log(d)
            console.log(d['avatar'], d)
            return 'https://sleepercdn.com/avatars/thumbs/' + d['avatar']
          })
          .attr('width', 20)
          .attr('height', 20)
          .attr('x', d => -25)
          .attr('y', d => 15)
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>