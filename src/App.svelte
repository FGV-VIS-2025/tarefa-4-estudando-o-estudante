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
    <h2>📊 Student Attitude and Behavior Dataset 🎓</h2>
<p>
  This dataset contains information collected from university students through a Google form.
  It includes details such as:
</p>
<ul>
  <li>Certification courses</li>
  <li>Gender</li>
  <li>Department</li>
  <li>Height (in cm)</li>
  <li>Weight (in kg)</li>
  <li>Marks in 10th and 12th grade</li>
  <li>College marks</li>
  <li>Hobbies</li>
  <li>Daily studying time</li>
  <li>Preferred study environment</li>
  <li>Salary expectations</li>
  <li>Satisfaction with their degree</li>
  <li>Willingness to pursue a career related to their degree</li>
  <li>Social media and video usage</li>
  <li>Traveling time</li>
  <li>Stress levels</li>
  <li>Financial status</li>
</ul>
<p>
  The dataset aims to provide insights into student behavior and can be used for analysis and research purposes.
</p>
  <h2> ParallelCoordinates</h2>
    <ParallelCoordinates />
  </section>
  <Footer />
</main>

