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
  
  // Adiciona id incremental
  data = raw.map((d, i) => ({
    id: i + 1,
    ...Object.fromEntries(Object.entries(d).map(([k, v]) => [k.trim(), parseValue(v)]))
  }));

  allDimensions = Object.keys(data[0]); // inclui o 'id' como opção
  selectedDimensions = allDimensions.filter(d => d !== 'id'); // mas não seleciona 'id' por padrão
  filteredData = [...data];
});


  function toggleDropdown() { dropdownOpen = !dropdownOpen; }
  function selectAll() { selectedDimensions = [...allDimensions]; }
  function clearAll() { selectedDimensions = []; }

  function dragstarted(event, d) {
    d3.select(this).raise().classed('active', true);

    d3.select(container).selectAll('.line')
      .transition().duration(200)
      .style('opacity', 0.3);
  }

  function dragged(event, d) {
    const dx = event.x;

    d3.select(this).attr('transform', `translate(${dx},0)`);

    selectedDimensions.sort((a, b) => {
      if (a === d) return dx - x(b);
      if (b === d) return x(a) - dx;
      return x(a) - x(b);
    });

    x.domain(selectedDimensions);

    d3.select(container).selectAll('.axis')
      .transition().duration(200)
      .attr('transform', d => `translate(${x(d)},0)`);

    d3.select(container).selectAll('.line')
      .transition().duration(200)
      .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
  }

  function dragended(event, d) {
  d3.select(this).classed('active', false);

  // Resetar filtros
  brushes = {};
  filteredData = [...data];

  d3.select(container).selectAll('.line')
    .transition().duration(300)
    .attr('stroke', '#4682b4')
    .attr('opacity', 0.7);

  // Apagar todos os brushes visuais
  d3.select(container).selectAll('.brush').remove();

  // Redesenhar brushes zerados
  const svg = d3.select(container).select('svg g'); // pega o grupo principal
  selectedDimensions.forEach(dim => {
    createBrush(svg, dim, 500 - 30 - 10); // você pode melhorar depois para usar height paramétrico
  });
}

function clearFilters() {
  brushes = {};
  filteredData = [...data];

  // Redefine as linhas
  d3.select(container).selectAll('.line')
    .transition().duration(300)
    .attr('stroke', '#4682b4')
    .attr('opacity', 0.7);

  // Remove brushes visuais
  d3.select(container).selectAll('.brush').remove();

  // Redesenha brushes vazios
  const svg = d3.select(container).select('svg g');
  selectedDimensions.forEach(dim => {
    createBrush(svg, dim, 500 - 30 - 10); 
  });
}

