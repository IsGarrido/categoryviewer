<template>
  <div>
      <canvas id="chart"></canvas>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */

//import Chart from 'chart.js'
import Chart from 'chart.js/auto';



export default {
  props: ['labels', 'datam', 'dataf'],
  mounted() {

const planetChartData = {
  type: "radar",
  data: {
    labels: this.labels,
    datasets: [
      {
        label: "Masculino",
        data: this.datam,
        backgroundColor: "rgba(102, 204, 255,.5)",
        borderColor: "#000000",
        borderWidth: 2
      },
      {
        label: "Femenino",
        data: this.dataf,
        backgroundColor: "rgba(255, 153, 255,.5)",
        borderColor: "#000000",
        borderWidth: 2
      }
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