<script>
  import { API_URL } from './config.js';

  export let categories;
  export let onSelectCategory;
  export let onBack;
  export let listName;
  
  function normalizeText(text) {
    if (!text) return '';
    return text
      .toLowerCase()
      .split(' ')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');
  }
</script>

<div class="container">
  <header>
    <button class="back-btn" on:click={onBack}>
      <span class="arrow">←</span>
    </button>
    <h1>{normalizeText(listName)}</h1>
    <p class="subtitle">Selecciona una categoría</p>
  </header>
  
  <div class="grid">
    {#each categories as category}
      <button class="category-card" on:click={() => onSelectCategory(category)}>
        <span class="category-name">{normalizeText(category)}</span>
        <span class="arrow-right">→</span>
      </button>
    {/each}
  </div>
</div>

<style>
  .container {
    min-height: 100vh;
    padding: 40px 24px;
    max-width: 480px;
    margin: 0 auto;
  }
  
  header {
    margin-bottom: 48px;
  }
  
  .back-btn {
    background: transparent;
    border: none;
    padding: 8px 0;
    margin-bottom: 24px;
    cursor: pointer;
    display: flex;
    align-items: center;
  }
  
  .arrow {
    font-size: 28px;
    color: #5B9FD8;
    font-weight: 400;
  }
  
  h1 {
    font-size: 36px;
    font-weight: 700;
    line-height: 1.2;
    margin-bottom: 12px;
    color: #1E293B;
    letter-spacing: -0.02em;
  }
  
  .subtitle {
    font-size: 18px;
    font-weight: 400;
    color: #64748B;
  }
  
  .grid {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }
  
  .category-card {
    background: #FFFFFF;
    border: none;
    padding: 32px 24px;
    border-radius: 20px;
    font-size: 18px;
    font-weight: 600;
    color: #1E293B;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
    text-align: left;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .category-card:active {
    transform: scale(0.98);
    box-shadow: 0 2px 8px rgba(91, 159, 216, 0.15);
  }
  
  .category-name {
    flex: 1;
  }
  
  .arrow-right {
    font-size: 24px;
    color: #5B9FD8;
    font-weight: 400;
  }
</style>