function createBrush(svg, dim, height) {
  const brush = d3.brushY()
    .extent([[-10, 0], [10, height]])
    .on('brush', function(event) {
      if (event.selection) {
        const [y0, y1] = event.selection;
        const scale = yScales[dim];

        if (scale.bandwidth) {
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
        } else {
          brushes[dim] = [
            Math.min(scale.invert(y0), scale.invert(y1)),
            Math.max(scale.invert(y0), scale.invert(y1))
          ];
        }
        updateFilteredData();
      }
    })
    .on('end', function(event) {
      if (!event.selection) {
        delete brushes[dim];
        updateFilteredData();
      }
    });

  const gBrush = svg.append('g')
    .attr('class', 'brush')
    .attr('transform', `translate(${x(dim)},0)`)
    .call(brush)
    .call(g => g.select('.overlay').attr('width', 30))
    .call(g => g.selectAll('.selection,.handle')
      .attr('stroke', '#4682b4')
      .attr('fill', '#4682b4')
      .attr('fill-opacity', 0.2));

  return gBrush;
}

  $: if (data.length && selectedDimensions) {
    drawParallel();
  }

  function drawParallel() {
  d3.select(container).selectAll('*').remove();

  const margin = { top: 30, right: 10, bottom: 10, left: 10 };
  const width = 900 - margin.left - margin.right;
  const height = 500 - margin.top - margin.bottom;

  const svg = d3.select(container)
    .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
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

  svg.selectAll('.line')
    .data(data)
    .enter().append('path')
      .attr('class', 'line')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', '#4682b4')
      .attr('stroke-width', 1)
      .attr('opacity', 0.7);

  svg.selectAll('.axis')
    .data(selectedDimensions, d => d)
    .enter()
    .append('g')
      .attr('class', 'axis')
      .attr('transform', d => `translate(${x(d)},0)`)
      .each(function(d) {
        d3.select(this).call(d3.axisLeft(yScales[d]));
      })
      .call(
        d3.drag()
          .on('start', dragstarted)
          .on('drag', dragged)
          .on('end', dragended)
      )
      .append('text')
        .style('text-anchor', 'middle')
        .attr('y', -9)
        .text(d => d)
        .style('fill', 'black');

        selectedDimensions.forEach(dim => {
  createBrush(svg, dim, height);
});
;
}


  function updateFilteredData() {
  filteredData = data.filter(d => {
    return selectedDimensions.every(dim => {
      if (!brushes[dim]) return true;

      const value = d[dim];
      const [min, max] = brushes[dim];

      if (yScales[dim].bandwidth) {
        const domain = yScales[dim].domain();
        const valueIndex = domain.indexOf(value);
        const minIndex = domain.indexOf(min);
        const maxIndex = domain.indexOf(max);

        return valueIndex >= Math.min(minIndex, maxIndex) &&
               valueIndex <= Math.max(minIndex, maxIndex);
      } else {
        return value >= Math.min(min, max) && value <= Math.max(min, max);
      }
    });
  });

  d3.select(container).selectAll('.line')
    .transition().duration(300)
    .attr('stroke', d =>
      filteredData.includes(d) ? '#000' : '#4682b4' // Preto para filtradas, azul claro para não-filtradas
    )
    .attr('opacity', d =>
      filteredData.includes(d) ? 0.9 : 0.05 // Forte para filtradas, quase invisível para não-filtradas
    );
}

</script>

<style>
.multiselect {
  position: relative;
  display: inline-block;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
}

.dropdown-btn {
  background: white;
  border: 1px solid #ddd;
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s, box-shadow 0.3s;
}
.dropdown-btn:hover {
  background: #f9f9f9;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}
.dropdown-btn:focus {
  outline: none;
}

.dropdown-panel {
  position: absolute;
  top: 110%;
  left: 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  margin-top: 0.5rem;
  width: 260px;
  z-index: 20;
}

.filter-input {
  width: calc(100% - 1rem);
  margin: 0.5rem;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.checkboxes {
  max-height: 200px;
  overflow-y: auto;
  padding: 0.5rem;
}
.checkboxes label {
  display: flex;
  align-items: center;
  margin-bottom: 0.4rem;
  font-size: 13px;
  color: #333;
}
.checkboxes input {
  margin-right: 0.6rem;
}

.actions {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem;
  border-top: 1px solid #eee;
}
.actions button {
  background: #f7f7f7;
  border: 1px solid #ccc;
  border-radius: 6px;
  padding: 0.4rem 0.8rem;
  font-size: 13px;
  cursor: pointer;
  transition: background 0.3s;
}
.actions button:hover {
  background: #ebebeb;
}

:global(svg) {
  font-family: 'Inter', sans-serif;
}

:global(.line) {
  stroke-linejoin: round;
  stroke-linecap: round;
  shape-rendering: geometricPrecision;
  transition: stroke 0.4s, opacity 0.4s;
}

:global(.axis path),
:global(.axis line) {
  fill: none;
  stroke: #ccc;
  shape-rendering: crispEdges;
}

:global(.axis text) {
  font-size: 12px;
  fill: #555;
}

:global(.axis.active text) {
  fill: #d43f3a;
  font-weight: bold;
}

:global(.brush .selection) {
  fill: #4682b4;
  fill-opacity: 0.25; 
  stroke: #4682b4;
  stroke-width: 1;
  shape-rendering: crispEdges;
}

:global(.brush .handle) {
  fill: #4682b4;
  cursor: ns-resize; 
}

:global(.brush .overlay) {
  cursor: crosshair; 
  fill: none;
  pointer-events: all;
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
        <button type="button" on:click={clearFilters}>Remover Filtros</button>
        
      </div>
    </div>
  {/if}
</div>

<div bind:this={container} style="height: 600px; margin-top: 1rem;"></div>