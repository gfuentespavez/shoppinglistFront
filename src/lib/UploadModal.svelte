<script>
  import axios from 'axios';
  import { API_URL } from './config.js';

  export let isOpen = false;
  export let onClose;
  export let onSuccess;
  export let workspaceCode;

  let files = [];
  let uploading = false;
  let message = '';
  let uploadedCount = 0;

  async function handleUpload() {
    if (files.length === 0) {
      message = 'Por favor selecciona al menos un PDF';
      return;
    }

    uploading = true;
    message = '';
    uploadedCount = 0;

    for (const file of files) {
      try {
        const formData = new FormData();
        formData.append('file', file);
        formData.append('workspaceCode', workspaceCode);

        await axios.post(`${API_URL}/api/upload`, formData, {
          headers: { 'Content-Type': 'multipart/form-data' }
        });

        uploadedCount++;
      } catch (error) {
        message = `Error al subir ${file.name}`;
        console.error(error);
        uploading = false;
        return;
      }
    }

    message = `✓ ${uploadedCount} archivo(s) subido(s) correctamente`;
    uploading = false;
    files = [];

    setTimeout(() => {
      if (onSuccess) onSuccess();
      handleClose();
    }, 1500);
  }

  function handleFileChange(event) {
    files = Array.from(event.target.files);
    message = '';
  }

  function handleClose() {
    if (!uploading) {
      files = [];
      message = '';
      onClose();
    }
  }

  function handleBackdropClick(e) {
    if (e.target === e.currentTarget) {
      handleClose();
    }
  }
</script>

{#if isOpen}
  <div class="backdrop" on:click={handleBackdropClick}>
    <div class="modal">
      <div class="modal-header">
        <h2>Subir lista</h2>
        <button class="close-btn" on:click={handleClose} disabled={uploading}>×</button>
      </div>

      <div class="modal-body">
        <label class="file-input-label">
          <input
                  type="file"
                  accept=".pdf"
                  multiple
                  on:change={handleFileChange}
                  disabled={uploading}
                  class="file-input"
          />
          <div class="file-input-button">
            {files.length > 0 ? `${files.length} archivo(s) seleccionado(s)` : 'Seleccionar PDFs'}
          </div>
        </label>

        {#if message}
          <p class="message" class:success={uploadedCount > 0 && !uploading}>{message}</p>
        {/if}

        <button
                class="upload-btn"
                on:click={handleUpload}
                disabled={uploading || files.length === 0}
        >
          {uploading ? 'Subiendo...' : 'Subir'}
        </button>
      </div>
    </div>
  </div>
{/if}

<style>
  .backdrop {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
    padding: 20px;
  }

  .modal {
    background: white;
    border-radius: 24px;
    max-width: 440px;
    width: 100%;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    animation: slideUp 0.3s ease;
  }

  @keyframes slideUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 24px 24px 16px;
    border-bottom: 1px solid #E2E8F0;
  }

  h2 {
    font-size: 24px;
    font-weight: 700;
    color: #1E293B;
  }

  .close-btn {
    background: transparent;
    border: none;
    font-size: 36px;
    color: #64748B;
    cursor: pointer;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 12px;
    transition: all 0.2s;
  }

  .close-btn:hover {
    background: #F1F5F9;
  }

  .close-btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .modal-body {
    padding: 24px;
  }

  .file-input {
    display: none;
  }

  .file-input-label {
    display: block;
    margin-bottom: 16px;
  }

  .file-input-button {
    background: #F1F5F9;
    border: 2px dashed #CBD5E1;
    padding: 32px 24px;
    border-radius: 16px;
    text-align: center;
    color: #64748B;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
  }

  .file-input-button:hover {
    background: #E2E8F0;
    border-color: #94A3B8;
  }

  .message {
    padding: 12px 16px;
    border-radius: 12px;
    background: #FEF2F2;
    color: #991B1B;
    font-size: 14px;
    margin-bottom: 16px;
    text-align: center;
  }

  .message.success {
    background: #ECFDF5;
    color: #065F46;
  }

  .upload-btn {
    width: 100%;
    background: #5B9FD8;
    color: white;
    border: none;
    padding: 16px;
    border-radius: 16px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s;
  }

  .upload-btn:hover:not(:disabled) {
    background: #4A8BC2;
  }

  .upload-btn:disabled {
    background: #CBD5E1;
    cursor: not-allowed;
  }
</style>
