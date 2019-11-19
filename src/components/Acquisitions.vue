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
        height: 800,
        width: 1200,
        margin: {top: 20, right: 20, bottom: 20, left: 50},
        colors: {
          'draft': '#80b1d3',
          'trade': '#fb8072',
          'waiver': '#fdb462'
        },
        teamColors: ['#a6cee3','#1f78b4','#b2df8a','#33a02c','#fb9a99','#e31a1c','#fdbf6f','#ff7f00','#cab2d6','#6a3d9a','#ffff99','#b15928'],
        players: [],
        currentWeek: "week3",
        radius: 4,
        teamRadius: 32,
        xScale: {},
        xAxis: {},
        winRange: {},
        trades: {}
      }
    },
    mounted () {
      this.placeTeams()
      this.initTeamGraph()
    },
    methods: {
      updateTeamRosters(week) {
        this.currentWeek = week
        this.placeTeams()

        let owners = acquisitions["weeks"][this.currentWeek]["players"]
        let teams = d3.selectAll('.teams')
        let keyedOwners = _.keyBy(owners, 'roster_id') //since the number of teams will stay the same we can just update x and y positions
        let g = d3.select('.main_group')
        //handle trades here
        let trades = g.selectAll('.trades')
          .data(this.trades)
          .join('line')
          .lower()
          .transition()
          .duration(1000)
          .attr('x1', (d) => {
            return keyedOwners[d.source].placeX
          })
          .attr('y1', (d) => {
            return keyedOwners[d.source].placeY
          })
          .attr('x2', (d) => {
            return keyedOwners[d.target].placeX
          })
          .attr('y2', (d) => {
            return keyedOwners[d.target].placeY
          })
          .attr('stroke', this.colors['trade'])
          .attr('class', 'trades')
        teams
          .transition()
          .duration(1000)
          .attr('cx', (d) => {
            return keyedOwners[d.roster_id].placeX
          })
          .attr('cy', (d) => {
            return keyedOwners[d.roster_id].placeY
          })
        
        let teamNames = d3.selectAll('.teamNames')
          .transition()
          .duration(1000)
          .attr('x', (d) => {
            return keyedOwners[d.roster_id].placeX
          })
          .attr('y', (d) => {
            return keyedOwners[d.roster_id].placeY + this.teamRadius + 10
          })

        let playerNodes = g.selectAll('.playerNodes')
          .data(this.players)
          .join('circle') //by joining I was able to properly append circle for each of the players instead of creating an empty selection
          .attr('r', this.radius)
          .attr('class', 'playerNodes')
          .attr('stroke', 'black')
        let that = this;
        let playerSimulation = d3.forceSimulation(this.players)
          .force("collide", d3.forceCollide([this.radius]))
          .force("x", d3.forceX((d) => {
            return keyedOwners[d.roster_id].placeX
          }).strength([1]))
          .force("y", d3.forceY((d) => {
            return keyedOwners[d.roster_id].placeY
          }).strength([1]))
          .on('tick', playerTick) //multiple ticks can correspond to different simulations

        playerSimulation.tick(300)

        function playerTick() {
          playerNodes
            .transition()
            .duration(1000)
            .attr('cx', (d) => d.x)
            .attr('cy', (d) => d.y)
            .attr('fill', (d) => that.colors[d.type])
        }

        // show and update axis
        let winLabels = g.selectAll('.winLabels')
          .data(this.winRange)
          .join('text')
          .transition()
          .duration(1000)
          .attr('x', (d) => this.xScale(d))
          .attr('y', this.height - 20)
          .text((d) => d)
          .attr('class', 'winLabels')
      },
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
        
        let owners = acquisitions["weeks"][this.currentWeek]["players"]
        
        let rosterIds = _.keyBy(owners, 'roster_id')
        let teams = g.selectAll('teams')
          .data(owners)
          .enter()
          .append('circle')
          .attr('r', this.teamRadius)
          .attr('cx', (d) => {
            return d.placeX
          })
          .attr('cy', (d) => {
            return d.placeY
          })
          .attr('fill', 'white')
          .attr('stroke', (d,i) => {
            return this.teamColors[i]
          })
          .attr('stroke-width', 2)
          .attr('class', 'teams')
        
        let teamNames = g.selectAll('teamNames')
          .data(owners)
          .enter()
          .append('text')
          .attr('x', (d) => {
            return d.placeX
          })
          .attr('y', (d) => {
            return d.placeY + this.teamRadius + 10
          })
          .style('text-anchor', 'middle')
          .style('font-size', '10px')
          .attr('class', 'teamNames')
          .text((d) => d.display)
        
        let playerNodes = g.selectAll('player')
          .data(this.players)
          .enter()
          .append('circle')
          .attr('r', this.radius)
          .attr('class', 'playerNodes')
          .attr('fill', d => this.colors[d.type])
          .attr('stroke', 'black')

        this.updateTeamRosters(this.currentWeek)

        // create button to switch from week to week
        // needs to be cleaned up and under one group, also needs to be rethunk on how to design the animation slider
        // remake all this stuff in tailwind css with the aesthetically pleasing buttons and vue watchers
        let that = this;
        let weeks = Object.keys(acquisitions["weeks"])
        let weekButtons = g.selectAll('weekButtons')
          .data(weeks)
          .enter()
          .append('rect')
          .attr('x', this.width - 100)
          .attr('y', (d, i) => i * 20)
          .attr('width', 20)
          .attr('height', 15)
          .attr('fill', (d) => {
            return d === this.currentWeek ? 'orange' : 'white'
          })
          .attr('stroke', 'black')
          .attr('class', 'weekButtons')
          .on('click', function(d) {
            that.updateTeamRosters(d)
            d3.selectAll('.weekButtons').attr('fill', 'white')
            d3.select(this).attr('fill', 'orange')
          })
        
        let weekTexts = g.selectAll('weekTexts')
          .data(weeks)
          .enter()
          .append('text')
          .attr('x', this.width - 93)
          .attr('y', (d, i) => i * 20 + 11)
          .text((d, i) => i)
          .style('text-anchor', 'start')
          .style('font-size', '9px')
        
        g.append('line')
          .attr('x1', -50)
          .attr('y1', this.height - 40)
          .attr('x2', this.width)
          .attr('y2', this.height - 40)
          .attr('stroke', 'black')

        // instead of running force simulation on beeswarm for each player, you could save the player x,y spots sinces it the same for each cluster
      },
      placeTeams() {
        let data = acquisitions["weeks"][this.currentWeek]["players"]
        let groupedWins = _.groupBy(data, 'wins')
        let winExtent = d3.extent(_.map(data, d => d.wins))
        let winRange = d3.range(winExtent[0], winExtent[1] + 1)
        this.winRange = winRange
        this.xScale = d3.scaleBand()
          .domain(winRange)
          .range([0, this.width])

        let placements = []
        let players = []
        let maxColumn = 0
        _.forEach(groupedWins, (teams) => {
          _.forEach(teams, (team, index) => {
            team['total'] = teams.length
            team['placeX'] = this.xScale(team.wins)
            team['index'] = index
            //get players inside nodes
            _.forEach(Object.keys(team['acquisitions']), (a) => {
              _.forEach(team['acquisitions'][a], (p) => {
                players.push({
                  "type": a,
                  "roster_id": team.roster_id,
                  "wins": team.wins,
                  "player_id": p,
                  "display": team.display
                })
              })
            })
          })
          if (maxColumn < teams.length) {
            maxColumn = teams.length
          }
          this.players = players
        })
        let yScale = d3.scaleBand()
          .domain(d3.range(0, maxColumn + 1))
          .range([this.height - 30, 0])
        console.log(maxColumn)
        let owners = acquisitions["weeks"][this.currentWeek]["players"]
        _.forEach(owners, (o) => {
          o['placeY'] = yScale(o.index)
          //yScale(o.index) + (o.total / 2 ) * yScale.bandwidth() - yScale.bandwidth() * 3
        })
        
        // pull the trade board
        let trades = acquisitions["weeks"][this.currentWeek]["trades"]
        if (trades) {
          this.trades = trades
        } else {
          this.trades = []
        }
        return data
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