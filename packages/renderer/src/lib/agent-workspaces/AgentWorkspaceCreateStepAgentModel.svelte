<script lang="ts">
import { faRobot } from '@fortawesome/free-solid-svg-icons';
import { Icon } from '@podman-desktop/ui-svelte/icons';

import CardSelector, { type CardSelectorOption } from '/@/lib/ui/CardSelector.svelte';

interface Props {
  agentOptions: CardSelectorOption[];
  selectedAgent: string;
}

let { agentOptions, selectedAgent = $bindable() }: Props = $props();
</script>

<div class="flex items-center gap-4 mb-5">
  <div class="w-10 h-10 rounded-lg flex items-center justify-center bg-[var(--pd-label-primary-bg)] text-[var(--pd-label-primary-text)]">
    <Icon icon={faRobot} size="lg" />
  </div>
  <div>
    <span class="text-lg font-semibold text-[var(--pd-modal-text)]">Coding Agent</span>
    <p class="text-xs text-[var(--pd-content-card-text)] opacity-70">Select the AI agent that will drive your workspace</p>
  </div>
</div>

<CardSelector label="Select Coding Agent" options={agentOptions} bind:selected={selectedAgent} />

<!-- TODO: model selection -->
