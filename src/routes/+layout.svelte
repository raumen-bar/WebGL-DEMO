<script>
	import './app.css';

	import { onMount } from 'svelte';
	import { screenType, isIframe, renderer } from '$lib/store/store';

	import Header from '$lib/components/header/header.svelte';
	import Footer from '$lib/components/footer/footer.svelte';

	let Geometry;

	onMount(async () => {

		if ($renderer === 'three') {
			const moduleThree = await import('$lib/graphics/three.svelte');
			Geometry = moduleThree.default;
		}

		if ($renderer === 'regl') {
			const moduleRegl = await import('$lib/graphics/regl.svelte');
			Geometry = moduleRegl.default;
		}

		if ($renderer === 'webgl') {
			const moduleWebGL = await import('$lib/graphics/webgl.svelte');
			Geometry = moduleWebGL.default;
		}

		function getDeviceType() {
			const width = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth;

			if ('ontouchstart' in window || navigator.maxTouchPoints > 0 || navigator.msMaxTouchPoints > 0) {
				// This is a device which supports touch
				if (width <= 767) {
					// Mobile
					return 1;
				} else {
					// Tablet
					return 2;
				}
			} else {
				// This is likely a laptop or desktop
				return 3;
			}
		}

		screenType.set(getDeviceType());
		isIframe.set(window.location !== window.parent.location);
	});

</script>

<svelte:head>
	<title>WebGL DEMO // AUFBAU</title>
	<meta name="description" content="AUFBAU :: WEB 1.0 Re-Imagined" />

	<link
		rel="preload"
		href="/fonts/NB-Architekt-Pro-Light.woff"
		as="font"
		type="font/woff"
		crossorigin="anonymous"
	/>

	<link
		rel="preload"
		href="/fonts/NB-Architekt-Pro-Bold.woff"
		as="font"
		type="font/woff"
		crossorigin="anonymous"
	/>

	<link rel="preload" href="icons/cv.svg" as="image">
	<link rel="preload" href="icons/insta.svg" as="image">
	<link rel="preload" href="icons/mail.svg" as="image">

</svelte:head>

{#key $renderer}
<svelte:component this={Geometry} />
{/key}

{#if $screenType}
	<main>
		<header>
			<Header />
		</header>

		<body>
			<slot />
		</body>

		<!-- {#if $screenType == 3}
		<footer>
			<Footer />
		</footer>
		{/if} -->
	</main>
{/if}

<style>
	main {
		display: flex;
		flex-direction: column;
		height: 100dvh;
	}
	
	header {
		position: absolute;
		top: 0;
		width: 100%;
	}

	footer {
		position: absolute;
		bottom: 0;
		width: 100%;
	}

	body {
		display: flex;
		flex-direction: column;
		/* padding: calc(1 * var(--margin)); */
		width: 100%;
		height: 100%;
	}
</style>
