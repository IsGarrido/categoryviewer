<template>
  <div>
      <canvas id="chart"></canvas>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
/* eslint-disable no-debugger */
//import Chart from 'chart.js'
import Chart from 'chart.js/auto';

let data = [
  {
    "Result Number": 5,
    "Different Words": 123,
    "Adjectives": 123,
    "%": "50"
  },
  {
    "Result Number": 10,
    "Different Words": 1324,
    "Adjectives": 568,
    "%": 42.9
  },
  {
    "Result Number": 15,
    "Different Words": 1744,
    "Adjectives": 700,
    "%": 40.13
  },
  {
    "Result Number": 20,
    "Different Words": 2097,
    "Adjectives": 794,
    "%": 37.86
  },
  {
    "Result Number": 25,
    "Different Words": 2422,
    "Adjectives": 872,
    "%": 36
  }
];

let options = {
    responsive: true,
    lineTension: 0,
    scales: {
      yAxes: [
        {
          ticks: {
            beginAtZero: true,
            padding: 25
          }
        }
      ]
    }
  };

    let labels = data.map( x => x["Result Number"]).map(String);
      let words = data.map( x => x["Different Words"]).map(String);
      let adjs = data.map( x => x["Adjectives"]).map(String);
      let prc = data.map( x => x["%"]).map(String);

export default {
  mounted() {

const planetChartData = {
  type: 'line',
  data: {
    labels,
    datasets: [
        { type: 'line',     label: 'Different Words',       data: words,    borderColor: 'rgba(255,0,0,1)' },
        { type: 'line',     label: 'Different Adjectives',  data: adjs,     borderColor: 'rgba(0,255,0,1)' },
        { type: 'line',     label: 'Prc',                   data: prc,      borderColor: 'rgba(0,0,255,1)' },
    ]
  },
  options: {
    responsive: true,
    lineTension: 0,
    scales: {
      yAxes: [
        {
          ticks: {
            beginAtZero: true,
            padding: 25
          }
        }
      ]
    }
  }
};

    const ctx = document.getElementById('chart');
    new Chart(ctx, planetChartData);
  },
  data(){
    return {   }
  }
}
</script>

<style scoped>

</style>