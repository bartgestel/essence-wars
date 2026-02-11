<script lang="ts">
  import { deckBuilderStore } from "$lib/stores/deckBuilderState.svelte";
  import { playSound } from "$lib/audio";
  import CharacterCountInput from "$lib/components/CharacterCountInput.svelte";

  let { onClose, onSaved }: { onClose: () => void; onSaved: () => void } = $props();

  let deckName = $state(deckBuilderStore.deckName === "New Deck" ? "" : deckBuilderStore.deckName);
  let deckDescription = $state(deckBuilderStore.deckDescription);
  let isSaving = $state(false);
  let error = $state<string | null>(null);

  const isNewDeck = $derived(!deckBuilderStore.currentDeckId);
  const canSave = $derived(deckName.trim().length > 0 && !isSaving);

  const maxLengths = {
    name: 50,
    description: 200
  };

  async function handleSave() {
    if (!canSave) return;

    isSaving = true;
    error = null;

    // Update store with new name/description
    deckBuilderStore.deckName = deckName.trim();
    deckBuilderStore.deckDescription = deckDescription.trim();

    try {
      await deckBuilderStore.saveDeck();
      playSound("cardSelect");
      onSaved();
      onClose();
    } catch (e) {
      error = e instanceof Error ? e.message : String(e);
      playSound("damage");
    } finally {
      isSaving = false;
    }
  }

  function handleBackdropClick(event: MouseEvent) {
    if (event.target === event.currentTarget) {
      onClose();
    }
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Escape") {
      onClose();
    } else if (event.key === "Enter" && canSave && !event.shiftKey) {
      event.preventDefault();
      handleSave();
    }
  }
</script>

<svelte:window onkeydown={handleKeydown} />

<div
  class="modal-backdrop"
  onclick={handleBackdropClick}
  onkeydown={handleKeydown}
  role="dialog"
  tabindex="-1"
  aria-modal="true"
  aria-labelledby="save-deck-title"
>
  <div class="modal-content">
    <header class="modal-header">
      <h2 id="save-deck-title">{isNewDeck ? "Save New Deck" : "Save Deck"}</h2>
      <button class="close-button" onclick={onClose} aria-label="Close">×</button>
    </header>

    <div class="modal-body">
      {#if error || deckBuilderStore.error}
        <div class="error-message">
          {error || deckBuilderStore.error}
        </div>
      {/if}

      <div class="form-group">
        <label for="deck-name">Deck Name *</label>
        <input
          id="deck-name"
          type="text"
          bind:value={deckName}
          placeholder="Enter deck name..."
          maxlength={maxLengths.name}
        />
        <CharacterCountInput currentLength={deckName.length} maxLength={maxLengths.name} />
      </div>

      <div class="form-group">
        <label for="deck-description">Description (optional)</label>
        <textarea
          id="deck-description"
          bind:value={deckDescription}
          placeholder="Describe your deck strategy..."
          rows="3"
          maxlength={maxLengths.description}
        ></textarea>
        <CharacterCountInput currentLength={deckDescription.length} maxLength={maxLengths.description} />
      </div>

      <div class="deck-summary">
        <div class="summary-row">
          <span class="label">Commander:</span>
          <span class="value">{deckBuilderStore.selectedCommander?.name ?? "None"}</span>
        </div>
        <div class="summary-row">
          <span class="label">Cards:</span>
          <span class="value">{deckBuilderStore.deckCards.length}</span>
        </div>
        {#if deckBuilderStore.playstyle}
          <div class="summary-row">
            <span class="label">Playstyle:</span>
            <span class="value">{deckBuilderStore.playstyle.primary.charAt(0).toUpperCase() + deckBuilderStore.playstyle.primary.slice(1)}</span>
          </div>
        {/if}
        {#if deckBuilderStore.validation && !deckBuilderStore.validation.isValid}
          <div class="validation-warning">
            This deck has validation issues but can still be saved.
          </div>
        {/if}
      </div>
    </div>

    <footer class="modal-footer">
      <button class="cancel-button" onclick={onClose}>Cancel</button>
      <button
        class="save-button"
        onclick={handleSave}
        disabled={!canSave}
      >
        {#if isSaving}
          Saving...
        {:else}
          Save Deck
        {/if}
      </button>
    </footer>
  </div>
</div>

<style>
  .modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    padding: 2rem;
  }

  .modal-content {
    background: var(--color-ui-bg, #1a1a2e);
    border-radius: 1rem;
    border: 1px solid rgba(255, 255, 255, 0.1);
    max-width: 450px;
    width: 100%;
    overflow: hidden;
  }

  .modal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 1rem 1.5rem;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .modal-header h2 {
    margin: 0;
    font-size: 1.25rem;
    font-weight: 600;
  }

  .close-button {
    background: transparent;
    border: none;
    color: rgba(255, 255, 255, 0.6);
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0.25rem 0.5rem;
    line-height: 1;
    transition: color 0.2s;
  }

  .close-button:hover {
    color: white;
  }

  .modal-body {
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .error-message {
    padding: 0.75rem 1rem;
    background: rgba(239, 68, 68, 0.2);
    border: 1px solid rgba(239, 68, 68, 0.3);
    border-radius: 0.5rem;
    color: #fca5a5;
    font-size: 0.875rem;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .form-group label {
    font-size: 0.875rem;
    font-weight: 500;
    color: rgba(255, 255, 255, 0.8);
  }

  .form-group input,
  .form-group textarea {
    padding: 0.75rem 1rem;
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 0.5rem;
    color: var(--color-ui-text);
    font-size: 1rem;
    font-family: inherit;
    resize: none;
  }

  .form-group input::placeholder,
  .form-group textarea::placeholder {
    color: rgba(255, 255, 255, 0.4);
  }

  .form-group input:focus,
  .form-group textarea:focus {
    outline: none;
    border-color: var(--color-ui-action, #e94560);
  }

  .deck-summary {
    padding: 1rem;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 0.5rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .summary-row {
    display: flex;
    justify-content: space-between;
    font-size: 0.875rem;
  }

  .summary-row .label {
    color: rgba(255, 255, 255, 0.6);
  }

  .summary-row .value {
    color: var(--color-ui-text);
    font-weight: 500;
  }

  .validation-warning {
    margin-top: 0.5rem;
    padding: 0.5rem;
    background: rgba(251, 191, 36, 0.1);
    border-radius: 0.25rem;
    font-size: 0.75rem;
    color: #fcd34d;
    text-align: center;
  }

  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 0.75rem;
    padding: 1rem 1.5rem;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
  }

  .cancel-button,
  .save-button {
    padding: 0.625rem 1.25rem;
    border-radius: 0.5rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s;
  }

  .cancel-button {
    background: transparent;
    border: 1px solid rgba(255, 255, 255, 0.2);
    color: rgba(255, 255, 255, 0.7);
  }

  .cancel-button:hover {
    border-color: rgba(255, 255, 255, 0.4);
    color: white;
  }

  .save-button {
    background: var(--color-ui-action, #e94560);
    border: none;
    color: white;
  }

  .save-button:hover:not(:disabled) {
    background: #ff5a75;
  }

  .save-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
