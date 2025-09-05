<script lang="ts">
	import { onMount, onDestroy } from 'svelte';

	// Пропсы компонента
	export let fileName: string;
	export let fileType: string = 'mp3';
	export let windowData: any = null; // Данные для восстановления окна
	export let initialX: number = 0;
	export let initialY: number = 0;

	let isSelected = false;
	let isDragging = false;
	let dragStartX = 0;
	let dragStartY = 0;
	let fileX = initialX;
	let fileY = initialY;
	let dragThreshold = 5; // Минимальное расстояние для начала перетаскивания
	let hasMoved = false;

	// Функция для переключения видимости окна
	function toggleWindow() {
		if (windowData) {
			const toggleEvent = new CustomEvent('fileToggle', {
				detail: {
					...windowData,
					fileId: windowData.fileId || `file-${Math.random()}`
				}
			});
			window.dispatchEvent(toggleEvent);
		}
	}

	// Функции перетаскивания (поддержка мыши и touch)
	function startDrag(event: MouseEvent | TouchEvent) {
		event.preventDefault();
		hasMoved = false;

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		dragStartX = clientX - fileX;
		dragStartY = clientY - fileY;

		document.addEventListener('mousemove', handleDrag);
		document.addEventListener('mouseup', stopDrag);
		document.addEventListener('touchmove', handleDrag, { passive: false });
		document.addEventListener('touchend', stopDrag);
	}

	function handleDrag(event: MouseEvent | TouchEvent) {
		event.preventDefault();

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		const deltaX = clientX - (dragStartX + fileX);
		const deltaY = clientY - (dragStartY + fileY);
		const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);

		// Начинаем перетаскивание только если мышь/палец сдвинулся достаточно далеко
		if (distance > dragThreshold && !isDragging) {
			isDragging = true;
			hasMoved = true;
		}

		if (isDragging) {
			fileX = clientX - dragStartX;
			fileY = clientY - dragStartY;

			// Ограничиваем перемещение границами экрана
			const isMobile = window.innerWidth <= 768;
			const fileWidth = isMobile ? 100 : 180;
			const fileHeight = isMobile ? 120 : 210;
			fileX = Math.max(0, Math.min(fileX, window.innerWidth - fileWidth));
			fileY = Math.max(0, Math.min(fileY, window.innerHeight - fileHeight));
		}
	}

	function stopDrag(event: MouseEvent | TouchEvent) {
		document.removeEventListener('mousemove', handleDrag);
		document.removeEventListener('mouseup', stopDrag);
		document.removeEventListener('touchmove', handleDrag);
		document.removeEventListener('touchend', stopDrag);

		// Если было перетаскивание, не открываем окно
		if (isDragging) {
			isDragging = false;
			hasMoved = false;
			return;
		}

		// Если не было перетаскивания, открываем окно
		if (!hasMoved) {
			toggleWindow();
		}
	}

	// Обработка клика
	function handleClick(event: MouseEvent) {
		// Клик обрабатывается в stopDrag
	}

	// Обработка двойного клика
	function handleDoubleClick(event: MouseEvent) {
		event.preventDefault();
		if (!isDragging && !hasMoved) {
			toggleWindow();
		}
	}

	onMount(() => {
		// Слушаем события от окон
		const handleWindowClose = (event: CustomEvent) => {
			windowData = event.detail;
		};

		window.addEventListener('windowClose', handleWindowClose as EventListener);

		return () => {
			window.removeEventListener('windowClose', handleWindowClose as EventListener);
		};
	});
</script>

<div
	class="desktop-file"
	class:selected={isSelected}
	class:dragging={isDragging}
	class:mp3={fileType === 'mp3'}
	class:wav={fileType === 'wav'}
	class:txt={fileType === 'txt'}
	class:jpg={fileType === 'jpg'}
	class:png={fileType === 'png'}
	style="left: {fileX}px; top: {fileY}px;"
	on:mousedown={startDrag}
	on:touchstart={startDrag}
	on:click={handleClick}
	on:dblclick={handleDoubleClick}
	role="button"
	tabindex="0"
	aria-label="Файл {fileName}"
	on:keydown={(e) => e.key === 'Enter' && toggleWindow()}
>
	<div class="file-icon">
		<div class="default-file-icon">
			<div class="file-corner"></div>
			<div class="file-content"></div>
		</div>
	</div>
	<div class="file-name">
		<div class="file-name-backdrop"></div>
		<span class="file-name-text">{fileName}</span>
	</div>
</div>

<style>
	.desktop-file {
		position: fixed;
		width: 180px;
		height: 210px;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: flex-start;
		padding: 18px 12px;
		cursor: grab;
		user-select: none;
		z-index: 500;
		transition:
			transform 0.2s ease,
			box-shadow 0.2s ease,
			opacity 0.2s ease;
	}

	.desktop-file.selected {
		background: rgba(0, 0, 255, 0.2);
		border-radius: 4px;
	}

	.desktop-file.dragging {
		z-index: 1000;
		cursor: grabbing;
	}

	.file-icon {
		width: 72px;
		height: 72px;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 48px;
		margin-bottom: 12px;
		background: #c0c0c0;
		border: 3px outset #c0c0c0;
		border-radius: 3px;
		box-shadow:
			3px 3px 0 #808080,
			inset 2px 2px 0 #ffffff;
		position: relative;
	}

	.file-name {
		position: relative;
		width: 100%;
		text-align: center;
		line-height: 1.2;
		max-width: 100%;
		overflow: hidden;
	}

	.file-name-backdrop {
		position: absolute;
		top: -3px;
		left: -6px;
		right: -6px;
		bottom: -3px;
		background: #000;
		border: 1px solid #fff;
		border-radius: 3px;
		z-index: -1;
		box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
	}

	.file-name-text {
		position: relative;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		font-size: 18px;
		font-weight: bold;
		color: #fff;
		text-shadow: 1px 1px 0 rgba(0, 0, 0, 0.8);
		z-index: 1;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	/* Дефолтная иконка файла */
	.default-file-icon {
		position: relative;
		width: 100%;
		height: 100%;
		background: #ffffff;
		border: 1px solid #000;
		overflow: hidden;
	}

	.file-corner {
		position: absolute;
		top: 0;
		right: 0;
		width: 0;
		height: 0;
		border-left: 12px solid transparent;
		border-top: 12px solid #e0e0e0;
	}

	.file-content {
		padding: 8px 6px 6px 8px;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	/* Адаптивность */
	@media (max-width: 480px) {
		.desktop-file {
			width: 100px;
			height: 120px;
			padding: 8px 4px;
		}

		.file-icon {
			width: 40px;
			height: 40px;
			font-size: 28px;
		}

		.default-file-icon {
			border-width: 1px;
		}

		.file-corner {
			border-left-width: 8px;
			border-top-width: 8px;
		}

		.file-content {
			padding: 6px 4px 4px 6px;
		}

		.file-name-text {
			font-size: 12px;
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}
	}
</style>
