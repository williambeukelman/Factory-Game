<style>
  :root {
    --term_green: #1cdc9a;
    --term_bg: #232627;
    --term_lightblue: #319de9;
    --term_border: #2f3739;
  }
  #terminal {
    background-color: var(--term_bg);
    height: 50vh;
    overflow: scroll;
    border: 10px solid var(--term_border);
    border-radius: 0.5rem;
    border-top: 30px solid var(--term_border);
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
</script>

<div id="terminal"></div>
