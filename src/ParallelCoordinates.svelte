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
  
  // Variáveis que precisamos adicionar
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
    data = raw.map(d =>
      Object.fromEntries(Object.entries(d).map(([k, v]) => [k.trim(), parseValue(v)]))
    );
    allDimensions = Object.keys(data[0]);
    selectedDimensions = [...allDimensions];
    filteredData = [...data];
  });

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }
  function selectAll() {
    selectedDimensions = [...allDimensions];
  }
  function clearAll() {
    selectedDimensions = [];
  }

  $: if (data.length && selectedDimensions) {
    drawParallel();
  }

  function drawParallel() {
    d3.select(container).selectAll('*').remove();

    const margin = { top: 30, right: 10, bottom: 10, left: 10 };
    const width  = 900 - margin.left - margin.right;
    const height = 500 - margin.top  - margin.bottom;

    const svg = d3.select(container)
      .append('svg')
        .attr('width',  width + margin.left + margin.right)
        .attr('height', height + margin.top  + margin.bottom)
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

    // Desenha linhas de fundo e filtradas como um tipo só com switch
    svg.selectAll('.line')
      .data(data) // Sempre trabalhe com todos os dados
      .enter().append('path')
        .attr('class', 'line')
        .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
        .attr('fill', 'none')
        .attr('stroke', '#4682b4') // Todas começam azuis
        .attr('stroke-width', 1)
        .attr('opacity', 0.7);

    // Adiciona eixos
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
    
    // Adiciona brushes
    selectedDimensions.forEach(dim => {
      const brush = d3.brushY()
        .extent([[-10, 0], [10, height]])
        .on('brush', function(event) {
        if (event.selection) {
          const [y0, y1] = event.selection;
          const scale = yScales[dim];
          
          if (scale.ticks) { // Numérico
            brushes[dim] = [
              Math.min(scale.invert(y0), scale.invert(y1)),
              Math.max(scale.invert(y0), scale.invert(y1))
            ];
          } else { // Categórico
            const domain = scale.domain();
            const pixelToIndex = d3.scaleLinear()
              .range([0, domain.length - 1])
              .domain([0, height]);
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

      svg.append('g')
        .attr('class', 'brush')
        .attr('transform', `translate(${x(dim)}, 0)`)
        .call(brush)
        .call(g => g.select('.overlay')
          .attr('width', 20)) // Aumenta a área sensível ao brush
        .call(g => g.selectAll('.selection,.handle')
          .attr('stroke', '#4682b4')
          .attr('fill', '#4682b4')
          .attr('fill-opacity', 0.2));
    });
  }

  function updateFilteredData() {
  filteredData = data.filter(d => {
    return selectedDimensions.every(dim => {
      if (!brushes[dim]) return true;
      
      const value = d[dim];
      const [min, max] = brushes[dim];
      
      if (yScales[dim].ticks) { // Numérico
        return value >= Math.min(min, max) && value <= Math.max(min, max);
      } else { // Categórico
        const domain = yScales[dim].domain();
        const valueIndex = domain.indexOf(value);
        const minIndex = domain.indexOf(min);
        const maxIndex = domain.indexOf(max);
        return valueIndex >= Math.min(minIndex, maxIndex) && 
               valueIndex <= Math.max(minIndex, maxIndex);
      }
    });
  });

  d3.select(container).selectAll('.line')
    .attr('stroke', d => 
      filteredData.includes(d) ? '#4682b4' : '#e0e0e0' // Azul se filtrado, cinza se não
    )
    .attr('opacity', d => 
      filteredData.includes(d) ? 0.7 : 0.05 // Opaco se filtrado, quase transparente se não
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

<!-- Seu HTML existente permanece o mesmo -->
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
