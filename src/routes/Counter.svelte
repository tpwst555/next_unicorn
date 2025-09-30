<script lang="ts">
	import { Spring } from 'svelte/motion';

	const count = new Spring(0);
	const offset = $derived(modulo(count.current, 1));

	function modulo(n: number, m: number) {
		// handle negative numbers
		return ((n % m) + m) % m;
	}

	// Generate buzzwords for extra credibility
  const metrics: readonly string[] = ['Active Users', 'ARR Multiple', 'Burn Rate', 'TAM Size', 'Unicorns Generated'] as const;
	let currentMetric = $state(0);
	
	function nextMetric() {
		currentMetric = (currentMetric + 1) % metrics.length;
	}
</script>

<div class="valuation-machine">
	<h3 class="metric-label">
		<span class="label-prefix">Totally Real</span>
		<button class="metric-switcher" onclick={nextMetric}>
			{metrics[currentMetric]}
		</button>
	</h3>
	
	<div class="counter">
		<button onclick={() => (count.target -= 1)} aria-label="Decrease valuation (why would you?)">
			<svg aria-hidden="true" viewBox="0 0 1 1">
				<path d="M0,0.5 L1,0.5" />
			</svg>
		</button>

		<div class="counter-viewport">
			<div class="counter-digits" style="transform: translate(0, {100 * offset}%)">
				<strong class="hidden" aria-hidden="true">{Math.floor(count.current + 1)}M</strong>
				<strong>{Math.floor(count.current)}M</strong>
			</div>
		</div>

		<button onclick={() => (count.target += 1)} aria-label="TO THE MOON 🚀">
			<svg aria-hidden="true" viewBox="0 0 1 1">
				<path d="M0,0.5 L1,0.5 M0.5,0 L0.5,1" />
			</svg>
		</button>
	</div>
	
	<p class="disclaimer">*Numbers guaranteed to impress VCs</p>
</div>

<style>
	.valuation-machine {
		margin: 2rem auto;
		max-width: 400px;
		text-align: center;
	}

	.metric-label {
		margin: 0 0 1rem;
		font-size: 0.9rem;
		text-transform: uppercase;
		letter-spacing: 0.1em;
		color: #666;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
	}

	.label-prefix {
		opacity: 0.7;
		font-weight: 400;
	}

	.metric-switcher {
		background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
		color: white;
		border: none;
		padding: 0.25rem 0.75rem;
		border-radius: 20px;
		font-size: 0.8rem;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.2s;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.metric-switcher:hover {
		transform: scale(1.05);
		box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
	}

	.counter {
		display: flex;
		align-items: center;
		justify-content: center;
		background: linear-gradient(135deg, rgba(102, 126, 234, 0.1) 0%, rgba(118, 75, 162, 0.1) 100%);
		border-radius: 16px;
		padding: 1rem;
		box-shadow: 0 4px 20px rgba(102, 126, 234, 0.15);
		border: 2px solid transparent;
		background-clip: padding-box;
		position: relative;
	}

	.counter::before {
		content: '';
		position: absolute;
		inset: -2px;
		border-radius: 16px;
		padding: 2px;
		background: linear-gradient(90deg, #667eea, #764ba2);
		mask: linear-gradient(#fff 0 0) content-box, linear-gradient(#fff 0 0);
		mask-composite: exclude;
		opacity: 0.5;
	}

	.counter button {
		width: 3em;
		height: 3em;
		padding: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		border: none;
		background: white;
		border-radius: 50%;
		touch-action: manipulation;
		font-size: 1.5rem;
		cursor: pointer;
		transition: all 0.2s;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
	}

	.counter button:hover {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		transform: scale(1.1);
		box-shadow: 0 4px 20px rgba(102, 126, 234, 0.4);
	}

	.counter button:hover path {
		stroke: white;
	}

	.counter button:active {
		transform: scale(0.95);
	}

	svg {
		width: 30%;
		height: 30%;
	}

	path {
		vector-effect: non-scaling-stroke;
		stroke-width: 3px;
		stroke: #667eea;
		stroke-linecap: round;
		transition: stroke 0.2s;
	}

	.counter-viewport {
		width: 8em;
		height: 4em;
		overflow: hidden;
		text-align: center;
		position: relative;
		margin: 0 1rem;
	}

	.counter-viewport strong {
		position: absolute;
		display: flex;
		width: 100%;
		height: 100%;
		font-weight: 900;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
		font-size: 3.5rem;
		align-items: center;
		justify-content: center;
		font-family: 'SF Pro Display', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
	}

	.counter-digits {
		position: absolute;
		width: 100%;
		height: 100%;
	}

	.hidden {
		top: -100%;
		user-select: none;
	}

	.disclaimer {
		margin-top: 1rem;
		font-size: 0.75rem;
		color: #999;
		font-style: italic;
		opacity: 0.7;
	}

	/* Add some ✨ magic ✨ */
	@keyframes shimmer {
		0% { background-position: -200% center; }
		100% { background-position: 200% center; }
	}

	.counter:hover::before {
		animation: shimmer 2s linear infinite;
		background: linear-gradient(90deg, 
			transparent, 
			rgba(255, 255, 255, 0.3), 
			transparent,
			#667eea,
			#764ba2,
			#667eea
		);
		background-size: 200% 100%;
	}

	/* Mobile friendly */
	@media (max-width: 640px) {
		.counter-viewport strong {
			font-size: 3rem;
		}
		
		.counter button {
			width: 2.5em;
			height: 2.5em;
		}
	}
</style>
