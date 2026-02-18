<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import releasesData from '$lib/releases.json';

	let {
		pattern = $bindable(0),
		rampIdx = $bindable(0),
		inverted = $bindable(false),
		speed = $bindable(1),
		asciiFont = $bindable(0),
		showPanel = $bindable(false)
	}: {
		pattern: number;
		rampIdx: number;
		inverted: boolean;
		speed: number;
		asciiFont: number;
		showPanel: boolean;
	} = $props();

	const asciiFonts = [
		"'JetBrains Mono', monospace",
		"'Courier New', monospace",
		"'Monaco', monospace",
		"'Menlo', monospace",
		"'Lucida Console', monospace",
		"'American Typewriter', monospace",
		"'Andale Mono', monospace",
		"'OCR A Std', 'OCR-A', monospace",
		"'Phosphate', fantasy",
		"'Copperplate', fantasy"
	];

	const EMAIL = 'hello@ye66ow.com';
	const trackOrder = [1, 0, 2];
	const DEFAULT_BG = '/background/FN.webp';

	let animId: number;
	let time = 0;
	let asciiOutput = $state('');
	let hoveredTrack = $state<number | null>(null);
	let playingTrack = $state<number | null>(null);
	let cols = $state(80);
	let rows = $state(55);

	// Mouse tracking (normalized 0-1)
	let mouseX = 0.5;
	let mouseY = 0.5;
	let mouseActive = false;

	// Logo: each letter in a different font, with glitch bursts
	const logoLetters = 'ye66ow'.split('');
	const logoFonts = [
		'Georgia',
		'Impact',
		'Courier New',
		'Arial Black',
		'Palatino',
		'Times New Roman',
		'Trebuchet MS',
		'Verdana',
		'Lucida Console',
		'Garamond',
		'Papyrus',
		'Copperplate',
		'Didot',
		'Bodoni MT',
		'Rockwell',
		'Futura',
		'Optima',
		'Baskerville',
		'American Typewriter',
		'Zapfino',
		'Marker Felt',
		'Chalkduster',
		'Phosphate',
		'Snell Roundhand',
		'Herculanum',
		'Brush Script MT',
		'Comic Sans MS',
		'serif',
		'fantasy',
		'cursive'
	];
	let letterFonts = $state(
		logoLetters.map(() => logoFonts[Math.floor(Math.random() * logoFonts.length)])
	);
	let fontInterval: ReturnType<typeof setInterval>;

	function randomFont() {
		return logoFonts[Math.floor(Math.random() * logoFonts.length)];
	}

	function glitchBurst() {
		// Rapid-fire font changes on random letters
		const burstCount = 3 + Math.floor(Math.random() * 6); // 3-8 rapid changes
		let i = 0;
		const burstInterval = setInterval(
			() => {
				const idx = Math.floor(Math.random() * logoLetters.length);
				letterFonts = letterFonts.map((f, j) => (j === idx ? randomFont() : f));
				i++;
				if (i >= burstCount) clearInterval(burstInterval);
			},
			60 + Math.random() * 80
		); // 60-140ms between each
	}

	function scheduleNextChange() {
		const delay = 600 + Math.random() * 1200; // 0.6-1.8s
		fontInterval = setTimeout(() => {
			if (Math.random() < 0.3) {
				glitchBurst();
			} else {
				const idx = Math.floor(Math.random() * logoLetters.length);
				letterFonts = letterFonts.map((f, i) => (i === idx ? randomFont() : f));
			}
			scheduleNextChange();
		}, delay);
	}

	// Character ramps — dense to light
	const ramps = [
		'@8OOoo0::..··    ',
		'#%*+=~-:..··     ',
		'█▓▒░░···..       ',
		'WMBHXoaw=+~-:..  ',
		'008OOoo°°··..    ',
		'////\\\\||--..·· ',
		'⣿⣷⣯⣟⡿⢿⣻⣽⣾⠿⠷⠟⠻⠽⠾⠀',
		'ンシツノ丿乀亅〆彡 ',
		'●◉◎○◌・ . ',
		'█▉▊▋▌▍▎▏ ',
		'ye66ow··..       ',
		'falsenostalgia·. ',
		'☾★✦✧·˚ ⊹  ',
		'♪♫♩♬~-·.. ',
		'optricia··..     ',
		'oO0oo°°··..      ',
		'still @live yoshi  __- ··..',
		'plastic river ЛТ ·. '
	];

	// Per-track presets: when a track starts, auto-apply these settings
	const trackPresets: Record<string, { pattern: number; ramp: number; speed: number }> = {
		'plastic-river': { pattern: 3, ramp: 17, speed: 0.25 },
		'still-alive': { pattern: 6, ramp: 16, speed: 0.5 },
		optricia: { pattern: 4, ramp: 3, speed: 4 }
	};

	// --- Image management (preload all) ---
	let currentImgSrc = DEFAULT_BG;
	const preloadedImages: Map<string, HTMLImageElement> = new Map();
	let imgBrightness: Float32Array = new Float32Array(0);
	let imgReady = false;

	function preloadAllImages() {
		const srcs = [DEFAULT_BG, ...releasesData.map((r: any) => r.bg)];
		for (const src of srcs) {
			const img = new Image();
			img.src = src;
			preloadedImages.set(src, img);
		}
		// Set initial image once default loads
		const defaultImg = preloadedImages.get(DEFAULT_BG)!;
		if (defaultImg.complete) {
			imgReady = true;
			sampleImage();
		} else {
			defaultImg.onload = () => {
				imgReady = true;
				sampleImage();
			};
		}
	}

	function switchImage(src: string) {
		if (src === currentImgSrc && imgReady) return;
		currentImgSrc = src;
		const img = preloadedImages.get(src);
		if (img && img.complete && img.naturalWidth > 0) {
			imgReady = true;
			sampleImage();
		} else if (img) {
			imgReady = false;
			img.onload = () => {
				if (currentImgSrc === src) {
					imgReady = true;
					sampleImage();
				}
			};
		}
	}

	function sampleImage() {
		const img = preloadedImages.get(currentImgSrc);
		if (!img || !img.complete || cols === 0 || rows === 0) return;
		const c = document.createElement('canvas');
		c.width = cols;
		c.height = rows;
		const ctx = c.getContext('2d')!;
		ctx.drawImage(img, 0, 0, cols, rows);
		const data = ctx.getImageData(0, 0, cols, rows).data;
		imgBrightness = new Float32Array(cols * rows);
		for (let i = 0; i < cols * rows; i++) {
			const r = data[i * 4];
			const g = data[i * 4 + 1];
			const b = data[i * 4 + 2];
			imgBrightness[i] = 1 - (0.299 * r + 0.587 * g + 0.114 * b) / 255;
		}
	}

	// --- Audio management ---
	const audioElements: Map<number, HTMLAudioElement> = new Map();
	let audioUnlocked = false;
	let pendingTrack: number | null = null;

	function initAudio() {
		releasesData.forEach((release: any, i: number) => {
			if (release.audio) {
				const audio = new Audio();
				audio.preload = 'auto';
				audio.src = release.audio;
				audio.loop = true;
				audio.volume = 0;
				audioElements.set(i, audio);
			}
		});
	}

	function unlockAudio() {
		if (audioUnlocked) return;
		audioUnlocked = true;
		// Play+pause each audio to unlock browser autoplay
		const promises: Promise<void>[] = [];
		audioElements.forEach((audio) => {
			promises.push(
				audio
					.play()
					.then(() => audio.pause())
					.catch(() => {})
			);
		});
		// Wait for unlock cycle to finish before playing pending track
		Promise.all(promises).then(() => {
			if (pendingTrack !== null) {
				const track = pendingTrack;
				pendingTrack = null;
				playTrack(track);
			}
		});
	}

	function playTrack(index: number) {
		if (!audioUnlocked) {
			pendingTrack = index;
			return;
		}
		pendingTrack = null;
		// Stop all other tracks
		audioElements.forEach((audio, i) => {
			if (i !== index) {
				fadeAudio(audio, 0, 300);
				setTimeout(() => {
					try {
						audio.pause();
					} catch {}
				}, 350);
			}
		});
		// Play this track
		const audio = audioElements.get(index);
		if (audio) {
			audio.currentTime = audio.currentTime || 0;
			audio.volume = 0;
			audio
				.play()
				.then(() => {
					fadeAudio(audio, 0.7, 500);
					playingTrack = index;
				})
				.catch(() => {
					// Autoplay blocked — will retry after unlock
					pendingTrack = index;
				});
		}
	}

	function stopAllTracks() {
		pendingTrack = null;
		audioElements.forEach((audio) => {
			fadeAudio(audio, 0, 400);
			setTimeout(() => {
				try {
					audio.pause();
				} catch {}
			}, 450);
		});
		playingTrack = null;
	}

	function fadeAudio(audio: HTMLAudioElement, target: number, durationMs: number) {
		const start = audio.volume;
		const diff = target - start;
		if (Math.abs(diff) < 0.01) {
			audio.volume = target;
			return;
		}
		const steps = 20;
		const stepMs = durationMs / steps;
		let step = 0;
		const interval = setInterval(() => {
			step++;
			try {
				audio.volume = Math.max(0, Math.min(1, start + diff * (step / steps)));
			} catch {}
			if (step >= steps) clearInterval(interval);
		}, stepMs);
	}

	// --- Track hover/click handling ---
	let hoverSavedPattern: number | null = null;
	let hoverSavedRamp: number | null = null;
	let hoverSavedSpeed: number | null = null;

	function onTrackEnter(index: number) {
		hoveredTrack = index;
		const release = releasesData[index] as any;
		if (release?.bg) switchImage(release.bg);
		// Apply track preset for hover preview
		const preset = trackPresets[release?.id];
		if (preset) {
			hoverSavedPattern = pattern;
			hoverSavedRamp = rampIdx;
			hoverSavedSpeed = speed;

			pattern = preset.pattern;
			rampIdx = preset.ramp;
			speed = preset.speed;
		}
	}

	function onTrackLeave() {
		hoveredTrack = null;
		// Restore settings from before hover
		if (hoverSavedPattern !== null) {

			pattern = hoverSavedPattern;
			hoverSavedPattern = null;
		}
		if (hoverSavedRamp !== null) {
			rampIdx = hoverSavedRamp;
			hoverSavedRamp = null;
		}
		if (hoverSavedSpeed !== null) {
			speed = hoverSavedSpeed;
			hoverSavedSpeed = null;
		}
		// Restore image
		if (playingTrack !== null) {
			const release = releasesData[playingTrack] as any;
			if (release?.bg) switchImage(release.bg);
		} else {
			switchImage(DEFAULT_BG);
		}
	}

	// Saved settings to restore when stopping track
	let savedPattern: number | null = null;
	let savedRamp: number | null = null;
	let savedSpeed: number | null = null;

	function onTrackClick(e: MouseEvent, index: number) {
		e.preventDefault();
		// Use pre-hover values as the "original" if hover is active
		const origPattern = hoverSavedPattern ?? pattern;
		const origRamp = hoverSavedRamp ?? rampIdx;
		const origSpeed = hoverSavedSpeed ?? speed;
		// Clear hover state since click takes over
		hoverSavedPattern = null;
		hoverSavedRamp = null;
		hoverSavedSpeed = null;

		if (!audioUnlocked) {
			const release = releasesData[index] as any;
			const preset = trackPresets[release?.id];
			if (preset) {
				if (savedPattern === null) savedPattern = origPattern;
				if (savedRamp === null) savedRamp = origRamp;
				if (savedSpeed === null) savedSpeed = origSpeed;
				pattern = preset.pattern;
				rampIdx = preset.ramp;
				speed = preset.speed;
			}
			switchImage(release?.cover || DEFAULT_BG);
			pendingTrack = index;
			unlockAudio();
			return;
		}
		if (playingTrack === index) {
			// Clicking same track — stop it
			stopAllTracks();
			switchImage(DEFAULT_BG);
			// Restore previous settings
			if (savedPattern !== null) {
				pattern = savedPattern;
				savedPattern = null;
			} else {
				pattern = origPattern;
			}
			if (savedRamp !== null) {
				rampIdx = savedRamp;
				savedRamp = null;
			} else {
				rampIdx = origRamp;
			}
			if (savedSpeed !== null) {
				speed = savedSpeed;
				savedSpeed = null;
			} else {
				speed = origSpeed;
			}
		} else {
			// Apply track preset if available
			const release = releasesData[index] as any;
			const preset = trackPresets[release?.id];
			if (preset) {
				if (savedPattern === null) savedPattern = origPattern;
				if (savedRamp === null) savedRamp = origRamp;
				if (savedSpeed === null) savedSpeed = origSpeed;
				pattern = preset.pattern;
				rampIdx = preset.ramp;
				speed = preset.speed;
			}
			playTrack(index);
		}
	}

	// --- Pattern 0: Metaballs ---
	const NUM_BLOBS = 7;
	const blobData = Array.from({ length: NUM_BLOBS }, (_, i) => ({
		sx: 0.1 + Math.random() * 0.15,
		sy: 0.07 + Math.random() * 0.12,
		ox: i * 1.7,
		oy: i * 2.3,
		r: 0.2 + Math.random() * 0.2
	}));

	function valBlobs(nx: number, ny: number, t: number): number {
		let sum = 0;
		for (let i = 0; i < NUM_BLOBS; i++) {
			const b = blobData[i];
			const bx = 0.5 + 0.42 * Math.sin(t * b.sx + b.ox);
			const by = 0.5 + 0.42 * Math.cos(t * b.sy + b.oy);
			const dx = nx - bx;
			const dy = (ny - by) * 1.8;
			sum += (b.r * b.r) / (dx * dx + dy * dy + 0.001);
		}
		return Math.max(0, (sum - 0.7) / 3.0);
	}

	// --- Pattern 1: Diagonal wave bands ---
	function valWaves(nx: number, ny: number, t: number): number {
		const w1 = Math.sin(nx * 12 + ny * 5 + t * 1.2);
		const w2 = Math.sin(nx * 7 - ny * 9 + t * 0.7);
		const w3 = Math.sin((nx + ny) * 8 + t * 1.0);
		const w4 = Math.cos(nx * 4 + ny * 14 + t * 0.5);
		const val = (w1 + w2 + w3 + w4) / 4;
		return Math.max(0, (val + 0.15) / 1.15);
	}

	// --- Pattern 2: Expanding ripples ---
	function valRipples(nx: number, ny: number, t: number): number {
		let val = 0;
		for (let i = 0; i < 5; i++) {
			const cx = 0.5 + 0.38 * Math.sin(t * 0.2 + i * 1.3);
			const cy = 0.5 + 0.38 * Math.cos(t * 0.18 + i * 1.8);
			const dist = Math.sqrt((nx - cx) ** 2 + ((ny - cy) * 1.8) ** 2);
			val += (Math.sin(dist * 35 - t * 2) + 1) * 0.5;
		}
		return Math.max(0, (val / 5 - 0.15) / 0.85);
	}

	// --- Pattern 3: Falling rain ---
	function valRain(nx: number, ny: number, t: number): number {
		let maxVal = 0;
		const col = Math.floor(nx * 300);
		for (let d = 0; d < 3; d++) {
			const hash = Math.sin(col * 127.1 + d * 311.7) * 43758.5453;
			const seed = hash - Math.floor(hash);
			const spd = 0.1 + seed * 0.35;
			const phase = seed * 12 + d * 3.7;
			const trailLen = 0.1 + seed * 0.25;
			const headY = ((t * spd + phase) % (1.0 + trailLen)) - trailLen * 0.3;
			const dist = ny - headY;
			if (dist >= 0 && dist <= trailLen) {
				maxVal = Math.max(maxVal, 1 - dist / trailLen);
			}
		}
		return maxVal;
	}

	// --- Pattern 4: Terrain contours ---
	function valTerrain(nx: number, ny: number, t: number): number {
		const v =
			Math.sin(nx * 8 + t * 0.25) * 0.25 +
			Math.sin(ny * 10 + t * 0.18) * 0.25 +
			Math.sin((nx + ny) * 6 + t * 0.3) * 0.2 +
			Math.sin(nx * 16 + ny * 10 + t * 0.4) * 0.15 +
			Math.sin(nx * 4 - ny * 7 + t * 0.22) * 0.15;
		return Math.max(0, (v + 0.45) / 0.95);
	}

	// --- Pattern 5: Moiré interference ---
	function valMoire(nx: number, ny: number, t: number): number {
		const cx = nx * 2 - 1;
		const cy = (ny * 2 - 1) * 1.5;
		const a1 = Math.sin(Math.sqrt(cx * cx + cy * cy) * 18 + t * 0.8);
		const a2 = Math.sin(Math.sqrt((cx - 0.5) ** 2 + cy * cy) * 16 - t * 0.6);
		const a3 = Math.sin(Math.sqrt((cx + 0.3) ** 2 + (cy - 0.4) ** 2) * 14 + t * 0.4);
		const a4 = Math.cos((cx * Math.cos(t * 0.1) + cy * Math.sin(t * 0.1)) * 10);
		const val = (a1 + a2 + a3 + a4) / 4;
		return Math.max(0, (val + 0.3) / 1.3);
	}

	// --- Pattern 6: Spiral vortex ---
	function valSpiral(nx: number, ny: number, t: number): number {
		const cx = nx - 0.5;
		const cy = (ny - 0.5) * 1.8;
		const dist = Math.sqrt(cx * cx + cy * cy);
		const angle = Math.atan2(cy, cx);
		const spiral = Math.sin(angle * 3 + dist * 20 - t * 2);
		const spiral2 = Math.sin(angle * 5 - dist * 15 + t * 1.3);
		const fade = Math.max(0, 1 - dist * 1.8);
		const val = (spiral + spiral2) * 0.5 * fade;
		return Math.max(0, (val + 0.3) / 1.3);
	}

	// --- Pattern 7: Scanner line ---
	function valScanner(nx: number, ny: number, t: number): number {
		const lineY = ((t * 0.15) % 1.4) - 0.2;
		const dist = Math.abs(ny - lineY);
		const beam = Math.max(0, 1 - dist * 6);
		const lineX = ((t * 0.1 + 0.5) % 1.4) - 0.2;
		const distX = Math.abs(nx - lineX);
		const beamX = Math.max(0, 1 - distX * 8);
		const trail = ny < lineY ? Math.max(0, 1 - (lineY - ny) * 3) * 0.5 : 0;
		return Math.min(1.2, beam + beamX * 0.6 + trail);
	}

	// --- Pattern 8: Heartbeat pulse ---
	function valPulse(nx: number, ny: number, t: number): number {
		const cx = nx - 0.5;
		const cy = (ny - 0.5) * 1.8;
		const dist = Math.sqrt(cx * cx + cy * cy);
		const beat1 = Math.sin(t * 3) * 0.5 + 0.5;
		const beat2 = Math.sin(t * 3 + 0.4) * 0.3 + 0.3;
		const beatPhase = Math.max(beat1, beat2);
		const ring1 = Math.max(0, 1 - Math.abs(dist - beatPhase * 0.5) * 12);
		const ring2 = Math.max(0, 1 - Math.abs(dist - beatPhase * 0.3) * 15);
		const ring3 = Math.max(0, 1 - Math.abs(dist - beatPhase * 0.7) * 10);
		const core = Math.max(0, 1 - dist * 5) * beatPhase;
		return Math.min(1.2, ring1 + ring2 * 0.7 + ring3 * 0.5 + core);
	}

	// --- Pattern 9: Pixel sort glitch ---
	function valSort(nx: number, ny: number, t: number): number {
		const bandY = Math.floor(ny * 20);
		const hash = Math.sin(bandY * 127.1 + 311.7) * 43758.5453;
		const seed = hash - Math.floor(hash);
		const shift = Math.sin(t * seed * 2 + seed * 10) * 0.3;
		const stretch = 1 + Math.sin(t * 0.5 + bandY * 0.3) * 0.5;
		const shiftedX = (nx + shift) * stretch;
		const glitchBand = Math.sin(bandY * 3.7 + t * 2) > 0.6 ? 1.3 : 1;
		const noise = Math.sin(shiftedX * 30 + bandY * 7 + t) * 0.3 + 0.5;
		const glitchLine = Math.sin(ny * 80 + t * 5) > 0.95 ? 1 : 0;
		return Math.max(0, Math.min(1.2, noise * glitchBand + glitchLine));
	}

	// --- Pattern 10: Voronoi cells ---
	const NUM_CELLS = 12;
	const cellData = Array.from({ length: NUM_CELLS }, (_, i) => ({
		sx: 0.08 + Math.random() * 0.12,
		sy: 0.06 + Math.random() * 0.1,
		ox: i * 2.1,
		oy: i * 1.7
	}));

	function valVoronoi(nx: number, ny: number, t: number): number {
		let minDist = 999;
		let secondDist = 999;
		for (let i = 0; i < NUM_CELLS; i++) {
			const c = cellData[i];
			const cx = 0.5 + 0.45 * Math.sin(t * c.sx + c.ox);
			const cy = 0.5 + 0.45 * Math.cos(t * c.sy + c.oy);
			const dx = nx - cx;
			const dy = (ny - cy) * 1.8;
			const d = Math.sqrt(dx * dx + dy * dy);
			if (d < minDist) {
				secondDist = minDist;
				minDist = d;
			} else if (d < secondDist) {
				secondDist = d;
			}
		}
		const edge = secondDist - minDist;
		const edgeVal = Math.max(0, 1 - edge * 12);
		const fill = Math.max(0, 1 - minDist * 4) * 0.4;
		return Math.min(1.2, edgeVal + fill);
	}

	// --- Pattern 11: Pulse + Scanner + Rain hybrid (Optricia) ---
	function valPulseScan(nx: number, ny: number, t: number): number {
		const cycle = (t * 0.15) % 1;
		const cx = nx - 0.5;
		const cy = (ny - 0.5) * 1.8;
		const dist = Math.sqrt(cx * cx + cy * cy);

		// Smooth ambient breathing (no noise — noise causes flicker)
		const ambient = 0.08 + 0.05 * Math.sin(t * 0.3);

		// Pulse rings — 2 smooth concentric
		const beat = Math.sin(t * 0.8) * 0.5 + 0.5;
		const ring1 = Math.max(0, 1 - Math.abs(dist - beat * 0.5) * 6);
		const ring2 = Math.max(0, 1 - Math.abs(dist - beat * 0.25) * 7);
		const core = Math.max(0, 1 - dist * 4) * beat * 0.4;
		const pulseVal = ring1 * 0.7 + ring2 * 0.4 + core;

		// Scanner — single slow beam
		const lineY1 = ((t * 0.08) % 1.6) - 0.3;
		const beam1 = Math.max(0, 1 - Math.abs(ny - lineY1) * 5);
		const trail1 = ny < lineY1 ? Math.max(0, 1 - (lineY1 - ny) * 3) * 0.3 : 0;
		const scanVal = beam1 + trail1;

		// Sparse rain — 2 drops
		let rainVal = 0;
		const col = Math.floor(nx * 200);
		for (let d = 0; d < 2; d++) {
			const hash = Math.sin(col * 127.1 + d * 311.7) * 43758.5453;
			const seed = hash - Math.floor(hash);
			const spd = 0.08 + seed * 0.2;
			const phase = seed * 12 + d * 3.7;
			const trailLen = 0.06 + seed * 0.12;
			const headY = ((t * spd + phase) % (1.0 + trailLen)) - trailLen * 0.3;
			const dv = ny - headY;
			if (dv >= 0 && dv <= trailLen) {
				rainVal = Math.max(rainVal, 1 - dv / trailLen);
			}
		}

		// Rare ripple event
		let rippleVal = 0;
		const rT = Math.floor(t * 0.1);
		const rSeed = Math.sin(rT * 123.456) * 43758.5453;
		if (rSeed - Math.floor(rSeed) > 0.5) {
			const rcx = Math.abs((Math.sin(rT * 45.67) * 43758.5453) % 1);
			const rcy = Math.abs((Math.sin(rT * 89.01) * 43758.5453) % 1);
			const rAge = t * 0.1 - rT;
			const rDist = Math.sqrt((nx - rcx) ** 2 + ((ny - rcy) * 1.8) ** 2);
			const rRing = Math.max(0, 1 - Math.abs(rDist - rAge * 0.4) * 10);
			rippleVal = rRing * Math.max(0, 1 - rAge * 2) * 0.5;
		}

		// Crossfade pulse/scanner
		const mix = Math.sin(cycle * Math.PI * 2) * 0.5 + 0.5;
		const pulseWeight = 0.3 + mix * 0.4;
		const scanWeight = 0.2 + (1 - mix) * 0.4;

		const combined =
			ambient + pulseVal * pulseWeight + scanVal * scanWeight + rainVal * 0.25 + rippleVal;
		return Math.min(1.0, combined);
	}

	const patternFns = [
		valBlobs,
		valWaves,
		valRipples,
		valRain,
		valTerrain,
		valMoire,
		valSpiral,
		valScanner,
		valPulse,
		valSort,
		valVoronoi,
		valPulseScan
	];

	function generateAscii() {
		const fn = patternFns[pattern] ?? patternFns[0];
		const currentRamp = ramps[rampIdx] ?? ramps[0];
		const boost = hoveredTrack !== null ? 1.15 : 1;
		const hasImg = imgReady && imgBrightness.length === cols * rows;
		let text = '';

		for (let y = 0; y < rows; y++) {
			for (let x = 0; x < cols; x++) {
				const nx = x / cols;
				const ny = y / rows;

				let mask = fn(nx, ny, time) * boost;
				mask += Math.sin(x * 0.35 + y * 0.25 + time * 1.5) * 0.03;

				if (mouseActive) {
					const mdx = nx - mouseX;
					const mdy = (ny - mouseY) * 1.8;
					const mDist = Math.sqrt(mdx * mdx + mdy * mdy);
					const mouseBoost = Math.max(0, 1 - mDist * 4) * 0.5;
					mask += mouseBoost;
				}

				mask = Math.max(0, Math.min(1.2, mask));

				let val: number;
				if (hasImg) {
					const brightness = imgBrightness[y * cols + x];
					// Image is the base, pattern adds movement on top
					val = brightness * 0.6 + mask * 0.3;
				} else {
					val = mask;
				}

				if (inverted) {
					val = Math.max(0, 1 - val);
				}

				if (val > 0.02) {
					const idx = Math.floor((1 - Math.min(1, val)) * (currentRamp.length - 1));
					text += currentRamp[Math.max(0, idx)];
				} else {
					text += ' ';
				}
			}
			text += '\n';
		}
		asciiOutput = text;
	}

	function calcDimensions() {
		if (!browser) return;
		if (window.innerWidth < 768) {
			// Mobile: full width, proportional height
			// top ~90px (logo + tracks), bottom ~30px (links)
			const cw = 5;
			const ch = 10;
			const asciiH = window.innerHeight - 120;
			cols = Math.floor(window.innerWidth / cw);
			// Keep aspect ratio reasonable — don't over-stretch vertically
			const maxRows = Math.floor(cols * 0.6);
			rows = Math.min(Math.floor(asciiH / ch), maxRows);
			cols = Math.max(30, Math.min(cols, 120));
			rows = Math.max(15, Math.min(rows, 60));
		} else {
			// Desktop: ASCII takes what it needs, right col gets the rest
			const rightW = Math.max(320, window.innerWidth * 0.22);
			const panelW = window.innerWidth - rightW;
			const cw = 7.22;
			const ch = 13;
			cols = Math.floor(panelW / cw);
			rows = Math.floor(window.innerHeight / ch);
			cols = Math.max(50, Math.min(cols, 200));
			rows = Math.max(25, Math.min(rows, 90));
		}
		sampleImage();
	}

	function handleMouseMove(e: MouseEvent) {
		const el = e.currentTarget as HTMLElement;
		const rect = el.getBoundingClientRect();
		mouseX = (e.clientX - rect.left) / rect.width;
		mouseY = (e.clientY - rect.top) / rect.height;
		mouseActive = true;
	}

	function handleMouseLeave() {
		mouseActive = false;
	}

	function animate() {
		time += 0.012 * speed;
		generateAscii();
		animId = requestAnimationFrame(animate);
	}

	function handleGlobalClick() {
		unlockAudio();
		window.removeEventListener('click', handleGlobalClick);
	}

	onMount(() => {
		if (!browser) return;
		preloadAllImages();
		initAudio();
		calcDimensions();
		animate();
		window.addEventListener('resize', calcDimensions);
		window.addEventListener('click', handleGlobalClick);
		scheduleNextChange();

		// Console ASCII logo
		const img = new Image();
		img.src = DEFAULT_BG;
		img.onload = () => {
			const c = document.createElement('canvas');
			const w = 60;
			const h = Math.round(w * (img.height / img.width) * 0.5);
			c.width = w;
			c.height = h;
			const ctx = c.getContext('2d')!;
			ctx.drawImage(img, 0, 0, w, h);
			const data = ctx.getImageData(0, 0, w, h).data;
			const chars = 'y6. ';
			let ascii = '';
			for (let y = 0; y < h; y++) {
				for (let x = 0; x < w; x++) {
					const i = (y * w + x) * 4;
					const brightness = (data[i] * 0.299 + data[i + 1] * 0.587 + data[i + 2] * 0.114) / 255;
					ascii += chars[Math.floor((1 - brightness) * (chars.length - 1))];
				}
				ascii += '\n';
			}
			console.log('%c' + ascii, 'color:#e6c619;font-size:10px;line-height:10px;font-family:monospace');
		};
	});

	onDestroy(() => {
		if (animId) cancelAnimationFrame(animId);
		clearTimeout(fontInterval);
		if (browser) {
			window.removeEventListener('resize', calcDimensions);
			window.removeEventListener('click', handleGlobalClick);
			audioElements.forEach((audio) => {
				audio.pause();
				audio.src = '';
			});
		}
	});
