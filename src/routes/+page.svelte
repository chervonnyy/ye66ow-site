<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import releasesData from '$lib/releases.json';
	import Window from '$lib/Window.svelte';
	import DesktopFile from '$lib/DesktopFile.svelte';

	let asciiContainer: HTMLPreElement;
	let animationId: number;
	let time = 0;
	let width = 120;
	let height = 60;

	// Фиксированный шрифт JetBrains Mono
	let currentFont = 'tech';

	// Переменная для скрытия блока релизов
	let isReleasesVisible = false;

	// Функция для переключения видимости блока
	function toggleReleases(event: Event) {
		console.log('toggleReleases called!', isReleasesVisible);
		event.preventDefault();
		event.stopPropagation();
		isReleasesVisible = !isReleasesVisible;
		console.log('isReleasesVisible now:', isReleasesVisible);
	}

	// Случайные позиции для окон
	let windowPositions = [
		{ x: 100, y: 150 },
		{ x: 300, y: 200 }
	];

	// Файлы на рабочем столе
	let desktopFiles: any[] = [];
	let openWindows: any[] = [];

	// Глобальная система управления z-index
	let globalZIndex = 1000;
	let windowZIndexes = new Map<string, number>();

	// Функция для получения следующего z-index
	function getNextZIndex(): number {
		globalZIndex += 1;
		return globalZIndex;
	}

	// Функция для поднятия окна наверх
	function bringWindowToFront(windowId: string): number {
		const newZIndex = getNextZIndex();
		windowZIndexes.set(windowId, newZIndex);
		return newZIndex;
	}

	// Функция для получения z-index окна
	function getWindowZIndex(windowId: string): number {
		return windowZIndexes.get(windowId) || 1000;
	}

	// Генерируем красивые позиции для окон с диагональным размещением
	function generateRandomPositions() {
		if (!browser) return;

		const isMobile = window.innerWidth <= 768;
		const windowWidth = isMobile ? Math.min(280, window.innerWidth - 40) : 400;
		const windowHeight = isMobile ? Math.min(350, window.innerHeight - 100) : 500;

		const maxX = window.innerWidth - windowWidth;
		const maxY = window.innerHeight - windowHeight;

		// Для мобильных устройств размещаем окна с диагональным смещением
		if (isMobile) {
			// Добавляем небольшую случайность для мобильных
			const randomOffsetX = (Math.random() - 0.5) * 20; // ±10px случайности
			const randomOffsetY = (Math.random() - 0.5) * 20;

			windowPositions = [
				{
					x: Math.max(20, Math.min(maxX, 20)),
					y: Math.max(50, Math.min(maxY, 50))
				},
				{
					x: Math.max(20, Math.min(maxX, 20 + 60 + randomOffsetX)), // Смещение по X с случайностью
					y: Math.max(50, Math.min(maxY, 50 + 80 + randomOffsetY)) // Смещение по Y с случайностью
				}
			];
		} else {
			// Для десктопа создаем красивое диагональное размещение
			const centerX = maxX / 2;
			const centerY = maxY / 2;

			// Добавляем небольшую случайность для более естественного вида
			const randomOffset1 = (Math.random() - 0.5) * 40; // ±20px случайности
			const randomOffset2 = (Math.random() - 0.5) * 40;

			// Первое окно - левый верхний угол с отступом
			const offset1 = 80 + randomOffset1;
			const x1 = Math.max(50, Math.min(maxX, centerX - offset1));
			const y1 = Math.max(50, Math.min(maxY, centerY - offset1));

			// Второе окно - правый нижний угол с отступом
			const offset2 = 120 + randomOffset2;
			const x2 = Math.max(50, Math.min(maxX, centerX + offset2));
			const y2 = Math.max(50, Math.min(maxY, centerY + offset2));

			windowPositions = [
				{ x: x1, y: y1 },
				{ x: x2, y: y2 }
			];
		}
	}

	// Обработчики событий для взаимодействия окон и файлов
	function handleWindowClose(event: Event) {
		const customEvent = event as CustomEvent;
		const windowData = customEvent.detail;

		// Скрываем только конкретное окно, не влияя на другие
		openWindows = openWindows.map((window) => {
			if (window.fileId === windowData.fileId) {
				return {
					...window,
					isVisible: false
				};
			}
			return window; // Остальные окна остаются без изменений
		});

		// Обновляем windowData в соответствующем файле
		const fileIndex = desktopFiles.findIndex((file) => file.fileId === windowData.fileId);
		if (fileIndex !== -1) {
			desktopFiles[fileIndex] = {
				...desktopFiles[fileIndex],
				windowData: {
					...desktopFiles[fileIndex].windowData,
					...windowData
				}
			};
		}
	}

	function handleFileToggle(event: Event) {
		const customEvent = event as CustomEvent;
		const fileData = customEvent.detail;

		// Ищем соответствующее окно по уникальному ID файла
		const windowIndex = openWindows.findIndex((window) => window.fileId === fileData.fileId);

		if (windowIndex !== -1) {
			// Переключаем видимость существующего окна независимо от других
			openWindows = openWindows.map((window, index) => {
				if (index === windowIndex) {
					return {
						...window,
						isVisible: !window.isVisible
					};
				}
				return window; // Остальные окна остаются без изменений
			});
		} else {
			// Создаем новое окно, если его нет
			const windowId = fileData.fileId;
			const zIndex = getNextZIndex();
			windowZIndexes.set(windowId, zIndex);

			openWindows = [
				...openWindows,
				{
					id: Date.now(),
					fileId: windowId,
					title: fileData.title,
					icon: fileData.icon,
					image: fileData.image,
					link: fileData.link,
					text1: fileData.text1,
					text2: fileData.text2,
					x: fileData.x || (window.innerWidth - 400) / 2,
					y: fileData.y || (window.innerHeight - 500) / 2,
					isVisible: true,
					zIndex: zIndex
				}
			];
		}
	}

	// Разные наборы символов для экспериментов
	const symbolSets = {
		geometric: [
			'█',
			'▓',
			'▒',
			'░',
			'·',
			'○',
			'●',
			'◐',
			'◑',
			'◒',
			'◓',
			'◔',
			'◕',
			'◖',
			'◗',
			'◘',
			'◙',
			'◚',
			'◛',
			'◜',
			'◝',
			'◞',
			'◟',
			'◠',
			'◡',
			'◢',
			'◣',
			'◤',
			'◥',
			'◦',
			'◯',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		// Новый набор для более живых паттернов - плавные геометрические формы
		flowing: [
			'◐',
			'◑',
			'◒',
			'◓',
			'◔',
			'◕',
			'◖',
			'◗',
			'◘',
			'◙',
			'◚',
			'◛',
			'◜',
			'◝',
			'◞',
			'◟',
			'◠',
			'◡',
			'◢',
			'◣',
			'◤',
			'◥',
			'◦',
			'◯',
			'◰',
			'◱',
			'◲',
			'◳',
			'◴',
			'◵',
			'◶',
			'◷',
			'◸',
			'◹',
			'◺',
			'◻',
			'◼',
			'◽',
			'◾',
			'◿',
			'○',
			'●',
			'◎',
			'◯',
			'◉',
			'◊',
			'◆',
			'◇',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		// Волновые символы - кристаллические и энергетические формы
		waves: [
			'✦',
			'✧',
			'✩',
			'✪',
			'✫',
			'✬',
			'✭',
			'✮',
			'✯',
			'✰',
			'✱',
			'✲',
			'✳',
			'✴',
			'✵',
			'✶',
			'✷',
			'✸',
			'✹',
			'✺',
			'✻',
			'✼',
			'✽',
			'✾',
			'✿',
			'❀',
			'❁',
			'❂',
			'❃',
			'❄',
			'❅',
			'❆',
			'★',
			'☆',
			'♦',
			'♠',
			'♣',
			'♥',
			'♡',
			'♢',
			'♤',
			'♧',
			'♨',
			'♩',
			'♪',
			'♫',
			'♬',
			'♭',
			'♮',
			'♯',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		blocks: [
			'█',
			'▓',
			'▒',
			'░',
			'▀',
			'▁',
			'▂',
			'▃',
			'▄',
			'▅',
			'▆',
			'▇',
			'▉',
			'▊',
			'▋',
			'▌',
			'▍',
			'▎',
			'▏',
			'▐',
			'░',
			'▒',
			'▓',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		lines: [
			'│',
			'┤',
			'╡',
			'╢',
			'╖',
			'╕',
			'╣',
			'║',
			'╗',
			'╝',
			'╜',
			'╛',
			'┐',
			'└',
			'┴',
			'┬',
			'├',
			'─',
			'┼',
			'╞',
			'╟',
			'╚',
			'╔',
			'╩',
			'╦',
			'╠',
			'═',
			'╬',
			'╧',
			'╨',
			'╤',
			'╥',
			'╙',
			'╘',
			'╒',
			'╓',
			'╫',
			'╪',
			'┘',
			'┌',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		stars: [
			'★',
			'☆',
			'✦',
			'✧',
			'✩',
			'✪',
			'✫',
			'✬',
			'✭',
			'✮',
			'✯',
			'✰',
			'✱',
			'✲',
			'✳',
			'✴',
			'✵',
			'✶',
			'✷',
			'✸',
			'✹',
			'✺',
			'✻',
			'✼',
			'✽',
			'✾',
			'✿',
			'❀',
			'❁',
			'❂',
			'❃',
			'❄',
			'❅',
			'❆',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		math: [
			'█',
			'▓',
			'▒',
			'░',
			'·',
			'○',
			'●',
			'◐',
			'◑',
			'◒',
			'◓',
			'◔',
			'◕',
			'◖',
			'◗',
			'◘',
			'◙',
			'◚',
			'◛',
			'◜',
			'◝',
			'◞',
			'◟',
			'◠',
			'◡',
			'◢',
			'◣',
			'◤',
			'◥',
			'◦',
			'◯',
			'◰',
			'◱',
			'◲',
			'◳',
			'◴',
			'◵',
			'◶',
			'◷',
			'◸',
			'◹',
			'◺',
			'◻',
			'◼',
			'◽',
			'◾',
			'◿',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		mathStable: [
			'█',
			'▓',
			'▒',
			'░',
			'▀',
			'▁',
			'▂',
			'▃',
			'▄',
			'▅',
			'▆',
			'▇',
			'▉',
			'▊',
			'▋',
			'▌',
			'▍',
			'▎',
			'▏',
			'▐',
			'░',
			'▒',
			'▓',
			'█',
			'▓',
			'▒',
			'░',
			'▀',
			'▁',
			'▂',
			'▃',
			'▄',
			'▅',
			'▆',
			'▇',
			'▉',
			'▊',
			'▋',
			'▌',
			'▍',
			'▎',
			'▏',
			'▐',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		organic: [
			'●',
			'○',
			'◐',
			'◑',
			'◒',
			'◓',
			'◔',
			'◕',
			'◖',
			'◗',
			'◘',
			'◙',
			'◚',
			'◛',
			'◜',
			'◝',
			'◞',
			'◟',
			'◠',
			'◡',
			'◢',
			'◣',
			'◤',
			'◥',
			'◦',
			'◯',
			'◰',
			'◱',
			'◲',
			'◳',
			'◴',
			'◵',
			'◶',
			'◷',
			'◸',
			'◹',
			'◺',
			'◻',
			'◼',
			'◽',
			'◾',
			'◿',
			' ',
			' ',
			' ',
			' ',
			' '
		],
		// Специальный набор для мобилки - решетки, цифры, пустые зоны
		mobile: [
			'#',
			'##',
			'###',
			'####',
			'#####',
			'0',
			'1',
			'2',
			'3',
			'4',
			'5',
			'6',
			'7',
			'8',
			'9',
			'█',
			'▓',
			'▒',
			'░',
			'·',
			'◐',
			'◑',
			'◒',
			'◓',
			'◔',
			'◕',
			'◖',
			'◗',
			'◘',
			'◙',
			'◚',
			'◛',
			'◜',
			'◝',
			'◞',
			'◟',
			'◠',
			'◡',
			'◢',
			'◣',
			'◤',
			'◥',
			'◦',
			'◯',
			'◰',
			'◱',
			'◲',
			'◳',
			'◴',
			'◵',
			'◶',
			'◷',
			'◸',
			'◹',
			'◺',
			'◻',
			'◼',
			'◽',
			'◾',
			'◿',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' ',
			' '
		]
	};

	let currentSymbolSet: keyof typeof symbolSets = 'stars';
	let asciiChars = symbolSets[currentSymbolSet];
	let lastChangeTime = 0;
	let changeInterval = 0;
	let autoModeEnabled = true;

	// Переменные для оптимизации производительности
	let isMobile = false;
	let frameSkip = 0;
	let maxFrameSkip = 1; // Пропускаем каждый второй кадр на мобильных
	let lastFrameTime = 0;
	let targetFPS = 60;

	// Переменные для интерактивности
	// Touch переменные удалены - они мешали кликам на мобилке

	// Функция для определения мобильного устройства и настройки оптимизации
	function detectMobileAndOptimize() {
		if (!browser) return;

		const userAgent = navigator.userAgent.toLowerCase();
		const isMobileDevice = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(
			userAgent
		);
		const isSmallScreen = window.innerWidth <= 768;
		const isVerySmallScreen = window.innerWidth <= 480;

		isMobile = isMobileDevice || isSmallScreen;

		if (isMobile) {
			// На мобильных устройствах используем оптимизированную анимацию ASCII
			maxFrameSkip = 3; // Пропускаем кадры для производительности
			targetFPS = 15; // Снижаем FPS для мобилки
		} else {
			maxFrameSkip = 0; // На десктопе обновляем каждый кадр
			targetFPS = 60;
		}
	}

	// Функция для ASCII рендера картинки в консоль
	function renderImageToASCII(imagePath: string) {
		if (!browser) return;

		const img = new Image();
		(img as any).crossOrigin = 'anonymous';
		img.onload = () => {
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d');
			if (!ctx) return;

			// Динамически определяем размеры консоли
			// Создаем временный элемент для измерения
			const testElement = document.createElement('pre');
			testElement.style.position = 'absolute';
			testElement.style.visibility = 'hidden';
			testElement.style.fontFamily = 'monospace';
			testElement.style.fontSize = '6px';
			testElement.style.lineHeight = '0.8';
			testElement.textContent = 'A'.repeat(200); // Тестовая строка
			document.body.appendChild(testElement);

			// Получаем ширину консоли в символах
			const consoleWidth = Math.floor(testElement.offsetWidth / 3.6); // Примерная ширина символа
			const maxWidth = Math.min(consoleWidth - 10, 200); // Оставляем отступы
			const maxHeight = Math.floor(maxWidth * 0.4); // Соотношение высоты к ширине

			// Убираем тестовый элемент
			document.body.removeChild(testElement);

			const aspectRatio = img.width / img.height;

			let width = maxWidth;
			let height = maxHeight;

			if (aspectRatio > 1) {
				height = Math.floor(maxWidth / aspectRatio);
			} else {
				width = Math.floor(maxHeight * aspectRatio);
			}

			canvas.width = width;
			canvas.height = height;

			// Рисуем изображение на канвасе с растягиванием
			// Создаем эффект растягивания по горизонтали
			const stretchFactor = 1.5; // Коэффициент растягивания
			const stretchedWidth = Math.floor(width * stretchFactor);

			// Создаем новый канвас для растянутого изображения
			const stretchCanvas = document.createElement('canvas');
			const stretchCtx = stretchCanvas.getContext('2d');
			if (!stretchCtx) return;

			stretchCanvas.width = stretchedWidth;
			stretchCanvas.height = height;

			// Рисуем растянутое изображение
			stretchCtx.drawImage(img, 0, 0, stretchedWidth, height);

			// Используем растянутое изображение для ASCII
			const imageData = stretchCtx.getImageData(0, 0, stretchedWidth, height);
			const data = imageData.data;

			// Обновляем размеры для ASCII
			width = stretchedWidth;

			// ASCII символы от темного к светлому
			const asciiChars = ' .:-=+*#%@';

			let asciiArt = '';
			for (let y = 0; y < height; y++) {
				for (let x = 0; x < width; x++) {
					const index = (y * width + x) * 4;
					const r = data[index];
					const g = data[index + 1];
					const b = data[index + 2];

					// Вычисляем яркость
					const brightness = (r + g + b) / 3;

					// Выбираем ASCII символ на основе яркости
					const charIndex = Math.floor((brightness / 255) * (asciiChars.length - 1));
					asciiArt += asciiChars[charIndex];
				}
				asciiArt += '\n';
			}

			// Выводим в консоль с цветом
			console.log(
				`%c${asciiArt}`,
				'color: #ffd700; font-family: monospace; font-size: 6px; line-height: 0.8; white-space: pre;'
			);
		};
		img.src = imagePath;
	}

	// Настройки для автоматической смены символов
	const symbolPriorities = {
		flowing: 0.35, // 35% вероятность - улучшенные плавные геометрические формы
		waves: 0.3, // 30% вероятность - кристаллические и энергетические эффекты
		stars: 0.15, // 15% вероятность
		organic: 0.1, // 10% вероятность - органичные формы
		math: 0.05, // 5% вероятность
		geometric: 0.03, // 3% вероятность
		blocks: 0.02 // 2% вероятность
	};

	// Функция для смены набора символов
	function changeSymbolSet(setName: keyof typeof symbolSets) {
		currentSymbolSet = setName;
		asciiChars = symbolSets[currentSymbolSet];
	}

	// Функция для случайного выбора набора символов
	function getRandomSymbolSet(): keyof typeof symbolSets {
		const random = Math.random();
		let cumulative = 0;

		for (const [setName, probability] of Object.entries(symbolPriorities)) {
			cumulative += probability;
			if (random <= cumulative) {
				return setName as keyof typeof symbolSets;
			}
		}

		return 'stars';
	}

	// Функции для обработки жестов удалены - они мешали кликам на мобилке

	// Функция для расчета расстояния между двумя точками
	function getDistance(x1: number, y1: number, x2: number, y2: number): number {
		return Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2);
	}

	// Функция getSwipeDirection удалена - больше не используется

	// Функция для автоматической смены символов
	function autoChangeSymbols() {
		if (!autoModeEnabled) return;

		const now = Date.now();

		if (now - lastChangeTime >= changeInterval) {
			const newSet = getRandomSymbolSet();
			changeSymbolSet(newSet);

			changeInterval = Math.random() * 5000 + 3000;
			lastChangeTime = now;
		}
	}

	function calculateDimensions() {
		if (!browser) return;

		// Получаем реальные размеры экрана
		const screenWidth = window.innerWidth;
		const screenHeight = window.innerHeight;

		// Получаем размеры viewport для более точного расчета
		const viewportWidth = Math.max(
			document.documentElement.clientWidth || 0,
			window.innerWidth || 0
		);
		const viewportHeight = Math.max(
			document.documentElement.clientHeight || 0,
			window.innerHeight || 0
		);

		// Для мобильных устройств используем более точные расчеты
		let actualWidth = Math.max(screenWidth, viewportWidth);
		let actualHeight = Math.max(screenHeight, viewportHeight);

		// Дополнительная проверка для iOS Safari
		if (navigator.userAgent.includes('iPhone') || navigator.userAgent.includes('iPad')) {
			// Используем screen.width и screen.height для iOS
			actualWidth = Math.max(actualWidth, screen.width || 0);
			actualHeight = Math.max(actualHeight, screen.height || 0);
		}

		// Учитываем safe area insets для iPhone notch/dynamic island
		const safeAreaTop = parseInt(
			getComputedStyle(document.documentElement).getPropertyValue('--safe-area-inset-top') || '0'
		);
		const safeAreaBottom = parseInt(
			getComputedStyle(document.documentElement).getPropertyValue('--safe-area-inset-bottom') || '0'
		);
		const safeAreaLeft = parseInt(
			getComputedStyle(document.documentElement).getPropertyValue('--safe-area-inset-left') || '0'
		);
		const safeAreaRight = parseInt(
			getComputedStyle(document.documentElement).getPropertyValue('--safe-area-inset-right') || '0'
		);

		// Добавляем safe area к размерам для полного покрытия
		actualWidth += safeAreaLeft + safeAreaRight;
		actualHeight += safeAreaTop + safeAreaBottom;

		// Адаптивный размер шрифта в зависимости от устройства
		let fontSize = 0.8;
		if (actualWidth <= 320) fontSize = 0.25;
		else if (actualWidth <= 480) fontSize = 0.35;
		else if (actualWidth <= 768) fontSize = 0.5;
		else if (actualWidth <= 1024) fontSize = 0.6;
		else fontSize = 0.8;

		// Более точный расчет размеров символов
		const charWidth = fontSize * 0.6;
		const charHeight = fontSize * 1.0;

		// Конвертируем в пиксели
		const charWidthPx = charWidth * 16;
		const charHeightPx = charHeight * 16;

		// Рассчитываем количество символов для полного покрытия экрана
		width = Math.floor(actualWidth / charWidthPx);
		height = Math.floor(actualHeight / charHeightPx);

		// Добавляем небольшой запас для обеспечения полного покрытия
		width = Math.ceil(actualWidth / charWidthPx) + 2;
		height = Math.ceil(actualHeight / charHeightPx) + 2;

		// Минимальные размеры
		width = Math.max(width, 20);
		height = Math.max(height, 10);

		// Максимальные размеры для предотвращения переполнения
		width = Math.min(width, 300);
		height = Math.min(height, 200);
	}

	function generateASCII() {
		if (!browser || !asciiContainer) {
			return;
		}

		let ascii = '';

		// Для мобильных устройств используем тот же красивый паттерн, но оптимизированный
		if (isMobile) {
			// Используем тот же паттерн что и на десктопе, но с оптимизацией
			const timeFactor = 0.3; // Медленнее для производительности
			const currentTime = time * timeFactor;
			const isVerySmallScreen = window.innerWidth <= 480;

			// Размер на весь экран для мобилки - подбираем под размер экрана
			const screenWidth = window.innerWidth;
			const screenHeight = window.innerHeight;

			// Рассчитываем количество символов для заполнения экрана
			// Увеличиваем плотность символов для полного покрытия
			const mobileWidth = Math.floor(screenWidth / 6) + 10; // Больше символов по ширине
			const mobileHeight = Math.floor(screenHeight / 8) + 10; // Больше символов по высоте

			// Используем специальный набор символов для мобилки
			const symbols = symbolSets.mobile;

			// Добавляем интерактивные переменные
			const centerX = mobileWidth / 2;
			const centerY = mobileHeight / 2;
			const maxDistance = Math.sqrt(centerX * centerX + centerY * centerY);

			for (let y = 0; y < mobileHeight; y++) {
				for (let x = 0; x < mobileWidth; x++) {
					// Создаем красивый и понятный паттерн для мобилки
					const distanceFromCenter = Math.sqrt((x - centerX) ** 2 + (y - centerY) ** 2);
					const angleFromCenter = Math.atan2(y - centerY, x - centerX);

					// Создаем волновые паттерны
					const wave1 = Math.sin(x * 0.1 + currentTime * 0.005) * 0.5;
					const wave2 = Math.sin(y * 0.15 + currentTime * 0.003) * 0.4;
					const wave3 = Math.sin((x + y) * 0.08 + currentTime * 0.002) * 0.3;

					// Создаем спиральные эффекты
					const spiral =
						Math.sin(distanceFromCenter * 0.1 + angleFromCenter * 2 + currentTime * 0.004) * 0.6;

					// Создаем рябь от центра
					const ripple = Math.sin(distanceFromCenter * 0.2 - currentTime * 0.008) * 0.4;

					// Создаем шум для детализации
					const noise = (Math.sin(x * 0.5 + y * 0.3 + currentTime * 0.01) + 1) * 0.2;

					// Комбинируем эффекты для создания красивого паттерна
					const combined = (wave1 + wave2 + wave3 + spiral + ripple + noise) / 6;

					// Выбираем символ на основе комбинированного значения
					const symbolIndex = Math.floor(((combined + 1) * (symbols.length - 1)) / 2);
					const symbol = symbols[Math.max(0, Math.min(symbolIndex, symbols.length - 1))];

					ascii += symbol;
				}
				ascii += '\n';
			}

			asciiContainer.textContent = ascii;
			return;
		}

		// Оптимизация для десктопа: упрощаем расчеты
		const isMathSet = currentSymbolSet === 'math';
		const isFlowingSet = currentSymbolSet === 'flowing';
		const isWavesSet = currentSymbolSet === 'waves';
		const isOrganicSet = currentSymbolSet === 'organic';
		const isVerySmallScreen = window.innerWidth <= 480;
		const timeFactor = 1; // Только для десктопа
		const currentTime = time * timeFactor;

		// Добавляем интерактивные переменные
		const centerX = width / 2;
		const centerY = height / 2;
		const maxDistance = Math.sqrt(centerX * centerX + centerY * centerY);

		for (let y = 0; y < height; y++) {
			for (let x = 0; x < width; x++) {
				// Создаем сложные абстрактные узоры
				let wave1 = 0,
					wave2 = 0,
					wave3 = 0,
					wave4 = 0,
					wave5 = 0,
					wave6 = 0,
					noise = 0,
					spiral = 0,
					ripple = 0,
					flow = 0;

				// Расстояние от центра для спиральных эффектов
				const distanceFromCenter = Math.sqrt((x - centerX) ** 2 + (y - centerY) ** 2);
				const angleFromCenter = Math.atan2(y - centerY, x - centerX);

				// Для новых живых паттернов - плавные геометрические формы
				if (isFlowingSet) {
					// Создаем более стабильные плавные паттерны
					wave1 = Math.sin(x * 0.05 + currentTime * 0.003) * 0.3;
					wave2 = Math.sin(y * 0.08 + currentTime * 0.002) * 0.25;
					wave3 = Math.sin((x + y) * 0.04 + currentTime * 0.001) * 0.2;
					flow = Math.sin((x - y) * 0.06 + currentTime * 0.004) * 0.15;
					spiral =
						Math.sin(distanceFromCenter * 0.03 + angleFromCenter * 1.5 + currentTime * 0.001) * 0.2;
					ripple = Math.sin(distanceFromCenter * 0.06 - currentTime * 0.002) * 0.15;
					noise = (Math.sin(x * 0.2 + y * 0.15 + currentTime * 0.005) + 1) * 0.05;
				} else if (isWavesSet) {
					// Кристаллические и энергетические эффекты
					wave1 = Math.sin(x * 0.08 + currentTime * 0.004) * 0.4;
					wave2 = Math.sin(y * 0.12 + currentTime * 0.003) * 0.35;
					wave3 = Math.sin((x + y) * 0.06 + currentTime * 0.002) * 0.25;
					wave4 = Math.sin((x - y) * 0.05 + currentTime * 0.003) * 0.15;
					ripple = Math.sin(distanceFromCenter * 0.05 - currentTime * 0.002) * 0.25;
					flow = Math.sin(angleFromCenter * 2 + currentTime * 0.002) * 0.15;
					noise = (Math.sin(x * 0.3 + y * 0.25 + currentTime * 0.006) + 1) * 0.08;
				} else if (isOrganicSet) {
					// Органичные, живые формы
					wave1 = Math.sin(x * 0.09 + currentTime * 0.007) * 0.4;
					wave2 = Math.sin(y * 0.14 + currentTime * 0.005) * 0.3;
					wave3 = Math.sin((x + y) * 0.07 + currentTime * 0.003) * 0.3;
					spiral =
						Math.sin(distanceFromCenter * 0.06 + angleFromCenter * 1.5 + currentTime * 0.004) * 0.4;
					ripple = Math.sin(distanceFromCenter * 0.12 - currentTime * 0.006) * 0.3;
					flow = Math.sin((x - y) * 0.1 + currentTime * 0.008) * 0.2;
					noise = (Math.sin(x * 0.3 + y * 0.25 + currentTime * 0.01) + 1) * 0.12;
				} else if (isMathSet) {
					// Для математических символов используем более стабильную анимацию
					wave1 = Math.sin(x * 0.05 + currentTime * 0.002) * 0.3;
					wave2 = Math.sin(y * 0.08 + currentTime * 0.001) * 0.2;
					wave3 = Math.sin((x + y) * 0.03 + currentTime * 0.001) * 0.2;
					wave4 = Math.sin((x - y) * 0.02 + currentTime * 0.002) * 0.1;
					wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.01 + currentTime * 0.0005) * 0.2;
					wave6 = Math.sin(x * y * 0.0005 + currentTime * 0.003) * 0.05;
					noise = (Math.sin(x * 0.2 + y * 0.15 + currentTime * 0.005) + 1) * 0.05;
				} else {
					// Упрощаем расчеты для мобильных устройств
					if (isMobile) {
						if (isVerySmallScreen) {
							// Максимальная оптимизация для очень маленьких экранов
							wave1 = Math.sin(x * 0.08 + currentTime * 0.002) * 0.4;
							wave2 = Math.sin(y * 0.12 + currentTime * 0.001) * 0.3;
							wave3 = 0;
							wave4 = 0;
							wave5 = 0;
							wave6 = 0;
							noise = 0;
						} else {
							wave1 = Math.sin(x * 0.08 + currentTime * 0.003) * 0.4;
							wave2 = Math.sin(y * 0.12 + currentTime * 0.002) * 0.3;
							wave3 = Math.sin((x + y) * 0.06 + currentTime * 0.001) * 0.3;
							wave4 = 0;
							wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.02 + currentTime * 0.0008) * 0.3;
							wave6 = 0;
							noise = (Math.sin(x * 0.3 + y * 0.2 + currentTime * 0.008) + 1) * 0.1;
						}
					} else {
						wave1 = Math.sin(x * 0.08 + currentTime * 0.005) * 0.4;
						wave2 = Math.sin(y * 0.12 + currentTime * 0.003) * 0.3;
						wave3 = Math.sin((x + y) * 0.06 + currentTime * 0.002) * 0.3;
						wave4 = Math.sin((x - y) * 0.04 + currentTime * 0.004) * 0.2;
						wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.02 + currentTime * 0.001) * 0.3;
						wave6 = Math.sin(x * y * 0.001 + currentTime * 0.006) * 0.1;
						noise = (Math.sin(x * 0.3 + y * 0.2 + currentTime * 0.01) + 1) * 0.1;
					}
				}

				// Объединяем все эффекты
				const intensity =
					(wave1 +
						wave2 +
						wave3 +
						(wave4 || 0) +
						(wave5 || 0) +
						(wave6 || 0) +
						noise +
						(spiral || 0) +
						(ripple || 0) +
						(flow || 0) +
						1) /
					2;
				const charIndex = Math.floor(intensity * (asciiChars.length - 1));

				// Проверяем, что charIndex в допустимых пределах
				const safeCharIndex = Math.max(0, Math.min(charIndex, asciiChars.length - 1));
				ascii += asciiChars[safeCharIndex] || ' ';
			}
			ascii += '\n';
		}

		asciiContainer.textContent = ascii;
	}

	function animate() {
		// На мобильных устройствах не запускаем анимацию
		if (isMobile) {
			return;
		}

		const now = performance.now();
		const deltaTime = now - lastFrameTime;

		// Контроль FPS
		if (deltaTime < 1000 / targetFPS) {
			animationId = requestAnimationFrame(animate);
			return;
		}

		lastFrameTime = now;

		time += 1;
		autoChangeSymbols();

		generateASCII();
		animationId = requestAnimationFrame(animate);
	}

	onMount(() => {
		// ASCII рендер картинки в консоль
		setTimeout(() => {
			renderImageToASCII('/img2.webp');
		}, 1000);

		if (browser) {
			// Небольшая задержка для правильного определения размеров экрана
			setTimeout(() => {
				// Генерируем позиции для окон с минимальным перекрытием
				generateRandomPositions();

				// Инициализируем начальные окна - все открыты
				const isMobile = window.innerWidth <= 768;
				openWindows = releasesData.map((release, index) => {
					const windowId = `file-${index}`;
					const zIndex = getNextZIndex();
					windowZIndexes.set(windowId, zIndex);

					return {
						id: Date.now() + index,
						fileId: windowId,
						title: release.title,
						icon: '🎵',
						image: release.cover['800'],
						link: release.link,
						text1: 'Music Release',
						text2: '@ye66ow',
						x: windowPositions[index]?.x || (isMobile ? 20 : 100),
						y: windowPositions[index]?.y || (isMobile ? 50 : 150),
						isVisible: true, // Все окна открыты при старте
						zIndex: zIndex
					};
				});

				// Создаем файлы на рабочем столе для каждого релиза
				desktopFiles = releasesData.map((release, index) => ({
					id: Date.now() + index + 1000,
					fileId: `file-${index}`,
					fileName: release.title,
					fileType: 'mp3',
					windowData: {
						fileId: `file-${index}`,
						title: release.title,
						icon: '🎵',
						image: release.cover['800'],
						link: release.link,
						text1: 'Music Release',
						text2: '@ye66ow',
						x: windowPositions[index]?.x || 100,
						y: windowPositions[index]?.y || 150
					},
					x: 50 + index * 200,
					y: 50
				}));
			}, 100); // Задержка 100ms для правильного определения размеров

			// Добавляем обработчики событий
			window.addEventListener('windowClose', handleWindowClose);
			window.addEventListener('fileToggle', handleFileToggle);

			// Определяем мобильное устройство и настраиваем оптимизацию
			detectMobileAndOptimize();
			// PWA: Prevent zoom on double tap
			let lastTouchEnd = 0;
			document.addEventListener(
				'touchend',
				(event) => {
					const now = new Date().getTime();
					if (now - lastTouchEnd <= 300) {
						event.preventDefault();
					}
					lastTouchEnd = now;
				},
				false
			);

			// PWA: Handle app install prompt
			let deferredPrompt: any;
			window.addEventListener('beforeinstallprompt', (e) => {
				e.preventDefault();
				deferredPrompt = e;
				// You can show install button here if needed
			});

			// PWA: Handle app installed
			window.addEventListener('appinstalled', () => {
				console.log('PWA was installed');
			});

			// Ждем полной загрузки страницы для корректного расчета размеров
			const initASCII = () => {
				calculateDimensions();
				lastChangeTime = Date.now();
				changeInterval = Math.random() * 5000 + 3000;

				setTimeout(() => {
					generateASCII();
					// Запускаем анимацию только на десктопе
					if (!isMobile) {
						animate();
					}
				}, 100);
			};

			// Инициализируем сразу и после полной загрузки
			initASCII();

			if (document.readyState === 'complete') {
				initASCII();
			} else {
				window.addEventListener('load', initASCII);
			}

			const handleResize = () => {
				// Добавляем небольшую задержку для стабилизации размеров
				setTimeout(() => {
					detectMobileAndOptimize(); // Переопределяем оптимизацию
					calculateDimensions();
					generateASCII();
					generateRandomPositions(); // Перегенерируем позиции окон

					// Обновляем позиции существующих окон
					openWindows = openWindows.map((window, index) => ({
						...window,
						x: windowPositions[index]?.x || window.x,
						y: windowPositions[index]?.y || window.y
					}));

					// Если переключились с мобильного на десктоп, запускаем анимацию
					if (!isMobile && !animationId) {
						animate();
					}
					// Если переключились с десктопа на мобильный, останавливаем анимацию
					else if (isMobile && animationId) {
						cancelAnimationFrame(animationId);
						animationId = 0;
					}
				}, 50);
			};

			const handleOrientationChange = () => {
				// Дополнительная задержка для изменения ориентации
				setTimeout(() => {
					calculateDimensions();
					generateASCII();
				}, 200);
			};

			const handleKeyPress = (e: KeyboardEvent) => {
				const key = e.key.toLowerCase();
				const symbolSetKeys: Record<string, keyof typeof symbolSets> = {
					'1': 'flowing',
					'2': 'waves',
					'3': 'organic',
					'4': 'stars',
					'5': 'math',
					'6': 'geometric',
					'7': 'blocks',
					'8': 'lines'
				};

				if (symbolSetKeys[key]) {
					// Временно отключаем автоматическую смену
					autoModeEnabled = false;
					changeSymbolSet(symbolSetKeys[key]);
					generateASCII();

					// Включаем автоматическую смену через 10 секунд
					setTimeout(() => {
						autoModeEnabled = true;
						lastChangeTime = Date.now();
						changeInterval = Math.random() * 5000 + 3000;
					}, 10000);
				}
			};

			// Обработчики touch событий удалены - они мешали кликам на мобилке

			window.addEventListener('resize', handleResize);
			window.addEventListener('orientationchange', handleOrientationChange);
			window.addEventListener('keydown', handleKeyPress);
			window.addEventListener('load', initASCII);

			// Touch обработчики удалены - они мешали кликам на мобилке

			return () => {
				window.removeEventListener('resize', handleResize);
				window.removeEventListener('orientationchange', handleOrientationChange);
				window.removeEventListener('keydown', handleKeyPress);
				window.removeEventListener('load', initASCII);
				window.removeEventListener('windowClose', handleWindowClose);
				window.removeEventListener('fileToggle', handleFileToggle);

				// Touch обработчики удалены
			};
		}
	});

	onDestroy(() => {
		if (animationId) {
			cancelAnimationFrame(animationId);
		}
	});
</script>

<svelte:head>
	<title>ye66ow</title>
	<meta name="description" content="ye66ow" />
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover"
	/>
	<meta name="apple-mobile-web-app-capable" content="yes" />
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
	<meta name="mobile-web-app-capable" content="yes" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Inter:wght@300;400;500;600;700&family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500;600;700&display=swap"
		rel="stylesheet"
	/>
	<style>
		:root {
			--safe-area-inset-top: env(safe-area-inset-top);
			--safe-area-inset-bottom: env(safe-area-inset-bottom);
			--safe-area-inset-left: env(safe-area-inset-left);
			--safe-area-inset-right: env(safe-area-inset-right);

			/* Варианты шрифтов для названий треков */
			--font-primary: 'Space Grotesk', 'Inter', 'Orbitron', sans-serif;
			--font-modern: 'Inter', 'Space Grotesk', sans-serif;
			--font-tech: 'JetBrains Mono', 'Space Grotesk', monospace;
			--font-futuristic: 'Orbitron', 'Space Grotesk', sans-serif;
			--font-clean: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
		}
	</style>
</svelte:head>

<!-- ASCII Art Background -->
<div class="ascii-background">
	<pre bind:this={asciiContainer} class="ascii-art">Loading...</pre>

	<!-- Заголовок с почтой -->
	<div class="email-header">
		<a href="mailto:hello@ye66ow.com" class="email-link">hello@ye66ow.com</a>
	</div>

	<!-- Окна с релизами -->
	{#each openWindows as window, index}
		{#if window.isVisible === true}
			<Window
				fileId={window.fileId}
				title={window.title}
				icon={window.icon}
				image={window.image}
				link={window.link}
				text1={window.text1}
				text2={window.text2}
				initialX={window.x}
				initialY={window.y}
				zIndex={window.zIndex}
				on:bringToFront={(e) => {
					const newZIndex = bringWindowToFront(window.fileId);
					openWindows = openWindows.map((w) =>
						w.fileId === window.fileId ? { ...w, zIndex: newZIndex } : w
					);
				}}
			/>
		{/if}
	{/each}

	<!-- Файлы на рабочем столе -->
	{#each desktopFiles as file}
		<DesktopFile
			fileName={file.fileName}
			fileType={file.fileType}
			windowData={file.windowData}
			initialX={file.x}
			initialY={file.y}
		/>
	{/each}
</div>

<style>
	/* Global styles */
	:global(html),
	:global(body) {
		margin: 0;
		padding: 0;
		min-height: 100vh;
		min-height: 100dvh; /* Dynamic viewport height for mobile */
		background: #000;
		color: #fff;
		font-family: 'Courier New', monospace;
		-webkit-text-size-adjust: 100%; /* Предотвращаем изменение размера текста в Safari */
		-webkit-touch-callout: none; /* Отключаем контекстное меню на iOS */
		-webkit-user-select: none; /* Отключаем выделение текста на iOS */
		user-select: none;
		/* Safe area support for iPhone notch/dynamic island */
		padding-top: env(safe-area-inset-top);
		padding-bottom: env(safe-area-inset-bottom);
		padding-left: env(safe-area-inset-left);
		padding-right: env(safe-area-inset-right);
	}

	.ascii-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		width: 100dvw; /* Dynamic viewport width for mobile */
		height: 100vh;
		height: 100dvh; /* Dynamic viewport height for mobile */
		background: #000;
		z-index: 1;
		/* Safe area support - растягиваем на всю область включая safe area */
		top: calc(-1 * env(safe-area-inset-top));
		left: calc(-1 * env(safe-area-inset-left));
		width: calc(100vw + env(safe-area-inset-left) + env(safe-area-inset-right));
		width: calc(100dvw + env(safe-area-inset-left) + env(safe-area-inset-right));
		height: calc(100vh + env(safe-area-inset-top) + env(safe-area-inset-bottom));
		height: calc(100dvh + env(safe-area-inset-top) + env(safe-area-inset-bottom));
		/* Делаем область тапа более отзывчивой */
		cursor: pointer;
		-webkit-tap-highlight-color: transparent;
	}

	.ascii-art {
		font-size: 0.8rem;
		line-height: 1;
		color: #fff;
		margin: 0;
		padding: 0;
		white-space: pre;
		user-select: none;
		opacity: 1;
		text-shadow:
			0 0 10px rgba(255, 255, 255, 0.3),
			0 0 20px rgba(0, 255, 255, 0.2),
			0 0 30px rgba(255, 0, 255, 0.1);
		width: 100vw;
		width: 100dvw; /* Dynamic viewport width for mobile */
		height: 100vh;
		height: 100dvh; /* Dynamic viewport height for mobile */
		display: block;
		overflow: hidden;
		position: absolute;
		top: 0;
		left: 0;
		z-index: 1;
		transform: translateZ(0); /* Аппаратное ускорение для лучшей производительности */
		-webkit-transform: translateZ(0);
		/* Safe area support - растягиваем ASCII арт на всю область включая safe area */
		top: calc(-1 * env(safe-area-inset-top));
		left: calc(-1 * env(safe-area-inset-left));
		width: calc(100vw + env(safe-area-inset-left) + env(safe-area-inset-right));
		width: calc(100dvw + env(safe-area-inset-left) + env(safe-area-inset-right));
		height: calc(100vh + env(safe-area-inset-top) + env(safe-area-inset-bottom));
		height: calc(100dvh + env(safe-area-inset-top) + env(safe-area-inset-bottom));

		/* Анимация для более живого вида */
		animation: subtleGlow 4s ease-in-out infinite alternate;
	}

	@keyframes subtleGlow {
		0% {
			text-shadow:
				0 0 10px rgba(255, 255, 255, 0.3),
				0 0 20px rgba(0, 255, 255, 0.2),
				0 0 30px rgba(255, 0, 255, 0.1);
		}
		100% {
			text-shadow:
				0 0 15px rgba(255, 255, 255, 0.4),
				0 0 25px rgba(0, 255, 255, 0.3),
				0 0 35px rgba(255, 0, 255, 0.2);
		}
	}

	/* Mobile Responsive */
	@media (max-width: 768px) {
		.ascii-art {
			/* Показываем ASCII на мобильных с крупными символами */
			display: block;
			font-size: 1.1rem;
			line-height: 0.9;
			opacity: 0.9;
			white-space: pre;
			overflow: hidden;
			width: 100vw;
			height: 100vh;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1;
			transform: scale(1.1);
			transform-origin: top left;
			margin: 0;
			padding: 0;
		}

		/* Простой фон для мобильных */
		.ascii-background {
			background: #0a0a0a;
		}
	}

	@media (max-width: 480px) {
		.ascii-art {
			/* Показываем ASCII на очень маленьких экранах с крупными символами */
			display: block;
			font-size: 0.9rem;
			line-height: 0.8;
			opacity: 0.8;
			white-space: pre;
			overflow: hidden;
			width: 100vw;
			height: 100vh;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1;
			transform: scale(1.2);
			transform-origin: top left;
			margin: 0;
			padding: 0;
		}

		/* Простой фон для очень маленьких экранов */
		.ascii-background {
			background: #0a0a0a;
		}
	}

	@media (max-width: 320px) {
		.ascii-art {
			/* Скрываем ASCII на очень маленьких экранах для производительности */
			display: none;
		}

		/* Простой фон для очень маленьких экранов */
		.ascii-background {
			background: #0a0a0a;
		}
	}

	/* Safari specific fixes */
	@supports (-webkit-touch-callout: none) {
		.ascii-background {
			width: 100vw;
			height: 100vh;
		}

		.ascii-art {
			width: 100vw;
			height: 100vh;
		}
	}

	/* iOS Safari specific fixes */
	@supports (-webkit-touch-callout: none) and (max-width: 768px) {
		:global(html),
		:global(body) {
			height: 100vh;
			height: -webkit-fill-available;
		}

		.ascii-background {
			height: 100vh;
			height: -webkit-fill-available;
		}

		.ascii-art {
			height: 100vh;
			height: -webkit-fill-available;
		}
	}

	/* iPhone Dynamic Island / Notch specific fixes */
	@supports (padding: max(0px)) {
		.ascii-background {
			/* Используем max() для fallback на случай, если safe-area-inset не поддерживается */
			top: calc(-1 * max(env(safe-area-inset-top), 0px));
			left: calc(-1 * max(env(safe-area-inset-left), 0px));
			width: calc(
				100vw + max(env(safe-area-inset-left), 0px) + max(env(safe-area-inset-right), 0px)
			);
			width: calc(
				100dvw + max(env(safe-area-inset-left), 0px) + max(env(safe-area-inset-right), 0px)
			);
			height: calc(
				100vh + max(env(safe-area-inset-top), 0px) + max(env(safe-area-inset-bottom), 0px)
			);
			height: calc(
				100dvh + max(env(safe-area-inset-top), 0px) + max(env(safe-area-inset-bottom), 0px)
			);
		}

		.ascii-art {
			top: calc(-1 * max(env(safe-area-inset-top), 0px));
			left: calc(-1 * max(env(safe-area-inset-left), 0px));
			width: calc(
				100vw + max(env(safe-area-inset-left), 0px) + max(env(safe-area-inset-right), 0px)
			);
			width: calc(
				100dvw + max(env(safe-area-inset-left), 0px) + max(env(safe-area-inset-right), 0px)
			);
			height: calc(
				100vh + max(env(safe-area-inset-top), 0px) + max(env(safe-area-inset-bottom), 0px)
			);
			height: calc(
				100dvh + max(env(safe-area-inset-top), 0px) + max(env(safe-area-inset-bottom), 0px)
			);
		}
	}

	/* PWA Specific Styles */
	/* Prevent text selection on mobile */
	.ascii-art {
		-webkit-touch-callout: none;
		-webkit-user-select: none;
		-khtml-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
		user-select: none;
	}

	/* Prevent pull-to-refresh on mobile */
	:global(body) {
		overscroll-behavior: none;
		-webkit-overflow-scrolling: touch;
	}

	/* Hide scrollbars but keep functionality */
	:global(html),
	:global(body) {
		scrollbar-width: none; /* Firefox */
		-ms-overflow-style: none; /* Internet Explorer 10+ */
	}

	:global(html::-webkit-scrollbar),
	:global(body::-webkit-scrollbar) {
		display: none; /* WebKit */
	}

	/* PWA: Fullscreen mode optimizations */
	@media (display-mode: fullscreen) {
		.ascii-art {
			font-size: 0.9rem;
		}
	}

	/* PWA: Standalone mode optimizations */
	@media (display-mode: standalone) {
		.ascii-art {
			font-size: 0.85rem;
		}
	}

	/* PWA: Minimal UI mode */
	@media (display-mode: minimal-ui) {
		.ascii-art {
			font-size: 0.8rem;
		}
	}

	/* PWA: Browser mode */
	@media (display-mode: browser) {
		.ascii-art {
			font-size: 0.8rem;
		}
	}

	/* Главный бренд - почта */
	.email-header {
		position: fixed;
		top: 25px;
		right: 50px;
		z-index: 100;
		pointer-events: none;
		padding: 8px;
		transform: translateX(20px);
		width: auto;
		max-width: calc(100vw - 100px);
	}

	.email-link {
		font-family: 'Courier New', 'Monaco', monospace;
		font-size: 24px;
		font-weight: bold;
		color: #ffffff;
		text-decoration: none;
		pointer-events: auto;
		text-shadow: 2px 2px 0 rgba(0, 0, 0, 0.8);
		letter-spacing: 2px;
		transition: all 0.3s ease;
		text-align: center;
		display: block;
		background: rgba(0, 0, 0, 0.2);
		backdrop-filter: blur(1px);
		-webkit-backdrop-filter: blur(12px);
		border-radius: 10px;
		overflow: hidden;
		padding: 12px 16px;
	}

	.email-link:hover {
		color: #ffff00;
		background: rgba(0, 0, 0, 0.3);
		backdrop-filter: blur(15px);
		-webkit-backdrop-filter: blur(15px);
		text-shadow: 2px 2px 0 rgba(0, 0, 0, 0.9);
	}
</style>
