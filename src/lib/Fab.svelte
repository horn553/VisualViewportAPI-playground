<script lang="ts">
	const DIRECTION_MAP = {
		'up-left': {
			anchorX: 'left',
			anchorY: 'top',
			enterX: '-120%',
			enterY: '-120%',
			originX: 'left',
			originY: 'top'
		},
		up: {
			anchorX: 'center',
			anchorY: 'top',
			enterX: '0%',
			enterY: '-120%',
			originX: 'center',
			originY: 'top'
		},
		'up-right': {
			anchorX: 'right',
			anchorY: 'top',
			enterX: '120%',
			enterY: '-120%',
			originX: 'right',
			originY: 'top'
		},
		left: {
			anchorX: 'left',
			anchorY: 'center',
			enterX: '-120%',
			enterY: '0%',
			originX: 'left',
			originY: 'center'
		},
		center: {
			anchorX: 'center',
			anchorY: 'center',
			enterX: '0%',
			enterY: '0%',
			originX: 'center',
			originY: 'center'
		},
		right: {
			anchorX: 'right',
			anchorY: 'center',
			enterX: '120%',
			enterY: '0%',
			originX: 'right',
			originY: 'center'
		},
		'down-left': {
			anchorX: 'left',
			anchorY: 'bottom',
			enterX: '-120%',
			enterY: '120%',
			originX: 'left',
			originY: 'bottom'
		},
		down: {
			anchorX: 'center',
			anchorY: 'bottom',
			enterX: '0%',
			enterY: '120%',
			originX: 'center',
			originY: 'bottom'
		},
		'down-right': {
			anchorX: 'right',
			anchorY: 'bottom',
			enterX: '120%',
			enterY: '120%',
			originX: 'right',
			originY: 'bottom'
		}
	} as const;

	type FabDirection = keyof typeof DIRECTION_MAP;
	type FabProps = {
		direction?: FabDirection;
		ariaLabel?: string;
	};

	let { direction = 'down-right', ariaLabel = 'Add' }: FabProps = $props();

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
	const directionConfig = $derived(DIRECTION_MAP[direction] ?? DIRECTION_MAP['down-right']);
	const uiVisible = $derived(hasEntered && !isViewportInteracting);
	const uiOpacity = $derived(uiVisible ? 1 : 0);
	const fabEnterX = $derived(uiVisible ? '0%' : directionConfig.enterX);
	const fabEnterY = $derived(uiVisible ? '0%' : directionConfig.enterY);
	const vvWidth = $derived(hasMetrics ? `${width}px` : '100vw');
	const vvHeight = $derived(hasMetrics ? `${height}px` : '100vh');
	const vvLeft = $derived(`${offsetLeft}px`);
	const vvTop = $derived(`${offsetTop}px`);
	const fabLeft = $derived(
		directionConfig.anchorX === 'left'
			? fabMargin
			: directionConfig.anchorX === 'center'
				? '50%'
				: 'auto'
	);
	const fabRight = $derived(directionConfig.anchorX === 'right' ? fabMargin : 'auto');
	const fabTop = $derived(
		directionConfig.anchorY === 'top'
			? fabMargin
			: directionConfig.anchorY === 'center'
				? '50%'
				: 'auto'
	);
	const fabBottom = $derived(directionConfig.anchorY === 'bottom' ? fabMargin : 'auto');
	const fabAnchorTranslateX = $derived(directionConfig.anchorX === 'center' ? '-50%' : '0%');
	const fabAnchorTranslateY = $derived(directionConfig.anchorY === 'center' ? '-50%' : '0%');
	const fabOriginX = $derived(directionConfig.originX);
	const fabOriginY = $derived(directionConfig.originY);

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
	style:--fab-left={fabLeft}
	style:--fab-right={fabRight}
	style:--fab-top={fabTop}
	style:--fab-bottom={fabBottom}
	style:--fab-anchor-translate-x={fabAnchorTranslateX}
	style:--fab-anchor-translate-y={fabAnchorTranslateY}
	style:--fab-origin-x={fabOriginX}
	style:--fab-origin-y={fabOriginY}
	style:--ui-opacity={uiOpacity}
>
	<div class="fab-slot">
		<div class="fab-motion">
			<button class="fab" type="button" aria-label={ariaLabel}>
				<span aria-hidden="true">+</span>
			</button>
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
		left: var(--fab-left);
		right: var(--fab-right);
		top: var(--fab-top);
		bottom: var(--fab-bottom);
		transform: translate3d(var(--fab-anchor-translate-x), var(--fab-anchor-translate-y), 0);
		pointer-events: none;
	}

	.fab-motion {
		transform: translate3d(var(--fab-enter-x), var(--fab-enter-y), 0) scale(var(--vv-scale-inv));
		transform-origin: var(--fab-origin-x) var(--fab-origin-y);
		opacity: var(--ui-opacity);
		transition:
			transform 0.1s ease,
			opacity 0.1s ease;
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
