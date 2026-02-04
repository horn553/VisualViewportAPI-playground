<script lang="ts">
	import Fab from '$lib/Fab.svelte';
	import Gradient from '$lib/Gradient.svelte';
	import { onMount } from 'svelte';

	let supported = false;
	let width = 0;
	let height = 0;
	let offsetLeft = 0;
	let offsetTop = 0;
	let scale = 1;
	let dpr = 1;
	let effectiveScale = 1;
	let scaleInv = 1;

	let baseDpr = 1;
	let rafId: number | null = null;

	const fmt = (value: number, digits = 2) => value.toFixed(digits);

	const updateMetrics = () => {
		const vv = window.visualViewport;
		supported = !!vv;

		const currentDpr = window.devicePixelRatio || 1;
		const dprRatio = currentDpr / baseDpr;
		dpr = currentDpr;

		if (vv) {
			width = vv.width;
			height = vv.height;
			offsetLeft = vv.offsetLeft;
			offsetTop = vv.offsetTop;
			scale = vv.scale;
		} else {
			width = window.innerWidth;
			height = window.innerHeight;
			offsetLeft = 0;
			offsetTop = 0;
			scale = 1;
		}

		effectiveScale = Math.max(0.01, scale * dprRatio);
		scaleInv = 1 / effectiveScale;
	};

	const scheduleUpdate = () => {
		if (rafId !== null) return;
		rafId = requestAnimationFrame(() => {
			rafId = null;
			updateMetrics();
		});
	};

	onMount(() => {
		baseDpr = window.devicePixelRatio || 1;
		updateMetrics();

		const vv = window.visualViewport;
		if (vv) {
			vv.addEventListener('resize', scheduleUpdate);
			vv.addEventListener('scroll', scheduleUpdate);
		}
		window.addEventListener('resize', scheduleUpdate);

		return () => {
			if (rafId !== null) cancelAnimationFrame(rafId);
			if (vv) {
				vv.removeEventListener('resize', scheduleUpdate);
				vv.removeEventListener('scroll', scheduleUpdate);
			}
			window.removeEventListener('resize', scheduleUpdate);
		};
	});
</script>

<Gradient />

<div
	class="vv-overlay"
	style={`--vv-left:${offsetLeft}px; --vv-top:${offsetTop}px; --vv-width:${width}px; --vv-height:${height}px; --vv-scale-inv:${scaleInv};`}
>
	<div class="fab-slot">
		<Fab />
	</div>

	<div class="vv-debug">
		<div class="row">
			<span class="label">supported</span>
			<span class="value">{supported ? 'yes' : 'no'}</span>
		</div>
		<div class="row">
			<span class="label">scale</span>
			<span class="value">{fmt(scale, 3)}</span>
		</div>
		<div class="row">
			<span class="label">offsetLeft</span>
			<span class="value">{fmt(offsetLeft, 2)}</span>
		</div>
		<div class="row">
			<span class="label">offsetTop</span>
			<span class="value">{fmt(offsetTop, 2)}</span>
		</div>
		<div class="row">
			<span class="label">width</span>
			<span class="value">{fmt(width, 2)}</span>
		</div>
		<div class="row">
			<span class="label">height</span>
			<span class="value">{fmt(height, 2)}</span>
		</div>
		<div class="row">
			<span class="label">devicePixelRatio</span>
			<span class="value">{fmt(dpr, 3)}</span>
		</div>
		<div class="row">
			<span class="label">effectiveScale</span>
			<span class="value">{fmt(effectiveScale, 3)}</span>
		</div>
	</div>
</div>

<style>
	.vv-overlay {
		position: fixed;
		left: var(--vv-left);
		top: var(--vv-top);
		width: var(--vv-width);
		height: var(--vv-height);
		pointer-events: none;
		z-index: 1000;
	}

	.fab-slot {
		position: absolute;
		right: calc(16px * var(--vv-scale-inv));
		bottom: calc(16px * var(--vv-scale-inv));
		transform: scale(var(--vv-scale-inv));
		transform-origin: bottom right;
		pointer-events: auto;
	}

	.vv-debug {
		position: absolute;
		top: calc(8px * var(--vv-scale-inv));
		left: calc(8px * var(--vv-scale-inv));
		transform: scale(var(--vv-scale-inv));
		transform-origin: top left;
		pointer-events: auto;
		background: rgba(255, 255, 255, 0.9);
		border-radius: 8px;
		padding: 8px 10px;
		font-family: 'Courier New', Courier, monospace;
		font-size: 12px;
		color: #111;
		box-shadow: 0 8px 18px rgba(0, 0, 0, 0.15);
	}

	.vv-debug .row {
		display: flex;
		gap: 8px;
		justify-content: space-between;
		white-space: nowrap;
	}

	.vv-debug .label {
		opacity: 0.7;
	}
</style>
