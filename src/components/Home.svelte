<script>
  // Callback para iniciar a visualização de Parallel Coordinates
  export let onStart = () => {};
</script>

<style>
  .hero {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 80vh;
    background: url('/assets/library.jpg') center/cover no-repeat;
    color: black;
    text-shadow: none;
  }
  .hero-content {
    max-width: 600px;
    text-align: center;
    padding: 2rem;
    background: none;
    border-radius: 0.5rem;
  }
  h1 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
  }
  p {
    font-size: 1.125rem;
    line-height: 1.6;
    margin-bottom: 1rem;
  }
  .faixa {
    background-color: #007bff;
    padding: 2rem 0;
  }
  .faixa-content {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 6rem;
  }
  .faixa-content img {
    width: 150px;
    height: auto;
    border-radius: 0.5rem;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    cursor: default;
  }
  /* Tornar a primeira imagem interativa */
  .faixa-content img:first-child {
    cursor: pointer;
  }
</style>

<div class="hero">
  <div class="hero-content">
    <h1>O estudo é uma característica de extrema importância na humanidade</h1>
    <p>
      Sem desenvolvermos essas habilidades, muitos dos avanços científicos, como a lei da gravitação universal de Newton (física), o cálculo diferencial e integral de Leibniz e Newton (matemática), a descoberta da estrutura do DNA (biologia), o desenvolvimento dos transistores (engenharia eletrônica) e a criação da programação de alto nível (ciência da computação), entre outros, não teriam sido possíveis.
    </p>
    <p>
      Aqui desenvolvemos uma plataforma onde você pode interagir com conjuntos de dados relacionados, fazer análises e, quem sabe, descobrir comportamentos interessantes a serem explorados.
    </p>
  </div>
</div>

<div class="faixa">
  <div class="faixa-content">
    <button on:click={onStart}> Prosseguir para visualização </button>
  </div>
</div>
