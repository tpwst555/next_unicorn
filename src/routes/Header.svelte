<script lang="ts">
	import { page } from '$app/state';
</script>

<header>
	<div class="corner">
		<a href="/" title="To the moon 🚀">
			<span class="logo">🦄</span>
		</a>
	</div>

	<nav>
		<svg viewBox="0 0 2 3" aria-hidden="true">
			<path d="M0,0 L1,2 C1.5,3 1.5,3 2,3 L2,0 Z" />
		</svg>
		<ul>
			<li aria-current={page.url.pathname === '/' ? 'page' : undefined}>
				<a href="/">Disrupt</a>
			</li>
			<li aria-current={page.url.pathname === '/pitch' ? 'page' : undefined}>
				<a href="/pitch">Pitch Deck</a>
			</li>
			<li aria-current={page.url.pathname === '/investors' ? 'page' : undefined}>
				<a href="/investors">Investors</a>
			</li>
			<li aria-current={page.url.pathname === '/exit' ? 'page' : undefined}>
				<a href="/exit">Exit Strategy</a>
			</li>
		</ul>
		<svg viewBox="0 0 2 3" aria-hidden="true">
			<path d="M0,0 L0,3 C0.5,3 0.5,3 1,2 L2,0 Z" />
		</svg>
	</nav>

	<div class="corner">
		<a href="https://github.com/yourusername/next_unicorn" title="Star us (we need the validation)">
			<span class="github-cta">⭐</span>
		</a>
	</div>
</header>

<style>
	header {
		display: flex;
		justify-content: space-between;
		background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
		box-shadow: 0 2px 10px rgba(102, 126, 234, 0.3);
	}

	.corner {
		width: 3em;
		height: 3em;
	}

	.corner a {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100%;
		transition: transform 0.2s;
	}

	.corner a:hover {
		transform: scale(1.2);
	}

	.logo {
		font-size: 2em;
		animation: float 3s ease-in-out infinite;
	}

	.github-cta {
		font-size: 1.5em;
		filter: grayscale(100%);
		transition: filter 0.3s;
	}

	.corner a:hover .github-cta {
		filter: grayscale(0%);
		animation: pulse 1s infinite;
	}

	@keyframes float {
		0%, 100% { transform: translateY(0px); }
		50% { transform: translateY(-5px); }
	}

	@keyframes pulse {
		0%, 100% { transform: scale(1); }
		50% { transform: scale(1.3); }
	}

	nav {
		display: flex;
		justify-content: center;
		--background: rgba(255, 255, 255, 0.95);
	}

	svg {
		width: 2em;
		height: 3em;
		display: block;
	}

	path {
		fill: var(--background);
	}

	ul {
		position: relative;
		padding: 0;
		margin: 0;
		height: 3em;
		display: flex;
		justify-content: center;
		align-items: center;
		list-style: none;
		background: var(--background);
		background-size: contain;
	}

	li {
		position: relative;
		height: 100%;
	}

	li[aria-current='page']::before {
		--size: 6px;
		content: '';
		width: 0;
		height: 0;
		position: absolute;
		top: 0;
		left: calc(50% - var(--size));
		border: var(--size) solid transparent;
		border-top: var(--size) solid #667eea;
	}

	nav a {
		display: flex;
		height: 100%;
		align-items: center;
		padding: 0 1rem;
		color: #333;
		font-weight: 700;
		font-size: 0.85rem;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		text-decoration: none;
		transition: all 0.2s;
	}

	nav a:hover {
		color: #667eea;
		transform: translateY(-1px);
	}

	/* Add some startup energy */
	@media (prefers-reduced-motion: no-preference) {
		nav a::after {
			content: '™';
			font-size: 0.5em;
			vertical-align: super;
			opacity: 0;
			transition: opacity 0.2s;
			margin-left: 1px;
		}

		nav a:hover::after {
			opacity: 0.5;
		}
	}
</style>
