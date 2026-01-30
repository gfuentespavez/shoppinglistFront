<script>
  import { onMount } from 'svelte';
  import axios from 'axios';
  import { API_URL } from './lib/config.js';
  import WorkspaceLogin from './lib/WorkspaceLogin.svelte';
  import ListSelector from './lib/ListSelector.svelte';
  import CategorySelector from './lib/CategorySelector.svelte';
  import ItemsView from './lib/ItemsView.svelte';
  import UploadModal from './lib/UploadModal.svelte';

  let workspaceCode = null;
  let view = 'lists';
  let selectedList = null;
  let selectedCategory = null;
  let categories = [];
  let items = [];
  let allItems = [];
  let isUploadModalOpen = false;
  let listSelectorComponent;
  let allLists = []; // Para generar la lista consolidada

  onMount(() => {
    // Verificar si ya hay un workspace guardado
    const savedWorkspace = localStorage.getItem('workspaceCode');
    if (savedWorkspace) {
      workspaceCode = savedWorkspace;
    }
  });

  function handleLogin(code) {
    workspaceCode = code;
  }

  function handleLogout() {
    localStorage.removeItem('workspaceCode');
    workspaceCode = null;
    view = 'lists';
    selectedList = null;
    selectedCategory = null;
  }

  // Función para normalizar nombres de items
  function normalizeItemName(name) {
    return name.trim().toLowerCase()
            .replace(/s$/, '') // quitar plural simple
            .replace(/es$/, ''); // quitar plural -es
  }

  // Función para extraer cantidad del texto
  function extractQuantity(text) {
    const match = text.match(/^(\d+)\s+/);
    return match ? parseInt(match[1]) : 1;
  }

  // Función para generar la lista consolidada
  async function generateConsolidatedList() {
    if (!allLists || allLists.length === 0) return null;

    // Objeto para agrupar items por nombre normalizado
    const consolidatedItems = {};

    // Recorrer todas las listas y sus items
    for (const list of allLists) {
      try {
        const response = await axios.get(`${API_URL}/api/lists/${list.id}/items`);
        const listItems = response.data;

        for (const item of listItems) {
          const normalizedName = normalizeItemName(item.name);
          const quantity = extractQuantity(item.name);

          if (!consolidatedItems[normalizedName]) {
            consolidatedItems[normalizedName] = {
              originalName: item.name.replace(/^\d+\s+/, ''), // nombre sin cantidad
              totalQuantity: 0,
              category: item.category,
              sourceItems: [] // referencias a items originales
            };
          }

          consolidatedItems[normalizedName].totalQuantity += quantity;
          consolidatedItems[normalizedName].sourceItems.push({
            listId: list.id,
            itemId: item.id,
            checked: item.checked
          });
        }
      } catch (error) {
        console.error(`Error cargando items de lista ${list.id}:`, error);
      }
    }

    // Convertir a array de items
    const consolidatedItemsArray = Object.entries(consolidatedItems).map(([key, value]) => ({
      id: `consolidated-${key}`,
      name: `${value.totalQuantity} ${value.originalName}`,
      category: value.category,
      checked: value.sourceItems.every(si => si.checked), // checked solo si TODOS están checked
      sourceItems: value.sourceItems
    }));

    return {
      id: 'CONSOLIDATED',
      name: '📋 Lista Consolidada',
      items: consolidatedItemsArray
    };
  }

  async function handleSelectList(list) {
    selectedList = list;

    // Si es la lista consolidada, generar sus items dinámicamente
    if (list.id === 'CONSOLIDATED') {
      const consolidatedList = await generateConsolidatedList();
      if (consolidatedList) {
        allItems = consolidatedList.items;
        categories = [...new Set(allItems.map(item => item.category))];
        view = 'categories';
      }
    } else {
      // Lista normal
      try {
        const response = await axios.get(`${API_URL}/api/lists/${list.id}/items`);
        allItems = response.data;
        categories = [...new Set(allItems.map(item => item.category))];
        view = 'categories';
      } catch (error) {
        console.error('Error cargando items:', error);
      }
    }
  }

  function handleSelectCategory(category) {
    selectedCategory = category;
    items = allItems.filter(item => item.category === category);
    view = 'items';
  }

  function backToLists() {
    view = 'lists';
    selectedList = null;
    selectedCategory = null;
  }

  function backToCategories() {
    view = 'categories';
    selectedCategory = null;
  }

  function openUploadModal() {
    isUploadModalOpen = true;
  }

  function closeUploadModal() {
    isUploadModalOpen = false;
  }

  function handleUploadSuccess() {
    if (listSelectorComponent) {
      listSelectorComponent.refresh();
    }
  }

  // Función para manejar el toggle de items consolidados
  async function handleToggleConsolidatedItem(item) {
    if (!item.sourceItems) return;

    const newCheckedState = !item.checked;

    // Actualizar todos los items originales
    for (const sourceItem of item.sourceItems) {
      try {
        await axios.put(`${API_URL}/api/items/${sourceItem.itemId}`, {
          checked: newCheckedState
        });
      } catch (error) {
        console.error(`Error actualizando item ${sourceItem.itemId}:`, error);
      }
    }

    // Actualizar el estado local
    item.checked = newCheckedState;
    items = items; // trigger reactivity
  }

  // Exponer función para que ItemsView pueda llamarla
  function handleItemToggle(item) {
    if (selectedList?.id === 'CONSOLIDATED') {
      return handleToggleConsolidatedItem(item);
    }
    // Si no es consolidada, ItemsView maneja el toggle normal
    return null;
  }
