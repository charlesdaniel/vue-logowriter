<template>
  <div>
    <form @submit.prevent>
      <input ref="command-input" type="text" v-model="command" placeholder="Enter Command" @keyup.enter.prevent="execCommand" @keyup.up.prevent="prefillLastCommand" />
      <div class="cmdInfo">
        <ul>
            <li v-for="(cmdInfo, index) in filteredCommands" :key="index"><b>{{cmdInfo[0]}}</b> - {{cmdInfo[1]}}</li>
        </ul>
      </div>
    </form>
  </div>
</template>

<script>
export default {
    name: 'CommandPane',
    props: ['value', 'prefillCommand'],
    data() {
        return {
            command: '',
            availableCommands: [
                [ 'fd U', 'Forward U units' ],
                [ 'bk U', 'Backward U units' ],
                [ 'lt D', 'Turn Left D degrees' ],
                [ 'rt D', 'Turn Right D degrees' ],
                [ 'pu', 'Pen Up (no line drawing)' ],
                [ 'pd', 'Pen Down (line drawing)' ],
                [ 'clear', '(or cs) Clear canvas and reset turtle' ],
                [ 'home', 'Reset turtle to home' ],
                [ 'repeat N [ CMDS ]', 'Repeat the CMDS sequence N times. CMDS are valid movements.' ],
                [ 'setbg C', 'Set the background color to C' ],
                [ 'setc C', '(or setpencolor or pc) Set the pen color to C. C can be a number 0..7 or #rgb or string like "black"'],
                [ 'setpos [X Y]', 'Set the absolute position of the turtle to X,Y'],
                [ 'seth D', '(or setangle) Set the turtle\'s absolute angle to D degrees'],
                [ 'ht', '(or hideturtle) Hide the turtle'],
                [ 'st', '(or showturtle) Show the turtle'],
            ]
        }
    },
    watch: {
        prefillCommand(val) {
            this.command = val
            this.$refs['command-input'].focus()
        }
    },
    methods: {
        execCommand() {
            let command = this.command.trim()
            this.$emit('append', command)
            this.command = ''
        },
        prefillLastCommand() {
            this.command = this.value[this.value.length - 1]
        }
    },
    computed: {
        filteredCommands() {
            return this.availableCommands.filter( a => (a[0].toLowerCase().indexOf(this.command) > -1 || a[1].toLowerCase().indexOf(this.command) > -1) )
        }
    },
    mounted() {
        this.$refs['command-input'].focus()
    }
}
</script>

<style scoped>
input {
    width: 100%;
    font-size: 1.2em;
}
.cmdInfo {
    height: 10%;
    overflow: auto;
}
</style>
