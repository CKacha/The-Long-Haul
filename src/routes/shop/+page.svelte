<script lang="ts">
	import { goto } from '$app/navigation';

	type ShopItem = { name: string; hours: string; image: string };

	const tiers: { name: string;  blurb: string; items: ShopItem[] }[] = [
		{
			name: 'Level 1',
            blurb: 'Start supplies for the ',
			items: [
				{ name: 'Hot Choco', hours: '~2 hrs', image: '/pics/hotchoco.webp' },
				{ name: 'Chrome License', hours: '~2 hrs', image: '/pics/chrome.webp' },
				{ name: 'AI Credits', hours: '~3 hrs', image: '/pics/ai.webp' },
				{ name: 'HC Pin', hours: '~4 hrs', image: '/pics/hcpin.webp' },
				{ name: 'Nebula', hours: '~4 hrs', image: '/pics/nebula.webp' },
				{ name: 'Pico 9', hours: '~5 hrs', image: '/pics/pico9.webp' },
				{ name: 'Small Blåhaj', hours: '~8 hrs', image: '/pics/smallblahaj.webp' },
				{ name: 'Ugee Tablet', hours: '~10 hrs', image: '/pics/ugee drawing tablet.webp' }
			]
		}, 
		{
			name: 'Level 2',
            blurb: 'Good gear for the longer nights & heavier miles',
			items: [
				{ name: 'Pinecil', hours: '~7 hrs', image: '/pics/pinecil.webp' },
				{ name: 'Big Blåhaj', hours: '~8 hrs', image: '/pics/bigblahaj.webp' },
				{ name: 'YubiKey', hours: '~15 hrs', image: '/pics/yubikey.webp' },
				{ name: 'Flipper Zero', hours: '~50 hrs', image: '/pics/flipperzero.webp' },
				{ name: 'A1 Mini', hours: '~50 hrs', image: '/pics/a1mini.webp' },
				{ name: 'MacBook Neo', hours: '~125 hrs', image: '/pics/macbookneo.webp' },
				{ name: 'iPad Air', hours: '~150 hrs', image: '/pics/ipadair.webp' },
				{ name: 'Framework 12', hours: '~150 hrs', image: '/pics/framework12.webp' }
			]
		},
		{
			name: 'Level 3',
            blurb: 'Serious Cargo for the real long-haulers!',
			items: [
				{ name: 'Nothing Headphones', hours: '~50 hrs', image: '/pics/nthingheaphones.webp' },
				{ name: 'A1', hours: '~60 hrs', image: '/pics/a1.webp' },
				{ name: 'Framework 13', hours: '~240 hrs', image: '/pics/framework13.webp' },
				{ name: 'Framework 16', hours: '~400 hrs', image: '/pics/framework16.webp' }
			]
		}
	];

	function goToDashboard() {
		goto('/dashboard');
	}
</script>

<svelte:head>
	<title>Shop | The Long Haul</title>
</svelte:head>

<section class="page-shell">
	<div class="page-overlay"></div>

	<header class="topbar">
		<div>
			<p class="eyebrow">The Long Haul</p>
			<h1 class="page-title">Camper Shop</h1>
			<p class="subtext">Spend your fuel on prizes! Unlock tiers by logging more hours.</p>
		</div>

		<button class="secondary-btn" onclick={() => goToDashboard()}>Back to Dashboard</button>
	</header>

	<div class="tiers">
		{#each tiers as tier}
			<section class="tier-section">
				<h2 class="tier-title">{tier.name}</h2>
				<div class="shop-grid">
					{#each tier.items as item}
						<div class="card shop-card">
							<img src={item.image} alt={item.name} />
							<span class="item-name">{item.name}</span>
							<span class="item-hours">{item.hours}</span>
						</div>
					{/each}
				</div>
			</section>
		{/each}
	</div>
</section>

<style>
	.tiers {
		position: relative;
		z-index: 1;
	}

	.tier-section {
		margin-bottom: 2.5rem;
	}

	.tier-title {
		font-size: 1.4rem;
		margin: 0 0 1rem;
		letter-spacing: 0.02em;
	}

	.shop-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
		gap: 1rem;
	}

	.shop-card {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.4rem;
		padding: 1rem;
	}

	.shop-card img {
		width: 90px;
		height: 90px;
		object-fit: contain;
		border-radius: 8px;
	}

	.item-name {
		font-size: 0.85rem;
		font-weight: 600;
		text-align: center;
	}

	.item-hours {
		font-size: 0.75rem;
		color: rgba(255, 255, 255, 0.6);
	}

	@media (max-width: 700px) {
		.shop-grid {
			grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
		}

		.shop-card img {
			width: 70px;
			height: 70px;
		}
	}
</style>
