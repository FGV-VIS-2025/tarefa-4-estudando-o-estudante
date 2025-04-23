<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let container;

    onMount(async () => {
    const data = await d3.csv(import.meta.env.BASE_URL + 'data/student_attitude.csv');
      const dimensions = [
        '10th Mark',
        '12th Mark',
        'college mark',
        'Height(CM)',
        'Weight(KG)',
        'daily studing time',
        'salary expectation',
        'Stress Level'
      ];
  
      const margin = { top: 30, right: 10, bottom: 10, left: 10 };
      const width = 900 - margin.left - margin.right;
      const height = 500 - margin.top - margin.bottom;
  
      const svg = d3.select(container)
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);
  
      const y = {};
      for (const dim of dimensions) {
        y[dim] = d3.scaleLinear()
          .domain(d3.extent(data, d => Number(d[dim]) || 0))
          .range([height, 0]);
      }
  
      const x = d3.scalePoint()
        .range([0, width])
        .padding(1)
        .domain(dimensions);
  
      function path(d) {
        return d3.line()(dimensions.map(p => [x(p), y[p](d[p])]));
      }
  
      svg.selectAll('path')
        .data(data)
        .enter().append('path')
        .attr('d', path)
        .attr('fill', 'none')
        .attr('stroke', '#4682b4')
        .attr('stroke-width', 1)
        .attr('opacity', 0.3);
  
      svg.selectAll('.axis')
        .data(dimensions).enter()
        .append('g')
        .attr('class', 'axis')
        .attr('transform', d => `translate(${x(d)})`)
        .each(function(d) {
          d3.select(this).call(d3.axisLeft().scale(y[d]));
        })
        .append('text')
        .style('text-anchor', 'middle')
        .attr('y', -9)
        .text(d => d)
        .style('fill', 'black');
    });
  </script>
  
  <div bind:this={container} style="height: 600px;"></div>
  
