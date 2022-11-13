<script lang="ts">
  import { onMount } from 'svelte';
  import { Cell, Stream, StreamSink } from 'sodiumjs';

  onMount(() => {
    const sMouse = new StreamSink<MouseEvent>();

    const sDrag = sMouse.filter(event => event.type === 'mousedown').map(mousedownEvent =>
        sMouse.filter(event => event.type === 'mousemove').map(mousemoveEvent => {
          mousedownEvent.target.style.left = mousemoveEvent.clientX + 'px';
          mousedownEvent.target.style.top = mousemoveEvent.clientY + 'px';
        })
    );

    const sIdle = new Stream<void>();
    const sEndDrag = sMouse.filter(event => event.type === 'mouseup').map(() => sIdle);

    const sDocUpdate = Cell.switchS(sDrag.orElse(sEndDrag).hold(sIdle));

    sDocUpdate.listen(() => {
    });

    document.onmousedown = event => {
      sMouse.send(event);
    };
    document.onmousemove = event => {
      sMouse.send(event);
    };
    document.onmouseup = event => {
      sMouse.send(event);
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
