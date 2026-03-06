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

        h2 {
            color: #222;
            margin-bottom: 10px;
        }

        #charts-container {
            display: flex;
            justify-content: center;
            width: 100%;
            max-width: 800px;
            margin-top: 20px;
        }

        .chart-wrapper {
            position: relative;
        }

        .gene-label {
            font-size: 11px;
            font-weight: 600;
            fill: #444;
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
            transition: opacity 0.2s;
        }

        .activity-arc {
            cursor: pointer;
            stroke: #fff;
            stroke-width: 1px;
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
    // 1. Configuration Constants
    const width = 750;
    const height = 750;
    const radius = Math.min(width, height) / 2;
    const innerRadius = 110;
    const ringWidth = (radius - innerRadius - 80) / 6;

    const colorScale = d3.scaleOrdinal()
        .domain([1, 2, 3, 4, 5, 6]) 
        .range(["#A36AC7", "#38C2E7", "#279F4D", "#FFD934", "#F89B2B", "#E63B34"]);

    const tooltip = d3.select("#tooltip");

    // 2. Load Data from CSV
    d3.csv("data.csv").then(data => {
        // Clean data: ensure numeric types
        data.forEach(d => {
            d.Activity = +d.Activity;
            d.Angle = +d.Angle;
        });

        drawChart("#kmt-chart", data, "KMT");
    }).catch(error => {
        console.error("Critical Error: Could not load data.csv. Ensure you are running this through a local server.", error);
    });

    // 3. Main Drawing Function
    function drawChart(selector, data, centerText) {
        const svg = d3.select(selector)
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2},${height / 2})`);

        // Draw center circle (The "Hub")
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

        // Arc Generator Logic
        // Activity 1 is closest to center, 6 is outer
        const arcGen = d3.arc()
            .innerRadius(d => innerRadius + (d.Activity - 1) * ringWidth)
            .outerRadius(d => innerRadius + d.Activity * ringWidth)
            .startAngle(d => (d.Angle - 4) * Math.PI / 180)
            .endAngle(d => (d.Angle + 4) * Math.PI / 180)
            .cornerRadius(3);

        // Render Arcs
        svg.selectAll(".activity-arc")
            .data(data)
            .enter()
            .append("path")
            .attr("class", "activity-arc")
            .attr("d", arcGen)
            .attr("fill", d => colorScale(d.Activity))
            .on("mouseover", function(event, d) {
                d3.select(this).transition().duration(150).attr("opacity", 0.7);
                tooltip.style("opacity", 1)
                    .html(`<strong>${d.Gene}</strong><br/>Activity Level: ${d.Activity}`)
                    .style("left", (event.pageX + 15) + "px")
                    .style("top", (event.pageY - 20) + "px");
            })
            .on("mousemove", function(event) {
                tooltip.style("left", (event.pageX + 15) + "px")
                    .style("top", (event.pageY - 20) + "px");
            })
            .on("mouseout", function() {
                d3.select(this).transition().duration(150).attr("opacity", 1);
                tooltip.style("opacity", 0);
            });

        // Add Labels for each Gene
        svg.selectAll(".gene-label")
            .data(data)
            .enter()
            .append("text")
            .attr("class", "gene-label")
            .attr("transform", d => {
                const angleRad = (d.Angle - 90) * Math.PI / 180;
                // Place label slightly further out than the arc
                const r = innerRadius + (d.Activity * ringWidth) + 10;
                return `translate(${Math.cos(angleRad) * r}, ${Math.sin(angleRad) * r})`;
            })
            // Align text based on which side of the circle it's on
            .attr("text-anchor", d => (d.Angle > 180 ? "end" : "start"))
            .attr("alignment-baseline", "middle")
            .text(d => d.Gene);
    }
</script>
</body>
</html>
