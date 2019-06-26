<template>
  <div id="app">
    <div id="editor"></div>
    <div id="shell">
      <div class="shell-content">
        <p>Interactive JavaScript Shell</p>
      </div>
      <div class="input-line"><span style="font-size: 10px;">></span>
        <form v-on:submit.prevent="runCommand()">
          <input v-model="shellInput" type="text" >
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import * as ace from '../node_modules/ace-builds/src-noconflict/ace';
import { AceHandler } from './aceHandler';

export default {
  name: 'app',
  data () {
    return {
      editor: null,
      shellInput: '',
      aceHandler: null,
      commands: ['help - see a list of commands', 
        'random - returns a random float between 0 and 1', 
        'clear - clears shell',
        "run VARIABLES - runs all code in the editor and prints the value of the variable names past"]
    }
  },
  mounted() {
    ace.config.set('basePath','../node_modules/ace-builds/src-noconflict');
    this.editor = ace.edit('editor');
    this.editor.setTheme('ace/theme/monokai');
    this.editor.session.setMode('ace/mode/javascript');

    this.aceHandler = new AceHandler();
  },
  methods: {
    runCommand() {
      var command = this.shellInput.split(" ")[0];

      var par = document.createElement('p')
      par.appendChild(document.createTextNode('> ' + this.shellInput));
      par.style = 'input';
      document.getElementsByClassName('shell-content')[0].appendChild(par);

      if( command.toLowerCase() === 'help') {
        this.commands.forEach((command) => {
          this.printOutput(command);
        });
      } else if( command.toLowerCase() === 'clear') {
        document.getElementsByClassName('shell-content')[0].innerHTML = '<p>Shell content cleared</p>'
      } else if( command.toLowerCase() === 'random') {
        this.printOutput(Math.random())
      } else if( command.toLowerCase() === 'run') {
        var args = this.shellInput.split(" ").splice(1);
        this.aceHandler.runCode(this.editor.getValue(), args).forEach((result) => {
          this.printOutput(result);     
        });
      } else if( this.shellInput.includes('(') && this.shellInput.includes(')')) {
        var functionName = this.shellInput.split('(')[0];
        var functionArgs = this.shellInput.split('(')[1];
        functionArgs = functionArgs.substring(0,functionArgs.length-1).split(",");
        // Convert to correct variable type
        functionArgs = functionArgs.map((arg) => {
          arg = arg.trim();
          
          // If string remove quotes
          if(arg.charAt(0) == '"' || arg.charAt(arg.length-1) == "'") {
            return arg.substring(1,arg.length-1);
          
          // If number param convert
          } else if (arg.charAt(0) != "{" ) {
            return parseInt(arg);
          } 
        })

        var result = this.aceHandler.runFunction(this.editor.getValue(), functionName, functionArgs);
        this.printOutput(result);
      } 
      else {
        this.printOutput("Error: Command Not Found. Type 'help' for a list of commands.");
      }

      // Clear shellInput
      this.shellInput = '';

      // Scroll shell with data
      document.getElementById('shell').scrollTop = document.getElementById('shell').scrollHeight;
    },
    printOutput(str) {
      var par = document.createElement('p')
      par.appendChild(document.createTextNode(str));
      par.className = 'output';
      document.getElementsByClassName('shell-content')[0].appendChild(par); 
    }
  }
}
</script>

<style>
html,body {
  margin: 0;
  padding: 0;
}

#app {
  display: flex;
  width: 100vw;
  height: 100vh;
}

#editor {
  width: 60%;
  height: 100%;
  font-size: 25px;
}

#shell {
  font-family: 'sans-serif';
  background: rgb(50,50,50);
  color: white;
  padding: 25px;
  height: calc(100% - 50px);
  width: calc(60% - 50px);
  overflow-y: auto;
}

.shell-content {

}

.input-line {

}

form {
  display:inline;

}

input {
  background: rgba(0,0,0,0);
  border: 0;
  color: white;
  margin: 0;
  width: 80%;
  padding: 5px;
}

.output {
  padding-left: 10px;
  color: rgb(200,200,200);
  margin: 5px;
}

</style>
