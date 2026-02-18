<script lang="ts">
	import VersionSwitcher from '$lib/versions/VersionSwitcher.svelte';
	import V1 from '$lib/versions/V1.svelte';

	let pattern = $state(0);
	let rampIdx = $state(10);
	let inverted = $state(false);
	let speed = $state(1);
	let asciiFont = $state(0);
	let showPanel = $state(false);
</script>

<svelte:head>
	<title>ye66ow</title>
	<meta name="description" content="peacemusic · new music out now" />
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"
	/>
	<meta name="apple-mobile-web-app-capable" content="yes" />

	<!-- Open Graph -->
	<meta property="og:type" content="website" />
	<meta property="og:title" content="ye66ow" />
	<meta property="og:description" content="peacemusic · new music out now" />
	<meta property="og:url" content="https://ye66ow.com" />
	<meta property="og:image" content="https://ye66ow.com/covers/still-alive-cover-1200.webp" />

	<!-- Twitter -->
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="twitter:title" content="ye66ow" />
	<meta name="twitter:description" content="peacemusic · new music out now" />
	<meta name="twitter:image" content="https://ye66ow.com/covers/still-alive-cover-1200.webp" />
</svelte:head>

<V1 bind:pattern bind:rampIdx bind:inverted bind:speed bind:asciiFont bind:showPanel />

<VersionSwitcher bind:current={pattern} bind:ramp={rampIdx} bind:inverted bind:speed bind:font={asciiFont} visible={showPanel} />
