---
layout: default
title: Home
---

# Welcome to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

<div style="text-align: center; margin-top: 20px;">
  <img src="assets/pics/rocket.png" alt="Rocket" style="height: 100px; margin: 0 10px;">
  <img src="assets/pics/longhorn.png" alt="Longhorn" style="height: 100px; margin: 0 10px;">
  <img src="assets/pics/planet.png" alt="Planet" style="height: 100px; margin: 0 10px;">
</div>

<h2>Select a Benchmark Result</h2>
<select id="model-selector">
  <option>Select a model</option>
</select>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const jsonPath = "{{ site.baseurl }}/assets/json/benchmark_results.json"; // Path to the single JSON file
  const dropdown = document.getElementById("model-selector");

  // Populate dropdown menu with model names
  async function populateDropdown() {
    try {
      const response = await fetch(jsonPath);
      const data = await response.json();

      // Extract unique model names
      const models = [...new Set(data.map((item) => item.model.model))];

      // Populate dropdown with models
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
        syntax_match_score: []
      };

      // Traverse filtered data for the selected model and extract metrics
      modelData.forEach((item) => {
        if (item.result) {
          item.result.forEach((result) => {
            if ("direct_match" in result && result.direct_match !== null) {
              metrics.direct_match.push(result.direct_match ? 1 : 0);
            }
            if ("fuzzy_match" in result && result.fuzzy_match !== null) {
              metrics.fuzzy_match.push(result.fuzzy_match / 100); // Normalize fuzzy_match
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

        // Extract values from `result_summary`
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

      renderChart(selectedModel, averages); // Render chart with processed data
    } catch (error) {
      console.error("Error fetching or processing JSON data:", error);
    }
  }

  // Render the chart
  function renderChart(selectedModel, averages) {
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
            label: `Metrics for ${selectedModel} (0-1 Range)`,
            data: Object.values(averages),
            backgroundColor: [
              "rgba(75, 192, 192, 0.2)",
              "rgba(54, 162, 235, 0.2)",
              "rgba(255, 206, 86, 0.2)",
              "rgba(153, 102, 255, 0.2)",
              "rgba(255, 159, 64, 0.2)",
              "rgba(201, 203, 207, 0.2)",
              "rgba(255, 99, 132, 0.2)"
            ],
            borderColor: [
              "rgba(75, 192, 192, 1)",
              "rgba(54, 162, 235, 1)",
              "rgba(255, 206, 86, 1)",
              "rgba(153, 102, 255, 1)",
              "rgba(255, 159, 64, 1)",
              "rgba(201, 203, 207, 1)",
              "rgba(255, 99, 132, 1)"
            ],
            borderWidth: 1
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: true,
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
            text: `Benchmark Evaluation Metrics for ${selectedModel}`
          }
        }
      }
    });
  }

  // Initialize the dropdown menu
  populateDropdown();
</script>

<h2>Team</h2>
<div id="team-section" style="display: flex; justify-content: center; flex-wrap: wrap; gap: 20px; margin-top: 20px;">

  <div class="team-member" style="text-align: center; max-width: 200px;">
    <img src="assets/pics/member1.png" alt="Member 1" style="width: 100px; height: 100px; border-radius: 50%;">
    <h3 style="margin: 10px 0;">Faculty 1</h3>
    <p style="color: gray; margin-bottom: 5px;">University name, Lab name</p>
    <button onclick="toggleDetails('member1')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member1" style="display: none; color: darkgray; margin-top: 10px;">details about faculty member and a link or something similar.</p>
  </div>

  <div class="team-member" style="text-align: center; max-width: 200px;">
    <img src="assets/pics/member2.png" alt="Member 2" style="width: 100px; height: 100px; border-radius: 50%;">
    <h3 style="margin: 10px 0;">Faculty 2</h3>
    <p style="color: gray; margin-bottom: 5px;">University name, Lab name</p>
    <button onclick="toggleDetails('member2')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member2" style="display: none; color: darkgray; margin-top: 10px;">details about faculty member and a link or something similar.</p>
  </div>

  <div class="team-member" style="text-align: center; max-width: 200px;">
    <img src="assets/pics/member3.png" alt="Member 3" style="width: 100px; height: 100px; border-radius: 50%;">
    <h3 style="margin: 10px 0;">Faculty 3</h3>
    <p style="color: gray; margin-bottom: 5px;">University name, Lab name</p>
    <button onclick="toggleDetails('member3')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member3" style="display: none; color: darkgray; margin-top: 10px;">details about faculty member and a link or something similar.</p>
  </div>

</div>

<script>
  function toggleDetails(memberId) {
    const details = document.getElementById(`details-${memberId}`);
    if (details.style.display === "none") {
      details.style.display = "block";
    } else {
      details.style.display = "none";
    }
  }
</script>
