<style>
  :root {
    --factory_border: #cbd5dd;
    --factory_bg: aliceblue;
  }
  .factory {
    height: var(--viewHeight, 48vh);
    overflow: scroll;
    background-color: var(--factory_bg);
    border: 10px solid var(--factory_border);
    border-radius: 0.5rem;
    border-top: 20px solid var(--factory_border);
    transition: 0.5s height cubic-bezier(0.19, 1, 0.22, 1);
  }
  canvas {
    border: 1px solid #ccc;
  }
</style>

<script>
  import { onMount } from "svelte";
  import { fabric } from "fabric";
  import { terminalHeight, gameHeight } from "../store.js";
  import block from "../assets/floor_blocks.png";
  import tile from "../assets/floor_tile.png";
  import rafers from "../assets/rafers.png";

  $: cssVarStyles = `--viewHeight:${$gameHeight + "vh"}`;

  //Define width and height of canas
  let cWidth = 2400;
  let cHeight = 720;

  onMount(async () => {
    //Game Canvas Code
    var canvas = new fabric.Canvas("c", { selection: false });
    canvas.backgroundColor = "rgba(0,0,0,0.6)";
    var grid = 60;

    // create grid
    for (var i = 0; i < cWidth / grid; i++) {
      canvas.add(
        new fabric.Line([i * grid, 0, i * grid, cHeight], {
          stroke: "#ccc",
          selectable: false
        })
      );
      canvas.add(
        new fabric.Line([0, i * grid, cWidth, i * grid], {
          stroke: "#ccc",
          selectable: false
        })
      );
    }

    // add objects
    canvas.add(
      new fabric.Rect({
        left: 100,
        top: 100,
        width: 50,
        height: 50,
        fill: "#faa",
        originX: "left",
        originY: "top",
        centeredRotation: true
      })
    );

    function addElement(x, y, e) {
      fabric.Image.fromURL(e, function(img) {
        canvas.add(img.set({ left: x, top: y }).scale(0.25));
      });
    }

    addElement(0, 0, tile);

    //Generate floor blocks
    for (var c = 0; c <= 2; c++) {
      for (var i = 0; i < cWidth / (grid * 3); i++) {
        addElement(i * (grid * 3), cHeight - grid * c, block);
      }
      //addElement(i * (grid * 3), cHeight - grid * 4, tile);
    }

    // snap to grid
    canvas.on("object:moving", function(options) {
      options.target.set({
        left: Math.round(options.target.left / grid) * grid,
        top: Math.round(options.target.top / grid) * grid
      });
    });
  }); // End of aync onMount
</script>

<div class="factory" style="{cssVarStyles}">
  <canvas id="c" width="{cWidth}" height="{cHeight}"></canvas>
</div>