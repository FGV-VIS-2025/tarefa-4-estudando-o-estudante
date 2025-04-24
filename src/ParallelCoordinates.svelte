<script>
  import { onMount } from 'svelte';
  import { interpolateTurbo } from 'd3';
  import * as d3 from 'd3';

  let container;
  let data = [];
  let allDimensions = [];
  let selectedDimensions = [];
  let filterText = '';
  let dropdownOpen = false;
  let colourVar = '';
  let colourScale;
  let isNumericColour = false;

function parseValue(str) {
  if (str === undefined || str === null || str === '') return null;
  
  const s = String(str).trim().replace(/,/g, '').replace(/\s+/g, '');
  
  // Check for percentage
  if (/^\d+(\.\d+)?%$/.test(s)) return +s.slice(0, -1) / 100;
  
  // Check for regular numbers
  if (!isNaN(s) && s !== '') return +s;
  
  // Return original if not a number
  return str;
}
  function computeColourScale() {
    
    const vals = data.map(d => d[colourVar]);
    isNumericColour = vals.every(v => typeof v === 'number' && !isNaN(v));

    if (isNumericColour) {
      const [minVal, maxVal] = d3.extent(vals);
      colourScale = d3.scaleSequential(interpolateTurbo).domain([minVal, maxVal]);
      data.forEach(d => d.__colour = d[colourVar]);
    } else {
      const uniqueVals = [...new Set(vals)];
      colourScale = d3.scaleOrdinal()
        .domain(uniqueVals)
        .range(d3.schemeTableau10); // Using a better color scheme for categories
      data.forEach(d => d.__colour = d[colourVar]);
    }

      console.log('Colour variable:', colourVar);
      console.log('Sample values:', vals.slice(0, 5));
      isNumericColour = vals.every(v => typeof v === 'number' && !isNaN(v));
      console.log('Is numeric:', isNumericColour);
  

  }

  onMount(async () => {
    const raw = await d3.csv(import.meta.env.BASE_URL + 'data/student_attitude.csv');
    data = raw.map(d => Object.fromEntries(Object.entries(d).map(([k, v]) => [k.trim(), parseValue(v)])));
    allDimensions = Object.keys(data[0]);
    selectedDimensions = [...allDimensions];
    colourVar = allDimensions[0];
  });

  function toggleDropdown() { dropdownOpen = !dropdownOpen; }
  const selectAll = () => selectedDimensions = [...allDimensions];
  const clearAll = () => selectedDimensions = [];

  $: if (data.length && selectedDimensions.length && colourVar) {
    computeColourScale();
    drawParallel();
  }

  function drawParallel() {
    d3.select(container).selectAll('*').remove();
    const margin = { top: 40, right: 20, bottom: 20, left: 20 };
    const width = 900 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    const svg = d3.select(container)
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom + 60)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    const x = d3.scalePoint().domain(selectedDimensions).range([0, width]).padding(1);
    const y = {};
    selectedDimensions.forEach(dim => {
      const vals = data.map(d => d[dim]);
      y[dim] = vals.every(v => typeof v === 'number')
        ? d3.scaleLinear().domain(d3.extent(vals)).nice().range([height, 0])
        : d3.scalePoint().domain([...new Set(vals)]).range([height, 0]).padding(0.5);
    });

    const line = d3.line();

    svg.append('g').attr('class', 'background')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), y[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', d => colourScale(d.__colour))
      .attr('stroke-width', 1)
      .attr('opacity', 0.15);

    const foreground = svg.append('g').attr('class', 'foreground')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), y[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', d => colourScale(d.__colour))
      .attr('stroke-width', 1.2)
      .attr('opacity', 0.6);

    const axis = svg.selectAll('.dimension')
      .data(selectedDimensions)
      .enter().append('g')
      .attr('class', 'dimension')
      .attr('transform', d => `translate(${x(d)})`)
      .call(d3.drag()
        .subject(d => ({ x: x(d) }))
        .on('start', function(event, d) { this.__origin__ = x(d); })
        .on('drag', function(event, d) {
          const dx = event.x;
          x.range([0, width]);
          d3.select(this).attr('transform', `translate(${dx})`);
          selectedDimensions.sort((a,b) => x(a) - x(b));
          foreground.attr('d', d => line(selectedDimensions.map(p => [x(p), y[p](d[p])])));
        })
        .on('end', function() {
          d3.select(this).attr('transform', d => `translate(${x(d)})`);
        })
      );

    axis.each(function(d) {
      d3.select(this).call(d3.axisLeft(y[d]));
    });

    axis.append('text')
      .attr('y', -9)
      .attr('text-anchor', 'middle')
      .text(d => d)
      .style('font-weight', d => d === colourVar ? 'bold' : null);

    axis.filter(d => y[d].bandwidth === undefined)
      .append('g')
      .attr('class', 'brush')
      .each(function(d) {
        d3.select(this).call(
          y[d].brush = d3.brushY()
            .extent([[-8,0],[8,height]])
            .on('brush end', brushed)
        );
      });

    function brushed() {
      const actives = [];
      svg.selectAll('.brush')
        .filter(function(d){ return d3.brushSelection(this); })
        .each(function(d){ actives.push({ dim: d, extent: d3.brushSelection(this).map(y[d].invert) }); });

      foreground.style('display', d => {
        return actives.every(active => {
          const val = d[active.dim];
          return val >= Math.min(...active.extent) && val <= Math.max(...active.extent);
        }) ? null : 'none';
      });
    }

    // LEGEND
    const legendGroup = svg.append("g").attr("transform", `translate(0, ${height + 30})`);
    
    if (isNumericColour) {
      // Numeric legend
      const defs = svg.append("defs");
      const gradient = defs.append("linearGradient")
        .attr("id", "legend-gradient")
        .attr("x1", "0%")
        .attr("y1", "0%")
        .attr("x2", "100%")
        .attr("y2", "0%");

      for (let i = 0; i <= 10; i++) {
        gradient.append("stop")
          .attr("offset", `${(i / 10) * 100}%`)
          .attr("stop-color", d3.interpolateTurbo(i / 10));
      }

      const legendWidth = 250, legendHeight = 12;

      legendGroup.append("rect")
        .attr("width", legendWidth)
        .attr("height", legendHeight)
        .style("fill", "url(#legend-gradient)")
        .style("stroke", "#ccc");

      legendGroup.append("g")
        .attr("transform", `translate(0, ${legendHeight})`)
        .call(d3.axisBottom(colourScale).ticks(5));
    } else {
      // Categorical legend
      const uniqueValues = [...new Set(data.map(d => d[colourVar]))];
      const itemHeight = 20;
      const itemsPerRow = 5;
      const itemWidth = 100;
      
      uniqueValues.forEach((value, i) => {
        const row = Math.floor(i / itemsPerRow);
        const col = i % itemsPerRow;
        
        legendGroup.append("rect")
          .attr("x", col * itemWidth)
          .attr("y", row * itemHeight)
          .attr("width", 15)
          .attr("height", 15)
          .attr("fill", colourScale(value));
          
        legendGroup.append("text")
          .attr("x", col * itemWidth + 20)
          .attr("y", row * itemHeight + 12)
          .text(value)
          .style("font-size", "12px");
      });
    }
    
    legendGroup.append("text")
      .attr("x", 125)
      .attr("y", -10)
      .attr("text-anchor", "middle")
      .style("font-size", "12px")
      .style("fill", "#333")
      .text(`Escala de cor: ${colourVar}`);
  }
