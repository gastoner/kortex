<script lang="ts">
import { faServer, faWrench } from '@fortawesome/free-solid-svg-icons';
import { Icon } from '@podman-desktop/ui-svelte/icons';

import type { ScrollableCardItem } from '/@/lib/ui/ScrollableCardSelector.svelte';
import ScrollableCardSelector from '/@/lib/ui/ScrollableCardSelector.svelte';

interface Props {
  skillItems: ScrollableCardItem[];
  selectedSkillIds: string[];
  mcpItems: ScrollableCardItem[];
  selectedMcpIds: string[];
}

let { skillItems, selectedSkillIds = $bindable(), mcpItems, selectedMcpIds = $bindable() }: Props = $props();
</script>

{#if skillItems.length > 0}
  <div class="flex items-center gap-4 mb-5">
    <div class="w-10 h-10 rounded-lg flex items-center justify-center bg-[var(--pd-label-tertiary-bg)] text-[var(--pd-label-tertiary-text)]">
      <Icon icon={faWrench} size="lg" />
    </div>
    <div>
      <span class="text-lg font-semibold text-[var(--pd-modal-text)]">Skills</span>
      <p class="text-xs text-[var(--pd-content-card-text)] opacity-70">Select the capabilities your agent should have</p>
    </div>
  </div>

  <ScrollableCardSelector items={skillItems} bind:selected={selectedSkillIds} placeholder="Search skills..." />
{/if}

{#if mcpItems.length > 0}
  <div class="flex items-center gap-4 mb-5 {skillItems.length > 0 ? 'mt-6' : ''}">
    <div class="w-10 h-10 rounded-lg flex items-center justify-center bg-[var(--pd-label-secondary-bg)] text-[var(--pd-label-secondary-text)]">
      <Icon icon={faServer} size="lg" />
    </div>
    <div>
      <span class="text-lg font-semibold text-[var(--pd-modal-text)]">MCP Servers</span>
      <p class="text-xs text-[var(--pd-content-card-text)] opacity-70">Connect to Model Context Protocol servers for extended capabilities</p>
    </div>
  </div>

  <ScrollableCardSelector items={mcpItems} bind:selected={selectedMcpIds} placeholder="Search MCP servers..." />
{/if}

<!-- TODO: secrets and environment variable configuration -->
