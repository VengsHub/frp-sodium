<script lang="ts">
  import { onMount } from 'svelte';
  import { Cell, Stream, StreamSink } from 'sodiumjs';

  onMount(() => {
    const sMouseDown = new StreamSink<MouseEvent>();
    const sMouseMove = new StreamSink<MouseEvent>();
    const sMouseUp = new StreamSink<MouseEvent>();

    // cell is needed for snapshot, even though we do not need one for this use case
    const cDoc = new Cell(document);

    const sDrag = sMouseDown.snapshot(cDoc, (mousedownEvent) =>
        sMouseMove.snapshot(cDoc, mousemoveEvent => {
          mousedownEvent.target.style.left = mousemoveEvent.clientX + 'px';
          mousedownEvent.target.style.top = mousemoveEvent.clientY + 'px';
        })
    );

    const sIdle = new Stream<void>();
    const sEndDrag = sMouseUp.map(() => sIdle);

    const sDocUpdate = Cell.switchS(sDrag.orElse(sEndDrag).hold(sIdle));

    sDocUpdate.listen(() => {});

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
    <div class="option">Drag me</div>
    <div class="option">Drag me</div>
    <div class="option">Drag me</div>
    <div class="option">Drag me</div>
</section>

<style lang="scss">
  .option {
    width: 40px;
    height: 40px;
    padding: 12px;
    border: 2px solid black;
    border-radius: 5px;
    font-size: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    user-select: none;
    position: absolute;

    &:hover {
      background-color: lightblue;
    }

    @for $i from 1 through 9 {
      &:nth-of-type(#{$i}) {
        left: 10 * $i + vw;
        top: 10 * $i + vh;
      }
    }
  }
</style>
