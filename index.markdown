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
                    label: 'Stellar Observations',
                    data: [30, 50, 70, 90],
                    backgroundColor: [
                        'rgba(128, 0, 128, 0.7)', // Purple
                        'rgba(0, 191, 255, 0.7)', // Blue
                        'rgba(255, 99, 71, 0.7)',  // Red
                        'rgba(50, 205, 50, 0.7)'  // Green
                    ],
                    borderColor: [
                        'rgba(128, 0, 128, 1)',
                        'rgba(0, 191, 255, 1)',
                        'rgba(255, 99, 71, 1)',
                        'rgba(50, 205, 50, 1)'
                    ],
                    borderWidth: 1
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: {
                        position: 'top',
                        labels: {
                            color: '#000000', // BLACK legend text
                            font: {
                                family: 'Orbitron', // Stylish font
                                size: 14
                            }
                        }
                    }
                },
                scales: {
                    x: {
                        ticks: {
                            color: '#ffffff', // White x-axis labels
                            font: {
                                family: 'Open Sans',
                                size: 12
                            }
                        },
                        grid: {
                            color: 'rgba(255, 255, 255, 0.2)' // Light gridlines
                        }
                    },
                    y: {
                        ticks: {
                            color: '#ffffff', // White y-axis labels
                            font: {
                                family: 'Open Sans',
                                size: 12
                            }
                        },
                        grid: {
                            color: 'rgba(255, 255, 255, 0.2)' // Light gridlines
                        },
                        beginAtZero: true
                    }
                }
            }
        });
    });
</script>

