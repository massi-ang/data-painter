<template>
<div id='canvas'>
  <canvas ref='select' @mousedown='onMouseDown' @mousemove='recordMouse' @mouseup='stopSelect'></canvas>
</div>
</template>

<script>
import { EventBus } from '../event-bus.js'
export default {
  props: ["series"],
  data() {
    return {
      ctx: null,
      selectionMode: false,
      startPosition: {
        x: null,
        y: null
      },
      offset: {
        x: 0,
        y: 0
      },
      t: 0,
      last: {
        x:0,
        y:0
      },
      refY:0,
      intvl: null,
      lastMousePos: {
        x: 0,
        y: 0
      },
      serie: [],
      color: null,
      eventRecording: false,
      events: []
    };
  },
  methods: {
    onMouseDown(e) {
      if (!this.eventRecording) {
        this.t = 0
        this.serie = []
        this.intvl = setInterval(()=>{ this.t++; this.draw(); }, 20)
        this.selectionMode = true;
        this.startPosition.x = 0;
        this.startPosition.y = this.offset.height/2;
        this.recordMouse(e)
        this.last = this.startPosition
        this.refY = e.clientY
        this.serie.push(0)
        this.color = randomColor()
      } else {
        this.recordEvent()
      }
    },
    recordEvent() {
      this.events.push(this.lastMousePos.x)
      this.drawEvents(this.events, this.color)
    },
    recordMouse(e) {
      this.lastMousePos.x = e.clientX
      this.lastMousePos.y = e.clientY
    },
    draw() {
      if (this.selectionMode) { 
        if (this.t > this.offset.width) {
          this.stopSelect()
          return
        }
        this.ctx.beginPath();
        this.ctx.moveTo(this.last.x, this.last.y)
        this.ctx.lineTo(this.last.x+1, this.last.y +(this.lastMousePos.y - this.refY ))
        this.ctx.strokeStyle = this.color
        this.last.x = this.last.x+1;
        this.last.y = this.last.y + this.lastMousePos.y -this.refY
        this.refY = this.lastMousePos.y
        this.serie.push(-(this.last.y - this.offset.height/2))
        this.ctx.stroke();
      } 
    },
    drawSeries() {
      this.clearCanvas()
      for (var i in this.series) {
        this.drawSerie(this.series[i].serie, this.series[i].color)
      }
    },
    drawSerie(s, color) {
      var _x = 0, _y = s[x]
      this.ctx.strokeStyle = color
      for (var x in s) {
        this.ctx.beginPath();
        this.ctx.moveTo(_x, -_y+this.offset.height/2)
        this.ctx.lineTo(x, -s[x]+this.offset.height/2)
        this.ctx.stroke()
        _x = x
        _y = s[x]
      }
    },

    drawEvents(e, color) {
      e.forEach((x) => {
        this.ctx.fillStyle = color
        this.ctx.moveTo(x, this.offset.height/2)
        this.ctx.arc(x-5, this.offset.height/2, 5, 0, 2*Math.PI, true )
        this.ctx.fill()
      })
    },
    
    stopSelect() {
      if (this.serie === undefined) {
        return
      }
      this.ctx.fillStyle = "#fff";
      clearInterval(this.intvl)
      this.selectionMode = false;
      this.startPosition.x = null;
      this.startPosition.y = null;
      this.$emit("done", { color: this.color, serie: this.serie})
      this.serie = undefined
    },

    clearCanvas() {
      console.log('Clear')
      this.ctx.clearRect(0,0,this.offset.width, this.offset.height)

      for (var y=0; y<=this.offset.height; y=y+this.offset.height/10) {
        this.ctx.beginPath()
        this.ctx.moveTo(0, y)
        this.ctx.strokeStyle = '#ddd'
        this.ctx.lineTo(this.offset.width, y)
        this.ctx.stroke()
      }
      this.ctx.beginPath()
      this.ctx.moveTo(0, this.offset.height/2)
      this.ctx.strokeStyle = '#999'
      this.ctx.lineTo(this.offset.width, this.offset.height/2)
      this.ctx.stroke()
    }
    
  },
  mounted() {
    this.$refs.select.height = this.$refs.select.parentElement.clientHeight;
    this.$refs.select.width = this.$refs.select.parentElement.clientWidth
    let rec = this.$refs.select.getBoundingClientRect()
    this.offset = rec
    this.ctx = this.$refs.select.getContext("2d");
    EventBus.$off("clear")
    EventBus.$on("clear", this.clearCanvas)
    EventBus.$on("startEventRecord", ()=> { 
      this.eventRecording=true
      this.events = []
      this.color = randomColor()})
    EventBus.$on("stopEventRecord", ()=>{
      this.eventRecording=false
      this.$emit("saveEvents", {color: this.color, data: this.events})
      })
    this.drawSeries()
    // this.ctx.fillRect(0,0,500,500);
  },
  watch: {
    series() {
      this.drawSeries()
    }
  }
}

function randomColor() {
  return `rgb( ${Math.floor(Math.random()*255)},${Math.floor(Math.random()*255)},${Math.floor(Math.random()*255)})`
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#canvas {
  background: #fdfdfd;
  width: 100%;
  height: 400px;
  box-shadow: 0px 2px 3px rgba(0, 0, 0, 0.2);
}
</style>
