<template>
  <div id="app">
    <button @click="playRandom">Start</button>
    <!-- <h1>{{playedNote}}</h1> -->
    <div id='notes-list'>
      <template v-for="note in frenchNotes">
        <p class='note-choice' :key='note' @click='guessResult(note)'>{{note}} </p>
      </template>
    </div>
    <h2 id='result' :style='{backgroundColor : result.color}'>{{result.text}}</h2>
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
        numNote: 0,
        result: {text: '?', color: 'white'}
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
        this.result.color = 'white'
        this.result.text = '?'
        let octave;
        let randomNum = Math.floor(Math.random() * 7)
        randomNum < 5 ? octave = 4 : octave = 5
        console.log(octave)
        let randomNote = this.notes[randomNum]
        let frenchNote = this.frenchNotes[randomNum]
        this.playedNote = frenchNote
        synth.playNote(synth.noteToMIDI(randomNote, octave), 2.0, 1.0, 0)
      },
      guessResult: function(note) {
        let indexOfGuessedNote = this.frenchNotes.indexOf(note)
        let indexOfPlayedNote = this.frenchNotes.indexOf(this.playedNote)
        if (indexOfGuessedNote > indexOfPlayedNote) {
          this.result.text = '<<<'
          this.result.color = 'red'
        }
        else if (indexOfGuessedNote < indexOfPlayedNote) {
          this.result.text = '>>>'
          this.result.color = 'red'
        }
        else {
          this.result.text = note
          this.result.color = 'green'
          let self = this
          setTimeout(() => {
            self.playRandom()
          }, 2000);

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
  /* margin-top: 50px; */
  width: 50%;
  margin: auto;
  margin-top: 50px;
  margin-bottom: 50px;
  display: flex;
  flex-direction: row;
}
.note-choice {
  width: 40px;
  height: 40px;
  border: 1px solid black;
  margin : 5px;
  /* align-self: auto; */
}

.note-choice:hover {
  background-color: grey;
}
</style>
