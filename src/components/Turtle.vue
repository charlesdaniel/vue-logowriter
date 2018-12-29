<template>
<div>
  <svg ref="svg" :style="{backgroundColor: backgroundColor}">
    <g :transform="transforms">
      <path class="line" :d="path.data" :stroke="path.color" strokeWidth="4" v-for="(path, index) in paths" :key="index" />
      <path v-if="!turtle.hide" id="turtle" d="M -5,-5 L 0,0 L 5,-5 L 0,10 Z" opacity="0.75" :fill="turtle.penDown ? turtle.penColor : 'none'" stroke="black" strokeWidth="3" :transform="turtleTransforms" />
    </g>
  </svg>
  <br/>
</div>
</template>

<script>
export default {
  name: 'Turtle',
  props: {
    scale: {
      type: String,
      default: '1',
    },
    value: {
      type: Array,
      default: [],
    },
  },
  data() {
    return {
      height: 0,
      width: 0,
      backgroundColor: 'white',
      turtle: {
        x: 0,
        y: 0,
        angle: 0.0,
        penDown: true,
        penColor: 'black',
        hide: false,
      },
      colors: ['black', 'silver', 'white', 'red', 'blue', 'yellow', 'green', 'orange'],
      _paths: [],
      pathIndex: -1,
      commandIndex: -1,
    }
  },
  methods: {
    resize() {
      let computed = getComputedStyle(this.$refs['svg'])
      this.width = parseFloat(computed.width)
      this.height = parseFloat(computed.height) 
    },
    getColor(input) {
      let colorIndex = parseInt(input, 10)
      if (isNaN(colorIndex)) {
        return input
      }
      return this.colors[colorIndex] || 'white'
    },
    toRad(deg) {
      return ((deg * Math.PI) / 180.0)
    }
  },
  computed: {
    turtleTransforms() {
      return `translate(${this.turtle.x}, ${this.turtle.y}) rotate(${this.turtle.angle - 90} )`
    },
    transforms() {
      const x = this.width / 2.0
      const y = this.height / 2.0
      return `translate(${x}, ${y}) rotate(-90) scale(${this.scale}, ${this.scale})`
    },
    paths() {
      let self = this
      if (! self._paths) {
        self._paths = [{ color: 'black', data: `M 0,0 ` }]
        self.pathIndex = 0
      }

      // TODO: This parser is waaay too hacky and brittle. Need to rewrite.
      function nextToken(cmd) {
        let str
        while(str = cmd.shift()) {
          str = str.replace('[', '').replace(']', '').replace(' ', '').replace('=', '')
          if(str) {
            return str
          }
        }
      }
      function processCommand(cmd) {
        cmd = cmd.slice(0)
        let instr
        let vars = {}

        while (instr = nextToken(cmd)) {
          switch(instr.toLowerCase()) {
            case 'pu':
              self.turtle.penDown = false
              break
            case 'pd':
              self.turtle.penDown = true
              break
            case 'rt':
              self.turtle.angle += parseFloat(nextToken(cmd))
              self.turtle.angle %= 360.0
              break
            case 'lt':
              self.turtle.angle -= parseFloat(nextToken(cmd))
              self.turtle.angle %= 360.0
              break
            case 'fd':
              let fd_length = nextToken(cmd)
              self.turtle.x += Math.cos(self.toRad(self.turtle.angle)) * parseFloat(fd_length)
              self.turtle.y += Math.sin(self.toRad(self.turtle.angle)) * parseFloat(fd_length)
              self._paths[self.pathIndex]['data'] += (self.turtle.penDown ? ` L ${self.turtle.x},${self.turtle.y}` : ` M ${self.turtle.x},${self.turtle.y}`)
              break
            case 'bk':
              let bk_length = nextToken(cmd)
              self.turtle.x -= Math.cos(self.toRad(self.turtle.angle)) * parseFloat(bk_length)
              self.turtle.y -= Math.sin(self.toRad(self.turtle.angle)) * parseFloat(bk_length)
              self._paths[self.pathIndex]['data'] += (self.turtle.penDown ? ` L ${self.turtle.x},${self.turtle.y}` : ` M ${self.turtle.x},${self.turtle.y}`)
              break
            case 'cs':
            case 'clear':
              self._paths = [{ color: 'black', data: 'M 0,0 ' }]
              self.$emit('input', [])
              self.commandIndex = -1
              self.turtle.x = 0
              self.turtle.y = 0
              self.turtle.angle = 0.0
              self.turtle.penDown = true
              self.pathIndex = self._paths.length - 1
              break
            case 'home':
              self.turtle.x = 0
              self.turtle.y = 0
              self.turtle.angle = 0.0
              self.turtle.penDown = true
              self._paths.push({ color: self.turtle.penColor, data: 'M 0,0' })
              self.pathIndex = self._paths.length - 1
              break
            case 'repeat':
              const iterations = parseInt(nextToken(cmd), 10)
              cmd[0] = cmd[0].replace('[', '')
              cmd[cmd.length - 1] = cmd[cmd.length - 1].replace(']', '')
              for(let i=0; i<iterations; i++) {
                processCommand(cmd)
              }
              break
            case 'setbg':
              self.backgroundColor = self.getColor(nextToken(cmd))
              break
            case 'pc':
            case 'setc':
            case 'setpencolor':
              self.turtle.penColor = self.getColor(nextToken(cmd))
              self._paths.push({ color: self.turtle.penColor, data: `M ${self.turtle.x},${self.turtle.y}` })
              self.pathIndex = self._paths.length - 1
              break
            case 'setpos':
              let newX = nextToken(cmd)
              let newY = nextToken(cmd)
              self.turtle.x = parseInt(newX, 10)
              self.turtle.y = parseInt(newY, 10)
              break
            case 'seth':
            case 'setangle':
              self.turtle.angle = parseFloat(nextToken(cmd))
              break
            case 'ht':
            case 'hideturtle':
              self.turtle.hide = true
              break
            case 'st':
            case 'showturtle':
              self.turtle.hide = false
              break
          }
        }
      }

      for(let i=this.commandIndex+1; i<this.value.length; i++) {
        if(this.value[i]) {
          this.commandIndex = i
          const cmd = String(this.value[i]).split(/[^\#\w\d]+/)
          processCommand(cmd)
        }
      }
      return self._paths
    }
  },
  mounted() {
    this.resize()
    window.addEventListener('resize', this.resize)
  }
}
</script>

<style scoped>
div {
  border: 1px solid black;
  height: 100%;
  width: 100%;
}
svg {
  height: 100%;
  width: 100%;
}
path.line {
  fill: none;
}
</style>