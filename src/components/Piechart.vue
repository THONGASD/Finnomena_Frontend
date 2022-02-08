<template>
  <div>
    <div id="chart-container">
      <canvas id="chart-js"></canvas>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import Chart from "chart.js";
// const Chart = require('chart.js');
export default {
  props: ["ChartData"],
  name: "Piechart",
  mounted() {
    this.createChart();
  },
  watch: {
    ChartData(val) {
      this.createChart();
      return val;
    },
  },
  methods: {
    createChart() {
      const ctx = document.getElementById("chart-js");
      ctx.remove();
      const canvas = document.createElement("canvas");
      console.log(canvas);
      canvas.setAttribute("id", "chart-js");
      canvas.setAttribute("width", "auto");
      canvas.setAttribute("height", "300");
      document.getElementById("chart-container").appendChild(canvas);
      new Chart(canvas, {
        type: "doughnut",
        data: {
          labels: this.ChartData.labels,
          datasets: [
            {
              label: "สัดส่วนประเภททรัพย์สินที่ลงทุนของกองทุน",
              data: this.ChartData.data,

              borderWidth: 1,
              backgroundColor: [
                "rgba(255, 99, 132, 0.2)",
                "rgba(255, 159, 64, 0.2)",
                "rgba(255, 205, 86, 0.2)",
                "rgba(75, 192, 192, 0.2)",
                "rgba(54, 162, 235, 0.2)",
                "rgba(153, 102, 255, 0.2)",
                "rgba(201, 203, 207, 0.2)",
              ],
              borderColor: [
                "rgb(255, 99, 132)",
                "rgb(255, 159, 64)",
                "rgb(255, 205, 86)",
                "rgb(75, 192, 192)",
                "rgb(54, 162, 235)",
                "rgb(153, 102, 255)",
                "rgb(201, 203, 207)",
              ],
            },
          ],
        },
        options: {
          aspectRatio: 1,
          scales: {
            y: {
              beginAtZero: true,
            },
          },
        },
      });
    },
  },
};
</script>

<style></style>
