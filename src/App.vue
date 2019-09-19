<template>
  <div id="app">
    <scoring-breakdown/>
  </div>
</template>

<script>
import ScoringBreakdown from './components/ScoringBreakdown.vue'

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
