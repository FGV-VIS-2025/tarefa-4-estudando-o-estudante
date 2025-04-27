<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let container;
  let data = [];
  let allDimensions = [];
  let selectedDimensions = [];
  let filterText = '';
  let dropdownOpen = false;
  let brushes = {};
  let colourVar = '';
  let colourScale;
  let isNumericColour = false;
  let x, yScales, line;
  let filteredData = [];

  function parseValue(str) {
    const s = str.trim();
    if (/^-?\d+(\.\d+)?$/.test(s)) return +s;
    if (/^\d+(\.\d+)?%$/.test(s)) return +s.slice(0, -1);
    return s;
  }

  onMount(async () => {
    const raw = await d3.csv(import.meta.env.BASE_URL + 'data/student_attitude.csv');
    data = raw.map(d => {
      const parsed = Object.fromEntries(
        Object.entries(d).map(([k, v]) => [k.trim(), parseValue(v)]
      ));
      parsed.id = Math.random().toString(36).substr(2, 9); // Adiciona um ID único
      return parsed;
    });
    allDimensions = Object.keys(data[0]).filter(d => d !== 'id');
    selectedDimensions = [...allDimensions];
    colourVar = allDimensions[0];
    filteredData = [...data];
  });

  function computeColourScale() {
    const vals = data.map(d => d[colourVar]);
    isNumericColour = vals.every(v => typeof v === 'number' && !isNaN(v));

    if (isNumericColour) {
      const [minVal, maxVal] = d3.extent(vals);
      data.forEach(d => d.__colour = d[colourVar]);
      return d3.scaleSequential(d3.interpolatePlasma).domain([minVal, maxVal]);
    } else {
      const categories = [...new Set(vals)];
      data.forEach(d => d.__colour = d[colourVar]);
      return d3.scaleOrdinal()
        .domain(categories)
        .range(d3.schemeTableau10);
    }
  }

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }

  function selectAll() {
    selectedDimensions = [...allDimensions];
  }

  function clearAll() {
    selectedDimensions = [];
  }

  $: if (data.length && selectedDimensions.length && colourVar) {
    colourScale = computeColourScale();
    drawParallel();
  }

  function drawParallel() {
    d3.select(container).selectAll('*').remove();

    const margin = { top: 30, right: 10, bottom: 60, left: 10 };
    const width = 900 - margin.left - margin.right;
    const height = 500 - margin.top - margin.bottom;

    const svg = d3.select(container)
      .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom + 60)
      .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);

    x = d3.scalePoint()
      .domain(selectedDimensions)
      .range([0, width])
      .padding(1);

    yScales = {};
    selectedDimensions.forEach(dim => {
      const vals = data.map(d => d[dim]);
      if (vals.every(v => typeof v === 'number')) {
        yScales[dim] = d3.scaleLinear()
          .domain(d3.extent(vals))
          .range([height, 0])
          .nice();
      } else {
        const cats = Array.from(new Set(vals));
        yScales[dim] = d3.scalePoint()
          .domain(cats)
          .range([height, 0])
          .padding(0.5);
      }
    });

    line = d3.line();

    // Linhas de fundo
    svg.append('g').attr('class', 'background')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', d => colourScale(d.__colour))
      .attr('stroke-width', 1)
      .attr('opacity', 0.15);

    // Linhas principais
    const foreground = svg.append('g').attr('class', 'foreground')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', d => colourScale(d.__colour))
      .attr('stroke-width', 1.5)
      .attr('opacity', 0.7);

    // Eixos
    svg.selectAll('.axis')
      .data(selectedDimensions).enter()
      .append('g')
        .attr('class', 'axis')
        .attr('transform', d => `translate(${x(d)},0)`)
      .each(function(d) {
        d3.select(this).call(d3.axisLeft(yScales[d]));
      })
      .append('text')
        .style('text-anchor', 'middle')
        .attr('y', -9)
        .text(d => d)
        .style('fill', 'black');

    // Brushes
    selectedDimensions.forEach(dim => {
      const brush = d3.brushY()
        .extent([[-10, 0], [10, height]])
        .on('brush', function(event) {
          if (event.selection) {
            const [y0, y1] = event.selection;
            const scale = yScales[dim];
            
            if (scale.ticks) {
              brushes[dim] = [
                Math.min(scale.invert(y0), scale.invert(y1)),
                Math.max(scale.invert(y0), scale.invert(y1))
              ];
            } else {
              const domain = scale.domain();
              const pixelToIndex = d3.scaleLinear()
                .range([0, domain.length - 1])
                .domain([height, 0]);

              const index0 = Math.round(pixelToIndex(y0));
              const index1 = Math.round(pixelToIndex(y1));
              
              brushes[dim] = [
                domain[Math.min(index0, index1)],
                domain[Math.max(index0, index1)]
              ];
            }
            updateFilteredData();
          }
        })
        .on('end', function(event) {
          if (!event.selection) {
            delete brushes[dim];
            filteredData = [...data];
            updateFilteredData();
          }
        });

      svg.append('g')
        .attr('class', 'brush')
        .attr('transform', `translate(${x(dim)}, 0)`)
        .call(brush)
        .call(g => g.select('.overlay')
          .attr('width', 20))
        .call(g => g.selectAll('.selection,.handle')
          .attr('stroke', '#4682b4')
          .attr('fill', '#4682b4')
          .attr('fill-opacity', 0.2));
    });

    // Legenda
    const legend = svg.append('g')
      .attr('transform', `translate(20, ${height + 20})`);

    if (isNumericColour) {
      const legendWidth = 200;
      const legendHeight = 20;
      
      const defs = svg.append('defs');
      const gradient = defs.append('linearGradient')
        .attr('id', 'color-gradient')
        .attr('x1', '0%')
        .attr('y1', '0%')
        .attr('x2', '100%')
        .attr('y2', '0%');
      
      gradient.selectAll('stop')
        .data(d3.range(0, 1.01, 0.1))
        .enter().append('stop')
        .attr('offset', d => `${d * 100}%`)
        .attr('stop-color', d => d3.interpolatePlasma(d));
      
      legend.append('rect')
        .attr('width', legendWidth)
        .attr('height', legendHeight)
        .style('fill', 'url(#color-gradient)');
      
      legend.append('g')
        .attr('transform', `translate(0, ${legendHeight})`)
        .call(d3.axisBottom(colourScale.copy()))
        .select('.domain')
        .attr('stroke', '#777');
    } else {
      const categories = [...new Set(data.map(d => d[colourVar]))];
      const itemWidth = 100;
      
      categories.forEach((cat, i) => {
        const g = legend.append('g')
          .attr('transform', `translate(${i * itemWidth}, 0)`);
        
        g.append('circle')
          .attr('r', 5)
          .attr('fill', colourScale(cat));
        
        g.append('text')
          .attr('x', 10)
          .attr('y', 5)
          .text(cat.length > 15 ? cat.substring(0, 12) + '...' : cat)
          .style('font-size', '10px');
      });
    }
  }

  function updateFilteredData() {
    filteredData = data.filter(d => {
      return selectedDimensions.every(dim => {
        if (!brushes[dim]) return true;
        
        const value = d[dim];
        const [min, max] = brushes[dim];
        
        if (yScales[dim].ticks) {
          return value >= Math.min(min, max) && value <= Math.max(min, max);
        } else {
          const domain = yScales[dim].domain();
          const valueIndex = domain.indexOf(value);
          const minIndex = domain.indexOf(min);
          const maxIndex = domain.indexOf(max);
          
          return valueIndex >= Math.min(minIndex, maxIndex) && 
                 valueIndex <= Math.max(minIndex, maxIndex);
        }
      });
    });

    d3.select(container).selectAll('.foreground path')
      .attr('stroke', d => 
        filteredData.some(fd => fd.id === d.id) ? colourScale(d.__colour) : '#e0e0e0'
      )
      .attr('opacity', d => 
        filteredData.some(fd => fd.id === d.id) ? 0.7 : 0.05
      );
  }