</script>

{#if !workspaceCode}
  <WorkspaceLogin onLogin={handleLogin} />
{:else}
  <main>
    {#if view === 'lists'}
      <ListSelector
              bind:this={listSelectorComponent}
              bind:allLists={allLists}
              onSelectList={handleSelectList}
              {workspaceCode}
      />
    {:else if view === 'categories'}
      <CategorySelector
              categories={categories}
              listName={selectedList.name}
              onSelectCategory={handleSelectCategory}
              onBack={backToLists}
      />
    {:else if view === 'items'}
      <ItemsView
              items={items}
              category={selectedCategory}
              listName={selectedList.name}
              isConsolidated={selectedList?.id === 'CONSOLIDATED'}
              onToggleConsolidated={handleItemToggle}
              onBack={backToCategories}
      />
    {/if}

    <button class="fab" on:click={openUploadModal}>
      +
    </button>

    <button class="logout-btn" on:click={handleLogout}>
      Cambiar código
    </button>

    <UploadModal
            isOpen={isUploadModalOpen}
            onClose={closeUploadModal}
            onSuccess={handleUploadSuccess}
            {workspaceCode}
    />
  </main>
{/if}

<style>
  main {
    min-height: 100vh;
    position: relative;
  }

  .fab {
    position: fixed;
    bottom: 32px;
    right: 32px;
    width: 64px;
    height: 64px;
    border-radius: 50%;
    background: #5B9FD8;
    color: white;
    border: none;
    font-size: 32px;
    font-weight: 300;
    cursor: pointer;
    box-shadow: 0 8px 24px rgba(91, 159, 216, 0.4);
    transition: all 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
  }

  .fab:active {
    transform: scale(0.92);
  }

  .fab:hover {
    box-shadow: 0 12px 32px rgba(91, 159, 216, 0.5);
  }

  .logout-btn {
    position: fixed;
    top: 24px;
    right: 24px;
    padding: 8px 16px;
    background: white;
    color: #64748B;
    border: 1px solid #E2E8F0;
    border-radius: 8px;
    font-size: 13px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
    z-index: 100;
  }

  .logout-btn:active {
    transform: scale(0.95);
    background: #F1F5F9;
  }
</style>