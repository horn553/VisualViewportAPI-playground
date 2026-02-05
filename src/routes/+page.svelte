<script lang="ts">
	import Fab from '$lib/Fab.svelte';
	import Gradient from '$lib/Gradient.svelte';

	let supported = $state(false);
	let width = $state(0);
	let height = $state(0);
	let offsetLeft = $state(0);
	let offsetTop = $state(0);
	let scale = $state(1);

	let debounceId: ReturnType<typeof setTimeout> | null = null;

	const fmt = (value: number, digits = 2) => value.toFixed(digits);
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

		const scaleInvValue = 1 / Math.max(0.01, scale);
		console.log({
			supported: supported ? 'yes' : 'no',
			scale: fmt(scale, 3),
			offsetLeft: fmt(offsetLeft, 2),
			offsetTop: fmt(offsetTop, 2),
			width: fmt(width, 2),
			height: fmt(height, 2),
			scaleInv: fmt(scaleInvValue, 3)
		});
	};

	const scheduleUpdate = () => {
		if (debounceId !== null) {
			clearTimeout(debounceId);
		}
		debounceId = setTimeout(() => {
			debounceId = null;
			updateMetrics();
		}, 300);
	};

	$effect(() => {
		if (typeof window === 'undefined') return;

		updateMetrics();

		const vv = window.visualViewport;
		if (vv) {
			vv.addEventListener('resize', scheduleUpdate);
			vv.addEventListener('scroll', scheduleUpdate);
		}
		window.addEventListener('resize', scheduleUpdate);

		return () => {
			if (debounceId !== null) clearTimeout(debounceId);
			if (vv) {
				vv.removeEventListener('resize', scheduleUpdate);
				vv.removeEventListener('scroll', scheduleUpdate);
			}
			window.removeEventListener('resize', scheduleUpdate);
		};
	});
</script>

<Gradient />

<Fab direction="up-left" />
<Fab direction="up" />
<Fab direction="up-right" />
<Fab direction="left" />
<Fab direction="center" />
<Fab direction="right" />
<Fab direction="down-left" />
<Fab direction="down" />
<Fab direction="down-right" />
