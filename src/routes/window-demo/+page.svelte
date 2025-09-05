<script lang="ts">
	import Window from '$lib/Window.svelte';
</script>

<svelte:head>
	<title>Window Demo - ye66ow</title>
	<meta name="description" content="Демонстрация ретро-окна Windows 98/XP" />
</svelte:head>

<div class="demo-container">
	<h1>Демонстрация ретро-окна</h1>
	
	<!-- Окно с изображением -->
	<Window 
		title="MyPhoto2006" 
		icon="🖼️"
		image="/img2.webp"
		text1="Image Loaded"
		text2="@vancespng"
	/>
	
	<!-- Окно с телефоном по умолчанию -->
	<Window 
		title="MyPhone2006" 
		icon="💾"
		text1="Connected"
		text2="@vancespng"
	/>
</div>

<style>
	.demo-container {
		min-height: 100vh;
		background: #008080;
		padding: 20px;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
	}
	
	h1 {
		color: #ffffff;
		text-align: center;
		margin-bottom: 40px;
		text-shadow: 2px 2px 0 rgba(0, 0, 0, 0.8);
	}
</style>
