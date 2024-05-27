<script>
  import { onMount } from 'svelte';
  import { scaleBand, scaleLinear } from 'd3-scale';
  import { select } from 'd3-selection';
  import { csv } from 'd3-fetch';
  import { axisBottom, axisLeft } from 'd3-axis';
  import * as d3 from 'd3';


  let width = 500;
  let height = 500;

  const margin = { top: 10, right: 30, bottom: 90, left: 40 };
  const innerWidth = width - margin.left - margin.right;
  const innerHeight = height - margin.top - margin.bottom;

  let selectedCountry = 'USA'; 

  onMount(() => {
    const svg = select('#my_dataviz')
      .append('svg')
      .attr('width', width)
      .attr('height', height)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    csv("/Users/suhanisharma/Desktop/personl-106-final/data/drinks.csv").then(data => {
      data.forEach(d => {
        d.beer_servings = +d.beer_servings;
        d.spirit_servings = +d.spirit_servings;
        d.wine_servings = +d.wine_servings;
        d.total_litres_of_pure_alcohol = +d.total_litres_of_pure_alcohol;
      });

      console.log(data);

      const x = scaleBand()
        .range([0, innerWidth])
        .domain(data.map(d => d.country))
        .padding(0.2);

      svg.append('g')
        .attr('transform', `translate(0,${innerHeight})`)
        .call(axisBottom(x))
        .selectAll('text')
        .attr('transform', 'translate(-10,0)rotate(-45)')
        .style('text-anchor', 'end');

      const y = scaleLinear()
        .domain([0, d3.max(data, d => d.total_litres_of_pure_alcohol)])
        .nice()
        .range([innerHeight, 0]);

      svg.append('g')
        .call(axisLeft(y));

      svg.selectAll('.bar')
        .data(data)
        .enter()
        .append('rect')
        .attr('class', 'bar')
        .attr('x', d => x(d.country))
        .attr('width', x.bandwidth())
        .attr('fill', '#69b3a2')
        .attr('y', d => y(0))
        .attr('height', d => innerHeight - y(0));

      svg.selectAll('rect')
        .transition()
        .duration(800)
        .attr('y', d => y(d.total_litres_of_pure_alcohol))
        .attr('height', d => innerHeight - y(d.total_litres_of_pure_alcohol))
        .delay((d, i) => i * 100);
    }).catch(error => {
      console.error('Error loading the CSV file:', error);
    });
  });
</script>

<style>
  .bar:hover {
    fill: orange;
  }

  .axis text {
    font-size: 12px;
  }

  .axis path,
  .axis line {
    fill: none;
    stroke: black;
    shape-rendering: crispEdges;
  }
</style>

<h1 class="body-header">Alcohol Type Consumption by Country</h1>

<p class="body-text">
  With an understanding of how alcohol consumption began in ancient times, let us explore how it looks in modern day.
</p>

<p class="body-text">
  <strong>Now it's your turn to explore!</strong>
</p>
  
<p class="body-text">
  <strong>Choose a country from the dropdown/type it in the box**</strong> 
  to generate a bar chart describing different consumptions rates of different 
  alcohol types of your chosen country.
</p>

<p class="body-text">
  The original data set can be found from this 
  <a href="https://github.com/fivethirtyeight/data/tree/master/alcohol-consumption">fivethirtyeight</a> link. 
</p>

<p class="body-text">
  <strong>**</strong>For this prototype our bar chart only shows information for one hard-coded
  country and we hope to improve the interaction of this in the final model.
</p>

<div id="my_dataviz"></div>
