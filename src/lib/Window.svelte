<script lang="ts">
	import { onMount } from 'svelte';

	// Пропсы компонента
	export let title: string = 'MyPhone2006';
	export let image: string = '';
	export let text1: string = 'Connected';
	export let text2: string = '@vancespng';
	export let icon: string = '💾';
	export let link: string = '';
	export let initialX: number = 0;
	export let initialY: number = 0;
	export let fileId: string = '';

	// Состояние окна
	let isMinimized = false;
	let isMaximized = false;
	let isClosed = false;
	let isInTrash = false;
	let signalStrength = 3;
	let isConnected = true;

	// Позиция и размеры окна
	let windowX = initialX;
	let windowY = initialY;
	let windowWidth = 400;
	let windowHeight = 500;
	let isMobile = false;
	let isDragging = false;
	let dragStartX = 0;
	let dragStartY = 0;
	let dragStartWindowX = 0;
	let dragStartWindowY = 0;

	// Функции управления окном
	function minimizeWindow(event?: Event) {
		if (event) {
			event.preventDefault();
			event.stopPropagation();
		}
		isMinimized = !isMinimized;
		isMaximized = false;
	}

	function maximizeWindow(event?: Event) {
		if (event) {
			event.preventDefault();
			event.stopPropagation();
		}
		if (isMaximized) {
			// Восстанавливаем размер
			isMaximized = false;
			windowX = dragStartWindowX;
			windowY = dragStartWindowY;
			windowWidth = 400;
			windowHeight = 500;
		} else {
			// Сохраняем текущую позицию
			dragStartWindowX = windowX;
			dragStartWindowY = windowY;
			// Разворачиваем на весь экран
			isMaximized = true;
			windowX = 0;
			windowY = 0;
			windowWidth = window.innerWidth;
			windowHeight = window.innerHeight;
		}
	}

	function closeWindow(event?: Event) {
		if (event) {
			event.preventDefault();
			event.stopPropagation();
		}
		// Эмитируем событие для родительского компонента
		// чтобы он мог создать файл на рабочем столе
		const closeEvent = new CustomEvent('windowClose', {
			detail: {
				fileId,
				title,
				icon,
				image,
				text1,
				text2,
				x: windowX, // Передаем текущую позицию
				y: windowY // Передаем текущую позицию
			}
		});
		window.dispatchEvent(closeEvent);

		// Закрываем окно
		isClosed = true;
	}

	function restoreFromTrash() {
		isClosed = false;
		isInTrash = false;
		// Восстанавливаем размеры
		windowWidth = 400;
		windowHeight = 500;
		// Возвращаем в центр экрана
		windowX = (window.innerWidth - 400) / 2;
		windowY = (window.innerHeight - 500) / 2;
	}

	// Функции перетаскивания (поддержка мыши и touch)
	function startDrag(event: MouseEvent | TouchEvent) {
		if (isMaximized || isInTrash) return;
		event.preventDefault();
		isDragging = true;

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		dragStartX = clientX;
		dragStartY = clientY;
		dragStartWindowX = windowX;
		dragStartWindowY = windowY;
		document.addEventListener('mousemove', handleDrag);
		document.addEventListener('mouseup', stopDrag);
		document.addEventListener('touchmove', handleDrag, { passive: false });
		document.addEventListener('touchend', stopDrag);
	}

	function handleDrag(event: MouseEvent | TouchEvent) {
		if (!isDragging) return;
		event.preventDefault();

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		windowX = dragStartWindowX + (clientX - dragStartX);
		windowY = dragStartWindowY + (clientY - dragStartY);

		// Ограничиваем перемещение границами экрана
		windowX = Math.max(0, Math.min(windowX, window.innerWidth - windowWidth));
		windowY = Math.max(0, Math.min(windowY, window.innerHeight - windowHeight));
	}

	function stopDrag() {
		isDragging = false;
		document.removeEventListener('mousemove', handleDrag);
		document.removeEventListener('mouseup', stopDrag);
		document.removeEventListener('touchmove', handleDrag);
		document.removeEventListener('touchend', stopDrag);
	}

	// Функция для открытия ссылки
	function openLink(event: Event) {
		if (link) {
			event.preventDefault();
			event.stopPropagation();
			window.open(link, '_blank', 'noopener,noreferrer');
		}
	}

	// Функция для получения даты релиза
	function getReleaseDate(title: string) {
		if (title.toLowerCase().includes('still alive')) {
			return '14 03 2024';
		} else if (title.toLowerCase().includes('plastic river')) {
			return '20 06 2024';
		}
		return '2024';
	}

	// Анимация сигнала
	onMount(() => {
		// Определяем мобильное устройство
		isMobile = window.innerWidth <= 768;

		// Устанавливаем размеры для мобильных устройств
		if (isMobile) {
			windowWidth = Math.min(280, window.innerWidth - 40);
			windowHeight = Math.min(350, window.innerHeight - 100);
		}

		const interval = setInterval(() => {
			signalStrength = Math.floor(Math.random() * 4) + 1;
		}, 2000);

		return () => clearInterval(interval);
	});
