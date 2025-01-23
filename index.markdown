---
layout: default
title: Home
---

# Welcome to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Bar Chart Example

<canvas id="benchmarkChart" width="400" height="200"></canvas>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const jsonUrl = 'https://raw.githubusercontent.com/Murtato1/astbench/main/assets/json/benchmark_1.json';

  async function fetchAndProcessData() {
    try {
      const response = await fetch(jsonUrl);
      const data = await response.json();

      const metrics = {
        direct_match: [],
        fuzzy_match: [],
        codebleu: [],
        codebertscore: [],
        codebertscore_rescaled: []
      };

      data.forEach(item => {
        item.result.forEach(result => {
          metrics.direct_match.push(result.direct_match ? 1 : 0);
          metrics.fuzzy_match.push(result.fuzzy_match);
          metrics.codebleu.push(result.codebleu.codebleu);
          metrics.codebertscore.push(result.codebertscore.F1);
          metrics.codebertscore_rescaled.push(result.codebertscore_rescaled.F1);
        });
      });

      const averages = {};
      for (const [key, values] of Object.entries(metrics)) {
        averages[key] = values.reduce((sum, val) => sum + val, 0) / values.length;
      }

      renderChart(averages);
    } catch (error) {
      console.error('Error fetching or processing data:', error);
    }
  }

  function renderChart(averages) {
    const ctx = document.getElementById('benchmarkChart').getContext('2d');
    new Chart(ctx, {
      type: 'bar',
      data: {
        labels: Object.keys(averages),
        datasets: [{
          label: 'Evaluation Metrics Averages',
          data: Object.values(averages),
          backgroundColor: [
            'rgba(75, 192, 192, 0.2)',
            'rgba(54, 162, 235
::contentReference[oaicite:0]{index=0}
