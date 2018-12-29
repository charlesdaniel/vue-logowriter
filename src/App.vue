<template>
  <div id="app">
    <div class="canvas">
      <Turtle v-model="commands" :scale="scale" />
    </div>
    <div class="history">
      <ul>
        <li @click="prefillCommand=cmd" v-for="(cmd, index) in commands" :key="index">{{cmd}}</li>
      </ul>
    </div>
    <div class="commands">
      Zoom: x{{scale}} <input type="range" min="1" max="10" v-model="scale">
      <CommandPane v-model="commands" :prefillCommand="prefillCommand" @append="appendCommand"/>
    </div>
  </div>
</template>

<script>
import CommandPane from './components/CommandPane.vue'
import Turtle from './components/Turtle.vue'

export default {
  name: 'app',
  components: {
    CommandPane,
    Turtle
  },
  data() {
    return {
      commands: [],
      availableCommands: [],
      colors: [],
      scale: '1',
      prefillCommand: '',
    }
  },
  methods: {
    appendCommand(command) {
      this.commands.push(command)
    },
  }
}
</script>

<style>
.canvas {
  border: 1px solid black;
  height: 500px;
  width: 75%;
  float: left;
}
.history {
  border: 1px solid blue;
  width: 20%;
  float: right;
  height: 500px;
  overflow: auto;
}
.commands {
  clear: both;
  width: 75%;
}
.history li {
  cursor: pointer;
}
.history li:hover {
  background-color: lightblue;
}
.history ul {
  list-style-type: none;
  padding-left: 10px;
}

</style>
