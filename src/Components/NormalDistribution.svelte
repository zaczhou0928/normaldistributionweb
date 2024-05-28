<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let mean = 0;
  let std = 1;
  let svg;
  let xAxisGroup, yAxisGroup, linePath;
  let xScale, yScale;

  function normalDistribution(x, mean, std) {
    return (1 / (std * Math.sqrt(2 * Math.PI))) * Math.exp(-0.5 * ((x - mean) / std) ** 2);
  }

  function setupChart() {
    const margin = { top: 20, right: 20, bottom: 40, left: 50 },
          width = 650 - margin.left - margin.right,
          height = 450 - margin.top - margin.bottom;

    // Set up scales
    xScale = d3.scaleLinear().range([0, width]);
    yScale = d3.scaleLinear().range([height, 0]);

    // Append the svg object to the body of the page
    const g = d3.select(svg)
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

    // Create axes groups
    xAxisGroup = g.append("g")
                  .attr("transform", `translate(0,${height})`);
    yAxisGroup = g.append("g");

    // Add the line path
    linePath = g.append("path")
                .attr("fill", "none")
                .attr("stroke", "steelblue")
                .attr("stroke-width", 2);
  }

  function updateGraph() {
    if (!xScale || !yScale) return; // Ensure scales are defined

    // Additional control for std and mean values
    if (std < 0.4) std = 0.4;
    if (mean < -6) mean = -6;
    if (mean > 6) mean = 6;

    // Update scales and data
    const xDomain = [-10, 10];
    xScale.domain(xDomain);
    yScale.domain([0, 1]);
    const data = d3.range(xDomain[0], xDomain[1], (xDomain[1] - xDomain[0]) / 100)
                  .map(x => ({ x, y: normalDistribution(x, mean, std) }));

    const line = d3.line().x(d => xScale(d.x)).y(d => yScale(d.y));
    linePath.datum(data).attr("d", line);
    xAxisGroup.call(d3.axisBottom(xScale));
    yAxisGroup.call(d3.axisLeft(yScale).ticks(5));
  }

  onMount(() => {
    setupChart();
    updateGraph();
  });

  // Reactively update the graph whenever mean or std changes
  $: if (mean || std) updateGraph();
</script>

<!-- Centralized Input controls -->
<div style="text-align: center;">
  <label for="mean">Mean:</label>
  <input type="number" id="mean" bind:value={mean} step="0.1" aria-label="Mean">

  <label for="std">Standard Deviation:</label>
  <input type="number" id="std" bind:value={std} min="0.1" step="0.1" aria-label="Standard Deviation">
</div>

<!-- SVG container for D3 graph -->
<svg bind:this={svg}></svg>

<style>
  input {
    margin: 0.5rem;
    padding: 0.2rem;
    width: 80px;
  }
  svg {
    width: 100%;
    height: 850px;
    overflow: visible;
    display: block;
    margin: auto;
  }
</style>
