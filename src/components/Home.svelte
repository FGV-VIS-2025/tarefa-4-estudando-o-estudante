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
  .big-btn {
    font-size: 1.1rem;
    padding: 0.9rem 2.2rem;
    border-radius: 10px;
    background: #66a1ee;
    color: #fff;
    border: none;
    cursor: pointer;
    transition: transform .15s ease;
  }
  .big-btn:hover {
    transform: scale(1.04);
  }

  /* Estilo para a descrição do dataset */
  .dataset-description {
    max-width: 800px;
    margin: 2rem auto;
    padding: 1.5rem;
    background: #f7f7f7;
    border-radius: 0.5rem;
    color: #333;
  }
  .dataset-description h2 {
    font-size: 1.75rem;
    margin-bottom: 1rem;
  }
  .dataset-description ul {
    list-style: none;
    padding: 0;
  }
  .dataset-description li {
    margin-bottom: 0.75rem;
    line-height: 1.4;
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

<!-- componente -->
<div class="faixa">
  <div class="faixa-content">
    <button class="big-btn" on:click={onStart}>
      Prosseguir para visualização
    </button>
  </div>
</div>

<!-- Descrição do Dataset -->
<section class="dataset-description">
  <h2>Descrição do Nosso Dataset</h2>
  <p>Conseguimos uma fonte de dados com as seguintes features:</p>
  <ul>
    <li>✅ <strong>Certification Course:</strong> Indica se o estudante concluiu algum curso de certificação.</li>
    <li>🚻 <strong>Gender:</strong> Gênero do estudante.</li>
    <li>📚 <strong>Department:</strong> Departamento ou área de estudo do estudante.</li>
    <li>📏 <strong>Height (CM):</strong> Altura do estudante em centímetros.</li>
    <li>⚖️ <strong>Weight (KG):</strong> Peso do estudante em quilogramas.</li>
    <li>📝 <strong>10th Mark:</strong> Nota obtida pelo estudante na 10ª série.</li>
    <li>🎓 <strong>12th Mark:</strong> Nota obtida pelo estudante na 12ª série.</li>
    <li>🏫 <strong>College Mark:</strong> Nota obtida pelo estudante na universidade.</li>
    <li>🎨 <strong>Hobbies:</strong> Hobbies ou interesses do estudante.</li>
    <li>⏰ <strong>Daily Studying Time:</strong> Tempo diário de estudo do estudante.</li>
    <li>📚🌳 <strong>Prefer to Study in:</strong> Ambiente ou local preferido para estudo pelo estudante.</li>
    <li>💰 <strong>Salary Expectation:</strong> Expectativa salarial futura do estudante.</li>
    <li>👍👎 <strong>Do you like your degree?:</strong> Opinião do estudante sobre gostar ou não do curso.</li>
    <li>🏢 <strong>Willingness to pursue a career based on their degree:</strong> Disposição do estudante em seguir carreira relacionada ao curso.</li>
    <li>📱📺 <strong>Social Media & Video:</strong> Engajamento do estudante com mídias sociais e plataformas de vídeo.</li>
    <li>🚗 <strong>Traveling Time:</strong> Tempo de deslocamento até a instituição de ensino.</li>
    <li>😓 <strong>Stress Level:</strong> Nível de estresse percebido pelo estudante.</li>
    <li>💵 <strong>Financial Status:</strong> Status financeiro ou background econômico do estudante.</li>
    <li>💼 <strong>Part-time Job:</strong> Indica se o estudante possui um emprego de meio período.</li>
  </ul>
</section>
