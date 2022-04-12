<style>
  :root {
    --factory_border: #cbd5dd;
    --factory_bg: aliceblue;
  }
  .factory {
    height: 50vh;
    overflow: scroll;
    background-color: var(--factory_bg);
    border: 10px solid var(--factory_border);
    border-radius: 0.5rem;
    border-top: 20px solid var(--factory_border);
  }
  .objSky {
    width: 3000px;
    height: 100%;
    background: linear-gradient(to top, cornflowerblue, rgb(141, 172, 230));
  }
  .objFactory {
    position: relative;
    left: 500px;
    width: 2000px;
    height: 100%;
    background-color: #a1a9af;
  }
  .objLight {
    margin-left: 200px;
    width: 400px;
    height: 400px;
    background: linear-gradient(135deg, white, transparent, transparent);
    transform: rotateZ(45deg);
  }
</style>

<script>
</script>

<div class="factory">
  <div class="objSky">
    <div class="objFactory d-flex ">
      <div class="objLight"></div>
      <div class="objLight"></div>
      <div class="objLight"></div>
    </div>
  </div>
</div>
