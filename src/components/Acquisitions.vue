<template>
  <div>
    <svg id="scoring"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import acquisitions from '../data/acquisitions.json';

  export default {
    data() {
      return {
        height: 600,
        width: 700,
        margin: {top: 20, right: 20, bottom: 20, left: 50},
        colors: {
          'draft': '#80b1d3',
          'trade': '#fb8072',
          'waiver': '#fdb462'
        },
        players: [],
        groupedWins: [],
        maxColumn: 0
      }
    },
    mounted () {
      this.placeTeams()
      this.initData()
      this.initTeamGraph()
    },
    methods: {
      initTeamGraph() {
        let height = this.height
        let width = this.width
        // sets svg with appropriate height and width
        // TODO should be responsive and mobile friendly
        let totalWidth = width + this.margin.left + this.margin.right
        let svg = d3.select('#scoring')
          .attr('width', totalWidth)
          .attr('height', height)
        
        // Group inside svg for the main visualization
        const g = svg.append('g')
          .attr('transform', "translate(" + this.margin.left + "," + this.margin.top + ")")
          .attr('class', 'main_group')
        
        let week = "week0"
        let owners = _.values(acquisitions["weeks"][week]["players"])
        let yScale = d3.scaleBand()
          .domain(d3.range(0, this.maxColumn + 1))
          .range([this.height, 0])
        let separation = 75
        let mid = height / 2

        let teamRadius = 48
        
        let radius = 8
        let rosterIds = _.keyBy(owners, 'roster_id')
        // putting a pause on trades
        // let tradeTransactions = acquisitions["weeks"][week]["trades"]
        // let trades = g.selectAll('trades')
        //   .data(tradeTransactions)
        //   .enter()
        //   .append('line')
        //   .attr('x1', (d) => {
        //     return rosterIds[d.source].placeX
        //   })
        //   .attr('y1', (d) => {
        //     return yScale(rosterIds[d.source].index)
        //   })
        //   .attr('x2', (d) => {
        //     return rosterIds[d.target].placeX
        //   })
        //   .attr('y2', (d) => {
        //     return yScale(rosterIds[d.target].index)
        //   })
        //   .attr('class', 'trades')
        let teams = g.selectAll('teams')
          .data(owners)
          .enter()
          .append('circle')
          .attr('r', teamRadius)
          .attr('cx', (d) => {
            return d.placeX
          })
          .attr('cy', (d) => {
            return yScale(d.index) + (d.total / 2 ) * yScale.bandwidth() - yScale.bandwidth()
          })
          .attr('fill', 'white')
          .attr('stroke', 'black')
        let playerNodes = g.selectAll('player')
          .data(this.players)
          .enter()
          .append('circle')
          .attr('r', radius)
          .attr('class', 'player')
          .attr('fill', d => this.colors[d.type])
          .attr('stroke', 'black')
      
        let playerSimulation = d3.forceSimulation(this.players)
          .force("collide", d3.forceCollide([radius]))
          .force("x", d3.forceX((d) => {
            return rosterIds[d.roster_id].placeX
          }).strength([1]))
          .force("y", d3.forceY((d) => {
            return yScale(rosterIds[d.roster_id].index) + (rosterIds[d.roster_id].total / 2 ) * yScale.bandwidth() - yScale.bandwidth()
          }).strength([1]))
          .on('tick', playerTick) //multiple ticks can correspond to different simulations
        
        function playerTick() {
          playerNodes
            .attr('cx', (d) => d.x)
            .attr('cy', (d) => d.y)
        }

        // Animate Teams over Week
        // Link Teams Via Trades
        // Center Vertical Teams
        // Separate Teams and Player functions
        
      },
      placeTeams() {
        let week = "week0"
        let data = acquisitions["weeks"][week]["players"]
        let groupedWins = _.groupBy(data, 'wins')
        let winExtent = d3.extent(_.map(data, d => d.wins))
        
        let xScale = d3.scaleBand()
          .domain(d3.range(winExtent[0], winExtent[1] + 1))
          .range([0, this.width])
        // each team needs an x and y placement
        let placements = []
        let maxColumn = 0
        _.forEach(groupedWins, (teams) => {
          _.forEach(teams, (team, index) => {
            team["total"] = teams.length
            team["placeX"] = xScale(team.wins)
            team["index"] = index
            // need to actually calculate y placement here, instead of computing every time
          })
          if (maxColumn < teams.length) {
            maxColumn = teams.length
          }
        })
        this.maxColumn = maxColumn
        this.groupedWins = groupedWins
        console.log(data)
      },
      initData() {
        let data = acquisitions["weeks"]["week0"]["players"]
        let acquisitionTypes = Object.keys(acquisitions["weeks"]["week0"]["players"]["1"]["acquisitions"])
        let players = []
        // this double for loop is also used when determine placeX and placeY 
        _.forEach(Object.keys(data), (player) => {
          _.forEach(acquisitionTypes, (d) => {
            players.push(_.map(data[player]["acquisitions"][d], (a) => {
              return {
                "player_id": a,
                "type": d,
                "roster_id": player,
                "wins": data[player]["wins"]
              }
            }))
          })
        })
        this.players = _.flatten(players)
      }
    },
  }
</script>

<style>
.trades {
  fill: none;
  stroke: #fb6a4a;
}
</style>