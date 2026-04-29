<script lang="ts">
import { faFolderOpen, faPlus, faShieldHalved } from '@fortawesome/free-solid-svg-icons';
import { Button, Input } from '@podman-desktop/ui-svelte';
import { Icon } from '@podman-desktop/ui-svelte/icons';

import CardSelector, { type CardSelectorOption } from '/@/lib/ui/CardSelector.svelte';

interface Props {
  fileAccessOptions: CardSelectorOption[];
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
</script>

<div class="flex items-center gap-4 mb-5">
  <div class="w-10 h-10 rounded-lg flex items-center justify-center bg-[var(--pd-label-quaternary-bg)] text-[var(--pd-label-quaternary-text)]">
    <Icon icon={faShieldHalved} size="lg" />
  </div>
  <div>
    <span class="text-lg font-semibold text-[var(--pd-modal-text)]">File System Access</span>
    <p class="text-xs text-[var(--pd-content-card-text)] opacity-70">Define which directories the agent can access on your host system</p>
  </div>
</div>

<CardSelector label="Access Level" options={fileAccessOptions} bind:selected={selectedFileAccess} />

{#if selectedFileAccess === 'custom'}
  <div class="mt-4 p-4 rounded-lg bg-[var(--pd-content-card-inset-bg)]">
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
