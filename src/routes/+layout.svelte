<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	let { children } = $props();

	onMount(() => {
		if (browser && 'serviceWorker' in navigator) {
			// Register service worker
			navigator.serviceWorker
				.register('/sw.js')
				.then((registration) => {
					console.log('SW registered: ', registration);
				})
				.catch((registrationError) => {
					console.log('SW registration failed: ', registrationError);
				});

			// Handle app updates
			navigator.serviceWorker.addEventListener('controllerchange', () => {
				window.location.reload();
			});
		}
	});
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<link rel="manifest" href="/site.webmanifest" />

	<!-- PWA Meta Tags -->
	<meta name="application-name" content="ye66ow" />
	<meta name="apple-mobile-web-app-title" content="ye66ow" />
	<meta name="msapplication-TileColor" content="#000000" />
	<meta name="msapplication-tap-highlight" content="no" />

	<!-- Apple Touch Icons -->
	<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
	<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />

	<!-- Standard Favicons -->
	<link rel="icon" type="image/png" sizes="32x32" href="/favicon-32.png" />
	<link rel="icon" type="image/png" sizes="192x192" href="/favicon-192.png" />
	<link rel="icon" type="image/png" sizes="512x512" href="/favicon-512.png" />
	<link rel="icon" href="/favicon.ico" />

	<!-- Theme Colors -->
	<meta name="theme-color" content="#000000" />
	<meta name="msapplication-navbutton-color" content="#000000" />
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
</svelte:head>

{@render children?.()}
