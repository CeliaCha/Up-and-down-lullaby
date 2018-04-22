<template>
  <div id="app">
    <b-card bg-variant="light" header="GUESS MYSTERY NOTES" class="text-center">
      <b-button lg @click="playRandomNote">PLAY</b-button>
      <p>Score : <span>{{score}}</span></p>
      <p>Erreurs : <span>{{errors}}</span></p>
      <div id='scores-list'>
        <template v-for="score in scoresList">
                        <li  class='note-name' :key='score'>{{score}} </li>
</template>
      </div>
      <div id='notes-list'>
        <transition-group name="flip-notes">
<template v-for="note in notes">
  <span class='note-name' :key='note.name' @click='guessNote(note)' :style='{backgroundColor : note.color}'>{{note.name}} </span>
</template>
        </transition-group>
    </div>
    <h4> <b-badge v-if="result.text !=='?'" id='result' :style='{backgroundColor : result.color}'>{{result.text}}</b-badge></h4>
    <div id="paper"></div>
    </b-card> 
  </div>
</template>

<script>
  import abcjs from "abcjs"
  import 'abcjs/abcjs-midi.css'
  import notesList from './notesList'
  import 'sonar-css/css/sonar.css'
  
  const AudioSynth = require('audiosynth')
  const AudioContext = window.AudioContext || window.webkitAudioContext
  const context = new AudioContext()
  const synth = new AudioSynth(context)
  synth.setOscWave(3)
  
  
  export default {
    name: 'App',
    mounted: function() {
      if (localStorage.getItem('scoresList')) this.scoresList = JSON.parse(localStorage.getItem('scoresList'));
      let tune = "M:4/4\n||"
      abcjs.renderAbc("paper", tune, {});
      this.shuffleNotes()
    },
    data() {
      return {
        notes: notesList,
        playedNote: {},
        errors: 0,
        score: 0,
        scoresList: [],
        timeStart: 0,
        result: {
          text: '?',
          color: ''
        },
        tune: {
          notes: 'M:4/4\n||',
          count: 0
        }
      }
    },
    watch: {
      scoresList: {
        handler() {
          console.log(this.scoresList);
          localStorage.setItem('scoresList', JSON.stringify(this.scoresList))
          deep: true
        }
      },
    },
    computed: {
      notesNames: function() {
        let array = []
        this.notes.forEach(note => array.push(note.name))
        return array
      }
    },
    methods: {
      shuffleNotes: function() {
        this.notes = _.shuffle(this.notes)
      },
      playRandomNote: function() {
        this.timeStart = Date.now()
        this.result.color = 'white'
        this.result.text = '?'
        let randomNum = Math.floor(Math.random() * 8)
        let note = this.notes[randomNum]
        this.playedNote = note
        synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
      },
      guessNote: function(note) {
        synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
        let indexOfGuessedNote = notesList.indexOf(note)
        let indexOfPlayedNote = notesList.indexOf(this.playedNote)
        if (indexOfGuessedNote > indexOfPlayedNote) {
          this.result.text = 'Note plus basse'
          this.result.color = 'red'
          this.errors++
            if (this.errors > 2) this.endGame()
        } else if (indexOfGuessedNote < indexOfPlayedNote) {
          this.result.text = 'Note plus haute'
          this.result.color = 'red'
          this.errors++
            if (this.errors > 2) this.endGame()
        } else {
          let timeToFind = Date.now() - this.timeStart
          let bonusTime = (50 - Math.round(timeToFind / 100))
          bonusTime > 0 ? this.score += 10 + bonusTime : this.score += 10
          this.result.text = 'Bravo !'
          this.result.color = 'green'
          if (this.tune.count % 4 === 0 && this.tune.count > 3) this.tune.notes += "|"
          if (this.tune.count % 12 === 0 && this.tune.count > 11) this.tune.notes += "\n"
          this.tune.notes += note.abcjs
          this.tune.count++
            abcjs.renderAbc("paper", this.tune.notes, {
              staffwidth: 600,
              responsive: "resize",
            })
          let self = this
          this.shuffleNotes()
          setTimeout(() => {
            self.playRandomNote()
          }, 2500)
        }
      },
      endGame: function() {
        this.tune.notes = ''
        this.tune.count = 0
        this.result.text = '?'
        this.result.color = 'white'
        let tune = "M:4/4\n||"
        abcjs.renderAbc("paper", tune, {})
        alert(`End game ! Your score : ${this.score}`)
        this.errors = 0
        this.scoresList.push(this.score)
        this.score = 0
      }
    }
  }
</script>


<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    max-width: 600px !important;
    margin: auto;
  }
  
  #notes-list {
    margin: auto;
    display: flex;
    flex-wrap: wrap;
    width: 210px;
    margin-top: 50px;
    margin-bottom: 50px;
  }
  
  .note-name {
    cursor: pointer;
    display: inline-block;
    margin: 1px;
    width: 50px;
    height: 50px;
    line-height: 50px;
    border-radius: 5px;
  }
  
  #paper {
    max-width: 500px;
  }
  
  .flip-notes-move {
    transition: transform 1s;
  }
</style>
