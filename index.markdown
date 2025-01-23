---
layout: default
title: Home
---

# Welcome to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Benchmark Results

<h3 id="current-file"></h3>
<p>Below is the visualization of the benchmark evaluation metrics:</p>

<canvas id="benchmarkChart" width="800" height="400"></canvas> <!-- Enlarged chart -->

<script>
  const jsonPath = "{{ site.baseurl }}/assets/json/gpt-4o.json";

  async function fetchAndProcessData() {
    try {
      // Update the displayed file name without ".json"
      document.getElementById("current-file").textContent = `Currently Displaying: ${jsonPath.split('/').pop().replace('.json', '')}`;

      const response = await fetch(jsonPath);
      const data = await response.json();

      const metrics = {
        direct_match: [],
        fuzzy_match: [],
        codebleu: [],
        codebertscore: [],
        codebertscore_rescaled: []
      };

      // Traverse every item in the JSON
      data.forEach((item) => {
        if (item.result) {
          item.result.forEach((result) => {
            // Check for each metric explicitly and push valid values
            if ("direct_match" in result && result.direct_match !== null) {
              metrics.direct_match.push(result.direct_match ? 1 : 0);
            }
            if ("fuzzy_match" in result && result.fuzzy_match !== null) {
              metrics.fuzzy_match.push(result.fuzzy_match);
            }
            if ("codebleu" in result && result.codebleu?.codebleu !== null) {
              metrics.codebleu.push(result.codebleu.codebleu);
            }
            if ("codebertscore" in result && result.codebertscore?.F1 !== null) {
              metrics.codebertscore.push(result.codebertscore.F1);
            }
            if (
              "codebertscore_rescaled" in result &&
              result.codebertscore_rescaled?.F1 !== null
            ) {
              metrics.codebertscore_rescaled.push(result.codebertscore_rescaled.F1);
            }
          });
        }
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
            label: "Metrics (0-1 Range)",
            data: Object.values(averages).map((value, index) =>
              index === 1 ? null : value // Ignore fuzzy_match for this dataset
            ),
            backgroundColor: "rgba(75, 192, 192, 0.2)",
            borderColor: "rgba(75, 192, 192, 1)",
            borderWidth: 1,
            yAxisID: "y"
          },
          {
            label: "Fuzzy Match (0-100 Range)",
            data: Object.values(averages).map((value, index) =>
              index === 1 ? value : null // Include only fuzzy_match for this dataset
            ),
            backgroundColor: "rgba(255, 159, 64, 0.2)",
            borderColor: "rgba(255, 159, 64, 1)",
            borderWidth: 1,
            yAxisID: "y1"
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: true, // Ensure consistent aspect ratio
        scales: {
          y: {
            type: "linear",
            position: "left",
            title: {
              display: true,
              text: "Metrics (0-1)"
            },
            beginAtZero: true
          },
          y1: {
            type: "linear",
            position: "right",
            title: {
              display: true,
              text: "Fuzzy Match (0-100)"
            },
            grid: {
              drawOnChartArea: false // Prevents overlap of grid lines
            },
            beginAtZero: true
          }
        },
        plugins: {
          legend: {
            display: true
          },
          title: {
            display: true,
            text: "Benchmark Evaluation Metrics with Dual Axes"
          }
        }
      }
    });
  }

  fetchAndProcessData(); // Fetch and render on page load
</script>

