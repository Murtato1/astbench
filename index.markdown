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

<h2>Select Benchmark Results</h2>
<div style="position: relative; display: inline-block;">
  <button id="dropdown-btn" style="padding: 8px 12px; background-color: #007BFF; color: white; border: none; cursor: pointer; border-radius: 5px;">
    Select Models ▼
  </button>
  <div id="model-dropdown" style="display: none; position: absolute; background: white; border: 1px solid #ccc; width: 300px; max-height: 250px; overflow-y: auto; padding: 5px;">
  </div>
</div>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  let jsonBasePath = "{{ site.baseurl }}/assets/json/";
  const datasets = {
    "benchmark_results_old.json": "Old Benchmark",
    "benchmark_results_new.json": "Colloquial Query Benchmark"
  };

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
  let allModels = {}; 

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

  const dropdownBtn = document.getElementById("dropdown-btn");
  const dropdownMenu = document.getElementById("model-dropdown");

  dropdownBtn.addEventListener("click", () => {
    dropdownMenu.style.display = dropdownMenu.style.display === "block" ? "none" : "block";
  });

  document.addEventListener("click", (event) => {
    if (!dropdownBtn.contains(event.target) && !dropdownMenu.contains(event.target)) {
      dropdownMenu.style.display = "none";
    }
  });

  async function populateDropdown() {
    dropdownMenu.innerHTML = "";
    allModels = {}; 

    for (const [file, datasetName] of Object.entries(datasets)) {
      try {
        const response = await fetch(jsonBasePath + file);
        const data = await response.json();

        data.forEach((item) => {
          let modelName = item.model?.model;
          if (modelName) {
            if (!allModels[modelName]) {
              allModels[modelName] = { old: false, colloquial: false };
            }
            allModels[modelName][datasetName === "Old Benchmark" ? "old" : "colloquial"] = true;
          }
        });
      } catch (error) {
        console.error(`Error loading ${file}:`, error);
      }
    }

    for (const [model, sources] of Object.entries(allModels)) {
      const container = document.createElement("div");
      container.style.marginBottom = "5px";

      const modelLabel = document.createElement("strong");
      modelLabel.textContent = model;
      container.appendChild(modelLabel);

      if (sources.old) {
        container.appendChild(createCheckbox(model, "Old Benchmark"));
      }
      if (sources.colloquial) {
        container.appendChild(createCheckbox(model, "Colloquial Query Benchmark"));
      }

      dropdownMenu.appendChild(container);
    }
  }

  function createCheckbox(model, dataset) {
    const label = document.createElement("label");
    label.style.display = "block";
    label.style.cursor = "pointer";
    label.style.padding = "3px";

    const checkbox = document.createElement("input");
    checkbox.type = "checkbox";
    checkbox.value = model;
    checkbox.dataset.source = dataset;
    checkbox.style.marginRight = "5px";

    checkbox.addEventListener("change", function () {
      if (this.checked) {
        fetchAndProcessData(model, dataset);
      } else {
        removeModelFromChart(`${model} (${dataset})`);
      }
    });

    label.appendChild(checkbox);
    label.appendChild(document.createTextNode(` ${dataset}`));
    return label;
  }

  async function fetchAndProcessData(selectedModel, dataset) {
    let selectedFile = dataset === "Colloquial Query Benchmark" ? "benchmark_results_new.json" : "benchmark_results_old.json";
    
    try {
      const response = await fetch(jsonBasePath + selectedFile);
      const data = await response.json();

      if (chartData.datasets.some(ds => ds.label === `${selectedModel} (${dataset})`)) {
        console.warn(`${selectedModel} (${dataset}) is already displayed.`);
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
          });
        }
      });

      updateChart(`${selectedModel} (${dataset})`, metrics);
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
