<script lang="ts">
	import { onMount } from 'svelte';
	import { fade } from 'svelte/transition';

	let showIntro = $state(true);
	let typedText = $state('');
	const fullText = 'Ready to go? Take the key and start the journey!';
	let typingDone = $state(false);

	let keyPhase = $state<'idle' | 'pickup' | 'move' | 'turn'>('idle');
	let flashing = $state(false);
	let revealMain = $state(false);
	let introFinished = $state(false);
	let introHidden = $state(false);

	let ignitionAudio: HTMLAudioElement | null = null;

	const STORAGE_KEY = 'the-long-haul-intro-seen';

	onMount(() => {
		const hasSeenIntro = localStorage.getItem(STORAGE_KEY) === 'true';

		if (hasSeenIntro) {
			showIntro = false;
			introHidden = true;
			revealMain = true;
			return;
		}

		ignitionAudio = new Audio('/media/ignition.mp3');
		ignitionAudio.volume = 0.85;

		startTyping();
	});

	function startTyping() {
		let i = 0;

		function typeNext() {
			if (i < fullText.length) {
				typedText = fullText.slice(0, i + 1);
				i += 1;
				setTimeout(typeNext, 45);
			} else {
				typingDone = true;
			}
		}

		typeNext();
	}

	async function handleKeyClick() {
		if (!typingDone || keyPhase !== 'idle' || introFinished) return;

		// Phase 1: key lifts up
		keyPhase = 'pickup';

		// Phase 2: key moves to right side of screen
		setTimeout(() => {
			keyPhase = 'move';
		}, 600);

		// Phase 3: key turns like an ignition + audio plays
		setTimeout(async () => {
			keyPhase = 'turn';
			try {
				await ignitionAudio?.play();
			} catch (e) {
				console.error('Audio failed:', e);
			}
		}, 1600);

		// Flash at 8s after audio starts (9.6s after click)
		setTimeout(() => {
			flashing = true;
			ignitionAudio?.pause();
		}, 9600);

		// Hide intro and reveal main page while flash is fully opaque
		setTimeout(() => {
			introHidden = true;
			showIntro = false;
			revealMain = true;
			introFinished = true;
			localStorage.setItem(STORAGE_KEY, 'true');
		}, 10200);

		// Remove flash overlay after full fade
		setTimeout(() => {
			flashing = false;
		}, 12000);
	}
</script>

<svelte:head>
	<title>The Long Haul</title>
	<meta
		name="description"
		content="One project. One journey. One final submission. The Long Haul YSWS."
	/>
</svelte:head>

