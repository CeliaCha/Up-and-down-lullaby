<template>
<div id="app">
    <b-card bg-variant="light" title="Devinez la note jouée par l'ordinateur" header="" class="text-center">

        <!-- Endgame screen -->
        <div v-if='displayEndGame' class="end-game">
            <p>Fin de partie !</p>
            <p>Votre score : {{score}}</p>
            <b-button @click='replay'>Rejouer toutes les notes</b-button>
            <b-button @click='resetGame'>OK</b-button>
        </div>

        <!-- Game screen -->
        <div v-else class='wrapper'>
            <b-button v-if="partyStarted === false" class='game-score' lg @click="startParty">NOUVELLE PARTIE</b-button>
            <b-button v-else class='game-score' lg>
                Score : <span>{{score}}</span> - Erreurs : <span>{{errors}}</span>
            </b-button>

            
            <div class='guess'>
                <!-- Notes to click -->
                <div id='notes-list'>
                    <transition-group name="flip-notes">
                        <template v-for="note in notes">
                            <span class='note-name' :key='note.name' @click='guessNote(note)' :style='{backgroundColor : note.color}'>{{note.name}} </span>
                          </template>
                    </transition-group>
                </div>

                <!--  Animation and message -->
                <div v-if="displaySonar === true" class="btn-sonar"></div>
                <div v-else class="btn-static">
                    <h4>
                        <b-badge v-if="message.text !==''" id='message' :style='{backgroundColor : message.color}'>{{message.text}}</b-badge>
                    </h4>
                </div>
            </div>

            <!-- Best scores board -->
            <div class='best-scores'>
                <h5>Meilleurs scores :</h5>
                <ol id='scores-list'>
                    <li v-for="score in bestScores" :key='score'> {{score}}</li>
                </ol>
            </div>
        </div>

        <div class='displaynotes' id="paper">
            <!-- Musical partition injected here -->
        </div>
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
    mounted: function () {
        if (localStorage.getItem('scoresList')) this.scoresList = JSON.parse(localStorage.getItem('scoresList'))
        abcjs.renderAbc("paper", "M:4/4\n||", {});
        this.shuffleNotes()
    },
    data() {
        return {
            notes: notesList,
            partyStarted: false,
            playedNote: {},
            playedNotes: [],
            timeStart: 0,
            errors: 0,
            score: 0,
            scoresList: [],
            countReplay: 0,
            displaySonar: false,
            displayEndGame: false,
            message: {
                text: '',
                color: ''
            },
            partition: {
                notes: 'M:4/4\n||',
                count: 0
            }
        }
    },
    computed: {
        bestScores: function () {
            let sortedScores = this.scoresList.sort((a, b) => b - a)
            return sortedScores.slice(0, 5)
        }
    },
    watch: {
        scoresList: {
            handler() {
                localStorage.setItem('scoresList', JSON.stringify(this.scoresList))
                deep: true
            }
        },
    },
    methods: {
        shuffleNotes: function () {
            this.notes = _.shuffle(this.notes)
        },
        startParty: function () {
            this.partyStarted = true
            this.playRandomNote()
        },
        playRandomNote: function () {
            this.timeStart = Date.now()
            this.message.color = ''
            this.message.text = ''
            let randomNum = Math.floor(Math.random() * 8)
            let note = this.notes[randomNum]
            this.playedNote = note
            this.playedNotes.push(note);
            this.displaySonar = true
            let self = this
            setTimeout(() => {
                self.displaySonar = false
            }, 1000)
            synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
        },
        guessNote: function (note) {
            synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
            let indexOfGuessedNote = notesList.indexOf(note)
            let indexOfPlayedNote = notesList.indexOf(this.playedNote)

            if (indexOfGuessedNote > indexOfPlayedNote) {
                this.message.text = 'Note plus basse'
                this.message.color = 'red'
                this.errors++
                    if (this.errors > 2) this.endGame()

            } else if (indexOfGuessedNote < indexOfPlayedNote) {
                this.message.text = 'Note plus haute'
                this.message.color = 'red'
                this.errors++
                    if (this.errors > 2) this.endGame()

            } else {
                // calculate bonus time and display message
                let timeToFind = Date.now() - this.timeStart
                let bonusTime = (50 - Math.round(timeToFind / 100))
                bonusTime > 0 ? this.score += 10 + bonusTime : this.score += 10
                this.message.text = 'Bravo !'
                this.message.color = 'green'

                // display new note in partition
                if (this.partition.count % 4 === 0 && this.partition.count > 3) this.partition.notes += "|"
                if (this.partition.count % 12 === 0 && this.partition.count > 11) this.partition.notes += "\n"
                this.partition.notes += note.abcjs
                this.partition.count++
                    abcjs.renderAbc("paper", this.partition.notes, {
                        staffwidth: 600,
                        responsive: "resize",
                    })

                // shuffle notes names and reload random note
                this.shuffleNotes()
                let self = this
                setTimeout(() => {
                    self.playRandomNote()
                }, 2500)
            }
        },
        endGame: function () {
            this.displayEndGame = true
            this.scoresList.push(this.score)
        },
        replay: function () {
            let self = this
            if (this.countReplay < this.playedNotes.length - 1) {
                setTimeout(() => {
                    let note = this.playedNotes[this.countReplay]
                    synth.playNote(synth.noteToMIDI(note.audio, note.octave), 2.0, 1.0, 0)
                    self.countReplay++
                        self.replay();
                }, 500);
            } else this.countReplay = 0
        },
        resetGame: function () {
            this.displayEndGame = false
            abcjs.renderAbc("paper", "M:4/4\n||", {})
            this.partition.notes = ''
            this.playedNotes = []
            this.partition.count = 0
            this.message.text = ''
            this.message.color = ''
            this.errors = 0
            this.score = 0
            this.partyStarted = false
        }
    }
}
</script>

