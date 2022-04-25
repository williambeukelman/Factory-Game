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
  import rafter from "../assets/rafers.png";
  import exhaust from "../assets/exhaust.png";
  import window from "../assets/window.png";
  import mixer from "../assets/mixer.png";
  import stamper from "../assets/stamper.png";
  import packager from "../assets/packager.png";
  import lamp from "../assets/hanging_lamp.png";
  import bricks from "../assets/bricks.png";

  $: cssVarStyles = `--viewHeight:${$gameHeight + "vh"}`;

  //Define width and height of canas
  let cWidth = 1200;
  let cHeight = 720;

  onMount(async () => {
    //Game Canvas Code
    var canvas = new fabric.Canvas("c", { selection: false });
    canvas.backgroundColor = "rgba(0,0,0,0.6)";
    var grid = 60;

    // create grid
    /*
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
            */

    //Function to load and create image tiles
    function addElement(x, y, e) {
      fabric.Image.fromURL(e, function(img) {
        canvas.add(img.set({ left: x, top: y }));
        canvas.sendBackwards(img);
      });
    }

    //Background Drawings
    for (var c = 0; c < (cHeight - grid * 2) / (grid * 2); c++) {
      for (var i = 0; i < cWidth / (grid * 2); i++) {
        addElement(i * (grid * 2), c * (grid * 2), bricks);
      }
    }

    //Draw Rafters
    for (var i = 0; i < cWidth / (grid * 2); i++) {
      addElement(i * (grid * 3.5), grid, rafter);
    }

    //Generate floor blocks
    for (var c = 0; c <= 2; c++) {
      for (var i = 0; i < cWidth / (grid * 3); i++) {
        addElement(i * (grid * 3), cHeight - grid * c, block);
        addElement(i * (grid * 3), grid * 10, tile);
      }
    }

    //Draw Lighting and Windows
    for (var i = 1; i < cWidth / (grid * 3.25); i++) {
      addElement(i * (grid * 3), grid * 3, window);
    }

    //Draw Background Decor
    addElement(grid * 1, grid * 7, exhaust);
    addElement(grid * 2.5, grid * 7, exhaust);
    addElement(grid * 4, grid * 7, exhaust);

    //Draw Machines
    addElement(grid * 1, grid * 8 + 19, mixer);
    addElement(grid * 4, grid * 8 + 8, stamper);
    addElement(grid * 7, grid * 8 + 2, packager);

    //Draw Gradients
    //Ceiling Blue Glow
    let ceilingGlow = new fabric.Rect({
      left: 0,
      top: 0,
      width: cWidth,
      height: cHeight / 1.5,
      originX: "left",
      originY: "top",
      centeredRotation: true,
      fill: new fabric.Gradient({
        type: "linear",
        gradientUnits: "percentage", // or 'percentage'
        coords: { x1: 0, y1: 0, x2: 0, y2: 1 },
        colorStops: [
          { offset: 0, color: "#3e009650" },
          { offset: 1, color: "#ffffff00" }
        ]
      })
    });

    //Game Darkness
    let gameDarkness = new fabric.Rect({
      left: 0,
      top: 0,
      width: cWidth,
      height: cHeight,
      originX: "left",
      originY: "top",
      centeredRotation: true,
      fill: "#00000090"
    });

    canvas.bringToFront(ceilingGlow);
    //canvas.add(gameDarkness);

    /*
        let cube = new fabric.Rect({
          left: 60,
          top: 60,
          width: 120,
          height: 120,
          originX: "left",
          originY: "top",
          centeredRotation: true,
          fill: "#000"
        });

        canvas.add(cube);

        //let mask = cube.cloneAsImage((img) => { return img; });
        cube.cloneAsImage((mask) => {
          gameDarkness.cloneAsImage((img) => { 
            const maskFilter = new fabric.Image.filters.BlendImage({
              image: mask,
              mode: 'multiply',
            });
            img.filters.push(maskFilter);
            img.applyFilters();
            canvas.add(img);
            //anvas.renderAll();
          });
        });
        */

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