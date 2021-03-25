<script>
	import { onMount } from 'svelte';
	import { select } from './select';
	import { loadImage } from './utils';
	import Welcome from './screens/Welcome.svelte';
	import Game from './screens/Game.svelte';

	let state = 'welcome';
	let celebsPromise;
	let selection;

	const start = async(event) => {
		const { celebs, lookup } = await celebsPromise;

		selection = select(celebs, lookup, event.detail.category);
		state = 'playing';
	};

	const loadCelebs = async () => {
		const res = await fetch('https://cameo-explorer.netlify.app/celebs.json');
		const data = await res.json();

		const lookup = new Map();
		data.forEach(celeb => {
			lookup.set(celeb.id, celeb);
		});
		
		const subset = new Set();
		data.forEach(celeb => {
			if (celeb.reviews >= 50) {
				subset.add(celeb);
				celeb.similar.forEach(id => {
					subset.add(lookup.get(id));
				});
			}
		});

		return {
			celebs: Array.from(subset),
			lookup
		}
	};

	onMount(() => {
		celebsPromise = loadCelebs();

		loadImage('/icons/right.svg');
		loadImage('/icons/wrong.svg');
	});
</script>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 800px;
		margin: 0 auto;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>

<main>
	{#if state === 'welcome'}
		<Welcome on:select={start} />
	{:else if state === 'playing'}
		<Game rounds={selection} on:restart={() => state = 'welcome'} />
	{/if}
</main>
