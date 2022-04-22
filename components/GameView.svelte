<style>
  :root {
    --factory_border: #cbd5dd;
    --factory_bg: aliceblue;
  }
  .factory {
    height: var(--viewHeight, 50vh);
    overflow: scroll;
    background-color: var(--factory_bg);
    border: 10px solid var(--factory_border);
    border-radius: 0.5rem;
    border-top: 20px solid var(--factory_border);
    transition: 0.5s height cubic-bezier(0.19, 1, 0.22, 1);
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
  .objShelf {
    opacity: 0.5;
    position: relative;
    top: -230px;
    left: 100px;
  }
  .objBox {
    position: relative;
    bottom: 133px;
    scale: 2.5;
    left: 267px;
  }
  .outside {
    width: 2000px;
  }
  .objTruck {
    scale: 1;
  }
</style>

<script>
  import { terminalHeight, gameHeight } from "../store.js";

  let objects = {
    truck: `<svg xmlns="http://www.w3.org/2000/svg" width="250" height="200" fill="currentColor" class="bi bi-truck" viewBox="0 0 16 16">
                                          <path d="M0 3.5A1.5 1.5 0 0 1 1.5 2h9A1.5 1.5 0 0 1 12 3.5V5h1.02a1.5 1.5 0 0 1 1.17.563l1.481 1.85a1.5 1.5 0 0 1 .329.938V10.5a1.5 1.5 0 0 1-1.5 1.5H14a2 2 0 1 1-4 0H5a2 2 0 1 1-3.998-.085A1.5 1.5 0 0 1 0 10.5v-7zm1.294 7.456A1.999 1.999 0 0 1 4.732 11h5.536a2.01 2.01 0 0 1 .732-.732V3.5a.5.5 0 0 0-.5-.5h-9a.5.5 0 0 0-.5.5v7a.5.5 0 0 0 .294.456zM12 10a2 2 0 0 1 1.732 1h.768a.5.5 0 0 0 .5-.5V8.35a.5.5 0 0 0-.11-.312l-1.48-1.85A.5.5 0 0 0 13.02 6H12v4zm-9 1a1 1 0 1 0 0 2 1 1 0 0 0 0-2zm9 0a1 1 0 1 0 0 2 1 1 0 0 0 0-2z"/>
                                        </svg>`,
    shelf: `<svg xmlns="http://www.w3.org/2000/svg" width="320" height="320" fill="currentColor" class="bi bi-bookshelf" viewBox="0 0 16 16">
                                          <path d="M2.5 0a.5.5 0 0 1 .5.5V2h10V.5a.5.5 0 0 1 1 0v15a.5.5 0 0 1-1 0V15H3v.5a.5.5 0 0 1-1 0V.5a.5.5 0 0 1 .5-.5zM3 14h10v-3H3v3zm0-4h10V7H3v3zm0-4h10V3H3v3z"/>
                                        </svg>`,
    box: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-archive" viewBox="0 0 16 16">
                                          <path d="M0 2a1 1 0 0 1 1-1h14a1 1 0 0 1 1 1v2a1 1 0 0 1-1 1v7.5a2.5 2.5 0 0 1-2.5 2.5h-9A2.5 2.5 0 0 1 1 12.5V5a1 1 0 0 1-1-1V2zm2 3v7.5A1.5 1.5 0 0 0 3.5 14h9a1.5 1.5 0 0 0 1.5-1.5V5H2zm13-3H1v2h14V2zM5 7.5a.5.5 0 0 1 .5-.5h5a.5.5 0 0 1 0 1h-5a.5.5 0 0 1-.5-.5z"/>
                                        </svg>`,
    truck_flat: `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-truck-flatbed" viewBox="0 0 16 16">
                                          <path d="M11.5 4a.5.5 0 0 1 .5.5V5h1.02a1.5 1.5 0 0 1 1.17.563l1.481 1.85a1.5 1.5 0 0 1 .329.938V10.5a1.5 1.5 0 0 1-1.5 1.5H14a2 2 0 1 1-4 0H5a2 2 0 1 1-4 0 1 1 0 0 1-1-1v-1h11V4.5a.5.5 0 0 1 .5-.5zM3 11a1 1 0 1 0 0 2 1 1 0 0 0 0-2zm9 0a1 1 0 1 0 0 2 1 1 0 0 0 0-2zm1.732 0h.768a.5.5 0 0 0 .5-.5V8.35a.5.5 0 0 0-.11-.312l-1.48-1.85A.5.5 0 0 0 13.02 6H12v4a2 2 0 0 1 1.732 1z"/>
                                        </svg>`,
    pinetree: `<svg xmlns="http://www.w3.org/2000/svg" width="160" height="160" fill="currentColor" class="bi bi-tree-fill" viewBox="0 0 16 16">
                                          <path d="M8.416.223a.5.5 0 0 0-.832 0l-3 4.5A.5.5 0 0 0 5 5.5h.098L3.076 8.735A.5.5 0 0 0 3.5 9.5h.191l-1.638 3.276a.5.5 0 0 0 .447.724H7V16h2v-2.5h4.5a.5.5 0 0 0 .447-.724L12.31 9.5h.191a.5.5 0 0 0 .424-.765L10.902 5.5H11a.5.5 0 0 0 .416-.777l-3-4.5z"/>
                                        </svg>`,
    bricks: `<svg xmlns="http://www.w3.org/2000/svg" width="160" height="160" fill="currentColor" class="bi bi-bricks" viewBox="0 0 16 16">
                                          <path d="M0 .5A.5.5 0 0 1 .5 0h15a.5.5 0 0 1 .5.5v3a.5.5 0 0 1-.5.5H14v2h1.5a.5.5 0 0 1 .5.5v3a.5.5 0 0 1-.5.5H14v2h1.5a.5.5 0 0 1 .5.5v3a.5.5 0 0 1-.5.5H.5a.5.5 0 0 1-.5-.5v-3a.5.5 0 0 1 .5-.5H2v-2H.5a.5.5 0 0 1-.5-.5v-3A.5.5 0 0 1 .5 6H2V4H.5a.5.5 0 0 1-.5-.5v-3zM3 4v2h4.5V4H3zm5.5 0v2H13V4H8.5zM3 10v2h4.5v-2H3zm5.5 0v2H13v-2H8.5zM1 1v2h3.5V1H1zm4.5 0v2h5V1h-5zm6 0v2H15V1h-3.5zM1 7v2h3.5V7H1zm4.5 0v2h5V7h-5zm6 0v2H15V7h-3.5zM1 13v2h3.5v-2H1zm4.5 0v2h5v-2h-5zm6 0v2H15v-2h-3.5z"/>
                                        </svg>`
  };

  //let viewHeight;
  //$: viewHeight = $gameHeight;
  $: cssVarStyles = `--viewHeight:${$gameHeight + "vh"}`;
</script>

<div class="factory" style="{cssVarStyles}">
<!--
  <div class="objSky d-flex flex-row">
    <div class="outside d-flex flex-row-reverse align-items-end">
      <span class="objTruck">{@html objects['truck']}</span>
      <span class="objPinetree">{@html objects['pinetree']}</span>
    </div>
    <div class="objFactory">
      <div class="ceiling d-flex">
        <div class="objLight"></div>
        <div class="objLight"></div>
        <div class="objLight"></div>
      </div>
      <div class="floor d-flex">
        <span>{@html objects['bricks']}</span>
        <span class="objBox">{@html objects['box']}</span>
        <span class="objShelf">
          {@html objects['shelf']}
        </span>
        <span class="objShelf">
        {@html objects['shelf']}
        </span>
      </div>
    </div>
  </div>
  -->
</div>
