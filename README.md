[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/oHw8ptbv)

# Ideia principal: parallel-coordinates
https://syntagmatic.github.io/parallel-coordinates/

# Estudando o Estudante: Desempenho, Perfil e Estilo de Vida

🔗 **Fonte dos dados**: [Kaggle - Student Attitude and Behavior Dataset](https://www.kaggle.com/datasets/susanta21/student-attitude-and-behavior)  
🔗 **Inspiração**: [Syntagmatic - Parallel Coordinates](https://syntagmatic.github.io/parallel-coordinates/)

Trabalhamos com um conjunto de dados sobre os hábitos e perfis de estudantes. Por conter uma grande quantidade de variáveis, escolhemos usar **gráficos de coordenadas paralelas**, que permitem visualizar múltiplas variáveis simultaneamente. 

Além disso, adicionamos visualizações complementares (Radar Chart e Scatter Plot) para ampliar a capacidade de análise e comparação entre grupos.

---

## Funcionalidades Principais (Resumo)

**Gráfico de Coordenadas Paralelas:**
- Seleção de colunas
- Filtragem por brushing 
- Reordenação dos eixos
- Identificação e seleção de pontos
- Coloração por variável (quantitativa ou categórica)
- Suporte a múltiplas paletas (Turbo, Viridis, etc.)

**Outros gráficos integrados:**
- Radar Chart para médias por gênero
- Scatter Plot com suporte a boxplot e dotplot

---

## Funcionalidades dos Gráficos Interativos

Nosso dashboard combina **três visualizações principais** — **Coordenadas Paralelas**, **Radar Chart** e **Gráfico Bivariado Adaptativo** — que compartilham dados e interações. Isso permite uma análise integrada e dinâmica de múltiplas variáveis.

---

### 1️⃣ Coordenadas Paralelas

**Objetivo**: Visualizar múltiplas variáveis simultaneamente para identificar padrões, correlações e outliers.

**Funcionalidades:**
- Seleção de variáveis (eixos)
- Reordenação dos eixos via arrasto
- Brushing (filtro por eixo)
- Coloração por variável (contínua ou categórica)
- Paletas de cor: Turbo, Viridis, Plasma, Inferno, Blues
- Seleção de linha individual com clique
- Remoção temporária de pontos
- Restauração de todos os dados removidos

---

### 2️⃣ Radar Chart

**Objetivo**: Comparar o perfil médio de estudantes por gênero (masculino e feminino) em múltiplas variáveis numéricas.

**Funcionalidades**:
- Cálculo automático das médias para cada grupo (masculino e feminino)
- Eixos radiais representam variáveis numéricas selecionadas
- Polígonos coloridos representam os grupos, com áreas distintas
- Tooltip interativo para cada ponto do polígono, exibindo a média correspondente
- Clique nos vértices exibe o valor numérico diretamente no gráfico


O Radar Chart só é exibido se ao menos 3 variáveis numéricas forem selecionadas. Ele se adapta automaticamente à seleção atual feita nas coordenadas paralelas.

---

### 3️⃣ Gráfico Bivariado Adaptativo

**Objetivo**: Explorar relações entre duas variáveis específicas (quantitativas ou categóricas).

**Funcionalidades:**
- Adaptação automática do tipo de gráfico:
  - Numérica × Numérica → *scatter plot*
  - Categórica × Numérica → *boxplot*
  - Categórica × Categórica → *dotplot*
- Tooltip com estatísticas ou frequência no *dotplot*
- Seleção de pontos com clique

---

Essas visualizações interativas permitem uma análise exploratória rica e fluida, facilitando a descoberta de padrões, comparações entre grupos e identificação de comportamentos extremos entre estudantes.


---
Nosso processo de desenvolvimento foi bastante colaborativo. Distribuímos as tarefas conforme a necessidade do momento, o que significa que todos os membros da equipe contribuíram tanto com o código quanto com decisões de design. Em diversas etapas, cada um participou desde o esboço inicial de ideias até a depuração de bugs. Essa abordagem flexível permitiu que todos tivessem uma compreensão ampla do projeto e colaborassem de forma efetiva em várias frentes.

Estimamos que foram investidas aproximadamente 20 horas por pessoa. Os aspectos que mais demandaram tempo foram integrar a interatividade tanto individual quanto compartilhada entre os gráficos (principalmente a compartilhada) e no começo do projeto, alguns problemas com a versão do Svelte

Uso de IA para depuração e ferramenta de pesquisa para funcionalidades e comandos do d3 e svelte, assistente para otimizar partes do código, e ajudar na resolução de problemas específicos