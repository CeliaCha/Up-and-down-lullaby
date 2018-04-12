<template>
  <div id="app">
    <button @click="playAll">Jouer une note</button>
    <h1>{{playedNote}}</h1>
    <template v-for="note in frenchNotes">
      <span :key='note' @click='guessResult(note)'>{{note}} </span>
    </template>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'

const AudioSynth = require('audiosynth');
const AudioContext = window.AudioContext || window.webkitAudioContext;
const context = new AudioContext();
const synth = new AudioSynth(context);
synth.setOscWave(3); 

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  data() {
      return {
        notes : ['C','D','E','F','G','A','B'],
        frenchNotes : ['do','ré','mi','fa','sol','la','si'],
        playedNote : 'Placeholder note',
        numNote: 0
      };
    },
    methods: {
      playAll: function() {
        let self = this
        let octave;
        if (this.numNote < 7) {
          this.numNote < 5 ? octave = 4 : octave = 5
          setTimeout(() => {
            synth.playNote(synth.noteToMIDI(self.notes[self.numNote], octave), 2.0, 1.0, 0)
            self.playedNote = self.frenchNotes[self.numNote]
            self.numNote ++
            self.playAll();
          }, 1000);
        }
      },
      playRandom: function() {
        let randomNum = Math.floor(Math.random() * 7)
        let randomNote = this.notes[randomNum]
        let frenchNote = this.frenchNotes[randomNum]
        this.playedNote = frenchNote
        synth.playNote(synth.noteToMIDI(randomNote, 4), 2.0, 1.0, 0)
      },
      guessResult: function(note) {
        let indexOfGuessedNote = this.frenchNotes.indexOf(note)
        let indexOfPlayedNote = this.frenchNotes.indexOf(this.playedNote)
        if (indexOfGuessedNote > indexOfPlayedNote) {
          console.log('Note trop haute')
        }
        else if (indexOfGuessedNote < indexOfPlayedNote) {
          console.log('Note trop basse')
        }
        else {
          console.log(`Bravo, il s'agit bien d'un ${note} !`)
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
</style>
