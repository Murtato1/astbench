---
layout: default
title: Home
---

# Welcome to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

<h2>Select a Benchmark Result</h2>
<select id="file-selector">
  <option>Select a model</option>
</select>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const jsonPath = "{{ site.baseurl }}/assets/json/benchmark_results.json"; // Path to your JSON file
  const dropdown = document.getElementById("file-selector");

  // Function to populate dropdown with model names from the JSON file
  async function populateDropdown() {
    try {
      const response = await fetch(jsonPath);
      const data = await response.json();

      // Extract unique model names from the JSON
      const models = [...new Set(data.map((item) => item.model.model))];

      // Populate the dropdown
      models.forEach((model) => {
        const option = document.createElement("option");
        option.value = model;
        option.textContent = model;
        dropdown.appendChild(option);
      });
    } catch (error) {
      console.error("Error populating dropdown:", error);
    }
  }

  // Event listener for dropdown selection
  dropdown.addEventListener("change", function () {
    const selectedModel = dropdown.value;
    if (selectedModel !== "Select a model") {
      fetchAndProcessData(selectedModel);
    }
  });

  // Fetch and process data for the selected model
  async function fetchAndProcessData(selectedModel) {
    try {
      const response = await fetch(jsonPath);
      const data = await response.json();

      // Filter data for the selected model
      const modelData = data.filter((item) => item.model.model === selectedModel);

      const metrics = {
        direct_match: [],
        fuzzy_match: [],
        codebleu: [],
        codebertscore: [],
        codebertscore_rescaled: [],
        code_success: [],
        syntax_match_score: [],
      };

      // Traverse model-specific data and extract metrics
      modelData.forEach((item) => {
        if (item.result) {
          item.result.forEach((result) => {
            if ("direct_match" in result && result.direct_match !== null) {
              metrics.direct_match.push(result.direct_match ? 1 : 0);
            }
            if ("fuzzy_match" in result && result.fuzzy_match !== null) {
              metrics.fuzzy_match.push(result.fuzzy_match / 100); // Normalize fuzzy_match to 0-1
            }
            if ("codebleu" in result && result.codebleu?.codebleu !== null) {
              metrics.codebleu.push(result.codebleu.codebleu);
            }
            if ("codebertscore" in result && result.codebertscore?.F1 !== null) {
              metrics.codebertscore.push(result.codebertscore.F1);
            }
            if ("codebertscore_rescaled" in result && result.codebertscore_rescaled?.F1 !== null) {
              metrics.codebertscore_rescaled.push(result.codebertscore_rescaled.F1);
            }
          });
        }

        if (item.result_summary) {
          if ("code_success" in item.result_summary) {
            metrics.code_success.push(item.result_summary.code_success);
          }
          if ("syntax_match_score" in item.result_summary) {
            metrics.syntax_match_score.push(item.result_summary.syntax_match_score);
          }
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

  // Render the chart
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
            data: Object.values(averages),
            backgroundColor: [
              "rgba(75, 192, 192, 0.2)",
              "rgba(54, 162, 235, 0.2)",
              "rgba(255, 206, 86, 0.2)",
              "rgba(153, 102, 255, 0.2)",
              "rgba(255, 159, 64, 0.2)",
              "rgba(201, 203, 207, 0.2)",
              "rgba(255, 99, 132, 0.2)",
            ],
            borderColor: [
              "rgba(75, 192, 192, 1)",
              "rgba(54, 162, 235, 1)",
              "rgba(255, 206, 86, 1)",
              "rgba(153, 102, 255, 1)",
              "rgba(255, 159, 64, 1)",
              "rgba(201, 203, 207, 1)",
              "rgba(255, 99, 132, 1)",
            ],
            borderWidth: 1,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: {
            beginAtZero: true,
          },
        },
        plugins: {
          legend: {
            display: true,
          },
          title: {
            display: true,
            text: `Benchmark Metrics for ${dropdown.value}`,
          },
        },
      },
    });
  }

  // Initialize the dropdown menu
  populateDropdown();
</script>
