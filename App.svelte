<script>
  import Terminal from "./components/Terminal.svelte";
  import GameView from "./components/GameView.svelte";
  import Resize from "./components/Resize.svelte";
</script>

<style>
  :root {
    --bg-body: #4a5457;
  }
  main {
    background-color: var(--bg-body);
    font-family: sans-serif;
    text-align: center;
  }
</style>

<main>
  <div class="container-fluid">
    <GameView/>
    <Resize/>
    <Terminal/>
  </div>
</main>