</script>

{#if !isClosed || isInTrash}
	<div
		class="retro-window"
		class:minimized={isMinimized}
		class:maximized={isMaximized}
		class:in-trash={isInTrash}
		style="left: {windowX}px; top: {windowY}px; width: {windowWidth}px; height: {windowHeight}px;"
		on:click={isInTrash ? restoreFromTrash : undefined}
		on:keydown={isInTrash ? (e) => e.key === 'Enter' && restoreFromTrash() : undefined}
		role={isInTrash ? 'button' : undefined}
		tabindex={isInTrash ? 0 : undefined}
		aria-label={isInTrash ? `Восстановить окно ${title}` : undefined}
	>
		<!-- Заголовок окна -->
		<div
			class="title-bar"
			on:mousedown={isInTrash ? undefined : startDrag}
			on:touchstart={isInTrash ? undefined : startDrag}
			role="button"
			tabindex="0"
			aria-label="Перетащить окно"
		>
			<div class="title-bar-left">
				<div class="floppy-icon">{icon}</div>
				<span class="window-title">{title}</span>
			</div>
			<div class="window-controls">
				{#if !isInTrash}
					<button
						class="control-btn minimize"
						on:click={minimizeWindow}
						on:touchend={minimizeWindow}
						title="Свернуть">_</button
					>
					<button
						class="control-btn maximize"
						on:click={maximizeWindow}
						on:touchend={maximizeWindow}
						title="Развернуть">□</button
					>
				{/if}
				<button
					class="control-btn close"
					on:click={closeWindow}
					on:touchend={closeWindow}
					title="Закрыть">×</button
				>
			</div>
		</div>

		{#if !isInTrash && !isMinimized}
			<!-- Основное содержимое -->
			<div class="window-content">
				{#if $$slots.default}
					<slot />
				{:else if image}
					<!-- Обложка на весь блок -->
					<div
						class="cover-image"
						on:click={openLink}
						on:touchend={openLink}
						on:keydown={(e) => e.key === 'Enter' && openLink(e)}
						role="button"
						tabindex="0"
						aria-label="Открыть ссылку на релиз {title}"
					>
						<img src={image} alt={title} />
					</div>
				{/if}
			</div>

			<!-- Статус бар -->
			<div class="status-bar">
				<div class="status-left">
					<span class="status-text">ye66ow</span>
				</div>
				<div class="status-divider">|</div>
				<div class="status-right">
					<span class="status-text">{getReleaseDate(title)}</span>
				</div>
			</div>
		{:else if isInTrash}
			<!-- Содержимое в корзине - только иконка и название -->
			<div class="trash-content">
				<div class="trash-icon">{icon}</div>
				<div class="trash-title">{title}</div>
			</div>
		{/if}
	</div>
{/if}

<style>
	.retro-window {
		position: fixed;
		background: #c0c0c0;
		border: 3px outset #c0c0c0;
		box-shadow:
			4px 4px 0 #404040,
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		z-index: 1000;
		transition: all 0.3s ease;
		user-select: none;
	}

	.retro-window.minimized {
		height: 30px !important;
		overflow: hidden;
	}

	.retro-window.in-trash {
		width: 80px !important;
		height: 80px !important;
		cursor: pointer;
		z-index: 999;
	}

	.retro-window.maximized {
		width: 100vw !important;
		height: 100vh !important;
		top: 0 !important;
		left: 0 !important;
		transform: none !important;
	}

	/* Заголовок окна */
	.title-bar {
		background: linear-gradient(90deg, #000080 0%, #0000ff 100%);
		color: #ffffff;
		padding: 4px 8px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 12px;
		font-weight: bold;
		height: 20px;
		box-shadow:
			inset 0 1px 0 rgba(255, 255, 255, 0.4),
			inset 0 -1px 0 rgba(0, 0, 0, 0.3);
		cursor: move;
		user-select: none;
	}

	.title-bar-left {
		display: flex;
		align-items: center;
		gap: 6px;
	}

	.floppy-icon {
		font-size: 14px;
	}

	.window-title {
		text-shadow: 1px 1px 0 rgba(0, 0, 0, 0.5);
	}

	.window-controls {
		display: flex;
		gap: 2px;
	}

	.control-btn {
		width: 16px;
		height: 14px;
		background: #c0c0c0;
		border: 1px outset #c0c0c0;
		font-size: 10px;
		font-weight: bold;
		color: #000;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.1s ease;
	}

	.control-btn:hover {
		background: #d4d0c8;
	}

	.control-btn:active {
		background: #a0a0a0;
		border: 1px inset #c0c0c0;
	}

	/* Основное содержимое */
	.window-content {
		height: calc(100% - 46px);
		display: flex;
		flex-direction: column;
		position: relative;
	}

	.top-section {
		background: #add8e6;
		height: 60px;
		display: flex;
		align-items: center;
		padding: 0 20px;
		border-bottom: 1px solid #808080;
	}

	.signal-indicator {
		display: flex;
		align-items: end;
		gap: 2px;
	}

	.signal-bar {
		width: 4px;
		background: #000;
		border-radius: 1px;
		animation: signalPulse 2s ease-in-out infinite;
	}

	@keyframes signalPulse {
		0%,
		100% {
			opacity: 0.6;
		}
		50% {
			opacity: 1;
		}
	}

	.bottom-section {
		background: #e6e6fa;
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	/* Телефон */
	.phone-container {
		position: relative;
		display: flex;
		align-items: center;
		gap: 20px;
	}

	.phone {
		width: 120px;
		height: 200px;
		position: relative;
		transform: rotate(-15deg);
		animation: phoneFloat 3s ease-in-out infinite;
	}

	@keyframes phoneFloat {
		0%,
		100% {
			transform: rotate(-15deg) translateY(0px);
		}
		50% {
			transform: rotate(-15deg) translateY(-5px);
		}
	}

	.phone-top {
		width: 100%;
		height: 60%;
		background: #ffb6c1;
		border: 2px solid #87ceeb;
		border-radius: 8px 8px 0 0;
		position: relative;
	}

	.phone-screen {
		width: 80%;
		height: 70%;
		background: #000080;
		margin: 8px auto;
		border-radius: 4px;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	.screen-content {
		color: #ffffff;
		font-size: 8px;
		text-align: center;
	}

	.screen-text {
		margin: 2px 0;
	}

	.phone-bottom {
		width: 100%;
		height: 40%;
		background: #ffb6c1;
		border: 2px solid #87ceeb;
		border-radius: 0 0 8px 8px;
		border-top: none;
		padding: 8px;
	}

	.keypad {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4px;
	}

	.nav-pad {
		width: 30px;
		height: 30px;
		background: #87ceeb;
		border-radius: 50%;
		position: relative;
		margin: 4px 0;
	}

	.nav-center {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		width: 8px;
		height: 8px;
		background: #000;
		border-radius: 50%;
	}

	.nav-up,
	.nav-down,
	.nav-left,
	.nav-right {
		position: absolute;
		background: #87ceeb;
		color: #000;
		font-size: 8px;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.nav-up {
		top: 2px;
		left: 50%;
		transform: translateX(-50%);
		width: 12px;
		height: 8px;
	}
	.nav-down {
		bottom: 2px;
		left: 50%;
		transform: translateX(-50%);
		width: 12px;
		height: 8px;
	}
	.nav-left {
		left: 2px;
		top: 50%;
		transform: translateY(-50%);
		width: 8px;
		height: 12px;
	}
	.nav-right {
		right: 2px;
		top: 50%;
		transform: translateY(-50%);
		width: 8px;
		height: 12px;
	}

	.keypad-keys {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 2px;
		width: 100%;
	}

	.key-row {
		display: contents;
	}

	.key {
		width: 20px;
		height: 16px;
		background: #87ceeb;
		border: 1px solid #000;
		border-radius: 2px;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 8px;
		font-weight: bold;
		color: #000;
	}

	/* Наушники */
	.headphones {
		position: absolute;
		left: 100px;
		top: 20px;
		z-index: 2;
	}

	.headphone-cord {
		width: 40px;
		height: 2px;
		background: #ffffff;
		position: absolute;
		top: 10px;
		left: -20px;
		transform: rotate(15deg);
	}

	.headphone-left {
		width: 12px;
		height: 12px;
		background: #ffffff;
		border-radius: 50%;
		position: absolute;
		top: 8px;
		left: -15px;
	}

	.headphone-right {
		width: 12px;
		height: 12px;
		background: #ffffff;
		border-radius: 50%;
		position: absolute;
		top: 8px;
		left: 5px;
	}

	/* Персонаж */
	.character {
		position: absolute;
		left: 120px;
		top: 30px;
		z-index: 3;
	}

	.character-body {
		font-size: 16px;
		animation: characterBounce 2s ease-in-out infinite;
	}

	@keyframes characterBounce {
		0%,
		100% {
			transform: translateY(0px);
		}
		50% {
			transform: translateY(-3px);
		}
	}

	/* Скроллбар */
	.scrollbar {
		position: absolute;
		right: 0;
		top: 0;
		width: 16px;
		height: 100%;
		background: #c0c0c0;
		border-left: 1px solid #808080;
	}

	.scrollbar-track {
		width: 100%;
		height: 100%;
		position: relative;
	}

	.scrollbar-thumb {
		width: 12px;
		height: 40px;
		background: #808080;
		border: 1px outset #c0c0c0;
		position: absolute;
		top: 20px;
		left: 2px;
	}

	/* Статус бар */
	.status-bar {
		background: #c0c0c0;
		padding: 2px 8px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 10px;
		font-weight: bold;
		border-top: 1px solid #808080;
		height: 18px;
	}

	.status-divider {
		color: #808080;
		font-weight: bold;
		margin: 0 8px;
	}

	.status-text {
		color: #000;
	}

	/* Стили для изображения */
	.cover-image {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #e6e6fa;
		cursor: pointer;
	}

	.cover-image img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		border: 2px inset #c0c0c0;
		box-shadow:
			inset 2px 2px 0 #ffffff,
			inset -2px -2px 0 #808080;
		pointer-events: none;
	}

	/* Стили для корзины */
	.trash-content {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 8px;
		background: #c0c0c0;
	}

	.trash-icon {
		font-size: 24px;
		margin-bottom: 4px;
	}

	.trash-title {
		font-size: 8px;
		font-weight: bold;
		color: #000;
		text-align: center;
		line-height: 1;
		word-break: break-all;
	}

	.in-trash .title-bar {
		height: 16px;
		padding: 2px 4px;
		font-size: 8px;
	}

	.in-trash .title-bar-left {
		gap: 2px;
	}

	.in-trash .floppy-icon {
		font-size: 8px;
	}

	.in-trash .window-title {
		font-size: 8px;
	}

	/* Адаптивность для мобильных устройств */
	@media (max-width: 768px) {
		.retro-window {
			max-width: 80vw;
			max-height: 70vh;
		}

		.title-bar {
			height: 24px;
			padding: 4px 8px;
			font-size: 12px;
		}

		.window-controls {
			gap: 3px;
		}

		.control-btn {
			width: 18px;
			height: 18px;
			font-size: 10px;
			touch-action: manipulation;
			-webkit-tap-highlight-color: transparent;
		}

		.window-content {
			padding: 8px;
			height: calc(100% - 42px);
		}

		.status-bar {
			height: 16px;
			padding: 2px 6px;
			font-size: 8px;
		}

		.cover-image img {
			max-width: 100%;
			max-height: 100%;
		}
	}

	/* Адаптивность для очень маленьких экранов */
	@media (max-width: 480px) {
		.retro-window {
			max-width: 95vw;
			max-height: 85vh;
		}

		.title-bar {
			height: 26px;
			padding: 4px 8px;
			font-size: 12px;
		}

		.control-btn {
			width: 20px;
			height: 20px;
			font-size: 12px;
			touch-action: manipulation;
			-webkit-tap-highlight-color: transparent;
		}
	}
</style>
