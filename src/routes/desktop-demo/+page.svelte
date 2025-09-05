<script lang="ts">
	import Window from '$lib/Window.svelte';
	import DesktopFile from '$lib/DesktopFile.svelte';

	let openWindows: any[] = [];
	let desktopFiles: any[] = [];

	// Обработчики событий
	function handleWindowClose(event: Event) {
		const customEvent = event as CustomEvent;
		const windowData = customEvent.detail;
		const fileName = `${windowData.title.toLowerCase().replace(/\s+/g, '-')}.${windowData.text1 === 'Music Release' ? 'mp3' : 'txt'}`;

		// Скрываем окно вместо удаления
		const windowIndex = openWindows.findIndex((window) => window.title === windowData.title);
		if (windowIndex !== -1) {
			openWindows[windowIndex] = {
				...openWindows[windowIndex],
				isVisible: false
			};
		}

		// Проверяем, есть ли уже файл с таким именем
		const existingFileIndex = desktopFiles.findIndex((file) => file.fileName === fileName);

		if (existingFileIndex === -1) {
			// Создаем новый файл только если его еще нет
			desktopFiles = [
				...desktopFiles,
				{
					id: Date.now(),
					fileName,
					fileIcon: windowData.icon,
					fileType: windowData.text1 === 'Music Release' ? 'mp3' : 'txt',
					windowData,
					x: windowData.x || Math.random() * (window.innerWidth - 120),
					y: windowData.y || Math.random() * (window.innerHeight - 140)
				}
			];
		} else {
			// Обновляем существующий файл новыми данными, сохраняя позицию
			desktopFiles[existingFileIndex] = {
				...desktopFiles[existingFileIndex],
				windowData,
				fileIcon: windowData.icon,
				fileType: windowData.text1 === 'Music Release' ? 'mp3' : 'txt'
			};
		}
	}

	function handleFileToggle(event: Event) {
		const customEvent = event as CustomEvent;
		const fileData = customEvent.detail;
		const fileName = `${fileData.title.toLowerCase().replace(/\s+/g, '-')}.${fileData.text1 === 'Music Release' ? 'mp3' : 'txt'}`;

		// Ищем соответствующее окно
		const windowIndex = openWindows.findIndex((window) => window.title === fileData.title);

		if (windowIndex !== -1) {
			// Переключаем видимость существующего окна
			openWindows[windowIndex] = {
				...openWindows[windowIndex],
				isVisible: !openWindows[windowIndex].isVisible
			};
		} else {
			// Создаем новое окно, если его нет
			openWindows = [
				...openWindows,
				{
					id: Date.now(),
					...fileData,
					x: fileData.x || (window.innerWidth - 400) / 2,
					y: fileData.y || (window.innerHeight - 500) / 2,
					isVisible: true
				}
			];
		}
	}

	// Инициализируем начальное окно
	function initDemo() {
		openWindows = [
			{
				id: 1,
				title: 'Test Window',
				icon: '🧪',
				image: '/img2.webp',
				text1: 'Music Release',
				text2: '@ye66ow',
				x: 200,
				y: 200,
				isVisible: true
			}
		];

		// Создаем файл на рабочем столе
		desktopFiles = [
			{
				id: 2,
				fileName: 'test-window.mp3',
				fileIcon: '🧪',
				fileType: 'mp3',
				windowData: {
					title: 'Test Window',
					icon: '🧪',
					image: '/img2.webp',
					text1: 'Music Release',
					text2: '@ye66ow',
					x: 200,
					y: 200
				},
				x: 50,
				y: 50
			}
		];
	}

	// Добавляем обработчики событий
	if (typeof window !== 'undefined') {
		window.addEventListener('windowClose', handleWindowClose);
		window.addEventListener('fileToggle', handleFileToggle);
		initDemo();
	}
</script>

<svelte:head>
	<title>Desktop Demo - ye66ow</title>
	<meta name="description" content="Демонстрация рабочего стола Windows 95" />
</svelte:head>

<div class="desktop-container">
	<!-- Окна -->
	{#each openWindows as window}
		{#if window.isVisible !== false}
			<Window
				title={window.title}
				icon={window.icon}
				image={window.image}
				text1={window.text1}
				text2={window.text2}
				initialX={window.x}
				initialY={window.y}
			/>
		{/if}
	{/each}

	<!-- Файлы на рабочем столе -->
	{#each desktopFiles as file}
		<DesktopFile
			fileName={file.fileName}
			fileIcon={file.fileIcon}
			fileType={file.fileType}
			windowData={file.windowData}
			initialX={file.x}
			initialY={file.y}
		/>
	{/each}

	<!-- Инструкции -->
	<div class="instructions">
		<h2>Инструкции:</h2>
		<ul>
			<li>Нажмите "×" чтобы закрыть окно → оно станет файлом на рабочем столе</li>
			<li>Кликните по файлу → он откроется как окно</li>
			<li>Перетаскивайте окна и файлы</li>
			<li>Кнопка "_" сворачивает окно в синюю полоску</li>
			<li>Кнопка "□" разворачивает окно на весь экран</li>
		</ul>
	</div>
</div>

<style>
	.desktop-container {
		min-height: 100vh;
		background: #008080;
		position: relative;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
	}

	.instructions {
		position: fixed;
		top: 20px;
		left: 20px;
		background: rgba(0, 0, 0, 0.8);
		color: #ffffff;
		padding: 20px;
		border-radius: 8px;
		max-width: 300px;
		z-index: 10;
	}

	.instructions h2 {
		margin: 0 0 15px 0;
		font-size: 16px;
	}

	.instructions ul {
		margin: 0;
		padding-left: 20px;
	}

	.instructions li {
		margin: 8px 0;
		font-size: 12px;
		line-height: 1.4;
	}
</style>
