<script>
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { API_URL } from './config.js';

  export let onSelectList;

  let lists = [];
  let loading = true;
  let editingId = null;
  let editingName = '';
  let listStats = {};
  let deletingId = null;

  function normalizeText(text) {
    if (!text) return '';
    return text
            .toLowerCase()
            .split(' ')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ');
  }

  async function loadLists() {
    loading = true;
    try {
      const response = await axios.get(`${API_URL}/api/lists`);
      lists = response.data;

      // Cargar estadísticas para cada lista
      for (const list of lists) {
        await loadStats(list.id);
      }
    } catch (error) {
      console.error('Error cargando listas:', error);
    }
    loading = false;
  }

  async function loadStats(listId) {
    try {
      const response = await axios.get(`${API_URL}/api/lists/${listId}/stats`);
      listStats[listId] = response.data;
      listStats = listStats; // trigger reactivity
    } catch (error) {
      console.error('Error cargando estadísticas:', error);
    }
  }

  function selectList(list) {
    if (editingId !== list.id) {
      onSelectList(list);
    }
  }

  function startEditing(list, event) {
    event.stopPropagation();
    editingId = list.id;
    editingName = list.name;
  }

  async function saveEdit(list) {
    if (editingName.trim() === '') {
      cancelEdit();
      return;
    }

    try {
      await axios.patch(`${API_URL}/api/lists/${list.id}`, {
        name: editingName
      });
      list.name = editingName;
      lists = lists;
      editingId = null;
    } catch (error) {
      console.error('Error actualizando nombre:', error);
    }
  }

  function cancelEdit() {
    editingId = null;
    editingName = '';
  }

  function handleKeydown(event, list) {
    if (event.key === 'Enter') {
      saveEdit(list);
    } else if (event.key === 'Escape') {
      cancelEdit();
    }
  }

  async function deleteList(list, event) {
    event.stopPropagation();

    if (!confirm(`¿Seguro que quieres eliminar "${normalizeText(list.name)}"?`)) {
      return;
    }

    deletingId = list.id;
    try {
      await axios.delete(`${API_URL}/api/lists/${list.id}`);
      lists = lists.filter(l => l.id !== list.id);
      delete listStats[list.id];
    } catch (error) {
      console.error('Error eliminando lista:', error);
      alert('Error al eliminar la lista');
    }
    deletingId = null;
  }

  function getProgress(listId) {
    const stats = listStats[listId];
    if (!stats || stats.total === 0) return 0;
    return Math.round((stats.checked / stats.total) * 100);
  }

  onMount(loadLists);

  export function refresh() {
    loadLists();
  }
</script>

<div class="container">
  <header>
    <h1>Listas de<br/>Materiales</h1>
    <p class="subtitle">Selecciona tu nivel</p>
  </header>

  {#if loading}
    <p class="loading">Cargando...</p>
  {:else if lists.length === 0}
    <p class="empty">No hay listas disponibles</p>
  {:else}
    <div class="grid">
      {#each lists as list}
        <div class="list-card" on:click={() => selectList(list)} class:deleting={deletingId === list.id}>
          {#if editingId === list.id}
            <input
                    type="text"
                    class="edit-input"
                    bind:value={editingName}
                    on:keydown={(e) => handleKeydown(e, list)}
                    on:blur={() => saveEdit(list)}
                    autofocus
            />
          {:else}
            <div class="card-content">
              <div class="card-header">
                <span class="list-name">{normalizeText(list.name)}</span>
                <div class="actions">
                  <button class="edit-btn" on:click={(e) => startEditing(list, e)}>
                    ✎
                  </button>
                  <button class="delete-btn" on:click={(e) => deleteList(list, e)}>
                    🗑️
                  </button>
                  <span class="arrow">→</span>
                </div>
              </div>

              {#if listStats[list.id]}
                <div class="stats">
                  <div class="progress-bar">
                    <div
                            class="progress-fill"
                            style="width: {getProgress(list.id)}%"
                    ></div>
                  </div>
                  <div class="stats-text">
                    <span class="checked">{listStats[list.id].checked} comprados</span>
                    <span class="unchecked">{listStats[list.id].unchecked} pendientes</span>
                  </div>
                </div>
              {/if}
            </div>
          {/if}
        </div>
      {/each}
    </div>
  {/if}
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

  h1 {
    font-size: 42px;
    font-weight: 700;
    line-height: 1.1;
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

  .list-card {
    background: #FFFFFF;
    border: none;
    padding: 24px;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  }

  .list-card:active {
    transform: scale(0.98);
    box-shadow: 0 2px 8px rgba(91, 159, 216, 0.15);
  }

  .list-card.deleting {
    opacity: 0.5;
    pointer-events: none;
  }

  .card-content {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .list-name {
    font-size: 20px;
    font-weight: 600;
    color: #1E293B;
    flex: 1;
  }

  .actions {
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .edit-btn, .delete-btn {
    background: #F1F5F9;
    border: none;
    width: 36px;
    height: 36px;
    border-radius: 10px;
    font-size: 18px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
  }

  .edit-btn {
    color: #5B9FD8;
  }

  .delete-btn {
    color: #EF4444;
  }

  .edit-btn:active, .delete-btn:active {
    background: #E2E8F0;
    transform: scale(0.95);
  }

  .arrow {
    font-size: 24px;
    color: #5B9FD8;
    font-weight: 400;
  }

  .stats {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .progress-bar {
    width: 100%;
    height: 8px;
    background: #F1F5F9;
    border-radius: 8px;
    overflow: hidden;
  }

  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #5B9FD8 0%, #4A8BC2 100%);
    transition: width 0.3s ease;
    border-radius: 8px;
  }

  .stats-text {
    display: flex;
    justify-content: space-between;
    font-size: 14px;
  }

  .checked {
    color: #5B9FD8;
    font-weight: 600;
  }

  .unchecked {
    color: #64748B;
    font-weight: 500;
  }

  .edit-input {
    width: 100%;
    background: #F1F5F9;
    border: 2px solid #5B9FD8;
    padding: 12px 16px;
    border-radius: 12px;
    font-size: 18px;
    font-weight: 600;
    color: #1E293B;
    font-family: inherit;
    outline: none;
  }

  .loading, .empty {
    text-align: center;
    color: #64748B;
    padding: 60px 20px;
    font-size: 16px;
  }