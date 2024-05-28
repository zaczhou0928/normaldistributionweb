<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { heightData } from "../heightdata.js";

  let selectedSampleSize = "10";
  let sampledData = heightData.slice(0, +selectedSampleSize);

  function updateSampleSize(event) {
    selectedSampleSize = event.target.value;
    if (selectedSampleSize === "all") {
      sampledData = heightData;
    } else {
      sampledData = heightData.slice(0, +selectedSampleSize);
    }
    setupChart();
  }

  function normalDistribution(x, mean, sigma) {
    const sqrt2pi = Math.sqrt(2 * Math.PI);
    return (1 / (sigma * sqrt2pi)) * Math.exp(-0.5 * Math.pow((x - mean) / sigma, 2));
  }

  function setupChart() {
    const margin = { top: 20, right: 40, bottom: 40, left: 50 };
    const width = 600 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    const svg = d3.select('#chart-container').select("svg");
    if (svg) {
      svg.remove();
    }

    const container = d3.select('#chart-container')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    const heights = sampledData.map(d => d["Height(Inches)"]);
    const mean = d3.mean(heights);
    const stdDev = d3.deviation(heights);

    const bins = d3.histogram()
      .domain(d3.extent(heights))
      .thresholds(d3.range(60, 85, 0.5))
      (heights);

    const xScale = d3.scaleLinear()
      .domain([55, 90])
      .range([0, width]);

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(bins, d => d.length)])
      .range([height, 0]);

    container.selectAll('rect')
      .data(bins)
      .enter().append('rect')
      .attr('x', d => xScale(d.x0))
      .attr('y', d => yScale(d.length))
      .attr('width', d => xScale(d.x1) - xScale(d.x0) - 1)
      .attr('height', d => height - yScale(d.length))
      .attr('fill', 'steelblue');

    const pdfScaleFactor = d3.max(bins, d => d.length) / normalDistribution(mean, mean, stdDev);

    const line = d3.line()
      .x(d => xScale(d))
      .y(d => yScale(normalDistribution(d, mean, stdDev) * pdfScaleFactor))
      .curve(d3.curveBasis);

    const points = d3.range(55, 90, 0.1);
    container.append("path")
      .datum(points)
      .attr("fill", "none")
      .attr("stroke", "red")
      .attr("d", line);

    container.append('g')
      .attr('transform', `translate(0,${height})`)
      .call(d3.axisBottom(xScale));

    container.append('g')
      .call(d3.axisLeft(yScale));

    // Adjusted position for the x-axis label
    container.append('text')
      .attr('x', width / 2)
      .attr('y', height + margin.bottom - 3)
      .attr('text-anchor', 'middle')
      .text('Height (inches)');

    // Adjusted position for the y-axis label
    container.append('text')
      .attr('transform', 'rotate(-90)')
      .attr('x', -height / 2)
      .attr('y', -margin.left + 12)
      .attr('text-anchor', 'middle')
      .text('Frequency');
  }

  onMount(() => {
    setupChart();
  });
</script>

<h2 class="body-header">Explore the Effect of Sample Size</h2>

<p class="body-text">
  To understand how the normal distribution works in real life, 
  let's look at a dataset of high school student heights. 
  Begin with a smaller sample size and gradually increase it. 
  Notice how the distribution of heights begins to resemble the 
  classic 'bell curve' of the normal distribution as the sample size grows. 
  This visualization helps illustrate the Central Limit Theorem in action: 
  larger samples tend to be normally distributed, regardless of the shape of the 
  population distribution.
</p>
 
<div id="chart-container"></div>

<div style="text-align: center; margin-top: 20px;">
  <select bind:value={selectedSampleSize} on:change={updateSampleSize}>
    <option value="10">Sample Size: 10</option>
    <option value="100">Sample Size: 100</option>
    <option value="1000">Sample Size: 1000</option>
    <option value="all">All Data</option>
  </select>
</div>

<style>
  #chart-container {
    width: 600px;
    height: 400px;
    margin-left: auto;
    margin-right: auto;
  }
</style>
