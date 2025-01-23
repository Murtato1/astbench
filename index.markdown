---
layout: default
title: Home
---

# Welcome2 to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Bar Chart Example

<select id="file-selector">
  <option>Select a result</option>
</select>
<div id="chart-container">
  <canvas id="barChart"></canvas>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
    const basePath = "{{ site.baseurl }}/assets/json/";

    // Populate dropdown menu dynamically
    const dropdown = document.getElementById("file-selector");

    // If directory listing is enabled (optional: use file_list.json instead if needed)
    async function getFiles() {
        try {
            const response = await fetch(basePath);
            const text = await response.text();

            // Parse directory listing for JSON files
            const parser = new DOMParser();
            const htmlDoc = parser.parseFromString(text, "text/html");
            const links = htmlDoc.querySelectorAll("a");

            // Filter and add JSON files to dropdown
            links.forEach(link => {
                const fileName = link.getAttribute("href");
                if (fileName.endsWith(".json")) {
                    const option = document.createElement("option");
                    option.value = fileName;
                    option.textContent = fileName.replace(".json", "");
                    dropdown.appendChild(option);
                }
            });
        } catch (error) {
            console.error("Error fetching files:", error);
        }
    }

    getFiles();

    // Event listener for dropdown selection
    dropdown.addEventListener("change", function () {
        const selectedFile = dropdown.value;
        if (selectedFile !== "Select a result") {
            fetch(basePath + selectedFile)
                .then(response => response.json())
                .then(data => updateChart(data));
        }
    });

    function processJSON(json) {
        const metrics = {
            "direct_match": [],
            "fuzzy_match": [],
            "codebleu": [],
            "codebertscore": [],
            "codebertscore_rescaled": []
        };

        json.forEach(item => {
            item.result.forEach(result => {
                metrics.direct_match.push(result.direct_match ? 1 : 0);
                metrics.fuzzy_match.push(result.fuzzy_match);
                metrics.codebleu.push(result.codebleu.codebleu);
                metrics.codebertscore.push(result.codebertscore.F1);
                metrics.codebertscore_rescaled.push(result.codebertscore_rescaled.F1);
            });
        });

        return Object.fromEntries(
            Object.entries(metrics).map(([key, values]) => [
                key, values.reduce((sum, value) => sum + value, 0) / values.length
            ])
        );
    }

    function updateChart(data) {
        const ctx = document.getElementById("barChart").getContext("2d");
        const averages = processJSON(data);

        // Destroy existing chart if it exists
        if (window.currentChart) {
            window.currentChart.destroy();
        }

        // Create new chart with updated data
        window.currentChart = new Chart(ctx, {
            type: "bar",
            data: {
                labels: Object.keys(averages),
                datasets: [{
                    label: "Evaluation Metrics",
                    data: Object.values(averages),
                    backgroundColor: ["#348AC7", "#7474BF", "#56CCF2", "#2F80ED", "#BB6BD9"]
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: { display: true },
                    title: { display: true, text: "Evaluation Metrics Averages" }
                }
            }
        });
    }
</script>

