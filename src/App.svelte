<script>
  import * as d3 from "d3";
  import * as topojson from "topojson-client";
  import { onMount } from "svelte";

  let width = 960;
  let height = 600;
  let svg;

  // Load polling data from CSV
  let pollingData = {};

  // State selected for detailed view
  let selectedState = null;
  let popupPosition = { x: 0, y: 0 };

  // Electoral college totals
  let electoralVotes = {
    democrat: 0,
    republican: 0,
    tossUp: 0
  };

  // Adjustable toss-up margin (slider value)
  let tossUpMargin = 4;

  // Function to determine color based on polling data and calculate electoral votes
  function getFillColor(stateId) {
    const data = pollingData[stateId];
    if (!data) return "#D3D3D3"; // Default color for no data (neutral tan)

    const { democrat, republican } = data;
    const diff = democrat - republican;

    if (Math.abs(diff) <= tossUpMargin) {
      electoralVotes.tossUp += data.electoral_votes;
      return "#D3D3D3"; // Toss-up
    } else if (diff > 0) { // Democrat leading
      electoralVotes.democrat += data.electoral_votes;
      if (diff <= tossUpMargin + 4) return "#99c4f0"; // Leans Democrat
      else if (diff <= tossUpMargin + 8) return "#446ea5"; // Likely Democrat
      else return "#2a4b83"; // Safe Democrat
    } else { // Republican leading
      electoralVotes.republican += data.electoral_votes;
      if (Math.abs(diff) <= tossUpMargin + 4) return "#f19d9d"; // Leans Republican
      else if (Math.abs(diff) <= tossUpMargin + 8) return "#c94f4f"; // Likely Republican
      else return "#a62929"; // Safe Republican
    }
  }

  // Function to determine state lean based on polling data
  function getStateLean(democrat, republican) {
    const diff = democrat - republican;
    if (Math.abs(diff) <= tossUpMargin) return "Toss-Up";
    else if (diff > 0) { // Democrat leading
      if (diff <= tossUpMargin + 4) return "Lean Democrat";
      else if (diff <= tossUpMargin + 8) return "Likely Democrat";
      else return "Safe Democrat";
    } else { // Republican leading
      if (Math.abs(diff) <= tossUpMargin + 4) return "Lean Republican";
      else if (Math.abs(diff) <= tossUpMargin + 8) return "Likely Republican";
      else return "Safe Republican";
    }
  }

  // Function to handle slider change
  function onTossUpMarginChange(event) {
    tossUpMargin = +event.target.value;
    electoralVotes = { democrat: 0, republican: 0, tossUp: 0 }; // Reset electoral votes
    refreshMap();
  }

  // Function to handle click on a state
  function onStateClick(event, d) {
    const stateData = pollingData[d.id];
    if (stateData) {
      selectedState = {
        name: d.properties.name,
        abbrev: stateData.abbrev,
        democrat: stateData.democrat,
        republican: stateData.republican,
        electoralVotes: stateData.electoral_votes,
        unknown: 100 - (stateData.democrat + stateData.republican), // Unknown votes
        lean: getStateLean(stateData.democrat, stateData.republican)
      };

      // Update popup position near clicked coordinates
      const { pageX, pageY } = event;
      popupPosition = { x: pageX, y: pageY - 20 };

      drawPieChart();
    } else {
      console.warn("No data for state:", d.id);
    }
  }

  // Function to draw a pie chart with percentage labels
  function drawPieChart() {
    if (!selectedState) return;

    const data = [
    { party: "Democrat", value: +selectedState.democrat.toFixed(2), color: "#2a4b83" },
    { party: "Republican", value: +selectedState.republican.toFixed(2), color: "#a62929" },
    { party: "Unknown", value: +selectedState.unknown.toFixed(2), color: "#D3D3D3" }
  ];

    const radius = 60;
    const pie = d3.pie().value(d => d.value)(data);
    const arc = d3.arc().innerRadius(0).outerRadius(radius);

    d3.select("#pie-chart").selectAll("*").remove();

    const svg = d3.select("#pie-chart")
      .attr("width", radius * 2)
      .attr("height", radius * 2)
      .append("g")
      .attr("transform", `translate(${radius},${radius})`);

    svg.selectAll("path")
      .data(pie)
      .join("path")
      .attr("d", arc)
      .attr("fill", d => d.data.color);

    svg.selectAll("text")
      .data(pie)
      .join("text")
      .attr("transform", d => `translate(${arc.centroid(d)})`)
      .attr("text-anchor", "middle")
      .attr("dy", "0.35em")
      .attr("fill", "#fff")
      .attr("font-size", "10px")
      .text(d => `${d.data.value}%`);
  }

  // Function to refresh the map based on updated toss-up margin
  function refreshMap() {
    svg.selectAll("path")
      .attr("fill", d => getFillColor(d.id));

    renderElectoralCollegeBar(); // Update electoral college bar
  }

  // Function to calculate and render the electoral college bar
  function renderElectoralCollegeBar() {
    const totalVotes = 538;
    const democratWidth = (electoralVotes.democrat / totalVotes) * 100;
    const republicanWidth = (electoralVotes.republican / totalVotes) * 100;
    const tossUpWidth = (electoralVotes.tossUp / totalVotes) * 100;

    d3.select("#electoral-college-bar")
      .style("display", "flex")
      .html(`
        <div style="width: ${democratWidth}%; background-color: #2a4b83; color: white; text-align: center;">
          Democrats ${electoralVotes.democrat}
        </div>
        <div style="width: ${tossUpWidth}%; background-color: #D3D3D3; color: black; text-align: center;">
          ${electoralVotes.tossUp}
        </div>
        <div style="width: ${republicanWidth}%; background-color: #a62929; color: white; text-align: center;">
          Republicans ${electoralVotes.republican}
        </div>
      `);
  }

  onMount(async () => {
    try {
      const csvData = await d3.csv("/polling-data.csv");
      csvData.forEach(row => {
          pollingData[row.stateid] = {
              democrat: +row.democrat,
              republican: +row.republican,
              abbrev: row.abbrev,
              electoral_votes: +row.electoral_votes
          };
      });

      const us = await fetch("https://cdn.jsdelivr.net/npm/us-atlas@3/states-10m.json")
        .then(d => d.json());

      const states = topojson.feature(us, us.objects.states).features;
      const projection = d3.geoAlbersUsa().scale(1300).translate([width / 2, height / 2]);
      const path = d3.geoPath().projection(projection);

      svg = d3.select("#us-map").attr("width", width).attr("height", height);

      svg.append("g")
        .selectAll("path")
        .data(states)
        .join("path")
        .attr("fill", d => getFillColor(d.id))
        .attr("d", path)
        .attr("stroke", "#333")
        .attr("stroke-width", 0.5)
        .on("click", onStateClick)
        .append("title")
        .text(d => {
          const data = pollingData[d.id] || { democrat: 0, republican: 0 };
          return `${d.properties.name || ''}: Democrat ${data.democrat}%, Republican ${data.republican}%`;
        });

      svg.append("g")
        .selectAll("text")
        .data(states)
        .join("text")
        .attr("transform", d => `translate(${path.centroid(d)})`)
        .attr("dy", "0.35em")
        .attr("text-anchor", "middle")
        .attr("font-size", "10px")
        .attr("fill", "#000")
        .style("font-weight", "bold")
        .text(d => pollingData[d.id] ? pollingData[d.id].abbrev : "");

      renderElectoralCollegeBar();
    } catch (error) {
      console.error("Error loading map data:", error);
    }
  });
