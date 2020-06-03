<template>
  <div id="app">
    <h1>UV-Vis Spectrometer Simulation</h1>

    <!--<line-chart></line-chart>-->
    <line-with-line></line-with-line>

    <!--<b-button size="sm" variant="info">Test</b-button>-->

  </div>
</template>

<script>
  //import LineChart from '@/components/LineChart'
  import LineWithLine from "./components/LineWithLine";

  export default {
    name: 'App',
    components: {
      //LineChart,
      LineWithLine
    },
  }

</script>
