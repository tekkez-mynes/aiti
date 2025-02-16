<template>
    <div class="hangman-game">
        <h2>Juego del Ahorcado</h2>
        <div class="controls">
            <select v-model="language">
                <option value="es">Español</option>
                <option value="en">Inglés</option>
            </select>
            <button @click="startGame" class="start-button">Iniciar Juego</button>
            <button @click="endGame">Terminar Juego</button>
            <button @click="downloadHistory">Descargar Historial</button>
        </div>
        <div class="emoji">{{ currentWord.emoji }}</div>
        <div class="word">
            <span v-for="(letter, index) in displayedWord" :key="index">{{ letter }}</span>
        </div>
        <div v-if="gameActive" class="letters">
            <span v-for="letter in alphabet" :key="letter" @click="guessLetter(letter)"
                :class="{ guessed: guessedLetters.includes(letter) }">{{ letter }}</span>
        </div>
        <div class="status">
            <p>Vidas restantes: {{ remainingLives }}</p>
            <p>Letras adivinadas: {{ guessedLetters.join(' ') }}</p>
            <p>Tiempo de juego: {{ formatTime }}</p>
        </div>
        <div v-if="gameOver" class="game-over-message">
            <p>{{ gameOverMessage }}</p>
            <button @click="cerrarMensaje">Cerrar</button>
        </div>
        <div v-if="mostrarHistorial" class="historial">
            <h3>Historial de Partidas</h3>
            <ul>
                <li v-for="(partida, index) in historial" :key="index">
                    Fecha: {{ partida.date }}, Palabra: {{ partida.word }}, Ganado: {{ partida.won ? 'Sí' : 'No' }},
                    Tiempo: {{ partida.time }}
                    <ul>
                        <li v-for="(evento, eventIndex) in partida.events" :key="eventIndex + '-' + evento.letter">
                            Letra: {{ evento.letter }}, Correcta: {{ evento.correcta ? 'Sí' : 'No' }}
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ElAhorcado',
    data() {
        return {
            words: {
                es: [
                    { word: 'manzana', emoji: '🍎' },
                    { word: 'auto', emoji: '🚗' },
                    { word: 'perro', emoji: '🐶' },
                    {word: 'gato', emoji: '🐱'},
                    {word: 'casa', emoji: '🏠'},
                    {word: 'sol', emoji: '☀️'},
                    {word: 'luna', emoji: '🌙'},
                    {word: 'flor', emoji: '🌸'},
                    {word: 'tren', emoji: '🚂'},
                    {word: 'avión', emoji: '✈️'},
                    {word: 'barco', emoji: '🚢'},
                    {word: 'bicicleta', emoji: '🚲'},
                    {word: 'computadora', emoji: '💻'},
                    {word: 'celular', emoji: '📱'},
                    {word: 'televisión', emoji: '📺'},
                    {word: 'radio', emoji: '📻'},
                    {word: 'libros', emoji: '📚'},
                    {word: 'cuaderno', emoji: '📒'},
                    {word: 'lápiz', emoji: '✏️'},
                    {word: 'ambulancia', emoji: '🚑'},
                    {word: 'bombero', emoji: '👨‍🚒'},
                    {word: 'doctor', emoji: '👨‍⚕️'},
                    {word: 'enfermera', emoji: '👩‍⚕️'},
                    {word: 'profesor', emoji: '👨‍🏫'},
                    {word: 'policía', emoji: '👮‍♂️'},
                    {word: 'cocinero', emoji: '👨‍🍳'},
                    {word: 'mecánico', emoji: '👨‍🔧'},
                    {word: 'pintor', emoji: '👨‍🎨'},
                    {word: 'música', emoji: '🎵'},
                    {word: 'carrito de compras', emoji: '🛒'},
                    {word: 'bandera', emoji: '🚩'},
                    {word: 'fuego', emoji: '🔥'},
                    {word: 'agua', emoji: '💧'},
                    {word: 'planeeta tierra', emoji: '🌍'},
                    {word: 'ampolleta', emoji: '💡'},
                    {word: 'reloj', emoji: '⏰'},
                    {word: 'teléfono', emoji: '☎️'},
                    {word: 'corazón', emoji: '❤️'},
                    {word: 'estrella', emoji: '⭐'},
                    {word: 'pizza', emoji: '🍕'},
                    {word: 'hamburguesa', emoji: '🍔'},
                    {word: 'papas fritas', emoji: '🍟'},
                    {word: 'helado', emoji: '🍦'},
                    {word: 'pastel', emoji: '🍰'},
                    {word: 'galleta', emoji: '🍪'},
                    {word: 'chocolate', emoji: '🍫'},
                    {word: 'dulces', emoji: '🍬'},
                    {word: 'refresco', emoji: '🥤'},
                    {word: 'café', emoji: '☕'},
                    {word: 'cara sonriente con lentes de sol', emoji: '😎'},
                    {word: 'cara sonriente con corazones en los ojos', emoji: '😍'},
                    {word: 'carcajadas', emoji: '😂'},
                    {word: 'confundido', emoji: '😕'},
                    {word: 'enfermo', emoji: '🤢'},
                    {word: 'dormido', emoji: '😴'},
                    {word: 'enojado', emoji: '😡'},
                    {word: 'asustado', emoji: '😱'},
                    {word: 'sorprendido', emoji: '😲'},
                    {word: 'cara sonriente', emoji: '🙂'},
                    {word: 'cara triste', emoji: '🙁'},
                    {word: 'cara llorando', emoji: '😢'},
                    {word: 'cara enojada', emoji: '😠'},
                    
                    // ...more words...
                ],
                en: [
                    { word: 'apple', emoji: '🍎' },
                    { word: 'car', emoji: '🚗' },
                    { word: 'dog', emoji: '🐶' },
                    // ...more words...
                ]
            },
            alphabet: 'abcdefghijklmnopqrstuvwxyz'.split(''),
            language: 'es',
            currentWord: {},
            displayedWord: [],
            guessedLetters: [],
            remainingLives: 5,
            gameOver: false,
            gameOverMessage: '',
            timer: 0,
            timerInterval: null,
            historial: [],
            mostrarHistorial: false,
            gameActive: false,
            correctWords: 0,
            events: [],
            usedWords: []
        }
    },
    computed: {
        formatTime() {
            const minutes = Math.floor(this.timer / 60)
            const seconds = this.timer % 60
            return `${minutes}:${seconds.toString().padStart(2, '0')}`
        }
    },
    methods: {
        startGame() {
            this.correctWords = 0
            this.usedWords = []
            this.nextWord()
            this.remainingLives = 5
            this.gameOver = false
            this.gameOverMessage = ''
            this.timer = 0
            this.gameActive = true
            this.events = []
            clearInterval(this.timerInterval)
            this.timerInterval = setInterval(() => {
                this.timer++
            }, 1000)
        },
        nextWord() {
            let randomIndex
            do {
                randomIndex = Math.floor(Math.random() * this.words[this.language].length)
            } while (this.usedWords.includes(this.words[this.language][randomIndex].word))

            this.currentWord = this.words[this.language][randomIndex]
            this.usedWords.push(this.currentWord.word)
            this.displayedWord = this.currentWord.word.split('').map((letter, index) => (index === 0 || index === this.currentWord.word.length - 1 || letter === ' ') ? letter : '_')
            this.guessedLetters = []
        },
        guessLetter(letter) {
            if (this.guessedLetters.includes(letter) || this.gameOver) return

            this.guessedLetters.push(letter)

            let correct = false
            if (this.language === 'es') {
                const normalizedWord = this.currentWord.word.normalize('NFD').replace(/[\u0300-\u036f]/g, '')
                correct = normalizedWord.includes(letter)
            } else {
                correct = this.currentWord.word.includes(letter)
            }
            
            this.events.push({ letter, correcta: correct })

            if (correct) {
                for (let i = 0; i < this.currentWord.word.length; i++) {
                    if (this.currentWord.word[i].normalize('NFD').replace(/[\u0300-\u036f]/g, '') === letter) {
                        this.displayedWord[i] = this.currentWord.word[i]
                    }
                }
                if (!this.displayedWord.includes('_')) {
                    this.correctWords++
                    if (this.correctWords >= 10) {
                        this.gameOver = true
                        this.gameOverMessage = '¡Felicidades! Has ganado.'
                        clearInterval(this.timerInterval)
                        this.saveGame(true)
                    } else {
                        setTimeout(() => {
                            this.nextWord()
                        }, 1000)
                    }
                }
            } else {
                this.remainingLives--
                if (this.remainingLives === 0) {
                    this.gameOver = true
                    this.gameOverMessage = `Juego terminado. La palabra era: ${this.currentWord.word}`
                    clearInterval(this.timerInterval)
                    this.saveGame(false)
                }
            }
        },
        endGame() {
            this.gameOver = true
            clearInterval(this.timerInterval)
            this.gameOverMessage = `Juego terminado. La palabra era: ${this.currentWord.word}`
            this.saveGame(false)
        },
        saveGame(won) {
            const partida = {
                word: this.currentWord.word,
                won: won,
                time: this.formatTime,
                date: new Date().toLocaleString(),
                events: this.events
            }
            this.historial.push(partida)
            localStorage.setItem('historialAhorcado', JSON.stringify(this.historial))
        },
        loadHistory() {
            const savedHistorial = localStorage.getItem('historialAhorcado')
            if (savedHistorial) {
                this.historial = JSON.parse(savedHistorial)
            }
        },
        downloadHistory() {
            const historialTXT = this.historial.map(partida =>
                `Fecha: ${partida.date}, Palabra: ${partida.word}, Ganado: ${partida.won ? 'Sí' : 'No'}, Tiempo: ${partida.time}\n` +
                partida.events.map(evento => `  Letra: ${evento.letter}, Correcta: ${evento.correcta ? 'Sí' : 'No'}`).join('\n')
            ).join('\n\n')
            const blob = new Blob([historialTXT], { type: 'text/plain' })
            const link = document.createElement('a')
            link.href = URL.createObjectURL(blob)
            link.download = 'historial_ahorcado.txt'
            link.click()
        },
        cerrarMensaje() {
            this.gameOver = false
        }
    },
    mounted() {
        this.loadHistory()
    }
}
</script>

