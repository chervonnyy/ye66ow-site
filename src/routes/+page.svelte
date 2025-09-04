<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import releasesData from '$lib/releases.json';

	let asciiContainer: HTMLPreElement;
	let animationId: number;
	let time = 0;
	let width = 120;
	let height = 60;

	// Фиксированный шрифт JetBrains Mono
	let currentFont = 'tech';

	// Переменная для скрытия блока релизов
	let isReleasesVisible = true;

	// Функция для переключения видимости блока
	function toggleReleases(event: Event) {
		console.log('toggleReleases called!', isReleasesVisible);
		event.preventDefault();
		event.stopPropagation();
		isReleasesVisible = !isReleasesVisible;
		console.log('isReleasesVisible now:', isReleasesVisible);
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
	let touchStartX = 0;
	let touchStartY = 0;
	let touchStartTime = 0;
	let lastTouchTime = 0;
	let gestureMode = 'tap'; // 'tap', 'swipe', 'pinch'
	let pinchStartDistance = 0;
	let currentScale = 1;
	let animationSpeed = 1;

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
			// На мобильных устройствах снижаем частоту обновления
			if (isVerySmallScreen) {
				// Для очень маленьких экранов еще больше оптимизируем
				maxFrameSkip = 3; // Пропускаем 3 из 4 кадров
				targetFPS = 20; // Очень низкий FPS
			} else {
				maxFrameSkip = 2; // Пропускаем 2 из 3 кадров
				targetFPS = 30; // Целевой FPS для мобильных
			}
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

	// Функция для переключения символов при тапе на мобильных
	function handleTapToChangeSymbols() {
		if (!browser) return;

		console.log('Tap detected! Switching symbols...'); // Отладочная информация

		// Временно отключаем автоматическую смену
		const wasAutoModeEnabled = autoModeEnabled;
		autoModeEnabled = false;

		// Выбираем случайный набор символов
		const newSet = getRandomSymbolSet();
		console.log('Switching to symbol set:', newSet); // Отладочная информация
		changeSymbolSet(newSet);
		generateASCII();

		// Включаем автоматическую смену через 5 секунд
		setTimeout(() => {
			autoModeEnabled = wasAutoModeEnabled;
			lastChangeTime = Date.now();
			changeInterval = Math.random() * 5000 + 3000;
		}, 5000);
	}

	// Функция для обработки свайпов
	function handleSwipe(direction: string) {
		if (!browser) return;

		// Временно отключаем автоматическую смену
		const wasAutoModeEnabled = autoModeEnabled;
		autoModeEnabled = false;

		// Выбираем набор символов в зависимости от направления свайпа
		let newSet: keyof typeof symbolSets;
		switch (direction) {
			case 'left':
				newSet = 'flowing';
				break;
			case 'right':
				newSet = 'waves';
				break;
			case 'up':
				newSet = 'organic';
				break;
			case 'down':
				newSet = 'stars';
				break;
			default:
				newSet = getRandomSymbolSet();
		}

		changeSymbolSet(newSet);
		generateASCII();

		// Включаем автоматическую смену через 8 секунд
		setTimeout(() => {
			autoModeEnabled = wasAutoModeEnabled;
			lastChangeTime = Date.now();
			changeInterval = Math.random() * 5000 + 3000;
		}, 8000);
	}

	// Функция для обработки пинча
	function handlePinch(scale: number) {
		if (!browser) return;

		// Изменяем скорость анимации в зависимости от масштаба
		animationSpeed = Math.max(0.1, Math.min(3, scale));
		currentScale = scale;

		// Временно ускоряем анимацию
		const originalTimeFactor = time;
		time *= animationSpeed;
		generateASCII();
		time = originalTimeFactor;
	}

	// Функция для расчета расстояния между двумя точками
	function getDistance(x1: number, y1: number, x2: number, y2: number): number {
		return Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2);
	}

	// Функция для определения направления свайпа
	function getSwipeDirection(startX: number, startY: number, endX: number, endY: number): string {
		const deltaX = endX - startX;
		const deltaY = endY - startY;
		const minSwipeDistance = 50;

		if (Math.abs(deltaX) > Math.abs(deltaY)) {
			if (Math.abs(deltaX) > minSwipeDistance) {
				return deltaX > 0 ? 'right' : 'left';
			}
		} else {
			if (Math.abs(deltaY) > minSwipeDistance) {
				return deltaY > 0 ? 'down' : 'up';
			}
		}

		return 'tap';
	}

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

		// Оптимизация для мобильных: упрощаем расчеты
		const isMathSet = currentSymbolSet === 'math';
		const isFlowingSet = currentSymbolSet === 'flowing';
		const isWavesSet = currentSymbolSet === 'waves';
		const isOrganicSet = currentSymbolSet === 'organic';
		const isVerySmallScreen = window.innerWidth <= 480;
		const timeFactor = isMobile ? (isVerySmallScreen ? 0.3 : 0.5) : 1;
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
		const now = performance.now();
		const deltaTime = now - lastFrameTime;

		// Контроль FPS
		if (deltaTime < 1000 / targetFPS) {
			animationId = requestAnimationFrame(animate);
			return;
		}

		lastFrameTime = now;

		// Пропускаем кадры на мобильных устройствах
		if (isMobile) {
			frameSkip++;
			if (frameSkip < maxFrameSkip) {
				animationId = requestAnimationFrame(animate);
				return;
			}
			frameSkip = 0;
		}

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
					animate();
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

			// Обработчик начала касания
			const handleTouchStart = (e: TouchEvent) => {
				e.preventDefault();
				const touch = e.touches[0];
				touchStartX = touch.clientX;
				touchStartY = touch.clientY;
				touchStartTime = Date.now();

				console.log('Touch start:', { x: touchStartX, y: touchStartY, touches: e.touches.length }); // Отладочная информация

				// Если два пальца - режим пинча
				if (e.touches.length === 2) {
					gestureMode = 'pinch';
					const touch1 = e.touches[0];
					const touch2 = e.touches[1];
					pinchStartDistance = getDistance(
						touch1.clientX,
						touch1.clientY,
						touch2.clientX,
						touch2.clientY
					);
					console.log('Pinch mode started');
				} else {
					gestureMode = 'tap';
					console.log('Tap mode started');
				}
			};

			// Обработчик движения касания
			const handleTouchMove = (e: TouchEvent) => {
				e.preventDefault();

				if (gestureMode === 'pinch' && e.touches.length === 2) {
					const touch1 = e.touches[0];
					const touch2 = e.touches[1];
					const currentDistance = getDistance(
						touch1.clientX,
						touch1.clientY,
						touch2.clientX,
						touch2.clientY
					);
					const scale = currentDistance / pinchStartDistance;
					handlePinch(scale);
				}
			};

			// Обработчик окончания касания
			const handleTouchEnd = (e: TouchEvent) => {
				e.preventDefault();
				const touch = e.changedTouches[0];
				const endX = touch.clientX;
				const endY = touch.clientY;
				const endTime = Date.now();

				console.log('Touch end:', {
					gestureMode,
					duration: endTime - touchStartTime,
					distance: getDistance(touchStartX, touchStartY, endX, endY)
				}); // Отладочная информация

				if (gestureMode === 'tap') {
					const duration = endTime - touchStartTime;
					const distance = getDistance(touchStartX, touchStartY, endX, endY);

					// Если касание было коротким и недалеко - это тап
					if (duration < 300 && distance < 30) {
						console.log('Tap detected!');
						handleTapToChangeSymbols();
					} else if (distance > 30) {
						// Иначе это свайп
						const direction = getSwipeDirection(touchStartX, touchStartY, endX, endY);
						console.log('Swipe detected:', direction);
						if (direction !== 'tap') {
							handleSwipe(direction);
						}
					}
				}

				gestureMode = 'tap';
			};

			// Обработчик клика для десктопа
			const handleClick = (e: MouseEvent) => {
				// Только для мобильных устройств или если это не клавиатурное управление
				if (window.innerWidth <= 768) {
					e.preventDefault();
					handleTapToChangeSymbols();
				}
			};

			window.addEventListener('resize', handleResize);
			window.addEventListener('orientationchange', handleOrientationChange);
			window.addEventListener('keydown', handleKeyPress);
			window.addEventListener('load', initASCII);

			// Добавляем обработчики для жестов
			document.addEventListener('touchstart', handleTouchStart, { passive: false });
			document.addEventListener('touchmove', handleTouchMove, { passive: false });
			document.addEventListener('touchend', handleTouchEnd, { passive: false });
			// document.addEventListener('click', handleClick); // Отключено для корректной работы кнопки сворачивания

			return () => {
				window.removeEventListener('resize', handleResize);
				window.removeEventListener('orientationchange', handleOrientationChange);
				window.removeEventListener('keydown', handleKeyPress);
				window.removeEventListener('load', initASCII);

				// Удаляем обработчики для жестов
				document.removeEventListener('touchstart', handleTouchStart);
				document.removeEventListener('touchmove', handleTouchMove);
				document.removeEventListener('touchend', handleTouchEnd);
				// document.removeEventListener('click', handleClick); // Отключено для корректной работы кнопки сворачивания
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

	<!-- Секция релизов -->
	{#if isReleasesVisible}
		<section class="releases-section">
			<div class="releases-container">
				<!-- Заголовок окна в стиле Windows -->
				<div class="window-titlebar">
					<div class="window-title">Music</div>
					<button class="minimize-btn" on:click={toggleReleases} title="Свернуть">
						<div class="minimize-icon">_</div>
					</button>
				</div>

				<div class="releases-list">
					{#each releasesData as release}
						<article class="release-item">
							<a href={release.link} target="_blank" rel="noopener noreferrer" class="release-link">
								<picture class="release-cover">
									<img
										src={release.cover['800']}
										srcset="{release.cover['1200']} 1200w,
												{release.cover['800']} 800w,
												{release.cover['512']} 512w"
										sizes="(min-width:1024px) 240px, 60vw"
										alt="{release.title} cover"
										loading="lazy"
										decoding="async"
									/>
								</picture>
								<h3 class="release-title" style="font-family: var(--font-{currentFont})">
									{release.title}
								</h3>
							</a>
						</article>
					{/each}
				</div>
			</div>
		</section>
	{:else}
		<!-- Кнопка Music когда блок свернут -->
		<div class="music-btn-container">
			<button class="music-btn" on:click={toggleReleases} title="Показать релизы">
				<span class="music-text">Music</span>
			</button>
		</div>
	{/if}
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
			font-size: 0.7rem; /* Увеличиваем для планшетов */
			line-height: 0.9;
		}
	}

	@media (max-width: 480px) {
		.ascii-art {
			font-size: 0.6rem; /* Увеличиваем для мобильных */
			line-height: 0.8;
		}
	}

	@media (max-width: 320px) {
		.ascii-art {
			font-size: 0.5rem; /* Увеличиваем для маленьких экранов */
			line-height: 0.7;
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

	/* Кнопка сворачивания в стиле ретро Windows */
	.minimize-btn {
		position: absolute;
		top: 8px;
		right: 8px;
		background: #c0c0c0;
		color: #000;
		border: 2px outset #c0c0c0;
		padding: 4px 8px;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		font-size: 14px;
		font-weight: bold;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 24px;
		height: 24px;
		transition: all 0.1s ease;
		z-index: 10;
		box-shadow:
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
	}

	.minimize-btn:hover {
		background: #d4d0c8;
		transform: translate(1px, 1px);
		box-shadow:
			1px 1px 0 #808080,
			inset 1px 1px 0 #ffffff;
	}

	.minimize-btn:active {
		background: #a0a0a0;
		transform: translate(2px, 2px);
		box-shadow:
			inset 1px 1px 2px #808080,
			inset -1px -1px 2px #ffffff;
		border: 2px inset #c0c0c0;
	}

	.minimize-icon {
		font-size: 16px;
		font-weight: bold;
		line-height: 1;
	}

	/* Кнопка Music когда блок свернут */
	.music-btn-container {
		position: fixed;
		top: 112px;
		right: 37px;
		z-index: 100;
		pointer-events: auto;
	}

	.music-btn {
		background: #c0c0c0;
		color: #000;
		border: 3px outset #c0c0c0;
		padding: 12px 24px;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		font-size: 16px;
		font-weight: bold;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		min-width: 120px;
		height: 40px;
		transition: all 0.1s ease;
		box-shadow:
			4px 4px 0 #404040,
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
		pointer-events: auto;
	}

	.music-btn:hover {
		background: #d4d0c8;
		box-shadow:
			3px 3px 0 #404040,
			1px 1px 0 #808080,
			inset 1px 1px 0 #ffffff;
	}

	.music-btn:active {
		background: #a0a0a0;
		box-shadow:
			inset 2px 2px 2px #404040,
			inset -1px -1px 2px #ffffff;
		border: 3px inset #c0c0c0;
	}

	.music-text {
		font-size: 16px;
		font-weight: bold;
		letter-spacing: 1px;
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

	/* Секция релизов */
	.releases-section {
		position: fixed;
		top: 100px;
		right: 20px;
		width: auto;
		height: calc(100vh - 120px);
		height: calc(100dvh - 120px);
		display: flex;
		align-items: flex-start;
		justify-content: flex-end;
		z-index: 50;
		pointer-events: none;
		overflow-y: auto;
		padding: 20px;
		box-sizing: border-box;
	}

	.releases-container {
		position: relative;
		background: #c0c0c0;
		padding: 0;
		width: fit-content;
		max-width: 320px;
		box-shadow:
			4px 4px 0 #404040,
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
		border: 3px outset #c0c0c0;
		pointer-events: auto;
	}

	/* Заголовок окна в стиле Windows 95 */
	.window-titlebar {
		background: linear-gradient(90deg, #000080 0%, #0000ff 100%);
		color: #ffffff;
		padding: 6px 10px;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		font-size: 13px;
		font-weight: bold;
		height: 24px;
		box-shadow:
			inset 0 1px 0 rgba(255, 255, 255, 0.4),
			inset 0 -1px 0 rgba(0, 0, 0, 0.3);
		border-bottom: 1px solid #404040;
		pointer-events: auto;
	}

	.window-title {
		flex: 1;
		text-shadow: 1px 1px 0 rgba(0, 0, 0, 0.5);
	}

	.window-titlebar .minimize-btn {
		position: static;
		width: 22px;
		height: 18px;
		padding: 0;
		margin: 0;
		background: #c0c0c0;
		border: 2px outset #c0c0c0;
		box-shadow:
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.window-titlebar .minimize-btn:hover {
		background: #d4d0c8;
		transform: none;
		box-shadow:
			2px 2px 0 #808080,
			inset 1px 1px 0 #ffffff;
	}

	.window-titlebar .minimize-btn:active {
		background: #a0a0a0;
		transform: none;
		box-shadow:
			inset 2px 2px 2px #808080,
			inset -1px -1px 2px #ffffff;
		border: 2px inset #c0c0c0;
	}

	/* Исправление для мобильных устройств */
	.window-titlebar .minimize-btn:focus {
		outline: none;
	}

	.window-titlebar .minimize-btn:focus-visible {
		outline: 2px solid #0000ff;
		outline-offset: 1px;
	}

	/* Дополнительные стили для мобильных устройств */
	@media (max-width: 768px) {
		.window-titlebar .minimize-btn {
			width: 24px;
			height: 20px;
			font-size: 14px;
			-webkit-tap-highlight-color: transparent;
			touch-action: manipulation;
			pointer-events: auto;
			z-index: 1000;
			position: relative;
		}
	}

	@media (max-width: 480px) {
		.window-titlebar .minimize-btn {
			width: 22px;
			height: 18px;
			font-size: 12px;
			-webkit-tap-highlight-color: transparent;
			touch-action: manipulation;
			pointer-events: auto;
			z-index: 1000;
			position: relative;
		}

		/* Предотвращаем перехват событий на мобилке */
		.releases-container {
			pointer-events: auto;
		}

		.window-titlebar {
			pointer-events: auto;
		}
	}

	/* Очень маленькие экраны */
	@media (max-width: 320px) {
		.email-header {
			right: 5px;
			max-width: calc(100vw - 10px);
			transform: translateX(5px);
		}

		.email-link {
			font-size: 14px;
			padding: 6px 8px;
		}
	}

	.window-titlebar .minimize-icon {
		font-size: 12px;
		line-height: 1;
	}

	.releases-list {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 24px;
		padding: 24px;
		background: #c0c0c0;
		border-top: 1px solid #808080;
	}

	.release-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.release-link {
		display: flex;
		flex-direction: column;
		align-items: center;
		text-decoration: none;
		color: inherit;
		gap: 0px;
	}

	.release-cover {
		width: 240px;
		height: 240px;
		overflow: hidden;
		position: relative;
		border: 3px inset #c0c0c0;
		box-shadow:
			inset 2px 2px 0 #ffffff,
			inset -2px -2px 0 #808080;
		background: #c0c0c0;
		transition: all 0.3s ease;
	}

	.release-cover img {
		width: 100%;
		height: 100%;
		object-fit: cover;
	}

	.release-item:hover .release-cover {
		border: 3px outset #c0c0c0;
		box-shadow:
			outset 2px 2px 0 #ffffff,
			outset -2px -2px 0 #808080;
		background: #d4d0c8;
		transform: scale(1.02);
	}

	.release-item:hover .release-title {
		color: #ffffff;
	}

	.release-title {
		font-family: 'MS Sans Serif', 'Courier New', monospace;
		font-size: 18px;
		font-weight: bold;
		color: #000;
		margin: 0;
		letter-spacing: 1px;
		text-align: center;
		width: 240px;
		padding: 10px 16px;
		transition: all 0.3s ease;
	}

	/* Адаптивность для секции релизов */
	@media (max-width: 768px) {
		.email-header {
			top: 15px;
			right: 15px;
			width: auto;
			max-width: calc(100vw - 30px);
			transform: translateX(15px);
		}

		.email-link {
			font-size: 18px;
			letter-spacing: 1.5px;
			padding: 10px 12px;
		}

		.minimize-btn {
			top: 6px;
			right: 6px;
			width: 20px;
			height: 20px;
			font-size: 12px;
		}

		.music-btn {
			font-size: 14px;
			padding: 10px 20px;
			min-width: 100px;
			height: 36px;
		}

		.releases-section {
			top: 85px;
			right: 15px;
			height: calc(100vh - 100px);
			height: calc(100dvh - 100px);
			padding: 15px;
		}

		.music-btn-container {
			top: 112px;
			right: 37px;
		}

		.releases-container {
			max-width: 240px;
		}

		.releases-list {
			padding: 15px;
			gap: 15px;
		}

		.releases-list {
			gap: 35px;
		}

		.release-cover {
			width: 200px;
			height: 200px;
		}

		.release-title {
			font-size: 18px;
			width: 200px;
		}
	}

	@media (max-width: 480px) {
		.email-header {
			top: 10px;
			right: 10px;
			width: auto;
			max-width: calc(100vw - 20px);
			transform: translateX(10px);
		}

		.email-link {
			font-size: 16px;
			letter-spacing: 1px;
			padding: 8px 10px;
		}

		.minimize-btn {
			top: 4px;
			right: 4px;
			width: 18px;
			height: 18px;
			font-size: 10px;
		}

		.music-btn {
			font-size: 12px;
			padding: 8px 16px;
			min-width: 80px;
			height: 32px;
		}

		.releases-section {
			top: 70px;
			right: 10px;
			height: calc(100vh - 80px);
			height: calc(100dvh - 80px);
			padding: 10px;
		}

		.music-btn-container {
			top: 70px;
			right: 10px;
		}

		.releases-container {
			max-width: 200px;
		}

		.releases-list {
			padding: 12px;
			gap: 12px;
		}

		.releases-list {
			gap: 30px;
		}

		.release-cover {
			width: 180px;
			height: 180px;
		}

		.release-title {
			font-size: 16px;
			width: 180px;
		}
	}
</style>
