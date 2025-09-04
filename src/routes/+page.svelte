<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	let asciiContainer: HTMLPreElement;
	let animationId: number;
	let time = 0;
	let width = 120;
	let height = 60;

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
		img.crossOrigin = 'anonymous';
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
		stars: 0.4, // 40% вероятность
		math: 0.3, // 30% вероятность
		geometric: 0.1, // 10% вероятность
		blocks: 0.1, // 10% вероятность
		lines: 0.05, // 5% вероятность
		organic: 0.05 // 5% вероятность
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

		// Временно отключаем автоматическую смену
		const wasAutoModeEnabled = autoModeEnabled;
		autoModeEnabled = false;

		// Выбираем случайный набор символов
		const newSet = getRandomSymbolSet();
		changeSymbolSet(newSet);
		generateASCII();

		// Включаем автоматическую смену через 5 секунд
		setTimeout(() => {
			autoModeEnabled = wasAutoModeEnabled;
			lastChangeTime = Date.now();
			changeInterval = Math.random() * 5000 + 3000;
		}, 5000);
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
		const isVerySmallScreen = window.innerWidth <= 480;
		const timeFactor = isMobile ? (isVerySmallScreen ? 0.3 : 0.5) : 1; // Еще больше замедляем на очень маленьких экранах
		const currentTime = time * timeFactor;

		for (let y = 0; y < height; y++) {
			for (let x = 0; x < width; x++) {
				// Создаем сложные абстрактные узоры
				let wave1, wave2, wave3, wave4, wave5, wave6, noise;

				// Для математических символов используем более стабильную анимацию
				if (isMathSet) {
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
							wave3 = 0; // Убираем еще больше расчетов
							wave4 = 0;
							wave5 = 0;
							wave6 = 0;
							noise = 0;
						} else {
							wave1 = Math.sin(x * 0.08 + currentTime * 0.003) * 0.4;
							wave2 = Math.sin(y * 0.12 + currentTime * 0.002) * 0.3;
							wave3 = Math.sin((x + y) * 0.06 + currentTime * 0.001) * 0.3;
							wave4 = 0; // Убираем сложные расчеты
							wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.02 + currentTime * 0.0008) * 0.3;
							wave6 = 0; // Убираем сложные расчеты
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

				const intensity = (wave1 + wave2 + wave3 + wave4 + wave5 + wave6 + noise + 1) / 2;
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
					'1': 'geometric',
					'2': 'blocks',
					'3': 'lines',
					'4': 'stars',
					'5': 'math',
					'6': 'organic',
					'7': 'mathStable'
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

			// Обработчик тапа для мобильных устройств
			const handleTap = (e: TouchEvent) => {
				// Предотвращаем стандартное поведение
				e.preventDefault();

				// Переключаем символы
				handleTapToChangeSymbols();
			};

			// Обработчик клика для десктопа (если нужно)
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

			// Добавляем обработчики для тапа/клика
			document.addEventListener('touchstart', handleTap, { passive: false });
			document.addEventListener('click', handleClick);

			return () => {
				window.removeEventListener('resize', handleResize);
				window.removeEventListener('orientationchange', handleOrientationChange);
				window.removeEventListener('keydown', handleKeyPress);
				window.removeEventListener('load', initASCII);

				// Удаляем обработчики для тапа/клика
				document.removeEventListener('touchstart', handleTap);
				document.removeEventListener('click', handleClick);
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
	<style>
		:root {
			--safe-area-inset-top: env(safe-area-inset-top);
			--safe-area-inset-bottom: env(safe-area-inset-bottom);
			--safe-area-inset-left: env(safe-area-inset-left);
			--safe-area-inset-right: env(safe-area-inset-right);
		}
	</style>
</svelte:head>

<!-- ASCII Art Background -->
<div class="ascii-background">
	<pre bind:this={asciiContainer} class="ascii-art">Loading...</pre>
</div>

<style>
	/* Global styles */
	:global(html),
	:global(body) {
		margin: 0;
		padding: 0;
		height: 100vh;
		height: 100dvh; /* Dynamic viewport height for mobile */
		overflow: hidden; /* Убираем скролл для полного покрытия */
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
		display: flex;
		align-items: center;
		justify-content: center;
		background: #000;
		z-index: 1;
		overflow: hidden;
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
		text-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
		width: 100vw;
		width: 100dvw; /* Dynamic viewport width for mobile */
		height: 100vh;
		height: 100dvh; /* Dynamic viewport height for mobile */
		display: block;
		overflow: hidden;
		position: absolute;
		top: 0;
		left: 0;
		transform: translateZ(0); /* Аппаратное ускорение для лучшей производительности */
		-webkit-transform: translateZ(0);
		/* Safe area support - растягиваем ASCII арт на всю область включая safe area */
		top: calc(-1 * env(safe-area-inset-top));
		left: calc(-1 * env(safe-area-inset-left));
		width: calc(100vw + env(safe-area-inset-left) + env(safe-area-inset-right));
		width: calc(100dvw + env(safe-area-inset-left) + env(safe-area-inset-right));
		height: calc(100vh + env(safe-area-inset-top) + env(safe-area-inset-bottom));
		height: calc(100dvh + env(safe-area-inset-top) + env(safe-area-inset-bottom));
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
</style>
