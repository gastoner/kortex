<script lang="ts">
import {
  faCode,
  faGears,
  faHome,
  faLock,
  faO,
  faRobot,
  faTriangleExclamation,
  faWrench,
} from '@fortawesome/free-solid-svg-icons';
import { Button } from '@podman-desktop/ui-svelte';
import { Icon } from '@podman-desktop/ui-svelte/icons';
import { toast } from 'svelte-sonner';

import AgentWorkspaceCreateStepAgentModel from '/@/lib/agent-workspaces/AgentWorkspaceCreateStepAgentModel.svelte';
import AgentWorkspaceCreateStepFileSystem from '/@/lib/agent-workspaces/AgentWorkspaceCreateStepFileSystem.svelte';
import AgentWorkspaceCreateStepNetworking from '/@/lib/agent-workspaces/AgentWorkspaceCreateStepNetworking.svelte';
import AgentWorkspaceCreateStepToolsSecrets from '/@/lib/agent-workspaces/AgentWorkspaceCreateStepToolsSecrets.svelte';
import AgentWorkspaceCreateStepWorkspace from '/@/lib/agent-workspaces/AgentWorkspaceCreateStepWorkspace.svelte';
import type { CardSelectorOption } from '/@/lib/ui/CardSelector.svelte';
import FormPage from '/@/lib/ui/FormPage.svelte';
import type { ScrollableCardItem } from '/@/lib/ui/ScrollableCardSelector.svelte';
import WizardStepper from '/@/lib/ui/WizardStepper.svelte';
import { handleNavigation } from '/@/navigation';
import { mcpRemoteServerInfos } from '/@/stores/mcp-remote-servers';
import { skillInfos } from '/@/stores/skills';
import { NavigationPage } from '/@api/navigation-page';

const agentOptions: CardSelectorOption[] = [
  {
    title: 'OpenCode',
    badge: 'Anomaly',
    value: 'opencode',
    icon: faO,
    description: 'Open-source terminal-based coding agent',
  },
  {
    title: 'Claude',
    badge: 'Anthropic',
    value: 'claude',
    icon: faRobot,
    description: `Anthropic's AI coding assistant`,
  },
  { title: 'Cursor', badge: 'Cursor', value: 'cursor', icon: faCode, description: 'AI-powered code editor agent' },
  {
    title: 'Goose',
    badge: 'Block',
    value: 'goose',
    icon: faWrench,
    description: 'Open-source autonomous coding agent',
  },
];

const fileAccessOptions: CardSelectorOption[] = [
  {
    title: 'Working Directory Only',
    badge: 'Recommended',
    value: 'workspace',
    icon: faGears,
    description: 'Restrict access to the project working directory',
  },
  {
    title: 'Home Directory',
    badge: '~/  access',
    value: 'home',
    icon: faHome,
    description: 'Allow access to the user home directory',
  },
  {
    title: 'Custom Paths',
    badge: 'Configurable',
    value: 'custom',
    icon: faGears,
    description: 'Specify custom paths the agent can access',
  },
  {
    title: 'Full System Access',
    badge: 'Caution',
    value: 'full',
    icon: faTriangleExclamation,
    description: 'Unrestricted filesystem access — use with care',
  },
];

const wizardSteps = [
  { id: 'workspace', title: 'Workspace' },
  { id: 'agent-model', title: 'Agent & Model' },
  { id: 'tools-secrets', title: 'Tools & Secrets' },
  { id: 'filesystem', title: 'File System' },
  { id: 'networking', title: 'Networking' },
];

let skillItems: ScrollableCardItem[] = $derived(
  $skillInfos.map(s => ({ id: s.name, name: s.name, description: s.description })),
);
let mcpItems: ScrollableCardItem[] = $derived(
  $mcpRemoteServerInfos.map(m => ({ id: m.id, name: m.name, description: m.description })),
);

// --- Form state ---
let sessionName = $state('');
let sourcePath = $state('');
let description = $state('');
let selectedAgent = $state('goose');
let selectedFileAccess = $state('workspace');
let selectedSkillIds = $state<string[]>([]);
let selectedMcpIds = $state<string[]>([]);
let customPaths = $state<string[]>(['']);

// --- Step 1 UI state ---
let nameManuallyEdited = $state(false);
let descriptionOpen = $state(false);

// Auto-suggest workspace name from the last path segment
$effect(() => {
  if (nameManuallyEdited) return;
  const src = sourcePath
    .trim()
    .replace(/\.git$/, '')
    .replace(/\/$/, '');
  if (!src) return;
  const last = src.split(/[/:]/).filter(Boolean).at(-1) ?? '';
  if (last) sessionName = last;
});

// --- Wizard navigation ---
let currentStepIndex = $state(0);
let creating = $state(false);
let error = $state('');

let currentStepId = $derived(wizardSteps[currentStepIndex]?.id ?? '');
let isLastStep = $derived(currentStepIndex === wizardSteps.length - 1);
let isCurrentStepComplete = $derived(
  currentStepId === 'workspace' ? sessionName.trim() !== '' && sourcePath.trim() !== '' : true,
);

function goNext(): void {
  if (currentStepIndex < wizardSteps.length - 1) currentStepIndex++;
}

function goBack(): void {
  if (currentStepIndex > 0) currentStepIndex--;
}

function handleStepClick(index: number): void {
  currentStepIndex = index;
}

function addCustomPath(): void {
  customPaths = [...customPaths, ''];
}

