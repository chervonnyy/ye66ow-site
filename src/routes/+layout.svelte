<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
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

	<!-- Fonts -->
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600;700;800&family=Outfit:wght@100;200;300;400&display=swap"
		rel="stylesheet"
	/>

	<!-- Theme Colors -->
	<meta name="theme-color" content="#000000" />
	<meta name="msapplication-navbutton-color" content="#000000" />
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
</svelte:head>

{@render children?.()}
