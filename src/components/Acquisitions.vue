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
        height: 500,
        width: 750,
        margin: {top: 20, right: 20, bottom: 20, left: 20},
        colors: {
          'draft': '#80b1d3',
          'trade': '#fb8072',
          'waiver': '#fdb462'
        },
        players: []
      }
    },
    mounted () {
      this.initData();
      this.initGraph();
    },
    methods: {
      initData() {
        let data = acquisitions["weeks"]["week0"]["players"]
        let acquisitionTypes = Object.keys(acquisitions["weeks"]["week0"]["players"]["1"]["acquisitions"])
        let players = []
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
        

        let radius = 8

        let xScale = d3.scaleLinear()
          .domain([0, 4])
          .range([0, width])
        let nodes = _.map(Object.keys(acquisitions["weeks"]["week0"]["players"]), (d) => {
          return {
            id: d,
            wins: acquisitions["weeks"]["week0"]["players"][d]["wins"]
          }
        })
        
        let bilinks = []
        let links = acquisitions["weeks"]["week0"]["trades"]


        let trades = g.selectAll('trades')
          .data(links)
          .enter()
          .append("line")
            .attr("class", "trades");


        let rosterIds = _.keyBy(nodes, 'id')
        let rosterRadius = 48
        let rosters = g.selectAll('rosters')
          .data(nodes.filter(function(d) { return d.id; }))
          .enter()
          .append('circle') 
          .attr('r', rosterRadius)
          .attr('fill', 'white')
          .attr('stroke', 'black')
          .attr('class', 'rosters')
        let playerNodes = g.selectAll('player')
          .data(this.players)
          .enter()
          .append('circle')
          .attr('r', radius)
          .attr('class', 'player')
          .attr('fill', d => this.colors[d.type])
          .attr('stroke', 'black')

        let teamSimulation = d3.forceSimulation(nodes)
          .force("collide", d3.forceCollide([rosterRadius]))
          .force("x", d3.forceX((d) => {
            return xScale(d.wins)
          }).strength([1]))
          .force("y", d3.forceY(height / 2).strength([1]))
          .force("link", d3.forceLink(links).id(function(d) {return d.id; }))
          .on('tick', tick) //multiple ticks can correspond to different simulations

        let playerSimulation = d3.forceSimulation(this.players)
          .force("collide", d3.forceCollide([radius]))
          .force("x", d3.forceX((d) => {
            return xScale(d.wins)
          }).strength([1]))
          .force("y", d3.forceY(height / 2).strength([1]))
          .on('tick', playerTick) //multiple ticks can correspond to different simulations
        

        function playerTick() {
          playerNodes
            .attr('cx', (d) => d.x)
            .attr('cy', (d) => d.y)
        }

        function tick() {
          rosters
            .attr('cx', (d) => d.x)
            .attr('cy', (d) => d.y)
          
          trades
            .attr('x1', function(d) {
              return d.source.x
            })
            .attr('y1', function(d) {
              return d.source.y
            })
            .attr('x2', function(d) {
              return d.target.x
            })
            .attr('y2', function(d) {
              return d.target.y
            })
            .attr('class', (d) => {
              return 'trades'
            })
          
          //need to make this line a curved path, will be more aesthetically pleasing
          //method1: intermediate "node" -> this was hard to replicate
          //method2: manually calculate path or create helper function, more research required
          //method3: preprocess the location of each node, save it to vuex state and use that to determine center for each node
        }
        console.log('links', links)
        console.log('bilinks', bilinks)
        console.log('nodes', nodes)
        
        //use d3 cluster to group nodes inside a circle

        //how to positions teams with the same number of wins? 
        //method1: force simulation for each team, use center of each circle as the forcex,y for the player nodes +less hardcode, flexible, generic
        //method2: manually calculate the y position for each team, take number of teams into account, make aesthetically pleasing, -lots of manual work, not memory intrusive

      }
    },
  }
</script>

<style>
.trades {
  fill: none;
  stroke: #bbb;
}
</style>