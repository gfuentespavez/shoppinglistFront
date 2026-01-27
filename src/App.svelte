<script>
  import axios from 'axios';
  import { API_URL } from './lib/config.js';
  import ListSelector from './lib/ListSelector.svelte';
  import CategorySelector from './lib/CategorySelector.svelte';
  import ItemsView from './lib/ItemsView.svelte';
  import UploadModal from './lib/UploadModal.svelte';

  let view = 'lists';
  let selectedList = null;
  let selectedCategory = null;
  let categories = [];
  let items = [];
  let allItems = [];
  let isUploadModalOpen = false;
  let listSelectorComponent;

  async function handleSelectList(list) {
    selectedList = list;

    try {
      const response = await axios.get(`${API_URL}/api/lists/${list.id}/items`);
      allItems = response.data;

      categories = [...new Set(allItems.map(item => item.category))];

      view = 'categories';
    } catch (error) {
      console.error('Error cargando items:', error);
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
</script>

<main>
  {#if view === 'lists'}
    <ListSelector bind:this={listSelectorComponent} onSelectList={handleSelectList} />
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
            onBack={backToCategories}
    />
  {/if}

  <button class="fab" on:click={openUploadModal}>
    +
  </button>

  <UploadModal
          isOpen={isUploadModalOpen}
          onClose={closeUploadModal}
          onSuccess={handleUploadSuccess}
  />
</main>

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
</style>