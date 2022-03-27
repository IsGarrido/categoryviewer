<template>
  <div>
    <canvas :id="named"></canvas>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
/* eslint-disable no-debugger */
import Chart from "chart.js/auto";
import ChartDataLabels from "chartjs-plugin-datalabels";

let data = [
  {
    "Result Number": 1,
    "Different Words": 289,
    Adjectives: 165,
    "%": 57.09,
    "Diff %": 57.09,
  },
  {
    "Result Number": 5,
    "Different Words": 827,
    Adjectives: 383,
    "%": 46.31,
    "Diff %": 10.78,
  },
  {
    "Result Number": 10,
    "Different Words": 1324,
    Adjectives: 568,
    "%": 42.9,
    "Diff %": 3.41,
  },
  {
    "Result Number": 15,
    "Different Words": 1744,
    Adjectives: 700,
    "%": 40.13,
    "Diff %": 2.77,
  },
  {
    "Result Number": 20,
    "Different Words": 2097,
    Adjectives: 794,
    "%": 37.86,
    "Diff %": 2.27,
  },
  {
    "Result Number": 25,
    "Different Words": 2422,
    Adjectives: 872,
    "%": 36,
    "Diff %": 1.86,
  },
  {
    "Result Number": 30,
    "Different Words": 2753,
    Adjectives: 955,
    "%": 34.68,
    "Diff %": 1.36,
  },
];

data.forEach(
  (item) => (item["TotalDiff"] = data[0]["Diff %"] - item["Diff %"])
);

for (let i = 0; i < data.length; i++) {
  let item = data[i];
  if (i === 0) item["AdjDiff"] = 0;
  else
    item["AdjDiff"] =
      (item["Adjectives"] / data[i - 1]["Adjectives"]) * 100 - 100;
}

console.log(JSON.stringify(data));

let options = {
  responsive: true,
  lineTension: 0,
  scales: {
    yAxes: [
      {
        ticks: {
          beginAtZero: true,
          padding: 25,
        },
      },
    ],
  },
};

let labels = data.map((x) => x["Result Number"]);
let words = data.map((x) => x["Different Words"]);
let adjs = data.map((x) => x["Adjectives"]);
let prc = data.map((x) => x["%"]);
let prcDiff = data.map((x) => x["Diff %"]);
let totalDiff = data.map((x) => x["TotalDiff"]);
let adjDiff = data.map((x) => x["AdjDiff"]);

export default {
  props: {
    EnableWordCount: {
      Type: Boolean,
      default: false,
    },
    EnablePrc: {
      Type: Boolean,
      default: false,
    },
    EnableTotalDiff: {
      Type: Boolean,
      default: false,
    },
    named: {
      Type: String,
      required: true,
    },
  },
  mounted() {
    let datasets = [];
    if (this.EnableWordCount) {
      datasets = [
        ...datasets,
        {
          type: "line",
          label: "Different Words",
          data: words,
          backgroundColor: "rgba(255,0,0,1)",
          borderColor: "rgba(255,0,0,1)",
        },
        {
          type: "line",
          label: "Different Adjectives",
          data: adjs,
          backgroundColor: "rgba(0,255,0,1)",
          borderColor: "rgba(0,255,0,1)",
        },
      ];
    }

    if (this.EnablePrc) {
      datasets = [
        ...datasets,
        {
          type: "line",
          label: "Prc",
          data: prc,
          backgroundColor: "rgba(0,0,255,1)",
          borderColor: "rgba(0,0,255,1)",
        },
        {
          type: "line",
          label: "Diff %",
          data: prcDiff,
          backgroundColor: "rgba(0,255,255,1)",
          borderColor: "rgba(0,255,255,1)",
        },
      ];
    }

    if (this.EnableTotalDiff) {
      datasets = [
        ...datasets,
        {
          type: "line",
          label: "Total Diff %",
          data: totalDiff,
          backgroundColor: "rgba(255,0,255,1)",
          borderColor: "rgba(255,0,255,1)",
        },
        {
          type: "line",
          label: "adj Diff %",
          data: adjDiff,
          backgroundColor: "rgba(125,125,125,1)",
          borderColor: "rgba(125,125,125,1)",
        },
      ];
    }

    const planetChartData = {
      type: "line",
      plugins: [ChartDataLabels],
      data: {
        labels,
        datasets,
      },
      options: {
        layout: {
          padding: 20,
        },
        responsive: true,
        lineTension: 0,
        scales: {
          yAxes: [
            {
              ticks: {
                beginAtZero: true,
                padding: 25,
              },
            },
          ],
        },
        plugins: {
          datalabels: {
            formatter: function (value, context) {
              return Math.round(value * 100) / 100;
            },
            backgroundColor: function (context) {
              return context.dataset.backgroundColor;
            },
            borderRadius: 4,
            color: "white",
            font: {
              weight: "bold",
            },
            padding: 6,
          },
        },
      },
    };

    const ctx = document.getElementById(this.named);
    new Chart(ctx, planetChartData);
  },
  data() {
    return {};
  },
};
</script>

<style scoped></style>
