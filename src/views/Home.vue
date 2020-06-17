<template>
  <div class="home">
    <button @click="sendClear()">Clear</button>
    <SeriePainter v-on:done="save" :series="paintSeries" v-on:saveEvents="saveEvents"/>
    <div v-for="(s, i) in series" :key="s.name">
      <input  type="checkbox" :id="i" @change="handleChange()" v-model="s.selected">
      <label :for="s.name">{{s.name}}</label>
      <span :style="{backgroundColor: s.color, color: s.color}">___</span>
      <button @click="deleteSerie(i)">Delete</button>
      <button @click="exportSerie(i)">Export</button>
    </div>
    <div v-for="(s, i) in events" :key="s.name">
      <input  type="checkbox" :id="i" @change="handleChange()" v-model="s.selected">
      <label :for="s.name">{{s.name}}</label>
      <span :style="{backgroundColor: s.color, color: s.color}">___</span>
      <button @click="deleteEvents(i)">Delete</button>
      <button @click="exportEvents(i)">Export</button>
    </div>
  <div> 
    <button @click="emitEventRec()">{{eventRecord}}EventRec</button>
  </div>
    <div>
      <button @click="rebase()">Rebase at Zero</button>
      <button @click="highCap()">Cap upper</button>
      <button @click="lowCap()">Cap lower</button>
       <button @click="smooth()">Smooth</button>
      <div>
        <label for="base">Base</label>
      <input id="base" type="text" v-model="base"/>
      <label for="low">Low</label>
      <input id="low" type="text" v-model="low"/>
      <label for="high">High</label>
      <input id="high" type="text" v-model="high"/>
      <label for="win">Window</label>
      <input id="win" type="text" v-model="window"/>
</div>
      </div>
  <div>
    {{exportedSerie}}
</div>
  </div>
</template>

<script>
// @ is an alias to /src
import SeriePainter from '@/components/SeriePainter.vue'
import { EventBus } from '../event-bus'

export default {
  name: 'Home',
  components: {
    SeriePainter
  },
  data() {
    return {
      series: [],
      events: [],
      exportedSerie: '',
      paintSeries: [],
      low:-100,
      high: 100,
      base: 0,
      window: 5,
      eventRecord: 'start'
    }
  },
  methods: {
    save(serie) {
      this.series.push({name: `S-${serie.color}`, ...serie, selected: true})
    },
    saveEvents(events) {
      console.log("save")
      this.events.push({name: `E-${Math.random()*99999}`, ...events, selected:true})
    },
    deleteSerie(i) {
      this.series = this.series.filter((v,idx)=>{if (idx!=i) {return true} else {return false}})
    },
    exportSerie(i) {
      console.log(this.series[i])
      
      this.exportedSerie = this.series[i].serie
      navigator.clipboard.writeText(JSON.stringify(this.exportedSerie))
    },
    deleteEvents(i) {
      this.events = this.events.filter((v,idx)=>{if (idx!=i) {return true} else {return false}})
    },
    exportEvents(i) {
      console.log(this.events[i])
      var toExport = {}
      this.events[i].data.forEach((x) => toExport[x.toString()] = 'EVENT')
      this.exportedSerie = toExport
      navigator.clipboard.writeText(JSON.stringify(this.exportedSerie))
    },
    sendClear() {
      EventBus.$emit('clear', {})
    },
    handleChange() {
      this.paintSeries = this.series.filter(s => s.selected) 
    },
    rebase() {
      this.series.filter(s => s.selected).map(s => rebase(s, parseInt(this.base)))
      this.handleChange()
    },
    highCap() {
      this.series.filter(s => s.selected).map(s => highCap(s, parseInt(this.high)))
      this.handleChange()
    },
    lowCap() {
      this.series.filter(s => s.selected).map(s => lowCap(s, parseInt(this.low)))
      this.handleChange()
    },
    smooth() {
      this.series.filter(s => s.selected).map(s => smooth(s, parseInt(this.window)))
      this.handleChange()
    },
    emitEventRec() {
      EventBus.$emit(this.eventRecord+'EventRecord', {})
      this.eventRecord = (this.eventRecord == 'start' ? 'stop': 'start')
    }
  }
}

function rebase(s, base) {
  var min = s.serie.reduce((x, y) => x <y ? x: y)
  s.serie = s.serie.map(v => v - min + base)
  
}

function lowCap(s, low) {
  s.serie = s.serie.map( v=> ( v < low ? low : v))
}

function highCap(s, high) {
  s.serie = s.serie.map( v=> ( v > high ? high : v))
}

function smooth(s,w) {
  var a = s.serie.slice(0,w)
  s.serie = s.serie.map( v=> {a.shift(); a.push(v); return a.reduce((x,y)=>x+y)/a.length})
}


</script>