</script>

<style>
  .multiselect {
    position: relative;
    display: inline-block;
    font-family: sans-serif;
  }
  .dropdown-btn {
    background: #fff;
    border: 1px solid #ccc;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
  }
  .dropdown-btn:focus { outline: none; }

  .dropdown-panel {
    position: absolute;
    top: 100%; left: 0;
    background: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    margin-top: 0.25rem;
    width: 260px;
    z-index: 10;
  }
  .filter-input {
    width: calc(100% - 1rem);
    margin: 0.5rem;
    padding: 0.3rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  .checkboxes {
    max-height: 180px;
    overflow-y: auto;
    padding: 0.5rem;
  }
  .checkboxes label {
    display: flex;
    align-items: center;
    margin-bottom: 0.3rem;
    font-size: 0.9rem;
  }
  .checkboxes input {
    margin-right: 0.5rem;
  }
  .actions {
    display: flex;
    justify-content: space-between;
    padding: 0.5rem;
    border-top: 1px solid #eee;
  }
  .actions button {
    background: #f5f5f5;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 0.3rem 0.6rem;
    cursor: pointer;
    font-size: 0.85rem;
  }
  .actions button:hover {
    background: #eaeaea;
  }
</style>
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
    {selectedDimensions.length === 0
      ? 'Escolher variáveis'
      : `${selectedDimensions.length} selecionada(s)`}
  </button>
  {#if dropdownOpen}
    <div class="dropdown-panel">
      <input
        class="filter-input"
        type="text"
        placeholder="Filtrar..."
        bind:value={filterText}
      />
      <div class="checkboxes">
        {#each allDimensions
                     .filter(dim => dim.toLowerCase().includes(filterText.toLowerCase()))
               as dim}
          <label>
            <input
              type="checkbox"
              bind:group={selectedDimensions}
              value={dim}
            />
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

<div bind:this={container} style="height: 600px; margin-top: 1rem;"></div>