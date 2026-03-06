---
title: KMTs
---

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gene Activity Sunburst with Tooltips</title>
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
            justify-content: space-evenly;
            width: 100%;
            max-width: 1400px;
            margin-top: 20px;
        }

        .chart-wrapper {
            position: relative;
        }

        .gene-label {
            font-size: 10px;
            fill: #333;
            pointer-events: none; /* Allows mouse to hit wedges underneath labels */
        }

        /* Tooltip Style */
        #tooltip {
            position: absolute;
            text-align: center;
            padding: 8px;
            font-size: 12px;
            background: rgba(0, 0, 0, 0.8);
            color: #fff;
            border-radius: 4px;
            pointer-events: none; /* Prevents tooltip from flickering under mouse */
            opacity: 0;
            z-index: 100;
            box-shadow: 0px 2px 5px rgba(0,0,0,0.2);
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

    <h2>Epigenetic Regulators: KMTs & KDMs</h2>

    <div id="charts-container">
        <div id="kmt-chart" class="chart-wrapper"></div>
        <div id="kdm-chart" class="chart-wrapper"></div>
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
    const width = 600;
    const height = 600;
    const radius = Math.min(width, height) / 2;
    const innerRadius = 100;
    const ringWidth = (radius - innerRadius - 40) / 6;

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
            .attr("fill", "#eeeeee");

        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("dy", ".35em")
            .attr("font-size", "36px")
            .attr("font-weight", "bold")
            .attr("fill", "#444")
            .text(centerText);

        // Generators
        const arcGen = d3.arc()
            .innerRadius(d => innerRadius + (6 - d.Activity) * ringWidth)
            .outerRadius(d => innerRadius + (7 - d.Activity) * ringWidth)
            .startAngle(d => (d.Angle - 3.5) * Math.PI / 180)
            .endAngle(d => (d.Angle + 3.5) * Math.PI / 180);

        const hoverArcGen = d3.arc()
            .innerRadius(d => innerRadius + (6 - d.Activity) * ringWidth)
            .outerRadius(d => (innerRadius + (7 - d.Activity) * ringWidth) + 10)
            .startAngle(d => (d.Angle - 4.5) * Math.PI / 180)
            .endAngle(d => (d.Angle + 4.5) * Math.PI / 180);

        // Draw Arcs
        svg.selectAll(".activity-arc")
            .data(data)
            .enter()
            .append("path")
            .attr("class", "activity-arc")
            .attr("d", arcGen)
            .attr("fill", d => colorScale(d.Activity))
            .attr("stroke", "#fff")
            .attr("stroke-width", 1)
            .on("mouseover", function(event, d) {
                d3.select(this)
                    .transition().duration(200)
                    .attr("d", hoverArcGen)
                    .attr("stroke", "#333");

                tooltip.transition().duration(100).style("opacity", 1);
                tooltip.html(`<strong>Gene:</strong> ${d.GeneLabel}<br/><strong>Ring:</strong> ${d.Ring}`)
                    .style("left", (event.pageX + 15) + "px")
                    .style("top", (event.pageY - 28) + "px");
            })
            .on("mousemove", function(event) {
                tooltip.style("left", (event.pageX + 15) + "px")
                       .style("top", (event.pageY - 28) + "px");
            })
            .on("mouseout", function() {
                d3.select(this)
                    .transition().duration(200)
                    .attr("d", arcGen)
                    .attr("stroke", "#fff");

                tooltip.transition().duration(100).style("opacity", 0);
            });

        // Add Gene Labels
        const genes = Array.from(d3.group(data, d => d.GeneLabel)).map(([key, value]) => ({
            label: key,
            angle: value[0].Angle
        }));

        svg.selectAll(".gene-label")
            .data(genes)
            .enter()
            .append("text")
            .attr("class", "gene-label")
            .attr("text-anchor", d => (d.angle > 180 && d.angle < 360) ? "end" : "start")
            .attr("transform", d => {
                const angle = d.angle;
                const r = radius - 30;
                const x = r * Math.sin(angle * Math.PI / 180);
                const y = -r * Math.cos(angle * Math.PI / 180);
                let rotation = angle - 90;
                if (angle > 180 && angle < 360) rotation = angle + 90;
                return `translate(${x}, ${y}) rotate(${rotation})`;
            })
            .text(d => d.label);
    }

    // Load and Render
    d3.csv("data.csv").then(data => {
        data.forEach(d => {
            d.Activity = +d.Activity;
            d.Angle = +d.Angle;
        });

        drawChart("#kmt-chart", data.filter(d => d.Circle === "KMTs"), "KMTs");
        drawChart("#kdm-chart", data.filter(d => d.Circle === "KDMs"), "KDMs");
    }).catch(err => {
        console.error("Error loading CSV: ", err);
    });

</script>
</body>
</html>