{#if showIntro && !introHidden}
	<section class="intro-screen">
		<div class="intro-content">
			<div class="intro-text-block">
				<h1 class="typed-line">
					{typedText}<span class:blink={!typingDone} class="cursor">|</span>
				</h1>
			</div>

			<div class="key-wrap">
				<button
					class="key-button"
					class:ready={typingDone}
					class:pickup={keyPhase === 'pickup'}
					class:move={keyPhase === 'move'}
					class:turn={keyPhase === 'turn'}
					onclick={handleKeyClick}
					aria-label="Start the journey"
				>
					<img src="/pics/key.png" alt="Ignition key" class="key-image" />
				</button>

				{#if typingDone && keyPhase === 'idle'}
					<p class="subprompt" in:fade={{ duration: 250 }}>
						(Click the key to start)
					</p>
				{/if}
			</div>
		</div>
	</section>
{/if}

{#if flashing}
	<div class="flash-overlay"></div>
{/if}

{#if revealMain}
	<section class="main-page" in:fade={{ duration: 700 }}>
		<a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="orpheus-link">
			<img src="/pics/flag-orpheus-top.png" alt="Hack Club" class="orpheus-flag" />
		</a>

		<div class="hero-section">
			<div class="hero">
				<p class="eyebrow">Hack Club YSWS Proposal</p>
				<h1>The Long Haul</h1>
				<p class="tagline">
					Build one giant project across the whole YSWS. No hopping between ideas. No tiny spam
					submits. Just one serious journey.
				</p>
			</div>
		</div>

		<div class="content-section">
			<div class="content-inner">
				<div class="premise-card">
					<h2>Premise</h2>
					<p>
						A YSWS where you're only allowed to submit <strong>one</strong> project throughout the
						entire event. The goal is to go all-in on one massive build and submit it at the end.
					</p>
				</div>

				<div class="reward-card">
					<h2>Rewards</h2>
					<p>
						Your payout is affected by project quality, time invested, and sidequests completed.
						Better projects earn more <strong>gas</strong>, which unlocks higher shop tiers.
					</p>
				</div>
			</div>
		</div>

		<footer class="site-footer">
			<p class="footer-initiative">A Hack Club initiative</p>
			<p class="footer-legal">
				The Hack Foundation is a 501(c)(3) non-profit organization d.b.a. Hack Club (EIN: 81-2908499)
			</p>
			<p class="footer-links">
				<a href="https://hackclub.com" target="_blank" rel="noopener noreferrer">About Hack Club</a>
			</p>
			<p class="footer-note">Dino imagery from rawr.hackclub.com!</p>
			<p class="footer-credit">By @Ckacha. Open source.</p>
		</footer>
	</section>
{/if}

<style>
	:global(html, body) {
		margin: 0;
		padding: 0;
		background: #000;
		color: #f5f5f5;
		font-family:
			Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
		scroll-behavior: smooth;
	}

	/* ── Intro screen ── */

	.intro-screen {
		position: fixed;
		inset: 0;
		background: #000;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 2rem;
		overflow: hidden;
		z-index: 10;
	}

	.intro-content {
		width: min(1150px, 100%);
		display: grid;
		grid-template-columns: minmax(0, 1.25fr) minmax(220px, 0.75fr);
		gap: 2rem;
		align-items: center;
	}

	.intro-text-block {
		display: flex;
		flex-direction: column;
		justify-content: center;
		min-width: 0;
	}

	.typed-line {
		margin: 0;
		font-size: clamp(2rem, 4.7vw, 4.6rem);
		line-height: 1.15;
		font-weight: 400;
		letter-spacing: -0.01em;
		font-family: 'Georgia', 'Times New Roman', serif;
		max-width: 11ch;
		min-height: 3.3em;
	}

	.cursor {
		display: inline-block;
		margin-left: 0.1em;
	}

	.cursor.blink {
		animation: blink 0.9s steps(1) infinite;
	}

	.subprompt {
		margin-top: 0.75rem;
		font-size: 0.85rem;
		color: #a8a8a8;
		letter-spacing: 0.08em;
		text-align: center;
	}

	.key-wrap {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.key-button {
		background: transparent;
		border: none;
		padding: 0;
		cursor: default;
		opacity: 0.72;
		filter: grayscale(1) brightness(0.92);
		transition:
			transform 0.6s cubic-bezier(0.25, 0.8, 0.25, 1),
			opacity 0.3s ease,
			filter 0.3s ease;
		position: relative;
		z-index: 15;
	}

	.key-button.ready {
		cursor: pointer;
		opacity: 1;
		filter: grayscale(0) brightness(1);
	}

	.key-button.ready:hover {
		transform: scale(1.03);
	}

	/* Phase 1: picked up — lifts and slightly enlarges */
	.key-button.pickup {
		transform: translateY(-30px) scale(1.1);
		opacity: 1;
		filter: grayscale(0) brightness(1.1);
		cursor: default;
		transition: transform 0.5s cubic-bezier(0.25, 0.8, 0.25, 1);
	}

	/* Phase 2: moves to right side of viewport */
	.key-button.move {
		position: fixed;
		top: 50%;
		right: 60px;
		transform: translate(0, -50%) rotate(0deg) scale(1);
		opacity: 1;
		filter: grayscale(0) brightness(1);
		cursor: default;
		transition:
			top 0.8s cubic-bezier(0.25, 0.8, 0.25, 1),
			right 0.8s cubic-bezier(0.25, 0.8, 0.25, 1),
			transform 0.8s cubic-bezier(0.25, 0.8, 0.25, 1);
	}

	/* Phase 3: turns like an ignition key */
	.key-button.turn {
		position: fixed;
		top: 50%;
		right: 60px;
		transform: translate(0, -50%) rotate(90deg) scale(1.05);
		opacity: 1;
		filter: grayscale(0) brightness(1.2);
		cursor: default;
		transition: transform 0.6s cubic-bezier(0.3, 0.9, 0.3, 1);
	}

	.key-image {
		display: block;
		width: min(300px, 30vw);
		height: auto;
		user-select: none;
		pointer-events: none;
	}

	/* ── Flash overlay ── */

	.flash-overlay {
		position: fixed;
		inset: 0;
		background: #fff;
		animation: flashReveal 2.4s ease forwards;
		pointer-events: none;
		z-index: 50;
	}

	/* ── Main page ── */

	.main-page {
		position: relative;
	}

	.orpheus-link {
		position: fixed;
		top: 0;
		left: 16px;
		z-index: 30;
		line-height: 0;
	}

	.orpheus-flag {
		width: 112px;
		height: auto;
		display: block;
		transition: transform 0.2s ease;
	}

	.orpheus-link:hover .orpheus-flag {
		transform: scale(1.05);
	}

	.hero-section {
		min-height: 100vh;
		background: url('/pics/background.png') center / cover no-repeat;
		display: flex;
		align-items: center;
		padding: 5rem 2rem;
	}

	.hero {
		width: min(1100px, 100%);
		margin: 0 auto;
	}

	.eyebrow {
		color: #8b949e;
		text-transform: uppercase;
		letter-spacing: 0.12em;
		font-size: 0.8rem;
		margin-bottom: 1rem;
	}

	.hero h1 {
		font-size: clamp(3rem, 7vw, 6rem);
		line-height: 0.95;
		margin: 0;
		letter-spacing: -0.04em;
	}

	.tagline {
		margin-top: 1.25rem;
		font-size: 1.2rem;
		line-height: 1.7;
		max-width: 780px;
		color: #d0d7de;
	}

	.content-section {
		background: linear-gradient(180deg, #6A9D2A 0%, #496900 100%);
		padding: 4rem 2rem;
	}

	.content-inner {
		width: min(1100px, 100%);
		margin: 0 auto;
	}

	.premise-card,
	.reward-card {
		margin-top: 1.5rem;
		max-width: 760px;
		padding: 1.25rem 1.35rem;
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.15);
		border-radius: 18px;
		backdrop-filter: blur(10px);
	}

	.premise-card:first-child {
		margin-top: 0;
	}

	.premise-card h2,
	.reward-card h2 {
		margin: 0 0 0.5rem;
		font-size: 1.05rem;
	}

	.premise-card p,
	.reward-card p {
		margin: 0;
		color: #e8eaed;
		line-height: 1.7;
	}

	/* ── Footer ── */

	.site-footer {
		background: #8492a6;
		padding: 2.5rem 2rem;
		text-align: center;
		color: #fff;
	}

	.footer-initiative {
		margin: 0 0 0.75rem;
		font-size: 1rem;
		font-weight: 600;
		letter-spacing: 0.04em;
	}

	.footer-legal {
		margin: 0 0 0.75rem;
		font-size: 0.78rem;
		color: rgba(255, 255, 255, 0.8);
		line-height: 1.5;
	}

	.footer-links a {
		color: #fff;
		text-decoration: underline;
		text-underline-offset: 3px;
		font-size: 0.85rem;
	}

	.footer-links a:hover {
		color: rgba(255, 255, 255, 0.8);
	}

	.footer-links {
		margin: 0 0 0.75rem;
	}

	.footer-note {
		margin: 0 0 0.35rem;
		font-size: 0.78rem;
		color: rgba(255, 255, 255, 0.7);
	}

	.footer-credit {
		margin: 0;
		font-size: 0.78rem;
		color: rgba(255, 255, 255, 0.7);
	}

	/* ── Keyframes ── */

	@keyframes blink {
		0%, 49% {
			opacity: 1;
		}
		50%, 100% {
			opacity: 0;
		}
	}

	@keyframes flashReveal {
		0% {
			opacity: 0;
		}
		15% {
			opacity: 1;
		}
		50% {
			opacity: 1;
		}
		100% {
			opacity: 0;
		}
	}

	/* ── Mobile ── */

	@media (max-width: 820px) {
		.intro-content {
			grid-template-columns: 1fr;
			text-align: center;
		}

		.intro-text-block {
			align-items: center;
		}

		.key-wrap {
			justify-content: center;
		}

		.typed-line {
			max-width: 100%;
			min-height: auto;
		}

		.key-image {
			width: min(260px, 60vw);
		}

		.key-button.move,
		.key-button.turn {
			right: 20px;
		}

		.orpheus-flag {
			width: 80px;
		}
	}
</style>