<style lang="scss">
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    max-width: 600px !important;
    margin: auto;
}

.wrapper {
    display: grid;
    width: 550px;
    height: 400px;
    margin: auto;
    border: 1px solid black;
    grid-template-columns: 440px 110px;
    grid-template-rows: 50px 350px;
}

.game-score {
    grid-column: 1 / 3;
    grid-row: 1 / 2;
}

.best-scores {
    grid-column: 2 / 3;
    grid-row: 2 / 3;
    padding-top: 30px;
    background-color: rgba(2, 65, 49, 0.87);
    color: white;
}

.guess {
    grid-column: 1 / 2;
    grid-row: 2 / 3;
}

.end-game {
    width: 550px;
    height: 400px;
    margin: auto;
    border: 1px solid black;
}

#notes-list {
    margin: auto;
    display: flex;
    flex-wrap: wrap;
    width: 250px;
    margin-top: 50px;
    margin-bottom: 50px;
}

.note-name {
    cursor: pointer;
    display: inline-block;
    margin: 1px;
    width: 60px;
    height: 60px;
    line-height: 60px;
    border-radius: 5px;
}

#paper {
    max-width: 500px;
}

.flip-notes-move {
    transition: transform 1s;
}

/* SONAR EFFECT - adapted from https://codepen.io/tieppt/pen/vKJNaE : */

.btn-static {
    margin: 10px;
    border: 0;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    display: inline-block;
    outline: none;
    position: relative;
}

.btn-sonar {
    margin: 10px;
    background: cyan;
    border: 0;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    display: inline-block;
    outline: none;
    position: relative;
    &::before {
        content: '';
        display: inline-block;
        position: absolute;
        width: 100%;
        height: 100%;
        border-radius: 50%;
        top: 0;
        left: 0;
        animation: sonar-effect 1s ease-in-out .1s infinite;
    }
}

@keyframes sonar-effect {
    0% {
        opacity: 0.3;
    }
    40% {
        opacity: 0.5;
        box-shadow: 0 0 0 5px rgb(118, 201, 201), 0 0 10px 10px rgb(118, 201, 201), 0 0 0 10px rgb(118, 201, 201);
    }
    100% {
        box-shadow: 0 0 0 5px rgb(118, 201, 201), 0 0 10px 10px rgb(118, 201, 201), 0 0 0 10px rgb(118, 201, 201);
        transform: scale(1.5);
        opacity: 0;
    }
}
</style>
