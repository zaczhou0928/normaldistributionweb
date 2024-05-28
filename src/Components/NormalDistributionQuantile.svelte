<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { jStat } from 'jstat';

  let mean = 0;
  let std = 1;
  let xValue = 0; // Variable for slider control
  let svg;
  let xAxisGroup, yAxisGroup, linePath, areaPath;
  let areaCovered = 0;
  let xScale, yScale;

  function normalDistribution(x, mean, std) {
    return (1 / (std * Math.sqrt(2 * Math.PI))) * Math.exp(-0.5 * ((x - mean) / std) ** 2);
  }

  function cdf(x, mean, std) {
    return jStat.normal.cdf(x, mean, std);
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

    // Add the area path
    areaPath = g.append("path")
                .attr("fill", "lightblue");
  }

  function updateGraph() {
    if (!xScale || !yScale) return; // Ensure scales are defined

    if (std < 0.4) {
      std = 0.4
    }

    if (mean < -6) {
      mean = -6
    }

    if (mean > 6) {
      mean = 6
    }

    const xDomain = [-10, 10];
    xScale.domain(xDomain);
    const data = d3.range(xDomain[0], xDomain[1], (xDomain[1] - xDomain[0]) / 100).map(x => ({
      x,
      y: normalDistribution(x, mean, std)
    }));
    yScale.domain([0, 1]);

    const line = d3.line()
                   .x(d => xScale(d.x))
                   .y(d => yScale(d.y));

    linePath.datum(data).attr("d", line);
    xAxisGroup.call(d3.axisBottom(xScale));
    yAxisGroup.call(d3.axisLeft(yScale).ticks(5));

    // Define the area up to the selected x-value
    const area = d3.area()
                   .x(d => xScale(d.x))
                   .y1(d => yScale(d.y))
                   .y0(yScale(0));

    areaPath.datum(data.filter(d => d.x <= xValue)).attr("d", area);
    areaCovered = cdf(xValue, mean, std);
  }

  onMount(() => {
    setupChart();
    updateGraph(); // Call updateGraph here to ensure initial drawing
  });

  // Reactively update the graph whenever mean, std, or xValue changes
  $: if (mean || std || xValue) updateGraph();
</script>

<div>
  <label for="mean">Mean:</label>
  <input type="range" id="mean" bind:value={mean} min="-10" max="10" step="0.1">

  <label for="std">Standard Deviation:</label>
  <input type="range" id="std" bind:value={std} min="0.1" max="5" step="0.1">

  <label for="xValue">X-Value:</label>
  <input type="range" id="xValue" bind:value={xValue} min="-10" max="10" step="0.1">

  <p>Mean: {mean}</p>
  <p>Standard Deviation: {std}</p>
  <p>X-Value: {xValue}</p>
  <p>Area Covered: {(areaCovered * 100).toFixed(2)}%</p>
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
