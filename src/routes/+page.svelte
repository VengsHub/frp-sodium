<script lang="ts">
  import { onMount } from 'svelte';

  import Counter from './Counter.svelte';
  import welcome from '$lib/images/svelte-welcome.webp';
  import welcome_fallback from '$lib/images/svelte-welcome.png';
  import { Cell, Stream, StreamSink } from 'sodiumjs';

  let elementToDrag: HTMLElement;

  onMount(() => {
    const sMouseDown = new StreamSink<MouseEvent>();
    const sMouseUp = new StreamSink<MouseEvent>();
    const sMouseMove = new StreamSink<MouseEvent>();

    const cDraggingElement = new Cell(elementToDrag);

    const sIdle = new Stream<void>();
    const sEndDrag = sMouseUp.map(() => sIdle);

    // map and snapshot count as a listen
    // snapshot doesnt happen until mousedown -> snapshot === listen -> no listen until mousedown?
    const sDrag = sMouseDown.snapshot(cDraggingElement, (event1, element) =>
        sMouseMove.snapshot(cDraggingElement, (event2, element) => {
          console.log('moving');
        })
    );

    const sDocUpdate = sDrag.orElse(sEndDrag).hold(sIdle);

    sDocUpdate.listen((event) => {
      // console.log('Received Drag:');
      // console.log(event);
      // event?.target.translate(event.x, event.y); Dit geht nicht.
    });

    document.onmousedown = event => {
      sMouseDown.send(event);
    };
    document.onmousemove = event => {
      sMouseMove.send(event);
    };
    document.onmouseup = event => {
      sMouseUp.send(event);
    };
  });
</script>

<svelte:head>
    <title>Home</title>
    <meta name="description" content="Svelte demo app"/>
</svelte:head>

<section>
    <h1>
		<span class="welcome">
			<picture>
				<source srcset={welcome} type="image/webp"/>
				<img src={welcome_fallback} alt="Welcome" bind:this={elementToDrag}/>
			</picture>
		</span>

        to your new<br/>SvelteKit app
    </h1>

    <h2>
        try editing <strong>src/routes/+page.svelte</strong>
    </h2>

    <Counter/>
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
