<template>
  <div id="app">
    <scoring-breakdown/>
  </div>
</template>

<script>
// import ScoringBreakdown from './components/BoxWhisker.vue'
// import ScoringBreakdown from './components/BarSparkLine.vue'
// import ScoringBreakdown from './components/ScoringBreakdown.vue'
// import ScoringBreakdown from './components/PointsForAgainst.vue'
// import ScoringBreakdown from './components/WeeklyScores.vue'
// import ScoringBreakdown from './components/Pyramids.vue'
import ScoringBreakdown from './components/Acquisitions.vue'

export default {
  name: 'app',
  components: {
    ScoringBreakdown
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
