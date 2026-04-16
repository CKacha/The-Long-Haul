<script lang="ts">
	import { goto } from '$app/navigation';

	const faqs = [
		{
			q: 'What is The Long Haul?',
			a: 'The Long Haul is a month-long You Ship We Ship (YSWS) where you build ONE project over the entire duration and submit it at the end.'
		},
		{
			q: 'What is YSWS?',
			a: 'A You Ship We Ship (YSWS) is a program where you build something and ship it. In return, you can earn rewards like stickers, hardware, and more.'
		},
		{
			q: 'Who can participate?',
			a: 'Anyone ages 13–18 can participate in The Long Haul.'
		},
		{
			q: 'What kind of projects are allowed?',
			a: 'Anything! Software, hardware, CAD, creative projects — as long as you can track your time and follow guidelines.'
		},
		{
			q: 'How do I track my work?',
			a: 'You can use tools like Lapse or Hackatime to verify your time spent working on your project.'
		}, 
		{
			q: 'How does the shop work?',
			a: 'The more time and quality you put into your project, the more rewards and tiers you unlock in the shop.'
		},
		{
			q: 'Is this legit?',
			a: 'Yes. Hack Club is a 501(c)(3) nonprofit that has run many successful programs like Summer of Making and High Seas.'
		},
		{
			q: 'I need help!',
			a: 'Join the Hack Club Slack or ask in the FAQ channel — someone will help you out quickly.'
		}
	];

	function navigate(path: string) {
		goto(path);
	}
</script>

<svelte:head>
	<title>FAQ | The Long Haul</title>
</svelte:head>

<section class="faq-page">
    <div class="grain"></div>

	<header class="faq-header">
		<div>
			<h1>FAQ</h1>
			<p class="subtext">Everything you need to know before starting your journey.</p>
		</div> 

		<div class="nav-buttons">
			<button on:click={() => navigate('/')}>Home</button>
			<button on:click={() => navigate('/shop')}>Shop</button>
			<button on:click={() => navigate('/projects')}>Projects</button>
		</div>
	</header>

	<section class="faq-container">
		{#each faqs as item}
			<div class="faq-item">
				<h3>{item.q}</h3>
				<p>{item.a}</p>
			</div>
		{/each}
	</section>
</section>

<style>
	.faq-page {
		min-height: 100vh;
		padding: 4rem 2rem;
		background: linear-gradient(180deg, #496900 0%, #6A9D2A 50%, #8492a6 100%);
		color: white;
	}

	.faq-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-end;
		gap: 2rem;
		flex-wrap: wrap;
		margin-bottom: 2.5rem;
	}

	.faq-header h1 {
		margin: 0;
		font-size: clamp(2.5rem, 5vw, 4rem);
	}

	.subtext {
		margin-top: 0.5rem;
		opacity: 0.85;
	}

	.nav-buttons {
		display: flex;
		gap: 0.75rem;
		flex-wrap: wrap;
	}

	.nav-buttons button {
		padding: 0.7rem 1rem;
		border-radius: 12px;
		border: 1px solid rgba(255, 255, 255, 0.2);
		background: rgba(255, 255, 255, 0.1);
		color: white;
		cursor: pointer;
		font-weight: 600;
		transition: transform 0.2s ease;
	}

	.nav-buttons button:hover {
		transform: translateY(-2px);
	}

	.faq-container {
		max-width: 800px;
		margin: 0 auto;
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}

	.faq-item {
		padding: 1.25rem;
		border-radius: 16px;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.12);
	}

	.faq-item h3 {
		margin: 0 0 0.4rem;
		font-size: 1.1rem;
	}

	.faq-item p {
		margin: 0;
		opacity: 0.9;
		line-height: 1.6;
	}
</style>