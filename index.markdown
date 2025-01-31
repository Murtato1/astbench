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

<h2>Select Dataset</h2>
<select id="dataset-selector" style="margin-bottom: 10px;">
  <option value="benchmark_results_old.json">Original Results</option>
  <option value="benchmark_results_new.json">Colloquial Query Results</option>
</select>

<h2>Select Benchmark Results</h2>
<div style="position: relative; display: inline-block;">
  <button id="dropdown-btn" style="padding: 8px 12px; background-color: #007BFF; color: white; border: none; cursor: pointer; border-radius: 5px;">
    Select Models ▼
  </button>
  <div id="model-dropdown" style="display: none; position: absolute; background: white; border: 1px solid #ccc; width: 200px; max-height: 200px; overflow-y: auto;">
  </div>
</div>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  let jsonBasePath = "{{ site.baseurl }}/assets/json/";
  let selectedJsonFile = "benchmark_results_1.json"; // Default dataset
  const datasetSelector = document.getElementById("dataset-selector");
  const dropdownBtn = document.getElementById("dropdown-btn");
  const dropdownMenu = document.getElementById("model-dropdown");

  let chartData = {
    labels: [],
    datasets: []
  };

  let colors = [
    "rgba(255, 99, 132, 0.5)", "rgba(54, 162, 235, 0.5)",
    "rgba(255, 206, 86, 0.5)", "rgba(75, 192, 192, 0.5)",
    "rgba(153, 102, 255, 0.5)", "rgba(255, 159, 64, 0.5)",
    "rgba(201, 203, 207, 0.5)"
  ];

  let borderColors = [
    "rgba(255, 99, 132, 1)", "rgba(54, 162, 235, 1)",
    "rgba(255, 206, 86, 1)", "rgba(75, 192, 192, 1)",
    "rgba(153, 102, 255, 1)", "rgba(255, 159, 64, 1)",
    "rgba(201, 203, 207, 1)"
  ];

  let usedColors = {}; 
  let currentColorIndex = 0;

  let ctx = document.getElementById("benchmarkChart").getContext("2d");
  let benchmarkChart = new Chart(ctx, {
    type: "bar",
    data: chartData,
    options: {
      responsive: true,
      maintainAspectRatio: true,
      scales: { y: { beginAtZero: true } },
      plugins: {
        legend: { display: true },
        title: { display: true, text: "Benchmark Evaluation Metrics" }
      }
    }
  });

  // Show/Hide Dropdown Menu
  dropdownBtn.addEventListener("click", () => {
    dropdownMenu.style.display = dropdownMenu.style.display === "block" ? "none" : "block";
  });

  // Close dropdown if clicked outside
  document.addEventListener("click", (event) => {
    if (!dropdownBtn.contains(event.target) && !dropdownMenu.contains(event.target)) {
      dropdownMenu.style.display = "none";
    }
  });

  // Handle dataset selection change
  datasetSelector.addEventListener("change", function () {
    selectedJsonFile = this.value; 
    chartData.datasets = []; // Reset chart
    chartData.labels = []; 
    benchmarkChart.update();
    usedColors = {}; 
    currentColorIndex = 0; 
    populateDropdown(); // Refresh model list
  });

  // Populate dropdown menu with model checkboxes
  async function populateDropdown() {
    try {
      const response = await fetch(jsonBasePath + selectedJsonFile);
      const data = await response.json();
      const models = [...new Set(data.map((item) => item.model.model).filter(m => m))];

      dropdownMenu.innerHTML = ""; // Clear old entries

      models.forEach((model) => {
        const label = document.createElement("label");
        label.style.display = "block";
        label.style.cursor = "pointer";
        label.style.padding = "5px";

        const checkbox = document.createElement("input");
        checkbox.type = "checkbox";
        checkbox.value = model;
        checkbox.style.marginRight = "5px";

        checkbox.addEventListener("change", function () {
          if (this.checked) {
            fetchAndProcessData(model);
          } else {
            removeModelFromChart(model);
          }
        });

        label.appendChild(checkbox);
        label.appendChild(document.createTextNode(model));
        dropdownMenu.appendChild(label);
      });
    } catch (error) {
      console.error("Error populating dropdown:", error);
    }
  }

  // Fetch and process data for the selected model
  async function fetchAndProcessData(selectedModel) {
    try {
      const response = await fetch(jsonBasePath + selectedJsonFile);
      const data = await response.json();

      if (chartData.datasets.some(ds => ds.label === selectedModel)) {
        console.warn(`${selectedModel} is already displayed.`);
        return;
      }

      if (!(selectedModel in usedColors)) {
        usedColors[selectedModel] = {
          backgroundColor: colors[currentColorIndex % colors.length],
          borderColor: borderColors[currentColorIndex % borderColors.length]
        };
        currentColorIndex++;
      }

      const modelData = data.filter((item) => item.model.model === selectedModel);

      const metrics = {
        direct_match: [], fuzzy_match: [], codebleu: [],
        codebertscore: [], codebertscore_rescaled: [],
        code_success: [], syntax_match_score: []
      };

      modelData.forEach((item) => {
        if (item.result) {
          item.result.forEach((result) => {
            if ("direct_match" in result && result.direct_match !== null) {
              metrics.direct_match.push(result.direct_match ? 1 : 0);
            }
            if ("fuzzy_match" in result && result.fuzzy_match !== null) {
              metrics.fuzzy_match.push(result.fuzzy_match / 100); 
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

      const averages = {};
      for (const [key, values] of Object.entries(metrics)) {
        averages[key] = values.length
          ? values.reduce((sum, val) => sum + val, 0) / values.length
          : 0;
      }

      updateChart(selectedModel, averages);
    } catch (error) {
      console.error("Error fetching or processing JSON data:", error);
    }
  }

  function removeModelFromChart(selectedModel) {
    chartData.datasets = chartData.datasets.filter(ds => ds.label !== selectedModel);
    benchmarkChart.update();
  }

  populateDropdown();
</script>
