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
	let musicAudio: HTMLAudioElement | null = null;
	let muted = $state(false);
	let volume = $state(0.5);
	let musicFadeInterval: ReturnType<typeof setInterval> | null = null;
	let bonusCat = $state(false);
	let bonusCatInterval: ReturnType<typeof setInterval> | null = null;

	function startBonusCatChance() {
		bonusCatInterval = setInterval(() => {
			if (!bonusCat && Math.random() < 0.005) {
				bonusCat = true;
				const pop = new Audio('/media/pop.mp3');
				pop.volume = muted ? 0 : volume;
				pop.play().catch(() => {});
				setTimeout(() => {
					bonusCat = false;
				}, 4000);
			}
		}, 1000);
	}

	const STORAGE_KEY = 'the-long-haul-intro-seen';

	function startMusic() {
		musicAudio = new Audio('/media/music.mp3');
		musicAudio.volume = 0;
		musicAudio.loop = false;

		musicAudio.addEventListener('timeupdate', () => {
			if (!musicAudio) return;
			const timeLeft = musicAudio.duration - musicAudio.currentTime;
			// Fade out in last 3 seconds
			if (timeLeft <= 3 && timeLeft > 0) {
				musicAudio.volume = Math.max(0, (timeLeft / 3) * (muted ? 0 : volume));
			}
		});

		musicAudio.addEventListener('ended', () => {
			if (!musicAudio) return;
			musicAudio.currentTime = 0;
			musicAudio.volume = 0;
			musicAudio.play().then(() => fadeInMusic()).catch(() => {});
		});

		musicAudio.play().then(() => fadeInMusic()).catch((e) => {
			console.error('Music autoplay failed:', e);
		});
	}

	function fadeInMusic() {
		if (!musicAudio) return;
		let vol = 0;
		const target = muted ? 0 : volume;
		if (musicFadeInterval) clearInterval(musicFadeInterval);
		musicFadeInterval = setInterval(() => {
			vol += 0.02;
			if (vol >= target) {
				vol = target;
				if (musicFadeInterval) clearInterval(musicFadeInterval);
			}
			if (musicAudio) musicAudio.volume = vol;
		}, 50);
	}

	function toggleMute() {
		muted = !muted;
		if (musicAudio) {
			musicAudio.volume = muted ? 0 : volume;
		}
	}

	function setVolume(e: Event) {
		const input = e.target as HTMLInputElement;
		volume = parseFloat(input.value);
		if (!muted && musicAudio) {
			musicAudio.volume = volume;
		}
	}

	onMount(() => {
		const hasSeenIntro = localStorage.getItem(STORAGE_KEY) === 'true';

		if (hasSeenIntro) {
			showIntro = false;
			introHidden = true;
			revealMain = true;
			startBonusCatChance();

			const resumeMusic = () => {
				startMusic();
				document.removeEventListener('click', resumeMusic);
				document.removeEventListener('keydown', resumeMusic);
				document.removeEventListener('touchstart', resumeMusic);
			};
			document.addEventListener('click', resumeMusic, { once: true });
			document.addEventListener('keydown', resumeMusic, { once: true });
			document.addEventListener('touchstart', resumeMusic, { once: true });
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

		keyPhase = 'pickup';

		setTimeout(() => {
			keyPhase = 'move';
		}, 600);

		setTimeout(async () => {
			keyPhase = 'turn';
			try {
				await ignitionAudio?.play();
			} catch (e) {
				console.error('Audio failed:', e);
			}
		}, 1600);

		setTimeout(() => {
			flashing = true;
			ignitionAudio?.pause();
		}, 9600);

		setTimeout(() => {
			introHidden = true;
			showIntro = false;
			revealMain = true;
			introFinished = true;
			localStorage.setItem(STORAGE_KEY, 'true');
			startMusic();
			startBonusCatChance();
		}, 10200);

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
		<div class="marquee-banner">
			<div class="marquee-track">
				{#each Array(20) as _}
					<span>THELONGHAUL</span>
				{/each}
			</div>
		</div>

		<div class="audio-controls">
			<button class="mute-btn" onclick={toggleMute} aria-label={muted ? 'Unmute' : 'Mute'}>
				{#if muted}
					<img src="https://icons.hackclub.com/api/icons/black/checkbox" alt="Muted" class="mute-icon" />
				{:else}
					<img src="https://icons.hackclub.com/api/icons/black/checkbox-checked" alt="Unmuted" class="mute-icon" />
				{/if}
			</button>
			<input
				type="range"
				min="0"
				max="1"
				step="0.01"
				value={volume}
				oninput={setVolume}
				class="volume-slider"
				aria-label="Volume"
			/>
		</div>

		<a href="https://hackclub.com" target="_blank" rel="noopener noreferrer" class="orpheus-link">
			<img src="/pics/flag-orpheus-top.png" alt="Hack Club" class="orpheus-flag" />
		</a>

		<div class="hero-section">
			<div class="hero">
				<h1>The Long Haul</h1>
				<p class="tagline">
					Build one giant project across a WHOLE MONTH! No hopping between ideas(maybe), just one long haul.
				</p>
				<p class="rsvp-prompt">Interested? RSVP!</p>
				<a
					href="https://forms.hackclub.com/t/8MSCEBXY2rus"
					target="_blank"
					rel="noopener noreferrer"
					class="rsvp-button"
				>
					RSVP
				</a>
			</div>
			<div class="cat-wrap-hero">
				<img src="/pics/breakdance cat.gif" alt="Dancing cat" class="dancing-cat" />
				<img src="/pics/breakdance cat.gif" alt="Dancing cat" class="dancing-cat" />
				<img src="/pics/breakdance cat.gif" alt="Dancing cat" class="dancing-cat" />
				{#if bonusCat}
					<img
						src="/pics/breakdance cat.gif"
						alt="Bonus cat!"
						class="dancing-cat bonus-cat"
						in:fade={{ duration: 200 }}
						out:fade={{ duration: 300 }}
					/>
				{/if}
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

			<div class="info-grid">
				<div class="info-card">
					<h3>What do I do?</h3>
					<p>
						Make a project! Wanted to make an app that sorts out all of your pictures?
						Want to make an alarm that forces you to solve math problems to turn off?
						ANYTHING GOES! (within reason :p)
					</p>
				</div>
				<div class="info-card">
					<h3>How do?</h3>
					<p>
						That depends! What kind of project do you want to make? It can be hardware,
						software, cad-ware - anything essentially! (as long as it fits the submission
						guidelines of course)
					</p>
				</div>
				<div class="info-card">
					<h3>What counts?</h3>
					<p>TBD! Follow basic Hack Club principals and you should be good.</p>
				</div>
				<div class="info-card">
					<h3> What do I get?</h3>
					<p>Anything in the shop, as long as you are in its tier and have enough fuel!</p>
				</div>
			</div>

			<div class="shop-preview">
				<h3>Shop items coming soon...</h3>
				<div class="carousel-track">
					<div class="carousel-item"> Item 1</div>
					<div class="carousel-item">Item 2</div>
					<div class="carousel-item">Item 3</div>
					<div class="carousel-item">Item 4</div>
					<div class="carousel-item">Item 1</div>
					<div class="carousel-item">Item 2</div>
					<div class="carousel-item">Item 3</div>
					<div class="carousel-item">Item 4</div>
				</div>
			</div>

			<div class="faq-section">
				<div class="faq-box">
					<h2>FAQ</h2>

					<div class="faq-item">
						<h4>What is YSWS?</h4>
						<p>
							A You Ship We Ship (YSWS) is a program where you take time and create some sort
							of project and ship it! Then, we'll ship something to you (prizes, food, etc!)
						</p>
					</div>

					<div class="faq-item">
						<h4>How do I join?</h4>
						<p>
							<a href="https://forms.hackclub.com/t/8MSCEBXY2rus" target="_blank" rel="noopener noreferrer">RSVP here!</a>
						</p>
					</div>

					<div class="faq-item">
						<h4>What is Hack Club?</h4>
						<p>
							Hack Club is a 501(c) non-profit organization aiming to get kids thinking.
							<a href="https://hackclub.com" target="_blank" rel="noopener noreferrer">Check them out here!</a>
						</p>
					</div>

					<div class="faq-item">
						<h4>I need help!</h4>
						<p>
							No problem! Join the
							<a href="https://hackclub.enterprise.slack.com/archives/C0AN2GXJ23F" target="_blank" rel="noopener noreferrer">FAQ channel</a>,
							or DM
							<a href="https://hackclub.slack.com/team/U0828FYS2UC" target="_blank" rel="noopener noreferrer">@Iamalive</a>
							on the Slack!
						</p>
					</div>
				</div>
			</div>
		</div>

		<footer class="site-footer">
			<div class="footer-inner">
				<a href="https://hackclub.com/" class="footer-banner">
					<img
						src="https://assets.hackclub.com/banners/2026.svg"
						alt="Hack Club"
						class="hc-banner"
					/>
				</a>
				<div class="footer-text">
					<p class="footer-initiative">A Hack Club initiative</p>
					<p class="footer-legal">
						The Hack Foundation is a 501(c)(3) non-profit organization d.b.a. Hack Club (EIN: 81-2908499)
					</p>
					<p class="footer-links">
						<a href="https://hackclub.com" target="_blank" rel="noopener noreferrer">About Hack Club</a>
					</p>
					<p class="footer-note">Dino imagery from rawr.hackclub.com!</p>
					<p class="footer-credit">By @Ckacha.</p>
				</div>
			</div>
		</footer>
	</section>
{/if}

<style>
	:global(html, body) {
		margin: 0;
		padding: 0;
		background: #6A9D2A;
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

	/* ── Marquee banner ── */

	.marquee-banner {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		height: 32px;
		background: #1a1a1a;
		overflow: hidden;
		z-index: 40;
		display: flex;
		align-items: center;
	}

	.marquee-track {
		display: flex;
		white-space: nowrap;
		animation: marquee 20s linear infinite;
	}

	.marquee-track span {
		padding: 0 2rem;
		font-size: 0.8rem;
		font-weight: 700;
		letter-spacing: 0.15em;
		color: #e8eaed;
		text-transform: uppercase;
	}

	@keyframes marquee {
		0% {
			transform: translateX(0);
		}
		100% {
			transform: translateX(-50%);
		}
	}

	/* ── Audio controls ── */

	.audio-controls {
		position: fixed;
		top: 38px;
		right: 16px;
		z-index: 40;
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}

	.mute-btn {
		background: rgba(0, 0, 0, 0.5);
		border: 1px solid rgba(255, 255, 255, 0.15);
		border-radius: 8px;
		padding: 0.3rem 0.5rem;
		cursor: pointer;
		font-size: 1rem;
		line-height: 1;
		color: #fff;
		transition: background 0.2s ease;
	}

	.mute-icon {
		width: 20px;
		height: 20px;
		display: block;
		filter: invert(1);
	}

	.mute-btn:hover {
		background: rgba(0, 0, 0, 0.7);
	}

	.volume-slider {
		width: 80px;
		height: 4px;
		appearance: none;
		background: rgba(255, 255, 255, 0.2);
		border-radius: 2px;
		outline: none;
		cursor: pointer;
	}

	.volume-slider::-webkit-slider-thumb {
		appearance: none;
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: #fff;
		cursor: pointer;
	}

	.volume-slider::-moz-range-thumb {
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: #fff;
		border: none;
		cursor: pointer;
	}

	.orpheus-link {
		position: fixed;
		top: 32px;
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
        height: 100vh;
        background: url('/pics/background.png') center / cover no-repeat;
        position: relative;
    }

	.cat-wrap-hero {
		position: absolute;
		top: 50%;
		left: 40%;
		transform: translate(-50%, -50%);
		pointer-events: none;
		display: flex;
		gap: 2rem;
		align-items: center;
	}

	.hero {
		max-width: 600px;
		padding: 31vh 0 0 calc(100vw / 3 * 0.1);
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

	.rsvp-prompt {
		margin: 2rem 0 0.75rem;
		font-size: 1.05rem;
		color: #e8eaed;
		font-weight: 500;
	}

	.rsvp-button {
		display: inline-block;
		padding: 1rem 3.5rem;
		background: #6A9D2A;
		color: #fff;
		font-size: 1.15rem;
		font-weight: 600;
		text-decoration: none;
		border-radius: 12px;
		border: 2px solid rgba(255, 255, 255, 0.4);
		letter-spacing: 0.04em;
		transition:
			background 0.2s ease,
			transform 0.15s ease,
			border-color 0.2s ease;
	}

	.rsvp-button:hover {
		background: #5a8a22;
		border-color: rgba(255, 255, 255, 0.7);
		transform: scale(1.03);
	}

	.content-section {
		background: linear-gradient(180deg, #496900 0%, #6A9D2A 50%, #8492a6 100%);
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

	/* ── Info grid ── */

	.info-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 1.25rem;
		max-width: 760px;
		margin-top: 3rem;
	}

	.info-card {
		padding: 1.25rem 1.35rem;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 14px;
	}

	.info-card h3 {
		margin: 0 0 0.5rem;
		font-size: 1rem;
	}

	.info-card p {
		margin: 0;
		color: #e8eaed;
		line-height: 1.6;
		font-size: 0.92rem;
	}

	/* ── Shop carousel ── */

	.shop-preview {
		max-width: 760px;
		margin-top: 3rem;
		overflow: hidden;
	}

	.shop-preview h3 {
		margin: 0 0 1rem;
		font-size: 1rem;
		color: #e8eaed;
	}

	.carousel-track {
		display: flex;
		gap: 1rem;
		animation: scroll 12s linear infinite;
	}

	.carousel-item {
		flex-shrink: 0;
		padding: 1rem 1.5rem;
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.15);
		border-radius: 12px;
		font-size: 0.95rem;
		white-space: nowrap;
	}

	@keyframes scroll {
		0% {
			transform: translateX(0);
		}
		100% {
			transform: translateX(-50%);
		}
	}

	/* ── Dancing cat ── */

	.dancing-cat {
		width: auto;
		height: auto;
		image-rendering: pixelated;
		transform: scale(2);
	}

	.bonus-cat {
		transform: scale(1);
		position: absolute;
		bottom: -20px;
		right: -40px;
	}

	/* ── FAQ ── */

	.faq-section {
		max-width: 760px;
		margin-top: 3rem;
	}

	.faq-box {
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 16px;
		padding: 2rem;
	}

	.faq-box h2 {
		margin: 0 0 1.25rem;
		font-size: 1.4rem;
	}

	.faq-item {
		padding: 1rem 0;
		border-top: 1px solid rgba(255, 255, 255, 0.1);
	}

	.faq-item:first-of-type {
		border-top: none;
		padding-top: 0;
	}

	.faq-item h4 {
		margin: 0 0 0.4rem;
		font-size: 0.95rem;
		font-weight: 600;
	}

	.faq-item p {
		margin: 0;
		color: #e0e4e8;
		line-height: 1.6;
		font-size: 0.9rem;
	}

	.faq-item a {
		color: #c5e89e;
		text-decoration: underline;
		text-underline-offset: 3px;
	}

	.faq-item a:hover {
		color: #fff;
	}

	/* ── Footer ── */

	.site-footer {
		background: #8492a6;
		padding: 2.5rem 2rem;
		color: #fff;
	}

	.footer-inner {
		width: min(1100px, 100%);
		margin: 0 auto;
		display: flex;
		align-items: flex-start;
		gap: 2rem;
	}

	.footer-banner {
		flex-shrink: 0;
		line-height: 0;
	}

	.hc-banner {
		width: 180px;
		height: auto;
	}

	.footer-text {
		text-align: center;
		flex: 1;
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

		.info-grid {
			grid-template-columns: 1fr;
		}

		.dancing-cat {
			transform: scale(2);
			margin: 2.5rem 0;
		}

		.footer-inner {
			flex-direction: column;
			align-items: center;
			text-align: center;
		}

		.hc-banner {
			width: 140px;
		}
	}
</style>
