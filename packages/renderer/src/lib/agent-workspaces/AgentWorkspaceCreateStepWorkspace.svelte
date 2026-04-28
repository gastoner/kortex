<script lang="ts">
import { faChevronDown, faFolderOpen } from '@fortawesome/free-solid-svg-icons';
import { Button, Input } from '@podman-desktop/ui-svelte';
import { Icon } from '@podman-desktop/ui-svelte/icons';

import { Textarea } from '/@/lib/chat/components/ui/textarea';

interface Props {
  sourcePath: string;
  sessionName: string;
  description: string;
  nameManuallyEdited: boolean;
  descriptionOpen: boolean;
  onBrowseSource: () => Promise<void>;
}

let {
  sourcePath = $bindable(),
  sessionName = $bindable(),
  description = $bindable(),
  nameManuallyEdited = $bindable(),
  descriptionOpen = $bindable(),
  onBrowseSource,
}: Props = $props();
</script>

<h2 class="text-lg font-semibold text-[var(--pd-modal-text)] mb-1">Workspace</h2>
<p class="text-sm text-[var(--pd-content-card-text)] opacity-60 mb-5">
  Point to a local project folder to set up your workspace.
</p>

<div class="space-y-4">
  <div>
    <label for="workspace-source" class="block text-sm font-semibold text-[var(--pd-modal-text)] mb-2">
      Project folder
    </label>
    <div class="flex gap-2 items-stretch">
      <Input
        id="workspace-source"
        bind:value={sourcePath}
        placeholder="/path/to/project"
        class="grow font-mono text-sm"
      />
      <Button onclick={onBrowseSource} aria-label="Browse for folder" icon={faFolderOpen} />
    </div>
    <p class="text-xs text-[var(--pd-content-card-text)] opacity-50 mt-1.5">
      Select a local directory to use as the workspace source.
    </p>
  </div>

  <div>
    <label for="workspace-name" class="block text-sm font-semibold text-[var(--pd-modal-text)] mb-2">
      Workspace name
    </label>
    <Input
      id="workspace-name"
      bind:value={sessionName}
      placeholder="e.g., Frontend Refactoring"
      class="w-full"
      oninput={(): boolean => (nameManuallyEdited = true)}
    />
  </div>

  <div class="rounded-xl border border-[var(--pd-content-card-border)]/85 bg-[var(--pd-content-card-bg)]/35 overflow-hidden">
    <button
      class="w-full flex items-center justify-between px-4 py-3 text-sm font-medium text-[var(--pd-modal-text)] hover:bg-[var(--pd-content-card-inset-bg)]/50 transition-colors cursor-pointer"
      onclick={(): boolean => (descriptionOpen = !descriptionOpen)}>
      <span>
        Description <span class="text-xs font-normal opacity-50">(optional)</span>
      </span>
      <span
        class="transition-transform duration-150 {descriptionOpen ? 'rotate-180' : ''}"
        aria-hidden="true">
        <Icon icon={faChevronDown} size="xs" />
      </span>
    </button>
    {#if descriptionOpen}
      <div class="px-4 pb-4">
        <label for="workspace-description" class="block text-xs text-[var(--pd-content-card-text)] opacity-60 mb-1.5">
          What should this workspace focus on?
        </label>
        <Textarea
          id="workspace-description"
          bind:value={description}
          placeholder="Short note for your team (optional)"
          rows={3}
          class="bg-muted min-h-[24px] resize-none rounded-lg !text-sm dark:border-zinc-700"
        />
      </div>
    {/if}
  </div>
</div>