</script>

<div class="page" class:inverted>
	<div
		class="ascii-side"
		onmousemove={handleMouseMove}
		onmouseleave={handleMouseLeave}
		role="presentation"
	>
		<pre
			class="ascii"
			style="font-family: {asciiFonts[asciiFont] ?? asciiFonts[0]}">{asciiOutput}</pre>
	</div>

	<div class="divider"></div>
	<div class="right-col">
		<h1 class="title">
			{#each logoLetters as letter, i}
				<span class="logo-letter" style="font-family: {letterFonts[i]}">{letter}</span>
			{/each}
		</h1>

		<nav class="tracks">
			{#each trackOrder as idx, i}
				{@const release = releasesData[idx]}
				{#if i > 0}<span class="dot" aria-hidden="true">·</span>{/if}
				<a
					href={release.link}
					target="_blank"
					rel="noopener noreferrer"
					class="track"
					class:active={hoveredTrack === idx}
					class:playing={playingTrack === idx}
					onmouseenter={() => onTrackEnter(idx)}
					onmouseleave={() => onTrackLeave()}
					onclick={(e) => onTrackClick(e, idx)}
				>
					{release.title}
				</a>
			{/each}
		</nav>

		<hr class="sep" />

		<nav class="links">
			<a href="https://open.spotify.com/artist/1YBaOVZvHYH45YR7qXnWQK?si=VyWFS9ecTnqqrRZfJagq6Q" target="_blank" rel="noopener noreferrer">Spotify</a>
			<span class="dot" aria-hidden="true">·</span>
			<a href="https://music.apple.com/us/artist/ye66ow/1797884681" target="_blank" rel="noopener noreferrer">Apple Music</a>
			<span class="dot" aria-hidden="true">·</span>
			<a href="https://www.instagram.com/ye66oowww" target="_blank" rel="noopener noreferrer">Instagram</a>
			<br class="mobile-only" />
			<a class="mobile-only" href="https://open.spotify.com/playlist/1TLECsOwyzfMvbtfrjUR8X?si=32fc1dbb3afa4135" target="_blank" rel="noopener noreferrer">Peacemusic</a>
			<span class="dot mobile-only" aria-hidden="true">·</span>
			<a class="mobile-only" href="mailto:{EMAIL}">Contacts</a>
		</nav>
		<a class="bottom-link" href="https://open.spotify.com/playlist/1TLECsOwyzfMvbtfrjUR8X?si=32fc1dbb3afa4135" target="_blank" rel="noopener noreferrer">Peacemusic</a>
		<span class="bottom-link contact-line">
			<a href="mailto:{EMAIL}">Contacts</a>
			<span class="email-hint">{EMAIL}</span>
		</span>
	</div>

	<!-- <button class="panel-toggle" onclick={() => (showPanel = !showPanel)}>
		{showPanel ? '×' : '≡'}
	</button> -->
</div>

<style>
	/* ===== Base / shared ===== */
	.page {
		position: fixed;
		inset: 0;
		display: flex;
		flex-direction: column;
		background: #fff;
		font-family: 'JetBrains Mono', 'Courier New', monospace;
		overflow: hidden;
	}

	.page.inverted {
		background: #000;
	}

	.ascii-side {
		display: flex;
		align-items: center;
		justify-content: center;
		overflow: hidden;
		cursor: crosshair;
	}

	.ascii {
		margin: 0;
		color: #000;
		user-select: none;
		letter-spacing: 0;
	}

	.page.inverted .ascii {
		color: #fff;
	}

	.logo-letter {
		display: inline-block;
		transition: font-family 0.3s ease;
	}

	.track {
		text-decoration: none;
		color: #111;
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		font-weight: 300;
		letter-spacing: 0.18em;
		text-transform: uppercase;
		cursor: pointer;
		transition: all 0.2s ease;
		border: none;
		background: none;
		white-space: nowrap;
	}

	.page.inverted .track {
		color: #ddd;
	}

	.track:hover,
	.track.active {
		color: #000;
	}

	.page.inverted .track:hover,
	.page.inverted .track.active {
		color: #fff;
	}

	.track.playing {
		color: #000;
		text-decoration: underline;
		text-underline-offset: 3px;
	}

	.page.inverted .track.playing {
		color: #fff;
	}

	.dot {
		color: #999;
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		user-select: none;
	}

	.page.inverted .dot {
		color: #555;
	}

	.links a {
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		font-weight: 300;
		color: #111;
		text-decoration: none;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		transition: color 0.15s;
	}

	.links a:hover {
		color: #555;
	}

	.page.inverted .links a {
		color: #ddd;
	}

	.page.inverted .links a:hover {
		color: #999;
	}

	.bottom-link {
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		font-weight: 300;
		color: #111;
		text-decoration: none;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		margin-top: 1.5rem;
		transition: color 0.15s;
	}

	.bottom-link:hover {
		color: #555;
	}

	.page.inverted .bottom-link {
		color: #ddd;
	}

	.page.inverted .bottom-link:hover {
		color: #999;
	}

	.contact-line a {
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		font-weight: 300;
		color: #111;
		text-decoration: none;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		transition: color 0.15s;
	}

	.contact-line a:hover {
		color: #555;
	}

	.page.inverted .contact-line a {
		color: #ddd;
	}

	.page.inverted .contact-line a:hover {
		color: #999;
	}

	.email-hint {
		font-family: 'Outfit', 'Helvetica Neue', Arial, sans-serif;
		font-weight: 300;
		color: #999;
		letter-spacing: 0.1em;
		text-transform: none;
		margin-left: 0.25rem;
		font-size: 0.85em;
	}

	.page.inverted .email-hint {
		color: #666;
	}

	.sep {
		border: none;
		border-top: 1px solid #222;
	}

	.page.inverted .sep {
		border-top-color: #555;
	}

	.divider {
		width: 1px;
		background: #222;
		align-self: stretch;
	}

	.page.inverted .divider {
		background: #555;
	}

	/* .panel-toggle styles hidden with button
	.panel-toggle { ... }
	*/

	/* ===== Mobile ===== */
	@media (max-width: 767px) {
		.right-col {
			display: contents;
		}

		.title {
			order: 1;
			font-size: 2.5rem;
			font-weight: 700;
			margin: 0;
			padding: calc(1.2rem + env(safe-area-inset-top, 0px)) 1rem 0.8rem;
			color: #111;
			letter-spacing: -0.02em;
			line-height: 1;
			display: flex;
			justify-content: center;
			gap: 0;
			user-select: none;
		}

		.page.inverted .title {
			color: #eee;
		}

		.tracks {
			order: 2;
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			justify-content: center;
			align-items: center;
			padding: 0 1rem 0;
			gap: 0.3rem 0;
		}

		.track {
			display: inline;
			font-size: 0.95rem;
			font-weight: 400;
			padding: 0;
		}

		.dot {
			font-size: 0.8rem;
			color: #444;
			margin: 0 0.6rem;
		}

		.page.inverted .dot {
			color: #888;
		}

		.sep {
			order: 2;
			display: block;
			margin: 0.8rem 0;
		}

		.divider {
			display: none;
		}

		.bottom-link {
			display: none;
		}

		.mobile-only {
			display: inline;
		}

		.links {
			order: 3;
			text-align: center;
			padding: 0 2.5rem 0.8rem;
			line-height: 1.2;
		}

		.links a {
			font-size: 0.7rem;
		}

		.links .dot {
			font-size: 0.55rem;
			color: #444;
			margin: 0 0.1rem;
		}

		.ascii-side {
			order: 4;
			flex: 1;
			min-height: 0;
		}

		.ascii {
			font-size: 8px;
			line-height: 10px;
		}
	}

	/* ===== Desktop: side-by-side 70/30 ===== */
	@media (min-width: 768px) {
		.page {
			flex-direction: row;
		}

		.ascii-side {
			flex: 0 0 auto;
			align-items: flex-start;
			justify-content: flex-start;
		}

		.ascii {
			font-size: 12px;
			line-height: 13px;
		}

		.right-col {
			flex: 1;
			min-width: 320px;
			display: flex;
			flex-direction: column;
			overflow-y: auto;
			padding: 2.5rem 0;
		}

		.title {
			font-size: 4rem;
			font-weight: 700;
			margin: 0 0 5.5rem;
			padding: 0 2rem;
			color: #111;
			letter-spacing: -0.02em;
			line-height: 1;
			display: inline-flex;
			width: auto;
			gap: 0;
			user-select: none;
		}

		.page.inverted .title {
			color: #eee;
		}

		.tracks {
			display: flex;
			flex-direction: column;
			gap: 0.2rem;
			padding: 0 2rem;
			margin-bottom: 5rem;
		}

		.track {
			display: block;
			font-size: 1.4rem;
			letter-spacing: 0.22em;
			padding: 0.4rem 0;
		}

		.track:hover,
		.track.active {
			letter-spacing: 0.28em;
		}

		.dot {
			display: none;
		}

		.mobile-only {
			display: none;
		}

		.sep, .divider {
			display: none;
		}

		.links {
			display: flex;
			flex-direction: column;
			gap: 0.5rem;
			padding: 0 2rem;
		}

		.links a {
			font-size: 0.85rem;
			letter-spacing: 0.18em;
		}

		.bottom-link {
			padding: 0 2rem;
			font-size: 0.85rem;
			letter-spacing: 0.18em;
		}

		.contact-line a {
			font-size: 0.85rem;
			letter-spacing: 0.18em;
		}

		.email-hint {
			font-size: 0.75rem;
		}
	}

	@media (min-width: 1200px) {
		.title {
			font-size: 5rem;
		}

		.track {
			font-size: 1.6rem;
		}
	}
</style>
