<script lang="ts">
import { faFolderOpen, faPlus, faShieldHalved } from '@fortawesome/free-solid-svg-icons';
import { Button, Input } from '@podman-desktop/ui-svelte';
import { Icon } from '@podman-desktop/ui-svelte/icons';

export interface FileAccessOption {
  value: string;
  name: string;
  description: string;
  access: string;
  notes: string;
  badge?: string;
}

interface Props {
  fileAccessOptions: FileAccessOption[];
  selectedFileAccess: string;
  customPaths: string[];
  onBrowseCustomPath: (index: number) => Promise<void>;
  onAddCustomPath: () => void;
  onRemoveCustomPath: (index: number) => void;
  onUpdateCustomPath: (index: number, value: string) => void;
}

let {
  fileAccessOptions,
  selectedFileAccess = $bindable(),
  customPaths,
  onBrowseCustomPath,
  onAddCustomPath,
  onRemoveCustomPath,
  onUpdateCustomPath,
}: Props = $props();

function selectOption(value: string): void {
  selectedFileAccess = value;
}
</script>

<div class="flex items-center gap-3 mb-5">
  <div class="w-9 h-9 rounded-[9px] flex items-center justify-center bg-[var(--pd-label-quaternary-bg)] text-[var(--pd-label-quaternary-text)]">
    <Icon icon={faShieldHalved} class="text-xl" />
  </div>
  <div>
    <span class="text-lg font-semibold text-[var(--pd-modal-text)]">File System Access</span>
    <p class="text-sm text-[var(--pd-content-card-text)] opacity-70 mt-0.5">Pick one scope for agent read/write on the host</p>
  </div>
</div>

<div class="rounded-xl border border-[var(--pd-content-card-border)] bg-[var(--pd-content-card-bg)] overflow-hidden">
  {#each fileAccessOptions as option, idx (option.value)}
    {#if idx > 0}
      <div class="mx-3 border-t border-[var(--pd-content-card-border)] opacity-30"></div>
    {/if}
    <button
      class="w-full flex items-center gap-3 px-4 py-3 cursor-pointer transition-colors text-left
        {selectedFileAccess === option.value
          ? 'bg-[var(--pd-content-card-hover-inset-bg)]'
          : 'hover:bg-[var(--pd-content-card-hover-inset-bg)]'}"
      onclick={selectOption.bind(null, option.value)}
      aria-label={option.name}>
      <div class="min-w-0 flex-1">
        <div class="flex items-center gap-2">
          <span class="text-[13px] font-medium text-[var(--pd-table-body-text-highlight)]">{option.name}</span>
          {#if option.badge}
            <span class="text-[10px] text-[var(--pd-table-body-text)] bg-[var(--pd-content-card-inset-bg)] rounded px-1.5 py-0.5">{option.badge}</span>
          {/if}
        </div>
        <div class="text-[11px] text-[var(--pd-table-body-text)] mt-0.5">{option.description}</div>
      </div>
      <div class="flex items-center gap-4 flex-shrink-0 text-xs text-[var(--pd-table-body-text)]">
        <span class="w-20">{option.access}</span>
        <span class="w-24">{option.notes}</span>
        <input
          type="radio"
          name="fileAccess"
          value={option.value}
          checked={selectedFileAccess === option.value}
          aria-label="Use {option.name}"
          class="accent-[var(--pd-button-primary-bg)] w-4 h-4 cursor-pointer pointer-events-none" />
      </div>
    </button>
  {/each}
</div>

{#if selectedFileAccess === 'custom'}
  <div class="mt-4 p-4 rounded-xl border border-[var(--pd-content-card-border)] bg-[var(--pd-content-card-bg)]">
    <p class="text-xs text-[var(--pd-content-card-text)] opacity-70 mb-3">Allowed paths for custom access. You can refine paths later in project or workspace settings.</p>
    {#each customPaths as path, index (index)}
      <div class="flex gap-3 mb-2 items-center">
        <Input
          value={path}
          placeholder="/path/to/allowed/directory"
          class="flex-1 font-mono text-sm"
          oninput={(e: Event): void => onUpdateCustomPath(index, (e.target as HTMLInputElement).value)}
        />
        <Button onclick={(): Promise<void> => onBrowseCustomPath(index)} aria-label="Browse for directory" icon={faFolderOpen} />
        {#if customPaths.length > 1}
          <Button class="text-red-400" onclick={(): void => onRemoveCustomPath(index)}>Remove</Button>
        {/if}
      </div>
    {/each}
    <Button class="mt-2" icon={faPlus} onclick={onAddCustomPath}>Add Another Path</Button>
  </div>
{/if}
