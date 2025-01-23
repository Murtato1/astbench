---
layout: default
title: Home
---

# Welcome 2to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Bar Chart Example

# Benchmark Results

Below is the visualization of the benchmark evaluation metrics:

<canvas id="benchmarkChart" width="400" height="200"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const jsonPath = "{{ site.baseurl }}/assets/json/benchmark_1.json";

  async function fetchAndProcessData() {
    try {
      const response = await fetch(jsonPath);
      const data = await response.json();

      const metrics = {
        direct_match: [],
        fuzzy_match: [],
        codebleu: [],
        codebertscore: [],
        codebertscore_rescaled: []
      };

      // Process JSON data
      data.forEach((item) => {
        item.result.forEach((result) => {
          if (result.direct_match !== null) {
            metrics.direct_match.push(result.direct_match ? 1 : 0);
          }
          if (result.fuzzy_match !== null) {
            metrics.fuzzy_match.push(result.fuzzy_match);
          }
          if (result.codebleu && result.codebleu.codebleu !== null) {
            metrics.codebleu.push(result.codebleu.codebleu);
          }
          if (result.codebertscore && result.codebertscore.F1 !== null) {
            metrics.codebertscore.push(result.codebertscore.F1);
          }
          if (
            result.codebertscore_rescaled &&
            result.codebertscore_rescaled.F1 !== null
          ) {
            metrics.codebertscore_rescaled.push(result.codebertscore_rescaled.F1);
          }
        });
      });

      // Calculate averages
      const averages = {};
      for (const [key, values] of Object.entries(metrics)) {
        averages[key] = values.length
          ? values.reduce((sum, val) => sum + val, 0) / values.length
          : 0;
      }

      renderChart(averages); // Render chart with processed data
    } catch (error) {
      console.error("Error fetching or processing JSON data:", error);
    }
  }

  function renderChart(averages) {
    const ctx = document.getElementById("benchmarkChart").getContext("2d");

    if (window.currentChart) {
      window.currentChart.destroy();
    }

    window.currentChart = new Chart(ctx, {
      type: "bar",
      data: {
        labels: Object.keys(averages),
        datasets: [
          {
            label: "Evaluation Metrics Averages",
            data: Object.values(averages),
            backgroundColor: [
              "rgba(75, 192, 192, 0.2)",
              "rgba(54, 162, 235, 0.2)",
              "rgba(255, 206, 86, 0.2)",
              "rgba(153, 102, 255, 0.2)",
              "rgba(255, 159, 64, 0.2)"
            ],
            borderColor: [
              "rgba(75, 192, 192, 1)",
              "rgba(54, 162, 235, 1)",
              "rgba(255, 206, 86, 1)",
              "rgba(153, 102, 255, 1)",
              "rgba(255, 159, 64, 1)"
            ],
            borderWidth: 1
          }
        ]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true
          }
        },
        plugins: {
          legend: {
            display: true
          },
          title: {
            display: true,
            text: "Benchmark Evaluation Metrics"
          }
        }
      }
    });
  }

  fetchAndProcessData(); // Fetch and render on page load
</script>
