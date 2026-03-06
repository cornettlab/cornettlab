---
title: Contact
nav:
  order: 5
  tooltip: Email, address, and location
---

# {% include icon.html icon="fa-regular fa-envelope" %}Contact

The Cornett Lab is housed within the Indiana University School of Medicine Department of Biochemistry and Molecular Biology. The lab is located within the Van Nuys Medical Science building on the Indiana University - Indianapolis campus.<br>

**Shipping Address:**<br>
ATTN: Cornett Lab<br>
Building VanNuys Med Sci Bldg<br>
Room# MS4075<br>
635 BARNHILL DR<br>
INDIANAPOLIS, IN 46202-5120<br>
United States<br>

{%
  include button.html
  type="email"
  tooltip="Email Evan"
  text="evcorn@iu.edu"
  link="evcorn@iu.edu"
%}
{%
  include button.html
  type="phone"
  tooltip="Evan's Office Phone"
  text="(317) 278-4503"
  link="+1-317-278-4503"
%}
{%
  include button.html
  type="address"
  text="Map"
  tooltip="Our location on Google Maps for easy navigation"
  link="https://www.google.com/maps?sll=39.776612,-86.178275&q=635+Barnhill+Drive+Indianapolis,+IN,+46202,+United+States&z=16"
%}

{% include section.html %}

{% capture col1 %}

{%
  include figure.html
  image="images/indianapolis.jpg"
  
%}

{% endcapture %}

{% capture col2 %}

{%
  include figure.html
  image="images/DNA.jpg"
  
%}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 %}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Gene Activity Sunburst Charts</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body {
            font-family: sans-serif;
            background-color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        #charts-container {
            display: flex;
            justify-content: space-around;
            width: 100%;
            margin-top: 50px;
        }

        .chart-wrapper {
            position: relative;
        }

        .gene-label {
            font-size: 10px;
            fill: black;
        }

        .arc-border {
            fill: none;
            stroke: #ccc;
            stroke-width: 0.5px;
        }

        #legend {
            margin-top: 30px;
            padding: 10px;
            border: 1px solid #ccc;
            background: #eee;
            border-radius: 5px;
        }

        .legend-item {
            display: flex;
            align-items: center;
            margin-bottom: 5px;
        }

        .color-box {
            width: 20px;
            height: 20px;
            margin-right: 10px;
            border: 1px solid #999;
        }
    </style>
</head>
<body>

<div id="charts-container">
    <div id="kmt-chart" class="chart-wrapper"></div>
    <div id="kdm-chart" class="chart-wrapper"></div>
</div>

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
    const ringWidth = (radius - innerRadius) / 6;

    // Color mapping based on numeric activity code in CSV
    const colorScale = d3.scaleOrdinal()
        .domain([1, 2, 3, 4, 5, 6]) // Mapping values 1-6 in CSV
        .range(["#A36AC7", "#38C2E7", "#279F4D", "#FFD934", "#F89B2B", "#E63B34"]);

    // Map angle sequentially for labels based on order
    let currentKMTAngle = 0;
    let currentKDMAngle = 0;

    // Function to draw one chart
    function drawChart(selector, data, centerText) {
        const svg = d3.select(selector)
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width / 2},${height / 2})`);

        // Center circle and text
        svg.append("circle")
            .attr("r", innerRadius)
            .attr("fill", "#ddd");

        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("dy", ".35em")
            .attr("font-size", "40px")
            .attr("font-weight", "bold")
            .text(centerText);

        // Define the arcs
        const arc = d3.arc()
            .innerRadius(d => innerRadius + (6 - d.Activity) * ringWidth)
            .outerRadius(d => innerRadius + (7 - d.Activity) * ringWidth)
            .startAngle(d => (d.Angle - 4) * Math.PI / 180) // 8-degree blocks
            .endAngle(d => (d.Angle + 4) * Math.PI / 180);

        // Draw colored boxes
        svg.selectAll(".activity-arc")
            .data(data)
            .enter()
            .append("path")
            .attr("class", "activity-arc")
            .attr("d", arc)
            .attr("fill", d => colorScale(d.Activity))
            .attr("stroke", "#ccc")
            .attr("stroke-width", 0.5);

        // Add gene labels
        // Group data by gene to get unique gene names and their positions
        const genes = Array.from(d3.group(data, d => d.GeneLabel)).map(([key, value]) => ({
            label: key,
            angle: value[0].Angle
        }));

        svg.selectAll(".gene-label")
            .data(genes)
            .enter()
            .append("text")
            .attr("class", "gene-label")
            .attr("text-anchor", "middle")
            .attr("transform", d => {
                const angle = d.angle;
                const r = radius + 15;
                const x = r * Math.sin(angle * Math.PI / 180);
                const y = -r * Math.cos(angle * Math.PI / 180);
                let rotation = angle - 90;
                if (angle > 180 && angle < 360) {
                    rotation = angle + 90;
                }
                return `translate(${x}, ${y}) rotate(${rotation})`;
            })
            .text(d => d.label);
    }

    // Load the CSV data and draw charts
    d3.csv("data.csv").then(data => {
        // Parse activity to integer
        data.forEach(d => {
            d.Activity = +d.Activity;
            d.Angle = +d.Angle; // Ensure angles are read as numbers
        });

        const kmtData = data.filter(d => d.Circle === "KMTs");
        const kdmData = data.filter(d => d.Circle === "KDMs");

        drawChart("#kmt-chart", kmtData, "KMTs");
        drawChart("#kdm-chart", kdmData, "KDMs");
    });

</script>

</body>
</html>

{% include section.html dark=true %}

{% capture col1 %}

{% endcapture %}

{% capture col2 %}

{% endcapture %}

{% capture col3 %}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
