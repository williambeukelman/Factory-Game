<style>
  @import url("https://cdn.jsdelivr.net/npm/hack-font@3.3.0/build/web/hack-subset.css");
  :root {
    --term_green: #1cdc9a;
    --term_bg: #232627;
    --term_lightblue: #319de9;
    --term_border: #2f3739;
    font-family: Hack, monospace;
  }
  #terminal {
    background-color: var(--term_bg);
    height: var(--terminalHeight, 50vh);
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
    padding: 0.1rem 0.5rem;
    border: none;
  }
</style>

<script>
  import { onMount } from "svelte";
  import { Terminal } from "xterm";

  let terminalHeight = "50vh";
  $: cssVarStyles = `--terminalHeight:${terminalHeight}`;

  onMount(async () => {
    var term = new Terminal({
      cursorBlink: "block",
      fontSize: 18,
      fontFamily: "Hack, monospace",
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
    var addrprompt = "\x1b[34m" + "~" + "\x1b[0m";
    var userprompt = "\x1b[32m" + "user" + "\x1b[0m";

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

    function prompt(term) {
      command = "";
      term.write("\r\n$ ");
    }

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
            ["file1", "file2", "file3", "file4", "file5"].join("\r\n")
          );
          term.prompt(term);
        },
        description: "Prints the current directory's file structure."
      }
    };

    function runCommand(term, text) {
      const command = text.trim().split(" ")[0];
      if (command.length > 0) {
        term.writeln("");
        if (command in commands) {
          commands[command].f();
          return;
        }
        term.writeln(`${command}: command not found`);
      }
      prompt(term);
    }

    term.prompt = function() {
      term.write("\r" + userprompt + ":" + addrprompt + shellprompt);
    };

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
    term.prompt();
  });

  let down_arrow = `
                      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-chevron-compact-down" viewBox="0 0 16 16">
                      <path fill-rule="evenodd" d="M1.553 6.776a.5.5 0 0 1 .67-.223L8 9.44l5.776-2.888a.5.5 0 1 1 .448.894l-6 3a.5.5 0 0 1-.448 0l-6-3a.5.5 0 0 1-.223-.67z"/>
                      </svg>
                      `;
  let up_arrow = `
                      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-chevron-compact-up" viewBox="0 0 16 16">
                      <path fill-rule="evenodd" d="M7.776 5.553a.5.5 0 0 1 .448 0l6 3a.5.5 0 1 1-.448.894L8 6.56 2.224 9.447a.5.5 0 1 1-.448-.894l6-3z"/>
                      </svg>
                      `;

  function maximize() {
    terminalHeight = "50vh";
  }

  function minimize() {
    terminalHeight = "0vh";
  }
</script>

<div class="titlebar">
  <button class="btn btn-outline-light"
  on:click={minimize}>{@html down_arrow}</button>
  <button class="btn btn-outline-light"
  on:click={maximize}>{@html up_arrow}</button>
</div>
<div id="terminal" style="{cssVarStyles}"></div>
