<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	let musicContainer: HTMLDivElement;
	let isPlaying = false;
	let currentTrack = 0;

	// Список треков (заглушка)
	const tracks = [
		{
			id: 1,
			title: "ye66ow - Track 1",
			duration: "3:45",
			genre: "Electronic"
		},
		{
			id: 2,
			title: "ye66ow - Track 2", 
			duration: "4:12",
			genre: "Ambient"
		},
		{
			id: 3,
			title: "ye66ow - Track 3",
			duration: "2:58",
			genre: "Experimental"
		}
	];

	function togglePlay() {
		isPlaying = !isPlaying;
		// Здесь будет логика воспроизведения
		console.log(isPlaying ? 'Playing' : 'Paused');
	}

	function selectTrack(index: number) {
		currentTrack = index;
		// Здесь будет логика выбора трека
		console.log('Selected track:', tracks[index].title);
	}

	onMount(() => {
		if (browser) {
			// Инициализация музыкального плеера
			console.log('Music page loaded');
		}
	});
</script>

<svelte:head>
	<title>Music - ye66ow</title>
	<meta name="description" content="ye66ow Music" />
</svelte:head>

<div class="music-page">
	<div class="container">
		<header class="header">
			<h1>🎵 Music</h1>
			<a href="/" class="back-button">← Back</a>
		</header>

		<div class="player-section">
			<div class="current-track">
				<h2>{tracks[currentTrack].title}</h2>
				<p class="track-info">{tracks[currentTrack].genre} • {tracks[currentTrack].duration}</p>
			</div>

			<div class="player-controls">
				<button class="control-btn prev-btn" on:click={() => selectTrack(Math.max(0, currentTrack - 1))}>
					⏮
				</button>
				<button class="control-btn play-btn" on:click={togglePlay}>
					{isPlaying ? '⏸' : '▶'}
				</button>
				<button class="control-btn next-btn" on:click={() => selectTrack(Math.min(tracks.length - 1, currentTrack + 1))}>
					⏭
				</button>
			</div>

			<div class="progress-bar">
				<div class="progress" style="width: 30%"></div>
			</div>
		</div>

		<div class="track-list">
			<h3>Track List</h3>
			{#each tracks as track, index}
				<div 
					class="track-item" 
					class:active={index === currentTrack}
					on:click={() => selectTrack(index)}
				>
					<div class="track-number">{index + 1}</div>
					<div class="track-details">
						<div class="track-title">{track.title}</div>
						<div class="track-meta">{track.genre} • {track.duration}</div>
					</div>
					<div class="track-status">
						{#if index === currentTrack && isPlaying}
							🎵
						{:else if index === currentTrack}
							⏸
						{/if}
					</div>
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	:global(html),
	:global(body) {
		margin: 0;
		padding: 0;
		height: 100vh;
		background: #000;
		color: #fff;
		font-family: 'Courier New', monospace;
		overflow-x: hidden;
	}

	.music-page {
		min-height: 100vh;
		background: linear-gradient(135deg, #000 0%, #1a0033 50%, #000 100%);
		padding: 20px;
	}

	.container {
		max-width: 800px;
		margin: 0 auto;
	}

	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 40px;
		padding-bottom: 20px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.2);
	}

	.header h1 {
		font-size: 2.5rem;
		margin: 0;
		text-shadow: 0 0 20px rgba(255, 0, 255, 0.5);
	}

	.back-button {
		color: #fff;
		text-decoration: none;
		padding: 10px 20px;
		border: 1px solid rgba(255, 255, 255, 0.3);
		border-radius: 8px;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
	}

	.back-button:hover {
		background: rgba(255, 255, 255, 0.1);
		border-color: rgba(255, 0, 255, 0.5);
		transform: translateX(-5px);
	}

	.player-section {
		background: rgba(0, 0, 0, 0.6);
		border-radius: 16px;
		padding: 30px;
		margin-bottom: 40px;
		backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.1);
	}

	.current-track h2 {
		font-size: 1.8rem;
		margin: 0 0 10px 0;
		color: #fff;
	}

	.track-info {
		color: rgba(255, 255, 255, 0.7);
		margin: 0 0 30px 0;
		font-size: 1rem;
	}

	.player-controls {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 20px;
		margin-bottom: 30px;
	}

	.control-btn {
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.3);
		color: #fff;
		width: 60px;
		height: 60px;
		border-radius: 50%;
		font-size: 1.5rem;
		cursor: pointer;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
	}

	.control-btn:hover {
		background: rgba(255, 0, 255, 0.3);
		border-color: rgba(255, 0, 255, 0.6);
		transform: scale(1.1);
	}

	.play-btn {
		width: 80px;
		height: 80px;
		font-size: 2rem;
		background: rgba(255, 0, 255, 0.2);
		border-color: rgba(255, 0, 255, 0.5);
	}

	.play-btn:hover {
		background: rgba(255, 0, 255, 0.4);
		transform: scale(1.15);
	}

	.progress-bar {
		width: 100%;
		height: 6px;
		background: rgba(255, 255, 255, 0.2);
		border-radius: 3px;
		overflow: hidden;
	}

	.progress {
		height: 100%;
		background: linear-gradient(90deg, #ff00ff, #00ffff);
		border-radius: 3px;
		transition: width 0.3s ease;
	}

	.track-list h3 {
		font-size: 1.5rem;
		margin: 0 0 20px 0;
		color: #fff;
	}

	.track-item {
		display: flex;
		align-items: center;
		padding: 15px;
		margin-bottom: 10px;
		background: rgba(255, 255, 255, 0.05);
		border-radius: 12px;
		cursor: pointer;
		transition: all 0.3s ease;
		border: 1px solid transparent;
	}

	.track-item:hover {
		background: rgba(255, 255, 255, 0.1);
		border-color: rgba(255, 0, 255, 0.3);
		transform: translateX(10px);
	}

	.track-item.active {
		background: rgba(255, 0, 255, 0.2);
		border-color: rgba(255, 0, 255, 0.5);
	}

	.track-number {
		width: 30px;
		text-align: center;
		font-weight: bold;
		color: rgba(255, 255, 255, 0.7);
		margin-right: 15px;
	}

	.track-details {
		flex: 1;
	}

	.track-title {
		font-size: 1.1rem;
		font-weight: bold;
		margin-bottom: 5px;
		color: #fff;
	}

	.track-meta {
		font-size: 0.9rem;
		color: rgba(255, 255, 255, 0.6);
	}

	.track-status {
		font-size: 1.2rem;
		width: 30px;
		text-align: center;
	}

	@media (max-width: 768px) {
		.music-page {
			padding: 15px;
		}

		.header h1 {
			font-size: 2rem;
		}

		.player-section {
			padding: 20px;
		}

		.current-track h2 {
			font-size: 1.5rem;
		}

		.control-btn {
			width: 50px;
			height: 50px;
			font-size: 1.2rem;
		}

		.play-btn {
			width: 70px;
			height: 70px;
			font-size: 1.8rem;
		}

		.track-item {
			padding: 12px;
		}

		.track-title {
			font-size: 1rem;
		}

		.track-meta {
			font-size: 0.8rem;
		}
	}
</style>

