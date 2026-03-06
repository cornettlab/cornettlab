---
title: KMTs
---

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

    <h2>Lysine Methyltransferases (KMTs)</h2>

    <div id="charts-container">
        <div id="kmt-chart" class="chart-wrapper"></div>
    </div>

    <div id="tooltip"></div>

    <div id="legend">
        <div class="legend-item"><div class="color-box" style="background-color: #A36AC7;"></div>Clinical</div>
        <div class="legend-item"><div class="color-box" style="background-color: #3
