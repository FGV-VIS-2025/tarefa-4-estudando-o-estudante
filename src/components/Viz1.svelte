<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';

  let data = [];

  onMount(async () => {
    try {
      // faz GET em /student_data.csv e já parseia para array de objetos
      data = await csv('/student_data.csv');
      console.log('🎉 Dados carregados:', data);
    } catch (e) {
      console.error('❌ Erro ao carregar CSV:', e);
    }
  });
</script>

{#if data.length === 0}
  <p>Carregando dados…</p>
{:else}
  <p>Foram carregados {data.length} registros.</p>
  <!-- aqui já dá pra começar a construir sua visualização -->
{/if}
