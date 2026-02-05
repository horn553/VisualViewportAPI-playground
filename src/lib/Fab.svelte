<script lang="ts">
	let supported = $state(false);
	let width = $state(0);
	let height = $state(0);
	let offsetLeft = $state(0);
	let offsetTop = $state(0);
	let scale = $state(1);
	let hasMetrics = $state(false);
	let hasEntered = $state(false);
	let isViewportInteracting = $state(false);

	let debounceId: ReturnType<typeof setTimeout> | null = null;
	let enterRafId: number | null = null;

	const effectiveScale = $derived(Math.max(0.01, scale));
	const scaleInv = $derived(1 / effectiveScale);
	const fabMargin = $derived(`${16 * scaleInv}px`);
	const uiVisible = $derived(hasEntered && !isViewportInteracting);
	const uiOpacity = $derived(uiVisible ? 1 : 0);
	const fabEnterX = $derived(uiVisible ? '0%' : '120%');
	const fabEnterY = $derived(uiVisible ? '0%' : '120%');
	const vvWidth = $derived(hasMetrics ? `${width}px` : '100vw');
	const vvHeight = $derived(hasMetrics ? `${height}px` : '100vh');
	const vvLeft = $derived(`${offsetLeft}px`);
	const vvTop = $derived(`${offsetTop}px`);

	const snapCssPx = (value: number, precision = 100) => Math.round(value * precision) / precision;
	const snapScale = (value: number, precision = 1000) => Math.round(value * precision) / precision;

	const updateMetrics = () => {
		const vv = window.visualViewport;
		supported = !!vv;

		if (vv) {
			width = snapCssPx(vv.width);
			height = snapCssPx(vv.height);
			offsetLeft = snapCssPx(vv.offsetLeft);
			offsetTop = snapCssPx(vv.offsetTop);
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
		isViewportInteracting = true;
		if (debounceId !== null) {
			clearTimeout(debounceId);
		}
		debounceId = setTimeout(() => {
			debounceId = null;
			updateMetrics();
			isViewportInteracting = false;
		}, 300);
	};

	$effect(() => {
		if (typeof window === 'undefined') return;

		updateMetrics();
		enterRafId = requestAnimationFrame(() => {
			enterRafId = null;
			hasEntered = true;
		});

		const vv = window.visualViewport;
		if (vv) {
			vv.addEventListener('resize', scheduleUpdate);
			vv.addEventListener('scroll', scheduleUpdate);
		}
		window.addEventListener('resize', scheduleUpdate);

		return () => {
			if (debounceId !== null) clearTimeout(debounceId);
			if (enterRafId !== null) cancelAnimationFrame(enterRafId);
			if (vv) {
				vv.removeEventListener('resize', scheduleUpdate);
				vv.removeEventListener('scroll', scheduleUpdate);
			}
			window.removeEventListener('resize', scheduleUpdate);
		};
	});
</script>

<div
	class="vv-overlay"
	style:--vv-left={vvLeft}
	style:--vv-top={vvTop}
	style:--vv-width={vvWidth}
	style:--vv-height={vvHeight}
	style:--vv-scale-inv={scaleInv}
	style:--fab-margin={fabMargin}
	style:--fab-enter-x={fabEnterX}
	style:--fab-enter-y={fabEnterY}
	style:--ui-opacity={uiOpacity}
>
	<div class="fab-slot">
		<button class="fab" type="button" aria-label="Add">
			<span aria-hidden="true">+</span>
		</button>
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
		transform: translate3d(var(--fab-enter-x), var(--fab-enter-y), 0) scale(var(--vv-scale-inv));
		transform-origin: bottom right;
		opacity: var(--ui-opacity);
		transition:
			transform 0.1s ease,
			opacity 0.1s ease,
			right 0.1s ease,
			bottom 0.1s ease;
		pointer-events: auto;
	}

	.fab {
		display: flex;
		justify-content: center;
		align-items: center;

		border: none;
		border-radius: 25px;
		width: 50px;
		height: 50px;

		background-color: white;
		color: #111;
		cursor: pointer;
		box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
		padding: 0;
	}

	span {
		font-size: 30px;
		font-weight: bold;
	}
</style>
