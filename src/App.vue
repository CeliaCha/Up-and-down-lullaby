<template>
  <div id="app">
    <button @click="playAll">Jouer gamme</button>
    <button @click="playRandom">Démarrer le jeu</button>
    <!-- <h1>{{playedNote.name}}</h1> -->
    <div id='notes-list'>
      <template v-for="note in notes">
        <p class='note-choice' :key='note' @click='guessResult(note)'>{{note.name}} </p>
      </template>
    </div>
    <h2 id='result' :style='{backgroundColor : result.color}'>{{result.text}}</h2>
    <div id="paper"></div>
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
  mounted: function () {
      let tune = "|:              "
			abcjs.renderAbc("paper", tune, {});
		},
  data() {
      return {
        notes : notesList,
        playedNote : {},
        numNote: 0,
        result: { text: '?', color: 'white' },
        tune: { notes:'|:', count : 0 }
      };
    },
    computed : {
      notesNames : function () {
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
            self.numNote ++
            self.playAll();
          }, 1000);
        }
      },
      playRandom: function () {
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
        }
        else if (indexOfGuessedNote < indexOfPlayedNote) {
          this.result.text = '>>>'
          this.result.color = 'red'
        }
        else {
          this.result.text = note.name
          this.result.color = 'green'
          if (this.tune.count % 4 === 0 && this.tune.count !== 0) this.tune.notes += "|"
          if (this.tune.count % 8 === 0 && this.tune.count !== 0) this.tune.notes += "\n"
          this.tune.notes += note.abcjs
          this.tune.count ++
          abcjs.renderAbc("paper", this.tune.notes, {});
          let self = this
          setTimeout(() => {
            self.playRandom()
          }, 1000);

        }
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
  margin-top: 60px;
}
#result {
  margin: auto;
  width:100px;
}

#notes-list {
  display: flex;
  flex-wrap: wrap;
  width: 50%;
  margin: auto;
  margin-top: 50px;
  margin-bottom: 50px;
  
}
.note-choice {
  width: 40px;
  height: 40px;
  border: 1px solid black;
  margin : 5px;
  /* align-self: auto; */
}
.note-choice:hover {
  background-color: cyan;
  cursor: pointer;
}
#paper {
  max-width: 600px;
  margin:auto;
}
</style>
