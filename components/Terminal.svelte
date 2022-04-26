<style>
  /*@import url("https://cdn.jsdelivr.net/npm/hack-font@3.3.0/build/web/hack-subset.css");
    */
  @import url("https://fonts.googleapis.com/css2?family=Roboto+Mono&display=swap");
  :root {
    --term_green: #1cdc9a;
    --term_bg: #232627;
    --term_lightblue: #319de9;
    --term_border: #2f3739;
    font-family: "Roboto Mono", monospace;
  }
  #terminal {
    background-color: var(--term_bg);
    height: var(--terminalHeight, 48vh);
    overflow: scroll;
    border: 10px solid var(--term_border);
    border-radius: 0.5rem;
    border-top: 2rem solid var(--term_border);
    transition: 0.5s height cubic-bezier(0.19, 1, 0.22, 1);
  }
  .titlebar {
    width: 100%;
    height: 2rem;
    background-color: transparent;
    display: flex;
    position: absolute;
    padding: 0rem 1rem;
  }
  .titlebar button {
    margin-right: 0.3rem;
    padding: 0.1rem 0.2rem;
    border: none;
  }
  .titlebar button:hover {
    filter: brightness(1.5);
  }
</style>

<script>
  import { onMount } from "svelte";
  import { Terminal } from "xterm";
  import bashEmulator from "bash-emulator";
  import { terminalHeight, gameHeight } from "../store.js";

  $: cssVarStyles = `--terminalHeight:${$terminalHeight + "vh"}`;

  //JSON Data
  let directory = [
    {
      name: "Factory",
      type: "Directory",
      contents: [
        {
          name: "Dock",
          type: "Directory",
          contents: [
            {
              name: "Inventory",
              type: "Directory",
              contents: [
                { name: "Sodium Hydroxide.item", type: "File", contents: "" },
                { name: "Vegetable Oil.item", type: "File", contents: "" },
                { name: "Lavender Oil.item", type: "File", contents: "" }
              ]
            },
            {
              name: "Employees",
              type: "Directory",
              contents: [{ name: "Emp001.employee", type: "File", contents: "" }]
            }
          ]
        },
        {
          name: "Floor",
          type: "Directory",
          contents: [
            {
              name: "Machines",
              type: "Directory",
              contents: [
                {
                  name: "Mixer",
                  type: "Directory",
                  contents: [
                    { name: "Mixer.machine", type: "File", contents: "" },
                    { name: "manual.txt", type: "File", contents: "" },
                    { name: ".config", type: "File", contents: "" }
                  ]
                },
                {
                  name: "Stamper",
                  type: "Directory",
                  contents: [
                    { name: "Stamper.machine", type: "File", contents: "" },
                    { name: "manual.txt", type: "File", contents: "" },
                    { name: ".config", type: "File", contents: "" }
                  ]
                },
                {
                  name: "Packager",
                  type: "Directory",
                  contents: [
                    { name: "Packager.machine", type: "File", contents: "" },
                    { name: "manual.txt", type: "File", contents: "" },
                    { name: ".config", type: "File", contents: "" }
                  ]
                }
              ]
            },
            { name: "Employees", type: "Directory", contents: [] }
          ]
        },
        { name: "Office", type: "Directory", contents: [] },
        {
          name: "Warhouse",
          type: "Directory",
          contents: [{ name: "Inventory", type: "Directory", contents: [] }]
        },
        { name: "Shipping", type: "Directory", contents: [] }
      ]
    }
  ];

  onMount(async () => {
    var term = new Terminal({
      cursorBlink: "block",
      fontSize: 16,
      fontFamily: "Roboto Mono, monospace",
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
    var userprompt = "\x1b[32m" + "user" + "\x1b[0m";

    function cB(text) {
      //Colors text blue
      return "\x1b[34m" + text + "\x1b[0m";
    }

    term.onData(e => {
      switch (e) {
        case "\u0003": // Ctrl+C
          term.write("^C");
          prompt(term);
          break;
        case "\r": // Enter
          runCommand(term, command);
          command = "";
          break;
        case "\u007F": // Backspace (DEL)
          // Do not delete the prompt
          if (term._core.buffer.x > 2) {
            term.write("\b \b");
            if (command.length > 0) {
              command = command.substr(0, command.length - 1);
            }
          }
          break;
        default:
          // Print all other characters for demo
          if (
            (e >= String.fromCharCode(0x20) && e <= String.fromCharCode(0x7b)) ||
            e >= "\u00a0"
          ) {
            command += e;
            term.write(e);
          }
      }
    });

    var command = "";
    var commands = {
      help: {
        f: () => {
          term.writeln(
            [
              "Here are the currently available commands.",
              "",
              ...Object.keys(commands).map(
                e => `  ${e.padEnd(10)} ${commands[e].description}`
              )
            ].join("\n\r")
          );
          prompt(term);
        },
        description: "Prints this help message"
      },
      ls: {
        f: () => {
          term.writeln(
            ["file1", "file2", "file3", "file4", "file5"].map(cB).join("\t")
          );
          term.prompt(term);
        },
        description: "Prints the current directory's file structure."
      }
    };

    function log(result) {
      if (result) {
        term.writeln(result);
        term.prompt(term);
      }
    }

    function error(result) {
      term.writeln(result);
      term.prompt(term);
    }

    function runCommand(term, text) {
      const command = text.trim().split(" ")[0];
      if (command.length > 0) {
        term.writeln("");
        emulator.run(text).then(log, error);
        return;
        /*if (command in commands) {
                commands[command].f();
                return;
              }
              term.writeln(`${command}: command not found`);
              */
      }
      term.prompt(term);
    }

    //text to insert everytime it goes back to user input
    term.prompt = function() {
      emulator.getDir().then(path => {
        term.write("\r" + userprompt + ":" + cB(path) + shellprompt);
      });
      //term.write("\r" + userprompt + ":" + addrprompt + shellprompt);
    };

    //Initialize bash emulator with file structure
    var emulator = bashEmulator({
      workingDirectory: "/",
      fileSystem: {
        "/": {
          type: "dir",
          modified: Date.now()
        },
        "/README.txt": {
          type: "file",
          modified: Date.now(),
          content: "empty"
        },
        "/home": {
          type: "dir",
          modified: Date.now()
        },
        "/home/user/journal.txt": {
          type: "file",
          modified: Date.now(),
          content: "this is private!"
        },
        "/home/user": {
          type: "dir",
          modified: Date.now()
        }
      }
    });

    let welcomeMsg = [
      "                        *** FACTORY GAME ***",
      "",
      " Welcome! This is the terminal, it will be used to control almost",
      " every aspect of your factory. I will explain how to use it.",
      "",
      " How about you start by looking around, you can change your location",
      " at any time with cd (change directory) and use ls (list) to see what",
      " is available.",
      ""
    ].join("\n\r");

    term.writeln(welcomeMsg);
    term.prompt(term);
  }); //End of async onMount

  let down_arrow = `
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-chevron-compact-down" viewBox="0 0 16 16">
        <path fill-rule="evenodd" d="M1.553 6.776a.5.5 0 0 1 .67-.223L8 9.44l5.776-2.888a.5.5 0 1 1 .448.894l-6 3a.5.5 0 0 1-.448 0l-6-3a.5.5 0 0 1-.223-.67z"/>
        </svg>`;

  let up_arrow = `
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-chevron-compact-up" viewBox="0 0 16 16">
        <path fill-rule="evenodd" d="M7.776 5.553a.5.5 0 0 1 .448 0l6 3a.5.5 0 1 1-.448.894L8 6.56 2.224 9.447a.5.5 0 1 1-.448-.894l6-3z"/>
        </svg>`;

  function middle() {
    $gameHeight = 48;
    $terminalHeight = 48;
  }

  function maximize() {
    $gameHeight = 7;
    $terminalHeight = 89;
  }

  function minimize() {
    $gameHeight = 89;
    $terminalHeight = 7;
  }
</script>

<div class="titlebar">
  <button class="btn minimize"
  on:click={minimize}>
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="#eb606b" class="bi bi-circle-fill" viewBox="0 0 16 16">
      <circle cx="8" cy="8" r="8"/>
    </svg>
  </button>
  <button class="btn middle"
  on:click={middle}>
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="#f7eb95" class="bi bi-circle-fill" viewBox="0 0 16 16">
      <circle cx="8" cy="8" r="8"/>
    </svg>
  </button>
  <button class="btn maximize"
  on:click={maximize}>
    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="#2ecc71" class="bi bi-circle-fill" viewBox="0 0 16 16">
      <circle cx="8" cy="8" r="8"/>
    </svg>
  </button>
<!--
  <button class="btn btn-outline-light"
  on:click={minimize}>{@html down_arrow}</button>
  <button class="btn btn-outline-light"
  on:click={maximize}>{@html up_arrow}</button>
-->
</div>
<div id="terminal" style="{cssVarStyles}"></div>
