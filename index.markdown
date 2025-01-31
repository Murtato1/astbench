---
layout: default
title: Home
---

# Welcome to AstroCodeBench

<div style="text-align: center; margin-top: 20px;">
  <img src="assets/pics/cosmic_logo.png" alt="Logo" style="height: 100px; margin: 0 10px;">
  <img src="assets/pics/ut_logo.png" alt="Longhorn" style="height: 100px; margin: 0 10px;">
</div>


AstroCodeBench is a benchmark designed to test LLM proficiency with using astronomy domain code packages. View all the currently benchmarked models below.

<h2>Select a Benchmark Result</h2>
<select id="model-selector">
  <option>Select a model</option>
</select>

<button id="clear-chart" style="margin-top: 10px; padding: 8px 12px; background-color: red; color: white; border: none; cursor: pointer; border-radius: 5px;">
  Clear Chart
</button>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const jsonPath = "{{ site.baseurl }}/assets/json/benchmark_results_new.json"; // Path to the single JSON file
  const dropdown = document.getElementById("model-selector");

  let chartData = {
    labels: [],
    datasets: []
  };

  let colors = [
    "rgba(255, 99, 132, 0.5)",
    "rgba(54, 162, 235, 0.5)",
    "rgba(255, 206, 86, 0.5)",
    "rgba(75, 192, 192, 0.5)",
    "rgba(153, 102, 255, 0.5)",
    "rgba(255, 159, 64, 0.5)",
    "rgba(201, 203, 207, 0.5)"
  ];
  
  let borderColors = [
    "rgba(255, 99, 132, 1)",
    "rgba(54, 162, 235, 1)",
    "rgba(255, 206, 86, 1)",
    "rgba(75, 192, 192, 1)",
    "rgba(153, 102, 255, 1)",
    "rgba(255, 159, 64, 1)",
    "rgba(201, 203, 207, 1)"
  ];

  let usedColors = {}; // Track assigned colors per model
  let currentColorIndex = 0;

  let ctx = document.getElementById("benchmarkChart").getContext("2d");
  let benchmarkChart = new Chart(ctx, {
    type: "bar",
    data: chartData,
    options: {
      responsive: true,
      maintainAspectRatio: true,
      scales: {
        y: { beginAtZero: true }
      },
      plugins: {
        legend: { display: true },
        title: { display: true, text: "Benchmark Evaluation Metrics" }
      }
    }
  });

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

      // Check if this model is already displayed, avoid duplicates
      if (chartData.datasets.some(ds => ds.label === selectedModel)) {
        console.warn(`${selectedModel} is already displayed.`);
        return;
      }

      // Assign a unique color to the model
      if (!(selectedModel in usedColors)) {
        usedColors[selectedModel] = {
          backgroundColor: colors[currentColorIndex % colors.length],
          borderColor: borderColors[currentColorIndex % borderColors.length]
        };
        currentColorIndex++;
      }

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

      updateChart(selectedModel, averages); // Render chart with processed data
    } catch (error) {
      console.error("Error fetching or processing JSON data:", error);
    }
  }

  // Update the chart with new model data
  function updateChart(selectedModel, averages) {
    if (chartData.labels.length === 0) {
      chartData.labels = Object.keys(averages);
    }
    
    chartData.datasets.push({
      label: selectedModel,
      data: Object.values(averages),
      backgroundColor: usedColors[selectedModel].backgroundColor,
      borderColor: usedColors[selectedModel].borderColor,
      borderWidth: 1
    });
    
    benchmarkChart.update();
  }

  document.getElementById("clear-chart").addEventListener("click", function () {
    chartData.datasets = []; // Remove all datasets
    chartData.labels = []; // Clear labels
    benchmarkChart.update(); // Update the chart
    usedColors = {}; // Reset color tracking
    currentColorIndex = 0; // Reset color index
    });


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
