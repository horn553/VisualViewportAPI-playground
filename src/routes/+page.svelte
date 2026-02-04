<script lang="ts">
	import Fab from '$lib/Fab.svelte';
	import Gradient from '$lib/Gradient.svelte';

	let supported = $state(false);
	let width = $state(0);
	let height = $state(0);
	let offsetLeft = $state(0);
	let offsetTop = $state(0);
	let scale = $state(1);
	let dpr = $state(1);
	let hasMetrics = $state(false);
	let baseDpr = $state(1);

	let rafId: number | null = null;

	const dprRatio = $derived(dpr / baseDpr);
	const effectiveScale = $derived(Math.max(0.01, scale * dprRatio));
	const scaleInv = $derived(1 / effectiveScale);
	const fabMargin = $derived(`${16 * scaleInv}px`);
	const debugMargin = $derived(`${8 * scaleInv}px`);
	const supportLabel = $derived(hasMetrics ? (supported ? 'yes' : 'no') : 'pending');
	const vvWidth = $derived(hasMetrics ? `${width}px` : '100vw');
	const vvHeight = $derived(hasMetrics ? `${height}px` : '100vh');
	const vvLeft = $derived(`${offsetLeft}px`);
	const vvTop = $derived(`${offsetTop}px`);

	const fmt = (value: number, digits = 2) => value.toFixed(digits);
	const snapCssPx = (value: number, dprValue: number) => Math.round(value * dprValue) / dprValue;
	const snapScale = (value: number, precision = 1000) => Math.round(value * precision) / precision;

	const updateMetrics = () => {
		const vv = window.visualViewport;
		supported = !!vv;

		const nextDpr = window.devicePixelRatio || 1;
		dpr = nextDpr;

		if (vv) {
			width = snapCssPx(vv.width, nextDpr);
			height = snapCssPx(vv.height, nextDpr);
			offsetLeft = snapCssPx(vv.offsetLeft, nextDpr);
			offsetTop = snapCssPx(vv.offsetTop, nextDpr);
			scale = snapScale(vv.scale);
		} else {
			width = window.innerWidth;
			height = window.innerHeight;
			offsetLeft = 0;
			offsetTop = 0;
			scale = 1;
		}

		hasMetrics = true;
	};

	const scheduleUpdate = () => {
		if (rafId !== null) return;
		rafId = requestAnimationFrame(() => {
			rafId = null;
			updateMetrics();
		});
	};

	$effect(() => {
		if (typeof window === 'undefined') return;

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
	style:--vv-left={vvLeft}
	style:--vv-top={vvTop}
	style:--vv-width={vvWidth}
	style:--vv-height={vvHeight}
	style:--vv-scale-inv={scaleInv}
	style:--fab-margin={fabMargin}
	style:--debug-margin={debugMargin}
>
	<div class="fab-slot">
		<Fab />
	</div>

	<div class="vv-debug">
		<div class="row">
			<span class="label">supported</span>
			<span class="value">{supportLabel}</span>
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
		left: 0;
		top: 0;
		width: var(--vv-width);
		height: var(--vv-height);
		transform: translate3d(var(--vv-left), var(--vv-top), 0);
		will-change: transform;
		pointer-events: none;
		z-index: 1000;
	}

	.fab-slot {
		position: absolute;
		right: var(--fab-margin);
		bottom: var(--fab-margin);
		transform: scale(var(--vv-scale-inv));
		transform-origin: bottom right;
		pointer-events: auto;
	}

	.vv-debug {
		position: absolute;
		top: var(--debug-margin);
		left: var(--debug-margin);
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
