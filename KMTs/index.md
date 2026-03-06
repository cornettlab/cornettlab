---
title: KMTs
---

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KMT Gene Activity Visualization</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #fcfcfc;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
        }

        #charts-container {
            display: flex;
            justify-content: center; /* Centered for single chart */
            width: 100%;
            max-width: 800px;
            margin-top: 20px;
        }

        .chart-wrapper {
            position: relative;
        }

        .gene-label {
            font-size: 10px;
            fill: #333;
            pointer-events: none;
        }

        /* Tooltip Style */
        #tooltip {
            position: absolute;
            text-align: center;
            padding: 8px;
            font-size: 12px;
            background: rgba(0, 0, 0, 0.85);
            color: #fff;
            border-radius: 4px;
            pointer-events: none;
            opacity: 0;
            z-index: 100;
            box-shadow: 0px 2px 5px rgba(0,0,0,0.3);
        }

        .activity-arc {
            cursor: pointer;
        }

        #legend {
            margin-top: 40px;
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            padding: 20px;
            background: white;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }

        .legend-item {
            display: flex;
            align-items: center;
            font-size: 14px;
        }

        .color-box {
            width: 18px;
            height: 18px;
            margin-right: 10px;
            border: 1px solid rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>

    <h2>Epigenetic Regulators: KMTs</h2>

    <div id="charts-container">
        <div id="kmt-chart" class="chart-wrapper"></div>
    </div>

    <div id="tooltip"></div>

    <div id="legend">
        <div class="legend-item"><div class="color-box" style="background-color: #A36AC7;"></div>Clinical</div>
        <div class="legend-item"><div class="color-box" style="background-color: #38C2E7;"></div>Preclinical</div>
        <div class="legend-item"><div class="color-box" style="background-color: #279F4D;"></div>Histone</div>
        <div class="legend-item"><div class="color-box" style="background-color: #FFD934;"></div>Non-histone</div>
        <div class="legend-item"><div class="color-box" style="background-color: #F89B2B;"></div>Cytoplasm</div>
        <div class="legend-item"><div class="color-box" style="background-color: #E63B34;"></div>Nucleus</div>
    </div>

<script>
    // Configuration
    const width = 700; // Slightly larger for a single focus chart
    const height = 700;
    const radius = Math.min(width, height) / 2;
    const innerRadius = 110;
    const ringWidth = (radius - innerRadius - 50) / 6;

    const colorScale = d3.scaleOrdinal()
        .domain([1, 2, 3, 4, 5, 6]) 
        .range(["#A36AC7", "#38C2E7", "#279F4D", "#FFD934", "#F89B2B", "#E63B34"]);

    const tooltip = d3.select("#tooltip");

    function drawChart(selector, data, centerText) {
        const svg = d3.select(selector)
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2},${height / 2})`);

        // Center circle
        svg.append("circle")
            .attr("r", innerRadius)
            .attr("fill", "#f0f0f0");

        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("dy", ".35em")
            .attr("font-size", "42px")
            .attr("font-weight", "bold")
            .attr("fill", "#222")
            .text(centerText);

        // Generators
        const arcGen = d3.arc()
            .innerRadius(d => innerRadius + (6 - d.Activity) * ringWidth)
            .outerRadius(d => innerRadius + (7 - d.Activity) * ringWidth)
            .startAngle(d => (d.Angle - 3.5) * Math.PI / 180)
            .endAngle(d => (d.Angle + 3.5) * Math.PI / 180);

        const hoverArcGen = d3.arc()
            .innerRadius(d => innerRadius + (6 - d.Activity) * ringWidth)
            .outerRadius(d => (innerRadius + (7 - d.Activity) * ringWidth) + 12)
            .startAngle(d => (d.Angle - 4.5) * Math.PI / 180)
            .endAngle(d => (d.Angle + 4.5) * Math.PI / 180);
