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
  let colourVariable = 'Salary Expectation'; // Valor inicial
let colourScale; // A escala de cor será criada dinamicamente
let selectedPalette = 'Turbo'; // Valor inicial
let hoveredDatum = null;
let selectedDatum = null;
let removedData = new Set();
let legendContainer;
let brushMode = 'color'; // 'color' ou 'hide'

function updateAxisLabels() {
  d3.select(container).selectAll('.axis').each(function(d) {
    if (brushes[d]) {
      d3.select(this).select('text')
        .style('fill', '#d43f3a')
        .style('font-weight', 'bold');
    } else {
      d3.select(this).select('text')
        .style('fill', 'black')
        .style('font-weight', 'normal');
    }
  });
}

function restoreAllData() {
  removedData.clear();      // Limpa todos os IDs removidos
  selectedDatum = null;     // Limpa qualquer seleção
  computeColourScale();     // Recalcula a escala de cores
  drawParallel();           // Redesenha o gráfico completo
}

function computeColourScale() {
  const vals = data.map(d => d[colourVariable]);
  const allNumbers = vals.every(v => typeof v === 'number' && !isNaN(v));

  if (allNumbers) {
    const extent = d3.extent(vals);
    let interpolator;

    switch (selectedPalette) {
      case 'Viridis':
        interpolator = d3.interpolateViridis;
        break;
      case 'Plasma':
        interpolator = d3.interpolatePlasma;
        break;
      case 'Inferno':
        interpolator = d3.interpolateInferno;
        break;
      case 'Blues':
        interpolator = d3.interpolateBlues;
        break;
      default:
        interpolator = d3.interpolateTurbo;
    }

    colourScale = d3.scaleSequential()
      .domain(extent)
      .interpolator(interpolator);

  } else {
    const categories = Array.from(new Set(vals));
    colourScale = d3.scaleOrdinal()
      .domain(categories)
      .range(d3.schemeSet2);
  }
}


  function parseValue(str) {
    const s = str.trim();
    if (/^-?\d+(\.\d+)?$/.test(s)) return +s;
    if (/^\d+(\.\d+)?%$/.test(s)) return +s.slice(0, -1);
    return s;
  }

  onMount(async () => {
  const raw = await d3.csv(import.meta.env.BASE_URL + 'data/student_attitude.csv');
  
  data = raw.map((d, i) => ({
    id: i + 1,
    ...Object.fromEntries(Object.entries(d).map(([k, v]) => [k.trim(), parseValue(v)]))
  }));

  allDimensions = Object.keys(data[0]); // inclui o 'id' como opção

  // Defina aqui quais eixos aparecem inicialmente:
  const initialSet = [
    "Gender",
    "Department",
    "college mark",
    "Stress Level"
  ];

  // Só adiciona no início os que existem no data
  selectedDimensions = initialSet.filter(dim => allDimensions.includes(dim));

  filteredData = [...data];
  colourVariable = 'college mark';

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
    .attr('stroke', d => colourScale(d[colourVariable]))

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
      }
    })
    .on('end', function(event) {
      if (!event.selection) {
        delete brushes[dim];
      }
      updateFilteredData();
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

$: if (data.length && selectedDimensions.length) {
  computeColourScale();
  drawParallel();
  drawLegend();
}

  $: if (data.length && selectedDimensions) {
    drawParallel();
  }

  function drawParallel() {
  d3.select(container).selectAll('*').remove();

  const margin = { top: 30, right: 10, bottom: 10, left: 10 };
  const width = 700 - margin.left - margin.right;
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
    const vals = data.filter(d => !removedData.has(d.id)).map(d => d[dim]);
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

  // Fundo cinza claro
  svg.append('rect')
    .attr('x', -margin.left)
    .attr('y', -margin.top)
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .attr('fill', '#f0f0f0');

  // Desenhar linhas
  const allLines = svg.selectAll('.line')
    .data(data.filter(d => !removedData.has(d.id)))
    .enter().append('path')
      .attr('class', 'line')
      .style('cursor', 'pointer')
      .attr('d', d => line(selectedDimensions.map(p => [x(p), yScales[p](d[p])])))
      .attr('fill', 'none')
      .attr('stroke', d => d === selectedDatum ? 'red' : colourScale(d[colourVariable]))
      .attr('stroke-width', d => d === selectedDatum ? 3 : 2.5)
      .attr('opacity', d => d === selectedDatum ? 1 : 0.7)
      .on('mouseover', function(event, d) {
        d3.selectAll('.line')
          .transition().duration(150)
          .attr('opacity', l => (l === d ? 1 : 0.05));

        d3.select(this)
          .transition().duration(150)
          .attr('stroke-width', 3);
      })
      .on('mouseout', function(event, d) {
        d3.selectAll('.line')
          .transition().duration(150)
          .attr('opacity', l => (l === selectedDatum ? 1 : 0.7));

        d3.select(this)
          .transition().duration(150)
          .attr('stroke-width', d => d === selectedDatum ? 3 : 2.5);
      })



      .on('click', function(event, d) {
        if (selectedDatum === d) {
          selectedDatum = null;
        } else {
          selectedDatum = d;
        }
        drawParallel(); // Redesenha com a nova seleção
      });

  // Se tiver linha selecionada, traz para frente
  if (selectedDatum) {
    allLines
      .filter(d => d === selectedDatum)
      .raise();
  }

  // Eixos
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
        .attr('class', 'axis-label')
        .style('text-anchor', 'middle')
        .attr('y', -9)
        .text(d => d);

  // Brushing
  selectedDimensions.forEach(dim => {
    createBrush(svg, dim, height);
  });

  updateAxisLabels();
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

  const lines = d3.select(container).selectAll('.line');

  if (brushMode === 'color') {
    lines
      .transition().duration(300)
      .attr('stroke', d => filteredData.includes(d) ? '#000' : '#4682b4')
      .attr('opacity', d => filteredData.includes(d) ? 0.9 : 0.05);
  } else if (brushMode === 'hide') {
    lines
      .transition().duration(300)
      .attr('opacity', d => filteredData.includes(d) ? 0.9 : 0);
  }
}

function drawLegend() {
  if (!colourScale || !legendContainer) return;

  d3.select(legendContainer).selectAll('*').remove();

  const width = 200;
  const height = 300;
  const svg = d3.select(legendContainer)
  .append('svg')
  .attr('width', width)
  .attr('height', height + 40)

  // const svg = d3.select(legendContainer)
  //   .append('svg')
  //   .attr('width', width)
  //   .attr('height', height + 40)
  //   .append('g')
  //   .attr('transform', 'translate(20,20)');

  // CONTÍNUA
  if (typeof colourScale.interpolator === 'function') {
    const defs = svg.append('defs');
    const gradient = defs.append('linearGradient')
      .attr('id', 'color-gradient')
      .attr('x1', '0%')
      .attr('y1', '100%')
      .attr('x2', '0%')
      .attr('y2', '0%');

    const domain = colourScale.domain();
    const n = 10;
    for (let i = 0; i <= n; i++) {
      gradient.append('stop')
        .attr('offset', `${(i / n) * 100}%`)
        .attr('stop-color', colourScale(domain[0] + i / n * (domain[1] - domain[0])));
    }

    svg.append('rect')
      .attr('width', 20)
      .attr('height', height)
      .style('fill', 'url(#color-gradient)');

    const scale = d3.scaleLinear()
      .domain([domain[0], domain[1]])
      .range([height, 0]);

    const axis = d3.axisRight(scale)
      .ticks(5)
      .tickSize(6)
      .tickFormat(d3.format(".2~g"));

    svg.append('g')
      .attr('transform', `translate(22,0)`)
      .call(axis)
      .select(".domain").remove();

  } else {
    // CATEGÓRICA
    const domain = colourScale.domain();
    const itemHeight = 20;

    domain.forEach((cat, i) => {
      svg.append('rect')
        .attr('x', 0)
        .attr('y', i * itemHeight)
        .attr('width', 14)
        .attr('height', 14)
        .attr('fill', colourScale(cat))
        .attr('stroke', '#ccc');

      svg.append('text')
        .attr('x', 20)
        .attr('y', i * itemHeight + 11)
        .text(cat)
        .style('font-size', '12px');
    });
  }
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
:global(.axis text) {
  font-size: 12px;
  fill: #555; /* Todos os ticks em cinza */
}

:global(.axis-label) {
  font-size: 13px;
  fill: black; /* Só o nome do eixo em preto */
  font-weight: bold;
}


</style>

<svelte:head>
  <title>Visualização: Student Attitude and Behavior</title>
  <meta name="description" content="Gráfico interativo de coordenadas paralelas com filtros, coloração e brushing para explorar dados de comportamento estudantil." />
</svelte:head>

<!-- 📘 Texto introdutório -->
<div style="max-width: 900px; margin: 2rem auto; font-family: 'Inter', sans-serif; line-height: 1.6; font-size: 15px; color: #333;">
  <p>
    Aqui podemos explorar padrões em cima do conjunto de dados <strong>Student Attitude and Behavior</strong>,
    disponível em <a href="https://www.kaggle.com/datasets/susanta21/student-attitude-and-behavior" target="_blank" style="color: #1a73e8;">Kaggle</a>.
    Abaixo apresentamos uma <strong>visualização de coordenadas paralelas interativa</strong> com diversas funcionalidades úteis para análise exploratória de dados multidimensionais.
  </p>

  <p>
    <strong>📊 O que são coordenadas paralelas?</strong><br>
    É uma técnica de visualização que permite representar dados com múltiplas variáveis (dimensões) simultaneamente. 
    Cada linha representa um indivíduo (ou ponto de dado) e cada eixo paralelo representa uma variável.
    Isso nos permite detectar padrões, correlações, outliers e agrupamentos entre variáveis de forma visual.
  </p>

  <p><strong>✨ Funcionalidades disponíveis:</strong></p>
  <ul>
    <li><strong>Escolha de colunas:</strong> selecione quais variáveis deseja visualizar.</li>
    <li><strong>Filtragem (Brushing):</strong> clique e arraste em um eixo para selecionar valores. Dois modos disponíveis:
      <ul>
        <li><em>Colorir selecionados</em>: destaca os pontos dentro do filtro.</li>
        <li><em>Esconder não selecionados</em>: oculta os dados fora do filtro.</li>
      </ul>
    </li>
    <li><strong>Coloração por variável:</strong> use qualquer variável quantitativa ou qualitativa para colorir as linhas.</li>
    <li><strong>Paletas de cor:</strong> escolha entre esquemas como Turbo, Viridis, Plasma e outros.</li>
    <li><strong>Interação com linhas:</strong> clique em uma linha para ver seus dados e removê-la temporariamente.</li>
    <li><strong>Restaurar visualização:</strong> botão dedicado para reexibir todos os dados removidos.</li>
    <li><strong>Reordenação de eixos:</strong> arraste os eixos com o mouse para reorganizar as variáveis.</li>
  </ul>
</div>

<h2 style="text-align: center; margin-top: 2rem; font-family: 'Inter', sans-serif;">
  Visualização de Coordenadas Paralelas Interativas
</h2>

<!-- 🎨 Layout geral: legenda | gráfico | controles -->
<div style="max-width: 1200px; margin: 0 auto; display: flex; justify-content: center; gap: 1rem; margin-top: 2rem; font-family: 'Inter', sans-serif;">
<div style="display: flex; flex-direction: column; align-items: flex-start; margin-top: 0.5rem;">
  <div style="font-size: 13px; font-weight: bold; margin-bottom: 0.5rem;">
    Legenda de Cores
  </div>
  <div bind:this={legendContainer} style="width: 60px; height: 500px;"></div>
</div>


  <!-- 📈 Gráfico com botão flutuante -->
  <div style="position: relative;">
    <div bind:this={container} style="height: 600px;"></div>

    <button 
      type="button" 
      on:click={clearFilters}
      style="position: absolute; bottom: 10px; right: 10px; background-color: #ff9800; color: white; padding: 0.5rem 1rem; border: none; border-radius: 6px; cursor: pointer;"
    >
      Remover Filtros
    </button>
  </div>

  <!-- 🎛️ Painel de controles à direita -->
  <div style="min-width: 200px; font-size: 14px;">
    
    <!-- Seletor de colunas -->
    <div class="multiselect" style="margin-bottom: 1.5rem;">
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

    <!-- Colorir por -->
    <div style="margin-bottom: 1rem;">
      <label for="color-select" style="display: block; margin-bottom: 0.3rem;"><strong>Colorir por:</strong></label>
      <select id="color-select" bind:value={colourVariable} on:change={() => { computeColourScale(); drawParallel(); drawLegend(); }}
        style="width: 180px; padding: 0.3rem; font-size: 13px;">
        {#each allDimensions as dim}
          <option value={dim}>{dim}</option>
        {/each}
      </select>
    </div>

    <!-- Paleta -->
    <div style="margin-bottom: 1rem;">
      <label for="palette-select" style="display: block; margin-bottom: 0.3rem;"><strong>Paleta:</strong></label>
      <select id="palette-select" bind:value={selectedPalette} on:change={() => { computeColourScale(); drawParallel(); drawLegend(); }}
        style="width: 180px; padding: 0.3rem; font-size: 13px;">
        <option value="Turbo">Turbo</option>
        <option value="Viridis">Viridis</option>
        <option value="Plasma">Plasma</option>
        <option value="Inferno">Inferno</option>
        <option value="Blues">Blues</option>
      </select>
    </div>

    <!-- Brushing -->
    <div>
      <label for="brush-mode-select" style="display: block; margin-bottom: 0.3rem;"><strong>Brushing:</strong></label>
      <select 
        id="brush-mode-select" 
        bind:value={brushMode}
        style="width: 180px; padding: 0.3rem; font-size: 13px;"
      >
        <option value="color">Colorir selecionados</option>
        <option value="hide">Esconder não selecionados</option>
      </select>
    </div>
  </div>
</div>

<!-- 🔁 Detalhes do ponto selecionado -->
{#if selectedDatum}
  <div style="max-width: 900px; margin: 2rem auto; padding: 0.5rem; border: 1px solid #ccc; border-radius: 6px;">
    <strong>Dados do ponto selecionado:</strong>
    <ul>
      {#each Object.entries(selectedDatum) as [key, value]}
        <li><strong>{key}:</strong> {value}</li>
      {/each}
    </ul>

    <button 
      on:click={() => {
        removedData.add(selectedDatum.id);
        selectedDatum = null;
        computeColourScale();
        drawParallel();
      }}
      style="margin-top: 0.5rem; background-color: #ff4d4f; color: white; padding: 0.4rem 0.8rem; border: none; border-radius: 5px; cursor: pointer;"
    >
      Remover este ponto
    </button>
  </div>
{/if}

<!-- ♻️ Botão de restaurar -->
<div style="text-align: center; margin-top: 1rem;">
  <button 
    on:click={restoreAllData}
    style="background-color: #4caf50; color: white; padding: 0.5rem 1rem; border: none; border-radius: 6px; cursor: pointer;"
  >
    Restaurar Todos
  </button>
</div>
