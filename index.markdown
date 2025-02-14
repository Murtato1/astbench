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
  <div id="model-dropdown" style="display: none; position: absolute; background: white; border: 1px solid #ccc; width: 200px; max-height: 200px; overflow-y: auto;">
  </div>
</div>

<input type="file" id="json-upload" accept=".json">

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>

let allModels = {}; // Store unique models and datasets
let uploadedData = []; // Store uploaded JSON data

const jsonSources = [
  { path: "{{ site.baseurl }}/assets/json/benchmark_results_new.json", prefix: "[New] " },
  { path: "{{ site.baseurl }}/assets/json/benchmark_results_old.json", prefix: "[Old] " }
];

const dropdownBtn = document.getElementById("dropdown-btn");
const dropdownMenu = document.getElementById("model-dropdown");
const fileInput = document.getElementById("json-upload");

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

let usedColors = {}; 
let currentColorIndex = 0;

let ctx = document.getElementById("benchmarkChart").getContext("2d");
let benchmarkChart = new Chart(ctx, {
  type: "bar",
  data: chartData,
  options: {
    responsive: true,
    maintainAspectRatio: true,
    scales: {
      y: { beginAtZero: true, max: 1 }
    },
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

// Handle File Upload
fileInput.addEventListener("change", function (event) {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();

  reader.onload = function (e) {
    try {
      const jsonData = JSON.parse(e.target.result);

      if (!Array.isArray(jsonData)) {
        throw new Error("Invalid JSON format: Root must be an array.");
      }

      processUploadedData(jsonData, file.name);
    } catch (error) {
      console.error("Error processing uploaded JSON:", error);
      alert("Error: Unable to read JSON file. Ensure it's in the correct format.");
    }
  };

  reader.readAsText(file);
});

async function fetchAllData() {
  try {
    const allData = [];
    for (const source of jsonSources) {
      const response = await fetch(source.path);
      const data = await response.json();
      data.forEach(entry => {
        entry.sourcePrefix = source.prefix;
      });
      allData.push(...data);
    }
    allData.push(...uploadedData); // Include uploaded JSON data
    return allData;
  } catch (error) {
    console.error("Error fetching JSON files:", error);
    return uploadedData; // At least return the uploaded data if remote fetch fails
  }
}

async function populateDropdown() {
  try {
    const data = await fetchAllData();
    const models = new Set();

    dropdownMenu.innerHTML = ""; 

    data.forEach((item) => {
      if (item.model && item.model.model) {
        const sourcePrefix = item.sourcePrefix || "[Uploaded] ";
        const modelLabel = `${sourcePrefix}${item.model.model}`;

        if (!models.has(modelLabel)) {
          models.add(modelLabel);

          const label = document.createElement("label");
          label.style.display = "block";
          label.style.cursor = "pointer";
          label.style.padding = "5px";

          const checkbox = document.createElement("input");
          checkbox.type = "checkbox";
          checkbox.value = modelLabel;
          checkbox.style.marginRight = "5px";

          checkbox.addEventListener("change", function () {
            if (this.checked) {
              fetchAndProcessData(item.model.model, sourcePrefix);
            } else {
              removeModelFromChart(modelLabel);
            }
          });

          label.appendChild(checkbox);
          label.appendChild(document.createTextNode(modelLabel));
          dropdownMenu.appendChild(label);
        }
      }
    });
  } catch (error) {
    console.error("Error populating dropdown:", error);
  }
}

async function fetchAndProcessData(selectedModel, sourcePrefix) {
  try {
    const data = await fetchAllData();
    const fullModelLabel = `${sourcePrefix}${selectedModel}`;

    if (chartData.datasets.some(ds => ds.label === fullModelLabel)) {
      console.warn(`${fullModelLabel} is already displayed.`);
      return;
    }

    if (!(fullModelLabel in usedColors)) {
      usedColors[fullModelLabel] = {
        backgroundColor: colors[currentColorIndex % colors.length],
        borderColor: borderColors[currentColorIndex % borderColors.length]
      };
      currentColorIndex++;
    }

    const modelData = data.filter((item) => item.model.model === selectedModel && item.sourcePrefix === sourcePrefix);

    let metrics = {};

    modelData.forEach((item) => {
      if (item.result_summary) {
        for (const [key, value] of Object.entries(item.result_summary)) {
          if (typeof value === "number") {
            if (!metrics[key]) {
              metrics[key] = [];
            }
            metrics[key].push(value > 1 ? value / 100 : value);
          }
        }
      }
    });

    let averages = {};
    for (const [key, values] of Object.entries(metrics)) {
      averages[key] = values.reduce((sum, val) => sum + val, 0) / values.length;
    }

    updateChart(fullModelLabel, averages);
  } catch (error) {
    console.error("Error fetching or processing JSON data:", error);
  }
}

function removeModelFromChart(selectedModel) {
  chartData.datasets = chartData.datasets.filter(ds => ds.label !== selectedModel);
  benchmarkChart.update();
}

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

function processUploadedData(jsonData, filename) {
  const sourcePrefix = `[Uploaded] ${filename.replace(".json", "")}`;

  jsonData.forEach(entry => {
    entry.sourcePrefix = sourcePrefix;
  });

  uploadedData = [...jsonData]; // Store uploaded data globally
  populateDropdown(); // Refresh dropdown with uploaded models
}

populateDropdown();


</script>