</script>

<style>

</style>

<h2>US State Polling Map</h2>



<!-- Electoral College Bar -->
<div id="electoral-college-bar"></div>

<!-- SVG container for the U.S. map -->
<svg id="us-map"></svg>

<div id="slider-container">
  <label for="toss-up-slider">Adjust Toss-Up Margin:</label>
  <input 
    type="range" 
    id="toss-up-slider" 
    min="0" 
    max="20" 
    step="1" 
    value={tossUpMargin} 
    on:input={onTossUpMarginChange}
  />
  <span>{Math.round(tossUpMargin * 100) / 100}%</span>

  <!-- Question Mark with Tooltip -->
  <span class="tooltip-container">
    <span class="question-mark">?</span>
    <span class="tooltip-text">The Toss-Up Margin determines the percentage point difference within which a state is considered a toss-up. For example, with a 4% margin, if the polling difference between parties is 4% or less, the state is labeled a toss-up.</span>
  </span>
</div>


<!-- Popup with detailed information -->
{#if selectedState}
  <div class="popup" style="top: {popupPosition.y}px; left: {popupPosition.x}px;">
    <h3>{selectedState.name} ({selectedState.abbrev})</h3>
    <p>Electoral Votes: {selectedState.electoralVotes}</p>
    <p>{selectedState.lean}</p>
    <svg id="pie-chart"></svg>
  </div>
{/if}

<p>Blue indicates Democrat lead, red indicates Republican lead. Opacity shows the strength of the lead.</p>

<!-- Legend for the map -->
<svg id="legend" width="400" height="120">
  <g>
    <rect x="10" y="10" width="20" height="20" fill="#99c4f0"></rect>
    <text x="40" y="25" font-size="12px">Lean Democrat</text>
    
    <rect x="10" y="40" width="20" height="20" fill="#446ea5"></rect>
    <text x="40" y="55" font-size="12px">Likely Democrat</text>
    
    <rect x="10" y="70" width="20" height="20" fill="#2a4b83"></rect>
    <text x="40" y="85" font-size="12px">Safe Democrat</text>
    
    <rect x="200" y="10" width="20" height="20" fill="#f19d9d"></rect>
    <text x="230" y="25" font-size="12px">Lean Republican</text>
    
    <rect x="200" y="40" width="20" height="20" fill="#c94f4f"></rect>
    <text x="230" y="55" font-size="12px">Likely Republican</text>
    
    <rect x="200" y="70" width="20" height="20" fill="#a62929"></rect>
    <text x="230" y="85" font-size="12px">Safe Republican</text>
    
    <rect x="10" y="100" width="20" height="20" fill="#D3D3D3"></rect>
    <text x="40" y="115" font-size="12px">Toss-Up</text>
  </g>
</svg>
