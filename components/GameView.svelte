<style>
  :root {
    --factory_border: #cbd5dd;
  }
  .factory {
    height: var(--viewHeight, 48vh);
    overflow: scroll;
    background: radial-gradient(transparent, black);
    border: 10px solid var(--factory_border);
    border-radius: 0.5rem;
    border-top: 20px solid var(--factory_border);
    transition: 0.5s height cubic-bezier(0.19, 1, 0.22, 1);
    display: flex;
    justify-content: center;
    align-items: center;
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
  import window_mask from "../assets/mask_large.png";
  import bookshelf from "../assets/bookshelf.png";
  import boxes from "../assets/boxes.png";

  $: cssVarStyles = `--viewHeight:${$gameHeight + "vh"}`;

  //Define width and height of canas
  let cWidth = 1800;
  let cHeight = 800;

  onMount(async () => {
    //Game Canvas Code
    var canvas = new fabric.Canvas("c", { selection: false });
    canvas.backgroundColor = "rgba(0,0,0,0.6)";
    var grid = 60;

    //Function to load and create image tiles
    function addElement(x, y, e, layer) {
      fabric.Image.fromURL(e, function(img) {
        img.set({ left: x, top: y });
        canvas.insertAt(img, layer);
      });
    }

    //Add to Group Function
    function addElementToGroup(x, y, e, group) {
      fabric.Image.fromURL(e, function(img) {
        img.set({ left: x, top: y });
        group.addWithUpdate(img);
      });
    }

    //Initiate Groups
    let brickGroup = new fabric.Group();
    let rafterGroup = new fabric.Group();
    let decorGroup = new fabric.Group();
    let floorGroup = new fabric.Group();
    let windowGroup = new fabric.Group();
    let machineGroup = new fabric.Group();

    //Background Drawings
    for (var c = 0; c < (cHeight - grid * 2) / (grid * 2); c++) {
      for (var i = 0; i < cWidth / (grid * 2); i++) {
        addElementToGroup(i * (grid * 2), c * (grid * 2), bricks, brickGroup);
      }
    }

    //Draw Rafters
    for (var i = 0; i < cWidth / (grid * 2); i++) {
      addElementToGroup(i * (grid * 3.5), grid, rafter, rafterGroup);
    }

    //Generate floor blocks
    for (var c = 0; c <= 2; c++) {
      for (var i = 0; i < cWidth / (grid * 3); i++) {
        addElementToGroup(i * (grid * 3), cHeight - grid * c, block, floorGroup);
        addElementToGroup(i * (grid * 3), grid * 11, tile, floorGroup);
      }
    }

    //Draw Lighting and Windows
    for (var i = 0.5; i < cWidth / (grid * 4); i++) {
      addElementToGroup(i * (grid * 8), grid * 3.5, window, windowGroup);
      addElementToGroup(i * (grid * 10), 0, lamp, windowGroup);
    }

    //Draw Background Decor
    addElementToGroup(grid * 5, grid * 8, exhaust, decorGroup);
    addElementToGroup(grid * 6.5, grid * 8, exhaust, decorGroup);
    addElementToGroup(grid * 8, grid * 8, exhaust, decorGroup);

    //Draw Machines
    addElementToGroup(grid * 2, grid * 8 + 12, bookshelf, machineGroup);
    addElementToGroup(grid * 8, grid * 9 + 19, mixer, machineGroup);
    addElementToGroup(grid * 11, grid * 9 + 8, stamper, machineGroup);
    addElementToGroup(grid * 14, grid * 9 + 2, packager, machineGroup);
    addElementToGroup(grid * 19, grid * 9.5 + 13, boxes, machineGroup);

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
          { offset: 0, color: "#3e009670" },
          { offset: 1, color: "#00000000" }
        ]
      })
    });

    //Ceiling Blue Glow
    let floorDarkness = new fabric.Rect({
      left: 0,
      top: cHeight - grid * 2,
      width: cWidth,
      height: grid * 2,
      originX: "left",
      originY: "top",
      centeredRotation: true,
      fill: new fabric.Gradient({
        type: "linear",
        gradientUnits: "percentage", // or 'percentage'
        coords: { x1: 0, y1: 0, x2: 0, y2: 1 },
        colorStops: [
          { offset: 0, color: "#00000000" },
          { offset: 1, color: "#00000099" }
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

    //Add elements to canvas
    const myTimeout = setTimeout(() => {
      canvas.add(brickGroup);
      canvas.add(rafterGroup);
      canvas.add(windowGroup);
      canvas.add(floorGroup);
      canvas.add(decorGroup);
      canvas.add(machineGroup);
      canvas.add(ceilingGlow);
      canvas.add(floorDarkness);

      fabric.Image.fromURL(
        window_mask,
        mask => {
          mask.set({ left: 0, top: 0 });
          gameDarkness.cloneAsImage(img => {
            const maskFilter = new fabric.Image.filters.BlendImage({
              image: mask,
              mode: "multiply"
            });
            img.filters.push(maskFilter);
            img.applyFilters();
            canvas.add(img);
            //canvas.add(mask);
            canvas.renderAll();
          });
        },
        { crossOrigin: "annonymous" }
      );

      clearTimeout(myTimeout);
    }, 1000);

    //canvas.add(gameDarkness);
  }); // End of aync onMount
</script>

<div class="factory" style="{cssVarStyles}">
  <canvas id="c" width="{cWidth}" height="{cHeight}"></canvas>
</div>