<script>
  import Home from './components/Home.svelte';
  import ParallelCoordinates from './ParallelCoordinates.svelte';

  let showHome = true;
</script>

<style>
  nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    background: #1e3a8a;
    color: white;
  }
  button.logo,
  button.link {
    background: none;
    border: none;
    color: inherit;
    font: inherit;
    cursor: pointer;
    padding: 0.5rem 1rem;
  }
  button.logo:hover,
  button.link:hover {
    background: rgba(255, 255, 255, 0.1);
  }
</style>

<nav>
  <button type="button" class="logo" on:click={() => showHome = true}>
    Comportamento Estudantil
  </button>
  <button type="button" class="link" on:click={() => showHome = false}>
    Visualização
  </button>
</nav>

{#if showHome}
  <Home onStart={() => showHome = false} />
{:else}
  <ParallelCoordinates />
{/if}
