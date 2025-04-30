<script>
    export let currentPage = "home";
  </script>
  
  <nav class="navbar">
    <div class="navbar-container">
      <div class="logo">
        <span>AcademicViz</span>
      </div>
      
      <div class="nav-links">
        <a href="#" class:active={currentPage === "home"} on:click|preventDefault={() => currentPage = "home"}>
          Home
        </a>
        <a href="#" class:active={currentPage === "analysis"} on:click|preventDefault={() => currentPage = "analysis"}>
          Análise Interativa
        </a>
        <a href="#" class:active={currentPage === "visualizations"} on:click|preventDefault={() => currentPage = "visualizations"}>
          Visualizações
        </a>
        
        <a href="https://github.com/seu-usuario/academic-viz" target="_blank" class="github-link">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
          </svg>
          GitHub
        </a>
      </div>
    </div>
  </nav>
  
  <style>
    .navbar {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      width: 100%;
      background: #2c3e50;
      color: white;
      z-index: 1000;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
  
    .navbar-container {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 1200px;
      margin: 0 auto;
      padding: 1rem 2rem;
    }
  
    .logo span {
      font-size: 1.5rem;
      font-weight: 700;
      color: #fff;
    }
  
    .nav-links {
      display: flex;
      gap: 1.5rem;
      align-items: center;
    }
  
    .nav-links a {
      color: rgba(255,255,255,0.8);
      text-decoration: none;
      padding: 0.5rem 0;
      position: relative;
      transition: color 0.3s ease;
      white-space: nowrap;
    }
  
    .nav-links a.active {
      color: white;
      font-weight: 500;
    }
  
    .nav-links a.active::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 2px;
      background: #42b983;
    }
  
    .nav-links a:hover {
      color: white;
    }
  
    .github-link {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      margin-left: 1rem;
      padding: 0.5rem 1rem;
      background: rgba(255,255,255,0.1);
      border-radius: 4px;
      transition: background 0.3s ease;
    }
  
    .github-link:hover {
      background: rgba(255,255,255,0.2);
    }
  </style>