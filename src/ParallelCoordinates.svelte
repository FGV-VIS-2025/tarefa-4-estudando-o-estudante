<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let container;
  let data = [];
  let allDimensions = [];
  let selectedDimensions = [];
  let filterText = '';
  let dropdownOpen = false;

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
    selectedDimensions = [...allDimensions]; // tudo marcado por padrão
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

  // redesenha o paralelo sempre que data OU seleção mudam
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

    const x = d3.scalePoint()
      .domain(selectedDimensions)
      .range([0, width])
      .padding(1);

    const yScales = {};
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

    const line = d3.line();

    svg.selectAll('path')
      .data(data)
      .enter().append('path')
        .attr('d', d => line(selectedDimensions.map(p => [ x(p), yScales[p](d[p]) ])))
        .attr('fill', 'none')
        .attr('stroke', '#4682b4')
        .attr('stroke-width', 1)
        .attr('opacity', 0.3);

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
