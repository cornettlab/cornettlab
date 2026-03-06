---
title: KMTs
---
<script>
    // Configuration
    const width = 700;
    const height = 700;
    const radius = Math.min(width, height) / 2;
    const innerRadius = 110;
    const ringWidth = (radius - innerRadius - 100) / 6; // Adjusted for label padding

    const colorScale = d3.scaleOrdinal()
        .domain([1, 2, 3, 4, 5, 6]) 
        .range(["#A36AC7", "#38C2E7", "#279F4D", "#FFD934", "#F89B2B", "#E63B34"]);

    const tooltip = d3.select("#tooltip");

    // Load the CSV file
    d3.csv("data.csv").then(data => {
        // Convert strings to numbers
        data.forEach(d => {
            d.Activity = +d.Activity;
            d.Angle = +d.Angle;
        });

        drawChart("#kmt-chart", data, "KMT");
    }).catch(error => {
        console.error("Error loading the CSV file:", error);
    });

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

        // Arc Generator
        const arcGen = d3.arc()
            .innerRadius(d => innerRadius + (d.Activity - 1) * ringWidth)
            .outerRadius(d => innerRadius + d.Activity * ringWidth)
            .startAngle(d => (d.Angle - 4) * Math.PI / 180)
            .endAngle(d => (d.Angle + 4) * Math.PI / 180)
            .cornerRadius(4);

        // Draw Arcs
        svg.selectAll(".activity-arc")
            .data(data)
            .enter()
            .append("path")
            .attr("class", "activity-arc")
            .attr("d", arcGen)
            .attr("fill", d => colorScale(d.Activity))
            .on("mouseover", function(event, d) {
                d3.select(this).transition().duration(200).attr("opacity", 0.7);
                tooltip.transition().duration(200).style("opacity", 1);
                tooltip.html(`<strong>${d.Gene}</strong><br/>Activity Level: ${d.Activity}`)
                    .style("left", (event.pageX + 15) + "px")
                    .style("top", (event.pageY - 28) + "px");
            })
            .on("mouseout", function() {
                d3.select(this).transition().duration(200).attr("opacity", 1);
                tooltip.transition().duration(200).style("opacity", 0);
            });

        // Add Gene Labels
        svg.selectAll(".gene-label")
            .data(data)
            .enter()
            .append("text")
            .attr("class", "gene-label")
            .attr("transform", d => {
                const angleRad = (d.Angle - 90) * Math.PI / 180;
                const r = innerRadius + (d.Activity * ringWidth) + 12;
                return `translate(${Math.cos(angleRad) * r}, ${Math.sin(angleRad) * r})`;
            })
            .attr("text-anchor", d => (d.Angle > 180 ? "end" : "start"))
            .attr("alignment-baseline", "middle")
            .text(d => d.Gene);
    }
</script>
</body>
</html>
