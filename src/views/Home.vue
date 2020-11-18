<template>
  <div class="section">
    <div class="level">
      <b-button @click="sendClear()">Clear</b-button>
      <b-button type="is-primary" @click="emitEventRec()">{{eventRecord}}EventRec</b-button>
    </div>
    <div class="level">
      <SeriePainter v-on:done="save" :series="paintSeries" v-on:saveEvents="saveEvents" />
    </div>
    <div class="columns">
      <div class="column">
    <section>
      <div class="block" v-for="(s, i) in series" :key="s.name">
        <b-checkbox :id="i" @change="handleChange()" v-model="s.selected">{{s.name}}<span :style="{backgroundColor: s.color, color: s.color}">___</span></b-checkbox>
        
        <b-button outlined type="is-primary" @click="deleteSerie(i)">Delete</b-button>
        <b-button @click="exportSerie(i)">Export</b-button>
      </div>
      <div v-for="(s, i) in events" :key="s.name">
        <b-checkbox :id="i" @change="handleChange()" v-model="s.selected" />
        <label :for="s.name">{{s.name}}</label>
        <span :style="{backgroundColor: s.color, color: s.color}">___</span>
        <b-button @click="deleteEvents(i)">Delete</b-button>
        <b-button @click="exportEvents(i)">Export</b-button>
      </div>
    </section>
      </div>
      <div class="column">

    <section class="section">
     <div class="buttons">
        <b-button type="is-info" @click="rebase()">Rebase at Zero</b-button>
        <b-button type="is-info" @click="highCap()">Cap upper</b-button>
        <b-button type="is-info" @click="lowCap()">Cap lower</b-button>
        <b-button type="is-info" @click="smooth()">Smooth</b-button>
        <b-button type="is-info" @click="average()">Average</b-button>
        <b-button type="is-info" @click="sample()">Sample</b-button>
     </div>
    </section>
      <section>
        <div class="block">
        <b-field label="Base" label-position="on-border">
          <b-input id="base" type="text" v-model="base" />
        </b-field>
        <b-field label="Low" label-position="on-border">
          <b-input id="low" type="text" v-model="low" />
        </b-field>
        <b-field label="High" label-position="on-border">
          <b-input id="high" type="text" v-model="high" />
        </b-field>
        <b-field label="Window" label-position="on-border">
        <b-input id="win" type="text" v-model="window" />
        </b-field>
        </div>
      </section>
   
    </div>
    </div>
    <section>
      <div>{{exportedSerie}}</div>
    </section>
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
  data () {
    return {
      series: [],
      events: [],
      exportedSerie: '',
      paintSeries: [],
      low: -100,
      high: 100,
      base: 0,
      window: 5,
      eventRecord: 'start',
    }
  },
  methods: {
    save (serie) {
      this.series.push({ name: `S-${serie.color}`, ...serie, selected: true })
    },
    saveEvents (events) {
      console.log("save")
      this.events.push({ name: `E-${Math.random() * 99999}`, ...events, selected: true })
    },
    deleteSerie (i) {
      this.series = this.series.filter((v, idx) => { if (idx != i) { return true } else { return false } })
    },
    exportSerie (i) {
      console.log(this.series[i])

      this.exportedSerie = this.series[i].serie
      navigator.clipboard.writeText(JSON.stringify(this.exportedSerie))
    },
    deleteEvents (i) {
      this.events = this.events.filter((v, idx) => { if (idx != i) { return true } else { return false } })
    },
    exportEvents (i) {
      console.log(this.events[i])
      var toExport = {}
      this.events[i].data.forEach((x) => toExport[x.toString()] = 'EVENT')
      this.exportedSerie = toExport
      navigator.clipboard.writeText(JSON.stringify(this.exportedSerie))
    },
    sendClear () {
      EventBus.$emit('clear', {})
    },
    handleChange () {
      this.paintSeries = this.series.filter(s => s.selected)
    },
    rebase () {
      this.series.filter(s => s.selected).map(s => rebase(s, parseInt(this.base)))
      this.handleChange()
    },
    highCap () {
      this.series.filter(s => s.selected).map(s => highCap(s, parseInt(this.high)))
      this.handleChange()
    },
    lowCap () {
      this.series.filter(s => s.selected).map(s => lowCap(s, parseInt(this.low)))
      this.handleChange()
    },
    smooth () {
      this.series.filter(s => s.selected).map(s => smooth(s, parseInt(this.window)))
      this.handleChange()
    },
    sample () {
      this.series.filter(s => s.selected).map(s => sample(s, parseInt(this.window)))
      this.handleChange()
    },
    average () {
      this.series.filter(s => s.selected).map(s => average(s, parseInt(this.window)))
      this.handleChange()
    },
    emitEventRec () {
      EventBus.$emit(this.eventRecord + 'EventRecord', {})
      this.eventRecord = (this.eventRecord == 'start' ? 'stop' : 'start')
    }
  }
}

function rebase (s, base) {
  var min = s.serie.reduce((x, y) => x < y ? x : y)
  s.serie = s.serie.map(v => v - min + base)

}

function lowCap (s, low) {
  s.serie = s.serie.map(v => (v < low ? low : v))
}

function highCap (s, high) {
  s.serie = s.serie.map(v => (v > high ? high : v))
}

function smooth (s, w) {
  var a = s.serie.slice(0, w)
  s.serie = s.serie.map(v => { a.shift(); a.push(v); return a.reduce((x, y) => x + y) / a.length })
}


function sample (s, w) {
  s.serie = s.serie.filter((v, i) => i % w === 0)
}


function average (s, w) {
  var i = 0
  var new_s = []
  while (i < s.serie.length) {
    let a = s.serie.slice(i, i + w)
    new_s.push(a.reduce((x, y) => x + y) / w)
    i += w
  }

  s.serie = new_s
}


</script>

<style scoped>
#input {
  width: 500px;
}
</style>