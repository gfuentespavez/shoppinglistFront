<script>
  import axios from 'axios';
  import { API_URL } from './config.js';

  export let items;
  export let category;
  export let listName;
  export let onBack;
  export let isConsolidated = false; // Nueva prop
  export let onToggleConsolidated = null; // Nueva prop

  function normalizeText(text) {
    if (!text) return '';
    return text
            .toLowerCase()
            .split(' ')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
  }

  async function toggleItem(item) {
    // Si es lista consolidada, usar la función especial
    if (isConsolidated && onToggleConsolidated) {
      await onToggleConsolidated(item);
      return;
    }

    // Toggle normal para listas regulares
    try {
      await axios.patch(`${API_URL}/api/items/${item.id}`, {
        checked: !item.checked
      });
      item.checked = !item.checked;
      items = items;
    } catch (error) {
      console.error('Error actualizando item:', error);
    }
  }
</script>

<div class="container">
  <header>
    <button class="back-btn" on:click={onBack}>
      <span class="arrow">←</span>
    </button>
    <h1>{normalizeText(category)}</h1>
    <p class="subtitle">{normalizeText(listName)}</p>
    {#if isConsolidated}
      <p class="consolidated-note">💡 Al marcar un item, se marca en todas las listas</p>
    {/if}
  </header>

  <div class="items-list">
    {#each items as item}
      <label class="item" class:checked={item.checked}>
        <input
                type="checkbox"
                checked={item.checked}
                on:change={() => toggleItem(item)}
        />
        <div class="item-content">
          <span class="quantity">{item.quantity}</span>
          <span class="description">{normalizeText(item.description)}</span>
        </div>
      </label>
    {/each}
  </div>
</div>

<style>
  .container {
    min-height: 100vh;
    padding: 40px 24px 60px;
    max-width: 480px;
    margin: 0 auto;
  }

  header {
    margin-bottom: 32px;
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
    font-size: 32px;
    font-weight: 700;
    line-height: 1.2;
    margin-bottom: 8px;
    color: #1E293B;
    letter-spacing: -0.02em;
  }

  .subtitle {
    font-size: 16px;
    font-weight: 400;
    color: #64748B;
  }

  .consolidated-note {
    font-size: 14px;
    color: #5B9FD8;
    margin-top: 12px;
    padding: 12px 16px;
    background: #EFF6FF;
    border-radius: 12px;
    font-weight: 500;
  }

  .items-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .item {
    display: flex;
    align-items: flex-start;
    gap: 16px;
    padding: 24px 20px;
    background: #FFFFFF;
    border-radius: 16px;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  }

  .item:active {
    transform: scale(0.98);
  }

  .item.checked {
    opacity: 0.4;
  }

  .item.checked .description {
    text-decoration: line-through;
  }

  input[type="checkbox"] {
    width: 28px;
    height: 28px;
    min-width: 28px;
    cursor: pointer;
    margin-top: 0;
    accent-color: #5B9FD8;
    border-radius: 8px;
  }

  .item-content {
    display: flex;
    gap: 12px;
    flex: 1;
  }

  .quantity {
    font-weight: 700;
    color: #5B9FD8;
    min-width: 28px;
    font-size: 18px;
  }

  .description {
    flex: 1;
    line-height: 1.6;
    color: #1E293B;
    font-size: 16px;
    font-weight: 500;
  }
</style>