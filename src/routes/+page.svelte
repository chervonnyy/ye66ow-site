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

		const screenWidth = window.innerWidth;
		const screenHeight = window.innerHeight;

		let fontSize = 0.8;
		if (screenWidth <= 320) fontSize = 0.3;
		else if (screenWidth <= 480) fontSize = 0.4;
		else if (screenWidth <= 768) fontSize = 0.5;

		const charWidth = fontSize * 0.6;
		const charHeight = fontSize * 1.0;

		const charWidthPx = charWidth * 16;
		const charHeightPx = charHeight * 16;

		width = Math.floor(screenWidth / charWidthPx);
		height = Math.floor(screenHeight / charHeightPx);

		height = Math.ceil(screenHeight / charHeightPx);

		width = Math.max(width, 20);
		height = Math.max(height, 10);
	}

	function generateASCII() {
		if (!browser || !asciiContainer) {
			return;
		}

		let ascii = '';
		for (let y = 0; y < height; y++) {
			for (let x = 0; x < width; x++) {
				// Создаем сложные абстрактные узоры
				let wave1, wave2, wave3, wave4, wave5, wave6, noise;

				// Для математических символов используем более стабильную анимацию
				if (currentSymbolSet === 'math') {
					wave1 = Math.sin(x * 0.05 + time * 0.002) * 0.3;
					wave2 = Math.sin(y * 0.08 + time * 0.001) * 0.2;
					wave3 = Math.sin((x + y) * 0.03 + time * 0.001) * 0.2;
					wave4 = Math.sin((x - y) * 0.02 + time * 0.002) * 0.1;
					wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.01 + time * 0.0005) * 0.2;
					wave6 = Math.sin(x * y * 0.0005 + time * 0.003) * 0.05;
					noise = (Math.sin(x * 0.2 + y * 0.15 + time * 0.005) + 1) * 0.05;
				} else {
					wave1 = Math.sin(x * 0.08 + time * 0.005) * 0.4;
					wave2 = Math.sin(y * 0.12 + time * 0.003) * 0.3;
					wave3 = Math.sin((x + y) * 0.06 + time * 0.002) * 0.3;
					wave4 = Math.sin((x - y) * 0.04 + time * 0.004) * 0.2;
					wave5 = Math.sin(Math.sqrt(x * x + y * y) * 0.02 + time * 0.001) * 0.3;
					wave6 = Math.sin(x * y * 0.001 + time * 0.006) * 0.1;
					noise = (Math.sin(x * 0.3 + y * 0.2 + time * 0.01) + 1) * 0.1;
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
			calculateDimensions();

			lastChangeTime = Date.now();
			changeInterval = Math.random() * 5000 + 3000;

			setTimeout(() => {
				generateASCII();
				animate();
			}, 100);

			const handleResize = () => {
				calculateDimensions();
				generateASCII();
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

			window.addEventListener('resize', handleResize);
			window.addEventListener('keydown', handleKeyPress);

			return () => {
				window.removeEventListener('resize', handleResize);
				window.removeEventListener('keydown', handleKeyPress);
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
	<title>ye66ow - ASCII Art</title>
	<meta name="description" content="ye66ow - ASCII art animation with geometric patterns" />
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
		overflow: auto;
		background: #000;
		color: #fff;
		font-family: 'Courier New', monospace;
	}

	.ascii-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		background: #000;
		z-index: 1;
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
		height: 100vh;
		display: block;
		overflow: hidden;
	}

	/* Mobile Responsive */
	@media (max-width: 768px) {
		.ascii-art {
			font-size: 0.5rem;
		}
	}

	@media (max-width: 480px) {
		.ascii-art {
			font-size: 0.4rem;
		}
	}

	@media (max-width: 320px) {
		.ascii-art {
			font-size: 0.3rem;
		}
	}
</style>
