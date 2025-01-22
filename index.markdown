---
layout: default
title: Home
---

# Welcome to AstroCopilot Benchmark

Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Bar Chart Example

<canvas id="myChart" width="400" height="200"></canvas>
<script>
    document.addEventListener("DOMContentLoaded", () => {
        const ctx = document.getElementById('myChart').getContext('2d');
        const myChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['Nebula A', 'Nebula B', 'Galaxy X', 'Galaxy Y'],
                datasets: [{
                    label: 'Stellar Data',
                    data: [50, 100, 150, 200],
                    backgroundColor: [
                        'rgba(128, 0, 128, 0.6)',  // Purple for Nebula A
                        'rgba(0, 191, 255, 0.6)', // Blue for Nebula B
                        'rgba(255, 69, 0, 0.6)',  // Red for Galaxy X
                        'rgba(34, 139, 34, 0.6)'  // Green for Galaxy Y
                    ],
                    borderColor: [
                        'rgba(128, 0, 128, 1)',
                        'rgba(0, 191, 255, 1)',
                        'rgba(255, 69, 0, 1)',
                        'rgba(34, 139, 34, 1)'
                    ],
                    borderWidth: 2
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                        labels: {
                            color: 'white' // White legend text
                        }
                    }
                },
                scales: {
                    x: {
                        ticks: {
                            color: 'white' // White axis labels
                        }
                    },
                    y: {
                        ticks: {
                            color: 'white' // White axis labels
                        },
                        beginAtZero: true
                    }
                }
            }
        });
    });
</script>
