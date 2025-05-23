<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let container;

  onMount(async () => {
    // Carrega os dados do CSV
    const data = await d3.csv('/data/student_attitute.csv', d => ({
      CertificationCourse: d['Certification Course'],
      Height: +d['Height(CM)'],
      Weight: +d['Weight(KG)'],
      Mark10: +d['10th Mark'],
      Mark12: +d['12th Mark'],
      CollegeMark: +d['college mark'],
      Salary: +d['salary expectation']
    }));

    // Dimensões a serem usadas no parallel coordinates
    const dimensions = ['Height', 'Weight', 'Mark10', 'Mark12', 'CollegeMark', 'Salary'];

    const margin = { top: 30, right: 10, bottom: 10, left: 10 };
    const width = 800 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Escala horizontal para cada dimensão
    const x = d3.scalePoint()
      .domain(dimensions)
      .range([0, width]);

    // Escalas verticais para cada dimensão
    const y = {};
    dimensions.forEach(dim => {
      y[dim] = d3.scaleLinear()
        .domain(d3.extent(data, d => d[dim]))
        .range([height, 0]);
    });

    // Define a área do SVG
    const svg = d3.select(container)
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom)
      .append('g')
      .attr('transform', `translate(${margin.left},${margin.top})`);

    // Função que gera o path de cada linha
    function path(d) {
      return d3.line()(dimensions.map(p => [x(p), y[p](d[p])]));
    }

    const brushes = {};

    // Função para coletar as extensões de cada brush
    function brushExtents() {
      let extents = {};
      dimensions.forEach(d => {
        const brush = brushes[d];
        if (brush && !brush.empty()) {
          extents[d] = brush.extent();
        }
      });
      return extents;
    }

    // Função que filtra as linhas com base nas extensões dos brushes
    function brushed() {
      const extents = brushExtents();
      svg.select('.foreground')
        .selectAll('path')
        .style('display', d => {
          return dimensions.every(dim => {
            const extent = extents[dim];
            if (extent) {
              return d[dim] >= extent[0] && d[dim] <= extent[1];
            }
            return true;
          }) ? null : 'none';
        });
    }

    // Linhas de fundo (background)
    svg.append('g')
      .attr('class', 'background')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', path);

    // Linhas frontais (foreground)
    svg.append('g')
      .attr('class', 'foreground')
      .selectAll('path')
      .data(data)
      .enter().append('path')
      .attr('d', path)
      .attr('stroke', 'steelblue')
      .attr('stroke-width', 1)
      .attr('fill', 'none')
      .attr('stroke-opacity', 0.6);

    // Eixos e brushes para cada dimensão
    const g = svg.selectAll('.dimension')
      .data(dimensions)
      .enter().append('g')
      .attr('class', 'dimension')
      .attr('transform', d => `translate(${x(d)})`);

    // Adiciona eixo
    g.append('g')
      .each(function(d) { d3.select(this).call(d3.axisLeft(y[d])); })
      .append('text')
      .style('text-anchor', 'middle')
      .attr('y', -9)
      .text(d => d);

    // Adiciona brushes
    g.append('g')
      .attr('class', 'brush')
      .each(function(d) {
        const brush = d3.brushY()
          .extent([[-10, 0], [10, height]])
          .on('brush', brushed)
          .on('end', brushed);
        
        d3.select(this).call(brush);
        brushes[d] = brush;
      });

  });
</script>

<div bind:this={container}></div>

<style>
  :global(.background path) {
    stroke: #ccc;
  }

  :global(.foreground path) {
    stroke: steelblue;
  }

  :global(.dimension path) {
    fill: none;
    stroke: #000;
    shape-rendering: crispEdges;
  }

  :global(.dimension text) {
    font-size: 12px;
    text-anchor: middle;
  }

  div {
    margin: 1rem 0;
    overflow-x: auto;
  }
</style>