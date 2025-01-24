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
<select id="file-selector">
  <option>Select a result</option>
</select>

<canvas id="benchmarkChart" width="800" height="400"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const basePath = "{{ site.baseurl }}/assets/json/"; // Path to your JSON directory
  const dropdown = document.getElementById("file-selector");

  // Populate dropdown menu with JSON file options
  async function populateDropdown() {
    try {
      const files = ["gpt-4o.json"]; // Add filenames here

      files.forEach((file) => {
        const option = document.createElement("option");
        option.value = file;
        option.textContent = file.replace(".json", ""); // Remove ".json" for display
        dropdown.appendChild(option);
      });
    } catch (error) {
      console.error("Error populating dropdown:", error);
    }
  }

  // Event listener for dropdown selection
  dropdown.addEventListener("change", function () {
    const selectedFile = dropdown.value;
    if (selectedFile !== "Select a result") {
      const jsonPath = basePath + selectedFile;
      fetchAndProcessData(jsonPath);
    }
  });

  // Fetch and process data from the selected JSON file
  async function fetchAndProcessData(jsonPath) {
    try {
      const response = await fetch(jsonPath);
      const data = await response.json();

      const metrics = {
        direct_match: [],
        fuzzy_match: [],
        codebleu: [],
        codebertscore: [],
        codebertscore_rescaled: [],
        code_success: [],
        syntax_match_score: []
      };

      // Traverse every item in the JSON
      data.forEach((item) => {
        if (item.result) {
          item.result.forEach((result) => {
            if ("direct_match" in result && result.direct_match !== null) {
              metrics.direct_match.push(result.direct_match ? 1 : 0);
            }
            if ("fuzzy_match" in result && result.fuzzy_match !== null) {
              metrics.fuzzy_match.push(result.fuzzy_match / 100); // Scale to 0-1
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

        // Extract values from `result_summary`
        if (item.result_summary) {
          if (item.result_summary.code_success !== null) {
            metrics.code_success.push(item.result_summary.code_success);
          }
          if (item.result_summary.syntax_match_score !== null) {
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
            text: "Benchmark Evaluation Metrics"
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
    <h3 style="margin: 10px 0;">Alex Johnson</h3>
    <p style="color: gray; margin-bottom: 5px;">UT Austin, Astro Labs</p>
    <button onclick="toggleDetails('member1')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member1" style="display: none; color: darkgray; margin-top: 10px;">Alex is a researcher focusing on space data processing and AI applications.</p>
  </div>

  <div class="team-member" style="text-align: center; max-width: 200px;">
    <img src="assets/pics/member2.png" alt="Member 2" style="width: 100px; height: 100px; border-radius: 50%;">
    <h3 style="margin: 10px 0;">Taylor Green</h3>
    <p style="color: gray; margin-bottom: 5px;">Bespoke Labs</p>
    <button onclick="toggleDetails('member2')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member2" style="display: none; color: darkgray; margin-top: 10px;">Taylor specializes in developing AI-driven tools for astrophysics research.</p>
  </div>

  <div class="team-member" style="text-align: center; max-width: 200px;">
    <img src="assets/pics/member3.png" alt="Member 3" style="width: 100px; height: 100px; border-radius: 50%;">
    <h3 style="margin: 10px 0;">Jordan Lee</h3>
    <p style="color: gray; margin-bottom: 5px;">UT Austin</p>
    <button onclick="toggleDetails('member3')" style="background: none; border: none; color: blue; cursor: pointer;">[expand]</button>
    <p id="details-member3" style="display: none; color: darkgray; margin-top: 10px;">Jordan is working on large-scale simulations of galactic formations.</p>
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
