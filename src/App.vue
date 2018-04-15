<template>
  <div id="app">
    <b-card bg-variant="light" header="GUESS MYSTERY NOTES" class="text-center">
      <b-button lg @click="playRandom">PLAY</b-button>
      <p class="card-text">Score :  <span>{{score}}</span></p>
  
      <div id='notes-list'>
        <b-button-group>
          <template v-for="note in notes">
                  <b-button :style='{backgroundColor : note.color}' :key='note.name' @click='guessResult(note)'>{{note.name}} </b-button>
          </template>
      </b-button-group>
    </div>

    <h4> <b-badge id='result' :style='{backgroundColor : result.color}'>{{result.text}}</b-badge></h4>
    <div id="paper"></div>
    </b-card> 
  </div>
</template>

<script>
  import abcjs from "abcjs"
  import 'abcjs/abcjs-midi.css'
  import notesList from './notesList'
  
  const AudioSynth = require('audiosynth')
  const AudioContext = window.AudioContext || window.webkitAudioContext
  const context = new AudioContext()
  const synth = new AudioSynth(context)
  synth.setOscWave(3)
  
  
  export default {
    name: 'App',
    mounted: function() {
      let tune = "M:4/4\n||"
      abcjs.renderAbc("paper", tune, {});
    },
    data() {
      return {
        notes: notesList,
        playedNote: {},
        numNote: 0,
        errors: 0,
        score: 0,
        result: {
          text: '?',
          color: 'white'
        },
        tune: {
          notes: 'M:4/4\n||',
          count: 0
        }
      };
    },
    computed: {
      notesNames: function() {
        let array = []
        this.notes.forEach(note => array.push(note.name))
        return array
      }
    },
    methods: {
      playAll: function() {
        let self = this
        if (this.numNote < 8) {
          setTimeout(() => {
            let note = this.notes[this.numNote]
            synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
            this.playedNote = note
            self.numNote++
              self.playAll();
          }, 1000);
        }
      },
      playRandom: function() {
        this.result.color = 'white'
        this.result.text = '?'
        let randomNum = Math.floor(Math.random() * 8)
        let note = this.notes[randomNum]
        this.playedNote = note
        synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
      },
      guessResult: function(note) {
        console.log(this.playedNote)
        let indexOfGuessedNote = this.notes.indexOf(note)
        let indexOfPlayedNote = this.notes.indexOf(this.playedNote)
        if (indexOfGuessedNote > indexOfPlayedNote) {
          this.result.text = '<<<'
          this.result.color = 'red'
          this.errors ++
          if (this.errors > 2) this.endGame()
        } 
        else if (indexOfGuessedNote < indexOfPlayedNote) {
          this.result.text = '>>>'
          this.result.color = 'red'
          this.errors ++
          if (this.errors > 2) this.endGame()
        } 
        else {
          this.score += 10
          this.result.text = note.name
          this.result.color = 'green'
          if (this.tune.count % 4 === 0 && this.tune.count > 3) this.tune.notes += "|"
          if (this.tune.count % 12 === 0 && this.tune.count > 11) this.tune.notes += "\n"
          this.tune.notes += note.abcjs
          this.tune.count++
            abcjs.renderAbc("paper", this.tune.notes, {
              staffwidth: 600,
              responsive: "resize",
            });
          let self = this
          setTimeout(() => {
            self.playRandom()
          }, 1000);
  
        }
      },
      endGame: function () {
        this.tune.notes = ''
        this.tune.count = 0
        this.result.text = '?'
        this.result.color = 'white'
        let tune = "M:4/4\n||"
        abcjs.renderAbc("paper", tune, {})
        alert(`End game ! Your score : ${this.score}`)
        this.errors = 0
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
  }
  
  #notes-list {
    margin-top: 50px;
    margin-bottom: 50px;
  }
  
  #paper {
    max-width: 500px;
  }
</style>
