<style>
  div {
    width: 100%;
    height: 1rem;
    background-color: red;
    opacity: 0;
    margin-bottom: -0.5rem;
    margin-top: -0.5rem;
    cursor: row-resize;
  }
</style>

<script>
  import { terminalHeight, gameHeight } from "../store.js";
  let viewHeight = 0;
  let dragging = false;

  function handleMousemove(event) {
    viewHeight = Math.round((event.clientY / window.innerHeight) * 100);
    if (dragging && viewHeight > 0 && viewHeight < 85) {
      $gameHeight = viewHeight;
      $terminalHeight = 95 - $gameHeight;
    }
  }

  function dragHandle(event) {
    dragging = true;
  }
  function releaseHandle(event) {
    dragging = false;
  }
</script>

<svelte:window 
  on:mousemove={handleMousemove}
  on:mouseup={releaseHandle}
  />
<div 
  on:mousedown={dragHandle}
  >
</div>