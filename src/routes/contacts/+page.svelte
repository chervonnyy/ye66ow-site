<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	let contactForm: HTMLFormElement;
	let name = '';
	let email = '';
	let message = '';
	let isSubmitting = false;

	const socialLinks = [
		{
			name: 'GitHub',
			url: 'https://github.com/ye66ow',
			icon: '💻'
		},
		{
			name: 'Twitter',
			url: 'https://twitter.com/ye66ow',
			icon: '🐦'
		},
		{
			name: 'Instagram',
			url: 'https://instagram.com/ye66ow',
			icon: '📷'
		},
		{
			name: 'Email',
			url: 'mailto:contact@ye66ow.com',
			icon: '✉️'
		}
	];

	function handleSubmit(event: Event) {
		event.preventDefault();
		isSubmitting = true;

		// Здесь будет логика отправки формы
		setTimeout(() => {
			isSubmitting = false;
			alert('Message sent! (This is a demo)');
			name = '';
			email = '';
			message = '';
		}, 2000);
	}

	onMount(() => {
		if (browser) {
			console.log('Contacts page loaded');
		}
	});
</script>

<svelte:head>
	<title>Contacts - ye66ow</title>
	<meta name="description" content="ye66ow Contacts" />
</svelte:head>

<div class="contacts-page">
	<div class="container">
		<header class="header">
			<h1>CONTACTS</h1>
			<a href="/" class="back-button">← BACK</a>
		</header>

		<div class="content">
			<div class="contact-info">
				<h2>GET IN TOUCH</h2>
				<p class="description">
					Ready to collaborate or just want to say hello? Drop me a message and I'll get back to you
					as soon as possible.
				</p>

				<div class="social-links">
					<h3>SOCIAL LINKS</h3>
					<div class="social-grid">
						{#each socialLinks as link}
							<a href={link.url} class="social-link" target="_blank" rel="noopener noreferrer">
								<span class="social-icon">{link.icon}</span>
								<span class="social-name">{link.name}</span>
							</a>
						{/each}
					</div>
				</div>
			</div>

			<div class="contact-form-section">
				<h2>SEND MESSAGE</h2>
				<form bind:this={contactForm} on:submit={handleSubmit} class="contact-form">
					<div class="form-group">
						<label for="name">NAME</label>
						<input
							type="text"
							id="name"
							bind:value={name}
							required
							placeholder="Your name"
							class="form-input"
						/>
					</div>

					<div class="form-group">
						<label for="email">EMAIL</label>
						<input
							type="email"
							id="email"
							bind:value={email}
							required
							placeholder="your@email.com"
							class="form-input"
						/>
					</div>

					<div class="form-group">
						<label for="message">MESSAGE</label>
						<textarea
							id="message"
							bind:value={message}
							required
							placeholder="Your message here..."
							rows="6"
							class="form-textarea"
						></textarea>
					</div>

					<button type="submit" class="submit-button" disabled={isSubmitting}>
						{isSubmitting ? 'SENDING...' : 'SEND MESSAGE'}
					</button>
				</form>
			</div>
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
		font-family: 'Orbitron', 'Courier New', monospace;
		overflow-x: hidden;
	}

	.contacts-page {
		min-height: 100vh;
		background: linear-gradient(135deg, #000 0%, #001a33 50%, #000 100%);
		padding: 20px;
	}

	.container {
		max-width: 1000px;
		margin: 0 auto;
	}

	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 60px;
		padding-bottom: 30px;
		border-bottom: 2px solid rgba(0, 255, 0, 0.3);
	}

	.header h1 {
		font-size: 3rem;
		margin: 0;
		text-shadow: 0 0 30px rgba(0, 255, 0, 0.5);
		letter-spacing: 4px;
		font-weight: 900;
	}

	.back-button {
		color: #fff;
		text-decoration: none;
		padding: 15px 30px;
		border: 2px solid rgba(255, 255, 255, 0.3);
		border-radius: 12px;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
		font-weight: bold;
		letter-spacing: 2px;
	}

	.back-button:hover {
		background: rgba(0, 255, 0, 0.2);
		border-color: rgba(0, 255, 0, 0.6);
		transform: translateX(-5px);
	}

	.content {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 60px;
		align-items: start;
	}

	.contact-info h2,
	.contact-form-section h2 {
		font-size: 2rem;
		margin: 0 0 30px 0;
		color: #fff;
		letter-spacing: 3px;
		font-weight: 700;
	}

	.description {
		font-size: 1.1rem;
		line-height: 1.6;
		color: rgba(255, 255, 255, 0.8);
		margin-bottom: 40px;
	}

	.social-links h3 {
		font-size: 1.3rem;
		margin: 0 0 20px 0;
		color: #fff;
		letter-spacing: 2px;
		font-weight: 600;
	}

	.social-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 15px;
	}

	.social-link {
		display: flex;
		align-items: center;
		gap: 12px;
		padding: 15px 20px;
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.2);
		border-radius: 12px;
		text-decoration: none;
		color: #fff;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
	}

	.social-link:hover {
		background: rgba(0, 255, 0, 0.2);
		border-color: rgba(0, 255, 0, 0.5);
		transform: translateY(-3px);
	}

	.social-icon {
		font-size: 1.5rem;
	}

	.social-name {
		font-weight: bold;
		letter-spacing: 1px;
	}

	.contact-form {
		background: rgba(0, 0, 0, 0.6);
		padding: 40px;
		border-radius: 20px;
		backdrop-filter: blur(20px);
		border: 1px solid rgba(255, 255, 255, 0.1);
	}

	.form-group {
		margin-bottom: 25px;
	}

	.form-group label {
		display: block;
		margin-bottom: 8px;
		font-weight: bold;
		letter-spacing: 1px;
		color: #fff;
	}

	.form-input,
	.form-textarea {
		width: 100%;
		padding: 15px 20px;
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.3);
		border-radius: 10px;
		color: #fff;
		font-family: 'Orbitron', 'Courier New', monospace;
		font-size: 1rem;
		transition: all 0.3s ease;
		backdrop-filter: blur(10px);
	}

	.form-input:focus,
	.form-textarea:focus {
		outline: none;
		border-color: rgba(0, 255, 0, 0.6);
		background: rgba(255, 255, 255, 0.15);
		box-shadow: 0 0 20px rgba(0, 255, 0, 0.2);
	}

	.form-input::placeholder,
	.form-textarea::placeholder {
		color: rgba(255, 255, 255, 0.5);
	}

	.submit-button {
		width: 100%;
		padding: 18px 30px;
		background: linear-gradient(45deg, rgba(0, 255, 0, 0.8), rgba(0, 255, 255, 0.8));
		border: none;
		border-radius: 12px;
		color: #000;
		font-family: 'Orbitron', 'Courier New', monospace;
		font-size: 1.1rem;
		font-weight: bold;
		letter-spacing: 2px;
		cursor: pointer;
		transition: all 0.3s ease;
		text-transform: uppercase;
	}

	.submit-button:hover:not(:disabled) {
		background: linear-gradient(45deg, rgba(0, 255, 0, 1), rgba(0, 255, 255, 1));
		transform: translateY(-2px);
		box-shadow: 0 8px 25px rgba(0, 255, 0, 0.4);
	}

	.submit-button:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}

	@media (max-width: 768px) {
		.contacts-page {
			padding: 15px;
		}

		.header h1 {
			font-size: 2.2rem;
			letter-spacing: 2px;
		}

		.back-button {
			padding: 12px 20px;
			font-size: 0.9rem;
		}

		.content {
			grid-template-columns: 1fr;
			gap: 40px;
		}

		.contact-info h2,
		.contact-form-section h2 {
			font-size: 1.6rem;
			letter-spacing: 2px;
		}

		.contact-form {
			padding: 25px;
		}

		.social-grid {
			grid-template-columns: 1fr;
		}

		.social-link {
			padding: 12px 15px;
		}
	}

	@media (max-width: 480px) {
		.header h1 {
			font-size: 1.8rem;
		}

		.contact-info h2,
		.contact-form-section h2 {
			font-size: 1.4rem;
		}

		.contact-form {
			padding: 20px;
		}

		.form-input,
		.form-textarea {
			padding: 12px 15px;
			font-size: 0.9rem;
		}

		.submit-button {
			padding: 15px 25px;
			font-size: 1rem;
		}
	}
</style>