<style scoped>
.hangman-game {
    text-align: center;
    max-width: 800px;
    margin: 0 auto;
    padding: 0px;
    background: linear-gradient(to bottom,
            #f58880,
            #f0c576,
            #ffff00,
            #ecf1a4,
            #f8bedb,
            #8cb8eb);
}

h2 {
    background: linear-gradient(to left, #f58880, #f0c576, #ffff00, #ecf1a4, #f8bedb, #8cb8eb);
    padding: 0;
    margin: 0;
}

.controls {
    margin-bottom: 20px;
}

.controls button {
    margin: 0 10px;
    padding: 10px 20px;
    cursor: pointer;
}

.emoji {
    font-size: 5em;
    margin-bottom: 20px;
}

.word {
    font-size: 2em;
    margin-bottom: 20px;
}

.letters {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 10px;
    margin-bottom: 20px;
}

.letters span {
    font-size: 1.5em;
    cursor: pointer;
    display: inline-block;
    width: 40px;
    height: 40px;
    line-height: 40px;
    border-radius: 50%;
    background-color: #f0c576;
    text-align: center;
    margin: 5px;
}

.letters span.guessed {
    text-decoration: line-through;
    color: #ccc;
}

.status {
    margin-bottom: 20px;
}

.game-over-message {
    background-color: rgba(0, 0, 0, 0.8);
    color: white;
    padding: 20px;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1000;
    text-align: center;
}

.game-over-message button {
    margin-top: 10px;
    padding: 5px 10px;
    cursor: pointer;
}

.historial {
    margin-top: 20px;
}

.historial h3 {
    margin-bottom: 10px;
}

.historial ul {
    list-style-type: none;
    padding: 0;
}

.historial li {
    margin: 5px 0;
    background: #fff;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}
</style>
