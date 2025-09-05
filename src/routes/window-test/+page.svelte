<script lang="ts">
	import Window from '$lib/Window.svelte';
</script>

<svelte:head>
	<title>Window Test - ye66ow</title>
	<meta name="description" content="Тестирование ретро-окна Windows 98/XP" />
</svelte:head>

<div class="test-container">
	<h1>Тестирование ретро-окна</h1>
	<p>Попробуйте:</p>
	<ul>
		<li>Кнопка "_" - сворачивает окно в синюю линию</li>
		<li>Кнопка "□" - разворачивает на весь экран</li>
		<li>Кнопка "×" - закрывает окно в корзину (правый нижний угол)</li>
		<li>Клик по окну в корзине - восстанавливает его</li>
		<li>Перетаскивание за заголовок</li>
	</ul>

	<!-- Тестовое окно -->
	<Window
		title="Test Window"
		icon="🧪"
		image="/img2.webp"
		text1="Testing Mode"
		text2="@ye66ow"
		initialX={200}
		initialY={200}
	/>
</div>

<style>
	.test-container {
		min-height: 100vh;
		background: #008080;
		padding: 20px;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		color: #ffffff;
	}

	h1 {
		text-align: center;
		margin-bottom: 20px;
		text-shadow: 2px 2px 0 rgba(0, 0, 0, 0.8);
	}

	ul {
		background: rgba(0, 0, 0, 0.3);
		padding: 20px;
		border-radius: 8px;
		max-width: 600px;
		margin: 0 auto 40px;
	}

	li {
		margin: 8px 0;
		line-height: 1.4;
	}
</style>
