<script>
  import Header from './components/Header.svelte';
  import Viz1 from './components/Viz1.svelte';
  import Footer from './components/Footer.svelte';
  import ParallelCoordinates from './ParallelCoordinates.svelte';
  import { onMount } from 'svelte';
  onMount(() => console.log('🧪 Teste montou'));

</script>

<main>
  <Header />
  <section class="content">
    <h2>Análise Visual</h2>
    <p>Explore a relação entre desempenho acadêmico e hábitos de estudo dos alunos com visualizações interativas.</p>
    <p> Viz 1 Test </p>
    <Viz1 />
    <ParallelCoordinates />
  </section>
  <Footer />
</main>

