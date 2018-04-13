<template>
  <div id="app">
    <button @click="playAll">Jouer gamme</button>
    <button @click="playRandom">Démarrer le jeu</button>
    <h1>{{playedNote}}</h1>
    <div id='notes-list'>
      <template v-for="note in notesNames">
        <p class='note-choice' :key='note' @click='guessResult(note)'>{{note}} </p>
      </template>
    </div>
    <h2 id='result' :style='{backgroundColor : result.color}'>{{result.text}}</h2>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import notesList from './notesList'

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
        notes : notesList,
        playedNote : 'Placeholder note',
        numNote: 0,
        result: {text: '?', color: 'white'}
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
            synth.playNote(synth.noteToMIDI(note.letter, note.octave), 2.0, 1.0, 0)
            this.playedNote = note.name
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
        this.playedNote = note.name
        synth.playNote(synth.noteToMIDI(note.letter, note.octave), 2.0, 1.0, 0)
      },
      guessResult: function(note) {
        let indexOfGuessedNote = this.notesNames.indexOf(note)
        let indexOfPlayedNote = this.notesNames.indexOf(this.playedNote)
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
</style>
