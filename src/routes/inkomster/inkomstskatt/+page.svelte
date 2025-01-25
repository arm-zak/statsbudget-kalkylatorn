<script>
    import { onMount } from 'svelte';
    import * as XLSX from 'xlsx';
    import { Chart, registerables } from 'chart.js';
    import dragDataPlugin from 'chartjs-plugin-dragdata';
    
    Chart.register(...registerables);
    Chart.register(dragDataPlugin);

    let chart;
    let chartData = {
        datasets: [{
        label: 'Sample Data',
        data: [],
        backgroundColor: 'rgba(75, 192, 192, 0.6)',
        borderColor: 'rgba(75, 192, 192, 1)',
        borderWidth: 1,
        pointRadius: 5,
        pointHoverRadius: 7,
        showLine: true
        }]
    };
    
    async function loadExcelData() {
        const response = await fetch('/Lonefordelning_2023.xlsx');
        const arrayBuffer = await response.arrayBuffer();
        const workbook = XLSX.read(arrayBuffer, { type: 'array' });
        const worksheet = workbook.Sheets[workbook.SheetNames[0]];
        const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

        // Assuming the first row contains headers and the subsequent rows contain data
        jsonData.slice(1).forEach(row => {
        const [x, y] = row;
        chartData.datasets[0].data.push({ x, y });
        });

        // Update the chart with the new data
        if (chart) {
        chart.update();
        }
    }

    onMount(() => {
        const ctx = document.getElementById('myChart').getContext('2d');
        chart = new Chart(ctx, {
            type: 'scatter',
            data: chartData,
            options: {
                responsive: true,
                scales: {
                x: {
                    type: 'linear',
                    position: 'bottom'
                },
                y: {
                    type: 'linear'
                }
                },
                plugins: {
                dragData: {
                    round: 1,
                    showTooltip: true,
                    onDragStart: function (e, datasetIndex, index, value) {
                    console.log(`Start dragging point ${index} in dataset ${datasetIndex}`);
                    },
                    onDrag: function (e, datasetIndex, index, value) {
                    console.log(`Dragging point ${index} in dataset ${datasetIndex} to ${value}`);
                    },
                    onDragEnd: function (e, datasetIndex, index, value) {
                    console.log(`End dragging point ${index} in dataset ${datasetIndex} to ${value}`);
                    }
                }
                }
            }
        });

        // Load the Excel data after the chart is created
        loadExcelData();
    });
</script>
  
<h1>Statens inkomster</h1>
<div class="container">
    <canvas id="myChart" width="100" height="100"></canvas>
</div>

<style>
canvas {
    max-width: 100%;
    height: auto;
}
</style>