<script lang="ts">
	import { goto } from '$app/navigation';

	const stats = [
		{ label: 'Guests Confirmed', value: '128' },
		{ label: 'Pending RSVPs', value: '34' },
		{ label: 'Tables Planned', value: '16' },
		{ label: 'Days Remaining', value: '12' }
	];

	const activity = [
		{ title: 'New RSVP received', detail: 'Sophia Chen confirmed attendance' },
		{ title: 'Venue update', detail: 'Main hall seating finalized' },
		{ title: 'Reminder sent', detail: 'Follow-up email sent to pending guests' }
	];

	const projects = [
		{
			title: 'Spring Launch',
			description: 'Main event setup, guest list tracking, and timeline management.',
			status: 'Active'
		},
		{
			title: 'VIP Guest Flow',
			description: 'Special seating, entry flow, and RSVP follow-up for VIP attendees.',
			status: 'In Review'
		},
		{
			title: 'Merch Table Setup',
			description: 'Planning product layout, stock count, and booth presentation.',
			status: 'Planned'
		}
	];

	function navigate(path: string) {
		goto(path);
	}
</script>

<svelte:head>
	<title>Dashboard | The Long Haul</title>
</svelte:head>

<section class="page-shell">
	<div class="page-overlay"></div>

	<header class="dash-header">
		<div>
			<p class="eyebrow">The Long Haul</p>
			<h1 class="page-title">Dashboard</h1>
			<p class="subtext">A simple overview page for your event, guests, projects, and recent activity.</p>
		</div>

		<div class="hero-actions">
			<button class="primary-btn">Create Invite</button>
			<button class="secondary-btn" on:click={() => navigate('/shop')}>Go to Shop</button>
		</div>
	</header>

	<section class="card section-nav">
		<div class="section-nav-head">
			<div>
				<p class="section-kicker">Sections</p>
				<h2>Jump to</h2>
			</div>

			<div class="section-buttons">
				<button class="nav-btn" on:click={() => navigate('/shop')}>Shop</button>
				<button class="nav-btn" on:click={() => navigate('/faq')}>FAQ</button>
				<button class="nav-btn" on:click={() => navigate('/projects')}>Projects</button>
			</div>
		</div>
	</section>

	<section class="stats-grid">
		{#each stats as stat}
			<div class="card stat-card">
				<p class="stat-label">{stat.label}</p>
				<h2>{stat.value}</h2>
			</div>
		{/each}
	</section>

	<section class="content-grid">
		<div class="card panel">
			<div class="panel-head">
				<h3>Recent Activity</h3>
				<span>Live updates</span>
			</div>

			<div class="activity-list">
				{#each activity as item}
					<div class="activity-item">
						<h4>{item.title}</h4>
						<p>{item.detail}</p>
					</div>
				{/each}
			</div>
		</div>

		<div class="card panel">
			<div class="panel-head">
				<h3>Quick Actions</h3>
				<span>Shortcuts</span>
			</div>

			<div class="quick-actions">
				<button class="action-tile">Manage Guests</button>
				<button class="action-tile">Edit Schedule</button>
				<button class="action-tile" on:click={() => navigate('/shop')}>Open Shop</button>
				<button class="action-tile" on:click={() => navigate('/projects')}>View Projects</button>
			</div>
		</div>
	</section>

	<section class="card panel projects-panel">
		<div class="panel-head">
			<h3>My Projects</h3>
			<span>Your current work</span>
		</div>

		<div class="projects-list">
			{#each projects as project}
				<div class="project-card">
					<div class="project-top">
						<h4>{project.title}</h4>
						<span class="project-status">{project.status}</span>
					</div>
					<p>{project.description}</p>
				</div>
			{/each}
		</div>

		<div class="projects-footer">
			<button class="secondary-btn" on:click={() => navigate('/projects')}>Go to Projects</button>
		</div>
	</section>
</section>

<style>
	.dash-header,
	.section-nav,
	.stats-grid,
	.content-grid,
	.projects-panel {
		position: relative;
		z-index: 1;
	}

	.dash-header {
		display: flex;
		justify-content: space-between;
		align-items: end;
		gap: 2rem;
		flex-wrap: wrap;
		margin-bottom: 1.5rem;
	}

	.hero-actions {
		display: flex;
		gap: 0.75rem;
		flex-wrap: wrap;
	}

	.section-nav {
		padding: 1.25rem;
		margin-bottom: 1.5rem;
	}

	.section-nav-head {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		flex-wrap: wrap;
	}

	.section-kicker {
		margin: 0 0 0.35rem;
		font-size: 0.8rem;
		opacity: 0.72;
		text-transform: uppercase;
		letter-spacing: 0.08em;
	}

	.section-nav h2 {
		margin: 0;
		font-size: 1.35rem;
	}

	.section-buttons {
		display: flex;
		flex-wrap: wrap;
		gap: 0.75rem;
	}

	.nav-btn,
	.action-tile {
		border: none;
		cursor: pointer;
		font-weight: 600;
		transition:
			transform 0.2s ease,
			opacity 0.2s ease;
	}

	.nav-btn:hover,
	.action-tile:hover {
		transform: translateY(-2px);
	}

	.nav-btn {
		padding: 0.9rem 1.1rem;
		border-radius: 14px;
		background: rgba(255, 255, 255, 0.12);
		color: white;
		border: 1px solid rgba(255, 255, 255, 0.14);
	}

	.stats-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
		gap: 1rem;
		margin-bottom: 1.5rem;
	}

	.content-grid {
		display: grid;
		grid-template-columns: 1.4fr 1fr;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.stat-card {
		padding: 1.25rem;
	}

	.stat-label {
		margin: 0 0 0.4rem;
		font-size: 0.9rem;
		opacity: 0.8;
	}

	.stat-card h2 {
		margin: 0;
		font-size: 2rem;
	}

	.panel {
		padding: 1.25rem;
	}

	.panel-head {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		margin-bottom: 1rem;
	}

	.panel-head h3 {
		margin: 0;
		font-size: 1.1rem;
	}

	.panel-head span {
		font-size: 0.85rem;
		opacity: 0.7;
	}

	.activity-list,
	.projects-list {
		display: flex;
		flex-direction: column;
		gap: 0.9rem;
	}

	.activity-item,
	.project-card {
		padding: 1rem;
		border-radius: 18px;
		background: rgba(0, 0, 0, 0.14);
	}

	.activity-item h4,
	.project-card h4 {
		margin: 0 0 0.35rem;
		font-size: 1rem;
	}

	.activity-item p,
	.project-card p {
		margin: 0;
		opacity: 0.82;
		font-size: 0.95rem;
	}

	.quick-actions {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 0.75rem;
	}

	.action-tile {
		padding: 1rem;
		border-radius: 18px;
		background: rgba(255, 255, 255, 0.12);
		color: white;
		border: 1px solid rgba(255, 255, 255, 0.14);
		text-align: left;
	}

	.project-top {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 1rem;
		margin-bottom: 0.45rem;
	}

	.project-status {
		font-size: 0.8rem;
		padding: 0.35rem 0.65rem;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.1);
		border: 1px solid rgba(255, 255, 255, 0.14);
		opacity: 0.9;
		white-space: nowrap;
	}

	.projects-footer {
		margin-top: 1rem;
		display: flex;
		justify-content: flex-end;
	}

	@media (max-width: 800px) {
		.content-grid {
			grid-template-columns: 1fr;
		}

		.quick-actions {
			grid-template-columns: 1fr;
		}

		.project-top,
		.section-nav-head {
			align-items: flex-start;
			flex-direction: column;
		}

		.projects-footer {
			justify-content: stretch;
		}
	}
</style>