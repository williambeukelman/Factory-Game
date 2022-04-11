<style>
  :root {
    --term_green: #1cdc9a;
    --term_bg: #232627;
    --term_lightblue: #319de9;
    --term_border: #2f3739;
  }
  .termUser {
    color: var(--term_green);
  }
  .termAddr {
    color: var(--term_lightblue);
    word-spacing: 1rem;
  }
  #terminal {
    background-color: var(--term_bg);
    height: 50vh;
    overflow: scroll;
    border: 10px solid var(--term_border);
    border-radius: 0.5rem;
    border-top: 30px solid var(--term_border);
  }
  .terminal .col {
    text-align: left;
  }
  .terminal .col p {
    display: flex;
  }
  .terminal input {
    flex-grow: 1;
    border: none;
    background: none;
    color: white;
  }
  #userInput:focus-visible {
    outline: none;
  }
</style>

<script>
  import { onMount } from "svelte";
  import { Terminal } from "xterm";
  import "xterm/css/xterm.css";

  onMount(async () => {
    var term = new Terminal({
      cursorBlink: "block",
      fontSize: 18,
      fontFamily: "Hack",
      theme: {
        background: "#232627",
        foreground: "#F5F8FA",
        red: "#c0392b",
        green: "#1cdc9a",
        yellow: "#fdbc4b",
        blue: "#3daee9",
        magenta: "#8e44ad",
        cyan: "#16a085",
        white: "#ffffff"
      }
    });
    term.open(document.getElementById("terminal"), false);
    var shellprompt = "$ ";
    var curr_line = "";
    var entries = [];

    //term.onKey();

    term.prompt = function() {
      term.write("\r" + shellprompt);
    };

    //term.writeln("");
    term.prompt();
  });
</script>

<div id="terminal"></div>
