<script lang="ts">
	import { onMount } from 'svelte';

	import Counter from './Counter.svelte';
	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';
	import { Cell, CellLoop, Stream, StreamSink, Transaction, Unit } from 'sodiumjs';

	onMount(() => {
		const sMouseDown = new StreamSink();
		const sMouseMove = new StreamSink();
		const sMouseUp = new StreamSink();

		// error happens from the above 6 lines already

		Transaction.run(() => {
			const doc = new CellLoop();
			const sStartDrag = sMouseDown.snapshot(doc, event => {
				const element = event.target;
				return sMouseMove.snapshot(doc, moveEvent => {
					console.log('mousemove', moveEvent);
					element.translate(moveEvent.x, moveEvent.y);
				});
			});

			const sIdle = new Stream<void>();
			const sEndDrag = sMouseUp.map(() => sIdle);
			const sDocUpdate = sStartDrag.orElse(sEndDrag).hold(sIdle);
			sDocUpdate.listen(() => console.log('document updated'));
		});

		document.onmousedown = event => {
			sMouseDown.send(event);
		}
		document.onmousemove = event => {
			// TODO doesn't work atm as it fires before listeners are set? -> sMouseMove.send(event);
		}
		document.onmouseup = event => {
			sMouseUp.send(event);
		}
		// did not work -> document.onmousedown = sMouseDown.send;
		// also did not work as it triggers send() on declaration -> document.onmousemove = event => sMouseMove.send(event);
	});
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
	<h1>
		<span class="welcome">
			<picture>
				<source srcset={welcome} type="image/webp" />
				<img src={welcome_fallback} alt="Welcome" />
			</picture>
		</span>

		to your new<br />SvelteKit app
	</h1>

	<h2>
		try editing <strong>src/routes/+page.svelte</strong>
	</h2>

	<Counter />
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.welcome {
		display: block;
		position: relative;
		width: 100%;
		height: 0;
		padding: 0 0 calc(100% * 495 / 2048) 0;
	}

	.welcome img {
		position: absolute;
		width: 100%;
		height: 100%;
		top: 0;
		display: block;
	}
</style>