</script>

<style>
  .multiselect { position: relative; display: inline-block; font-family: sans-serif; }
  .dropdown-btn { background:#fff; border:1px solid #ccc; padding:0.5rem 1rem; border-radius:4px; cursor:pointer; }
  .dropdown-panel {
    position: absolute;
    top: 100%;
    left: 0;
    background: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    margin-top: 0.25rem;
    width: 400px;
    max-height: 350px;
    overflow-y: auto;
    z-index: 10;
  }
  .filter-input { width:calc(100% - 1rem); margin:0.5rem; padding:0.3rem; border:1px solid #ccc; border-radius:4px; }
  .checkboxes { max-height:180px; overflow-y:auto; padding:0.5rem; }
  .checkboxes label {
    display: flex;
    align-items: flex-start;
    font-size: 0.9rem;
    word-break: break-word;
    line-height: 1.3rem;
  }
  .checkboxes input { margin-right:0.5rem; }
  .actions { display:flex; justify-content:space-between; padding:0.5rem; border-top:1px solid #eee; }
  .actions button { background:#f5f5f5; border:1px solid #ccc; border-radius:4px; padding:0.3rem 0.6rem; cursor:pointer; font-size:0.85rem; }
  .actions button:hover { background:#eaeaea; }
  select { padding:0.25rem 0.5rem; border:1px solid #ccc; border-radius:4px; font-size:0.85rem; margin-right:0.5rem; }
</style>

<!-- === UI === -->
<div style="margin-bottom:0.5rem">
  <label style="font-size:0.85rem; margin-right:0.25rem">Color by:</label>
  <select bind:value={colourVar}>
    {#each allDimensions as dim}
      <option value={dim}>{dim}</option>
    {/each}
  </select>
</div>

<div class="multiselect">
  <button class="dropdown-btn" on:click={toggleDropdown}>
    {selectedDimensions.length === 0 ? 'Escolher variáveis' : `${selectedDimensions.length} selecionada(s)`}
  </button>
  {#if dropdownOpen}
    <div class="dropdown-panel">
      <input class="filter-input" type="text" placeholder="Filtrar..." bind:value={filterText} />
      <div class="checkboxes">
        {#each allDimensions.filter(dim => dim.toLowerCase().includes(filterText.toLowerCase())) as dim}
          <label>
            <input type="checkbox" bind:group={selectedDimensions} value={dim} />
            {dim}
          </label>
        {/each}
      </div>
      <div class="actions">
        <button type="button" on:click={selectAll}>Selecionar tudo</button>
        <button type="button" on:click={clearAll}>Limpar tudo</button>
      </div>
    </div>
  {/if}
</div>

<div bind:this={container} style="height:600px; margin-top:1rem;"></div>
