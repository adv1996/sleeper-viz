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
  import teams from '../data/week9_output.json';
  // import scores from '../data/espn_weekly_scores.json';
  // import teams from '../data/espn_data.json';

  export default {
    data() {
      return {
        height: 500,
        width: 750,
        margin: {top: 25, right: 20, bottom: 20, left: 20},
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
        
        // making a vertical box and whisker plot
        let boxScores = scores["games"]
        let scoreRange = d3.extent(boxScores, (d) => d.points)
        let yScale = d3.scaleLinear()
          .domain([scoreRange[0] - 10, scoreRange[1] + 10])
          .range([height - this.margin.top - this.margin.bottom, 0]) // need to resolve this whole margin issue
        let data = _.groupBy(boxScores, 'roster_id')
        function sortNumber(a, b) {
          return a - b;
        }
        let leagueMean = d3.mean(boxScores, d => d.points)
        
        let bwdata = _.map(data, (d) => {
          let points = _.map(d, d => d.points).sort(sortNumber)
          console.log(points, d3.quantile(points, 0.25), d3.quantile(points, 0.75))
          return {
            "max": d3.max(points),
            "min": d3.min(points),
            "sum": d3.sum(points),
            "median": d3.quantile(points, 0.5),
            "first_quantile": d3.quantile(points, 0.25),
            "third_quantile": d3.quantile(points, 0.75),
            "mean": d3.mean(points),
            "roster_id": d[0].roster_id
          }
        })
        bwdata = _.orderBy(bwdata, ['max'], ['desc'])
        let teamOrder = _.map(bwdata, (d) => {
          return d.roster_id
        })
        let xScale = d3.scaleBand()
          .domain(teamOrder)
          .range([this.margin.left, width])
        // phase one make a mix and max line
        let boxes = g.selectAll('boxes')
          .data(bwdata)
          .enter()
          .append('g')
        
        boxes.selectAll('lines')
          .data(d => [d])
          .enter()
          .append('line')
          .attr('x1', d => xScale(d.roster_id))
          .attr('y1', d => yScale(d.min))
          .attr('x2', d => xScale(d.roster_id))
          .attr('y2', d => yScale(d.max))
          .attr('stroke', 'black')
        
        let offset = 5

        boxes.selectAll('bw')
          .data(d => [d])
          .enter()
          .append('rect')
          .attr('x', d => xScale(d.roster_id) - offset)
          .attr('y', d => yScale(d.third_quantile))
          .attr('width', offset * 2)
          .attr('height', d => yScale(d.first_quantile) - yScale(d.third_quantile))
          .attr('stroke', 'black')
          .attr('fill', '#bdbdbd')
        
        boxes.selectAll('medianlines')
          .data(d => [d])
          .enter()
          .append('line')
          .attr('x1', d => xScale(d.roster_id) - offset)
          .attr('y1', d => yScale(d.median))
          .attr('x2', d => xScale(d.roster_id) + offset)
          .attr('y2', d => yScale(d.median))
          .attr('stroke', 'black')

        boxes.selectAll('points')
          .data(d => [
            {
              "value": d.max,
              "roster_id": d.roster_id
            },
            {
              "value": d.min,
              "roster_id": d.roster_id
            }
          ])
          .enter()
          .append('circle')
          .attr('cx', (d) => {
            return xScale(d.roster_id)
          })                        
          .attr('cy', d => yScale(d.value))
          .attr('r', 3)
          .attr('fill', 'black')
        
        g
          .append('line')
          .attr('x1', 0)
          .attr('y1', yScale(leagueMean))
          .attr('x2', this.width - this.margin.right)
          .attr('y2', yScale(leagueMean))
          .attr('stroke', 'red')
          .style('stroke-dasharray', "6,6")
        
        g.append('text')
          .attr('x', this.width / 2 - 40)
          .attr('y', 20)
          .text('Scores Through Week 9')
        
        // let espnRosterIds = _.groupBy(teams['teams'], 'id')

        boxes.selectAll('avatars')
          .data(d => [d])
          .enter()
          .append('image')
          .attr('xlink:href', (d) => {
            let avatar = teams["players"][d.roster_id].avatar
            return 'https://sleepercdn.com/avatars/thumbs/' + avatar
          })
          .attr('width', 17)
          .attr('height', 17)
          .attr('x', d => xScale(d.roster_id) - 8)
          .attr('y', d => yScale(d.max) - 25)
        // boxes.selectAll('labels')
        //   .data(d => [d])
        //   .enter()
        //   .append('text')
        //   .attr('x', d => xScale(d.roster_id))
        //   .attr('y', d => yScale(d.max) - 15)
        //   .style('font-size', '12px')
        //   .style('text-anchor', 'middle')
        //   .text((d) => {
        //     return espnRosterIds[d.roster_id][0].abbrev
        //   })
        let yAxis = d3.axisLeft()
          .scale(yScale)
          .tickSizeInner([0])
          .tickSizeOuter([0])
        
        g.append('g')
          .attr('transform', "translate(" + 0 + "," + 0 + ")")
          .call(yAxis)
        
        d3.selectAll(".domain").remove();
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>