function removeCustomPath(index: number): void {
  if (customPaths.length <= 1) return;
  customPaths = customPaths.filter((_, i) => i !== index);
}

function updateCustomPath(index: number, value: string): void {
  customPaths = customPaths.map((p, i) => (i === index ? value : p));
}

async function handleBrowseCustomPath(index: number): Promise<void> {
  try {
    const result = await window.openDialog({ title: 'Select a directory', selectors: ['openDirectory'] });
    const selected = result?.[0];
    if (selected) updateCustomPath(index, selected);
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : String(err);
    error = message;
    toast.error(`Failed to browse for directory: ${message}`);
  }
}

async function handleBrowseSource(): Promise<void> {
  try {
    const result = await window.openDialog({ title: 'Select a working directory', selectors: ['openDirectory'] });
    const selected = result?.[0];
    if (selected) {
      sourcePath = selected;
      if (!nameManuallyEdited) {
        const lastSegment = selected.replace(/\/$/, '').split('/').at(-1) ?? '';
        if (lastSegment) sessionName = lastSegment;
      }
    }
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : String(err);
    error = message;
    toast.error(`Failed to browse for directory: ${message}`);
  }
}

function cancel(): void {
  handleNavigation({ page: NavigationPage.AGENT_WORKSPACES });
}

async function startWorkspace(): Promise<void> {
  if (!sessionName.trim() || !sourcePath.trim()) return;

  creating = true;
  error = '';
  try {
    await window.createAgentWorkspace({
      sourcePath,
      agent: selectedAgent,
      name: sessionName,
    });
    handleNavigation({ page: NavigationPage.AGENT_WORKSPACES });
  } catch (err: unknown) {
    error = err instanceof Error ? err.message : String(err);
  } finally {
    creating = false;
  }
}
</script>

<FormPage title="Create Agent Workspace">
  {#snippet content()}
    <div class="px-5 pb-5 min-w-full">
      <div class="bg-[var(--pd-content-card-bg)] py-6">
        <div class="flex flex-col px-6 max-w-4xl mx-auto space-y-5">

          <!-- Page header -->
          <div class="mb-2">
            <span class="text-xs font-semibold uppercase tracking-widest text-[var(--pd-label-primary-text)] bg-[var(--pd-label-primary-bg)] px-2 py-0.5 rounded mb-2 inline-block">
              Coding Agent
            </span>
            <h1 class="text-2xl font-bold text-[var(--pd-modal-text)] mb-1">Create Coding Agent Workspace</h1>
            <p class="text-sm text-[var(--pd-content-card-text)] opacity-70 max-w-2xl leading-relaxed">
              Add your code location first, then tune agent, tools, and sandbox access…
            </p>
          </div>

          <!-- Stepper -->
          <WizardStepper steps={wizardSteps} currentIndex={currentStepIndex} onStepClick={handleStepClick} />

          <!-- Step content -->
          <div class="rounded-xl border border-[var(--pd-content-card-border)] bg-[var(--pd-content-card-inset-bg)] p-6">
            {#if currentStepId === 'workspace'}
              <AgentWorkspaceCreateStepWorkspace
                bind:sourcePath
                bind:sessionName
                bind:description
                bind:nameManuallyEdited
                bind:descriptionOpen
                onBrowseSource={handleBrowseSource} />
            {:else if currentStepId === 'agent-model'}
              <AgentWorkspaceCreateStepAgentModel {agentOptions} bind:selectedAgent />
            {:else if currentStepId === 'tools-secrets'}
              <AgentWorkspaceCreateStepToolsSecrets
                {skillItems}
                bind:selectedSkillIds
                {mcpItems}
                bind:selectedMcpIds />
            {:else if currentStepId === 'filesystem'}
              <AgentWorkspaceCreateStepFileSystem
                {fileAccessOptions}
                bind:selectedFileAccess
                {customPaths}
                onBrowseCustomPath={handleBrowseCustomPath}
                onAddCustomPath={addCustomPath}
                onRemoveCustomPath={removeCustomPath}
                onUpdateCustomPath={updateCustomPath} />
            {:else if currentStepId === 'networking'}
              <AgentWorkspaceCreateStepNetworking />
            {/if}
          </div>

          {#if error}
            <div class="text-sm text-red-400 bg-red-900/20 rounded-lg p-3">{error}</div>
          {/if}

          <!-- Footer actions -->
          <div class="flex items-center justify-between pt-4 border-t border-[var(--pd-content-card-border)]">
            <div class="flex items-center gap-3 text-sm text-[var(--pd-content-card-text)] opacity-70">
              <Icon icon={faLock} size="sm" />
              <span>Workspace will run in a secured sandbox environment</span>
            </div>
            <div class="flex gap-3">
              <Button onclick={cancel}>Cancel</Button>
              {#if currentStepIndex > 0}
                <Button onclick={goBack}>Back</Button>
              {/if}
              {#if isLastStep}
                <Button disabled={creating} onclick={startWorkspace}>
                  {creating ? 'Creating...' : 'Start Workspace'}
                </Button>
              {:else}
                {#if currentStepId === 'workspace'}
                  <Button type="secondary" disabled={!isCurrentStepComplete || creating} onclick={startWorkspace}>
                    {creating ? 'Creating...' : 'Start Workspace'}
                  </Button>
                {/if}
                <Button disabled={!isCurrentStepComplete} onclick={goNext}>Next</Button>
              {/if}
            </div>
          </div>

        </div>
      </div>
    </div>
  {/snippet}
</FormPage>
