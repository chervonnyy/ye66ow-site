<script lang="ts">
	import { onMount } from 'svelte';

	let {
		current = $bindable(0),
		ramp = $bindable(0),
		inverted = $bindable(false),
		speed = $bindable(1),
		font = $bindable(0),
		visible = false,
		onChange
	}: {
		current: number;
		ramp: number;
		inverted: boolean;
		speed: number;
		font: number;
		visible: boolean;
		onChange?: (v: number) => void;
	} = $props();

	const fontLabels = ['JetBrains', 'Courier', 'Monaco', 'Menlo', 'Lucida', 'Typewriter', 'Andale', 'OCR-A', 'Phosphate', 'Copper'];

	const patterns = ['blob', 'wave', 'ring', 'rain', 'topo', 'moiré', 'spiral', 'scan', 'pulse', 'sort', 'cell', 'p+scan'];
	const charSets = ['@8O', '#%*', '█▓▒', 'WMB', '0Oo', '//|', '⣿⣷⣯', 'ンシツ', '●◉◎', '█▉▊', 'ye66ow', 'fnstlg', '☾★✦', '♪♫♩', 'optrcα', 'oO0°·', 'yoshi', 'река'];
	const speeds = [0.25, 0.5, 1, 2, 4];

	function selectPattern(index: number) {
		current = index;
		onChange?.(index);
	}

	function selectRamp(index: number) {
		ramp = index;
	}

	function toggleInvert() {
		inverted = !inverted;
	}

	function cycleSpeed() {
		const idx = speeds.indexOf(speed);
		const next = (idx + 1) % speeds.length;
		speed = speeds[next];
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.target instanceof HTMLInputElement || e.target instanceof HTMLTextAreaElement) return;
		const num = parseInt(e.key);
		if (num >= 1 && num <= 9) {
			selectPattern(num - 1);
		}
		if (e.key === '0') selectPattern(9);
		if (e.key === 'ArrowLeft') selectPattern(Math.max(0, current - 1));
		if (e.key === 'ArrowRight') selectPattern(Math.min(patterns.length - 1, current + 1));
		if (e.key === 'ArrowUp') { e.preventDefault(); selectRamp(Math.max(0, ramp - 1)); }
		if (e.key === 'ArrowDown') { e.preventDefault(); selectRamp(Math.min(charSets.length - 1, ramp + 1)); }
		if (e.key === 'i') toggleInvert();
		if (e.key === 's') cycleSpeed();
	}

	onMount(() => {
		window.addEventListener('keydown', handleKeydown);
		return () => {
			window.removeEventListener('keydown', handleKeydown);
		};
	});
</script>

{#if visible}
<div class="switcher">
	<div class="row scrollable">
		{#each patterns as label, i}
			<button
				class="btn"
				class:active={current === i}
				onclick={() => selectPattern(i)}
			>
				{label}
			</button>
		{/each}
	</div>
	<div class="row scrollable">
		{#each charSets as label, i}
			<button
				class="btn char-btn"
				class:active={ramp === i}
				onclick={() => selectRamp(i)}
			>
				{label}
			</button>
		{/each}
	</div>
	<div class="row scrollable">
		{#each fontLabels as label, i}
			<button
				class="btn char-btn"
				class:active={font === i}
				onclick={() => font = i}
			>
				{label}
			</button>
		{/each}
	</div>
	<div class="row controls">
		<button
			class="btn ctrl-btn"
			class:active={inverted}
			onclick={toggleInvert}
		>
			◐ inv
		</button>
		<button
			class="btn ctrl-btn"
			onclick={cycleSpeed}
		>
			{speed}x
		</button>
	</div>
	<span class="hint">1-9 pattern · ↑↓ chars · i invert · s speed</span>
</div>
{/if}

<style>
	.switcher {
		position: fixed;
		bottom: 24px;
		left: 50%;
		transform: translateX(-50%);
		z-index: 99999;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4px;
		max-width: 95vw;
	}

	.row {
		display: flex;
		gap: 3px;
		background: rgba(0, 0, 0, 0.85);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border-radius: 40px;
		padding: 4px;
		border: 1px solid rgba(255, 255, 255, 0.12);
		box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
	}

	.scrollable {
		overflow-x: auto;
		max-width: 95vw;
		scrollbar-width: none;
		-ms-overflow-style: none;
	}

	.scrollable::-webkit-scrollbar {
		display: none;
	}

	.controls {
		gap: 6px;
	}

	.btn {
		font-family: 'JetBrains Mono', 'Courier New', monospace;
		font-size: 0.7rem;
		font-weight: 600;
		padding: 6px 12px;
		border: none;
		border-radius: 30px;
		cursor: pointer;
		transition: all 0.2s ease;
		background: transparent;
		color: rgba(255, 255, 255, 0.45);
		letter-spacing: 0.02em;
		white-space: nowrap;
		flex-shrink: 0;
	}

	.btn:hover {
		color: rgba(255, 255, 255, 0.8);
		background: rgba(255, 255, 255, 0.1);
	}

	.btn.active {
		background: #fff;
		color: #000;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
	}

	.char-btn {
		font-size: 0.65rem;
		padding: 4px 10px;
		letter-spacing: 0.05em;
	}

	.ctrl-btn {
		font-size: 0.65rem;
		padding: 5px 12px;
	}

	.hint {
		font-family: 'JetBrains Mono', monospace;
		font-size: 0.55rem;
		color: rgba(255, 255, 255, 0.25);
		letter-spacing: 0.04em;
	}

	@media (max-width: 480px) {
		.switcher {
			bottom: 12px;
		}
		.btn {
			font-size: 0.6rem;
			padding: 5px 8px;
		}
		.char-btn {
			font-size: 0.55rem;
			padding: 3px 7px;
		}
		.hint {
			display: none;
		}
	}
</style>
