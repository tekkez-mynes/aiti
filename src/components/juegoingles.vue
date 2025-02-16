<template>
    <div class="container">
        <h2>Juego Inglés / Español</h2>
        <div class="button-group">
            <button @click="iniciarJuego" class="start-button">Iniciar Juego</button>
            <button @click="terminarJuego">Terminar Juego</button>
            <button @click="descargarHistorial">Descargar Historial</button>
            <button @click="mostrarHistorial = !mostrarHistorial">Ver Historial</button>
        </div>
        <div v-if="!juegoIniciado">
            <p>Haz click en "iniciar juego" para comenzar a jugar</p>
        </div>
        <div v-else class="game-section">
            <p class="emoji">{{ emojiActual.emoji }}</p>
            <label for="respuestaIngles">Nombre en inglés:</label>
            <input type="text" v-model="respuestaIngles" id="respuestaIngles" class="response-input" />
            <label for="respuestaEspanol">Nombre en español:</label>
            <input type="text" v-model="respuestaEspanol" id="respuestaEspanol" class="response-input" />
            <button @click="comprobarRespuesta">Comprobar</button>
            <p v-if="mostrarFeedback && respuestaCorrecta">¡Correcto!</p>
            <p v-if="mostrarFeedback && !respuestaCorrecta">Incorrecto</p>
            <p>Tiempo de juego: {{ formatTime }}</p>
            <br>
            <ul v-if="juegoTerminado">
                <li v-for="resultado in resultados" :key="resultado.emoji">
                    {{ resultado.emoji }} - Inglés: {{ resultado.ingles }} - Español: {{ resultado.espanol }} ({{
                        resultado.correcta ? 'Correcto' : 'Incorrecto' }})
                </li>
                <li>Porcentaje de aciertos: {{ porcentajeAciertos }}%</li>
            </ul>
        </div>
        <div v-if="juegoTerminado" class="game-over-message">
            <p>Juego Terminado</p>
            <button @click="cerrarMensaje">Cerrar</button>
        </div>
        <div v-if="mostrarHistorial" class="historial">
            <h3>Historial de Resultados</h3>
            <ul>
                <li v-for="resultado in historialResultados" :key="resultado.fecha">
                    Fecha: {{ resultado.fecha }}, Emoji: {{ resultado.emoji }}, Inglés: {{ resultado.ingles }}, Español:
                    {{ resultado.espanol }}, Correcta: {{ resultado.correcta ? 'Sí' : 'No' }}
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
export default {
    name: 'JuegoEmojis',
    data() {
        return {
            emojis: [
                { emoji: '🍎', ingles: 'apple', espanol: 'manzana' },
                { emoji: '🚗', ingles: 'car', espanol: 'auto' },
                { emoji: '🐶', ingles: 'dog', espanol: 'perro' },
                { emoji: '🍇', ingles: 'grapes', espanol: 'uvas' },
                { emoji: '🍔', ingles: 'burguer', espanol: 'hamburguesa' },
                { emoji: '🍟', ingles: 'fries', espanol: 'papas fritas' },
                { emoji: '🚀', ingles: 'rocket', espanol: 'cohete' },
                { emoji: '🌈', ingles: 'rainbow', espanol: 'arcoiris' },
                { emoji: '🦄', ingles: 'unicorn', espanol: 'unicornio' },
                { emoji: '🎈', ingles: 'balloon', espanol: 'globo' },
                { emoji: '🎉', ingles: 'party', espanol: 'fiesta' },
                { emoji: '🎁', ingles: 'gift', espanol: 'regalo' },
                { emoji: '🎓', ingles: 'graduation', espanol: 'graduación' },
                { emoji: '🎄', ingles: 'christmas', espanol: 'navidad' },
                { emoji: '🎃', ingles: 'halloween', espanol: 'halloween' },
                { emoji: '🍩', ingles: 'donut', espanol: 'dona' },
                { emoji: '🍦', ingles: 'ice cream', espanol: 'helado' },
                { emoji: '🍭', ingles: 'lollipop', espanol: 'paleta' },
                { emoji: '🍪', ingles: 'cookie', espanol: 'galleta' },
                { emoji: '🍫', ingles: 'chocolate', espanol: 'chocolate' },
                { emoji: '🍬', ingles: 'candy', espanol: 'caramelo' },
                { emoji: '🍸', ingles: 'cocktail', espanol: 'cóctel' },
                { emoji: '🍺', ingles: 'beer', espanol: 'cerveza' },
                { emoji: '🍷', ingles: 'wine', espanol: 'vino' },
                { emoji: '🍹', ingles: 'drink', espanol: 'bebida' },
                { emoji: '🍾', ingles: 'champagne', espanol: 'champaña' },
                { emoji: '🍴', ingles: 'cutlery', espanol: 'cubiertos' },
                { emoji: '🍽', ingles: 'plate', espanol: 'plato' },
                { emoji: '🍳', ingles: 'egg', espanol: 'huevo' },
                { emoji: '🍲', ingles: 'soup', espanol: 'sopa' },
                { emoji: '🍜', ingles: 'noodles', espanol: 'fideos' },
                { emoji: '🍛', ingles: 'curry', espanol: 'curry' },
                { emoji: '🍙', ingles: 'rice ball', espanol: 'bola de arroz' },
                { emoji: '🍚', ingles: 'rice', espanol: 'arroz' },
                { emoji: '🍘', ingles: 'rice cracker', espanol: 'galleta de arroz' },
                { emoji: '🍨', ingles: 'ice cream', espanol: 'helado' },
                { emoji: '🍰', ingles: 'cake', espanol: 'pastel' },
                { emoji: '🎶', ingles: 'music', espanol: 'música' },
                { emoji: '🎵', ingles: 'music', espanol: 'música' },
                { emoji: '🎼', ingles: 'music', espanol: 'música' },
                { emoji: '🎤', ingles: 'microphone', espanol: 'micrófono' },
                { emoji: '🎧', ingles: 'headphones', espanol: 'audífonos' },
                { emoji: '🎺', ingles: 'trumpet', espanol: 'trompeta' },
                { emoji: '🎻', ingles: 'violin', espanol: 'violín' },
                { emoji: '🥁', ingles: 'drum', espanol: 'tambor' },
                { emoji: '🎸', ingles: 'guitar', espanol: 'guitarra' },
                { emoji: '🎹', ingles: 'piano', espanol: 'piano' },
                { emoji: '🎮', ingles: 'videogame', espanol: 'videojuego' },
                { emoji: '🎲', ingles: 'dice', espanol: 'dado' },
                { emoji: '🎯', ingles: 'target', espanol: 'objetivo' },
                { emoji: '🎳', ingles: 'bowling', espanol: 'boliche' },
                { emoji: '🎰', ingles: 'slot machine', espanol: 'tragamonedas' },
                { emoji: '🎱', ingles: 'pool', espanol: 'billar' },
                { emoji: '🎫', ingles: 'ticket', espanol: 'boleto' },
                { emoji: '🎟', ingles: 'ticket', espanol: 'boleto' },
                { emoji: '🎚', ingles: 'level', espanol: 'nivel' },
                { emoji: '🎛', ingles: 'control', espanol: 'control' },
                { emoji: '🎥', ingles: 'camera', espanol: 'cámara' },
                { emoji: '📽', ingles: 'projector', espanol: 'proyector' },
                { emoji: '📺', ingles: 'tv', espanol: 'televisión' },
                { emoji: '📷', ingles: 'camera', espanol: 'cámara' },
                { emoji: '📸', ingles: 'camera', espanol: 'cámara' },
                { emoji: '📹', ingles: 'camera', espanol: 'cámara' },
                { emoji: '📼', ingles: 'videotape', espanol: 'videocinta' },
                { emoji: '🔈', ingles: 'speaker', espanol: 'altavoz' },
                { emoji: '🔉', ingles: 'speaker', espanol: 'altavoz' },
                { emoji: '🔊', ingles: 'speaker', espanol: 'altavoz' },
                { emoji: '🔇', ingles: 'mute', espanol: 'silencio' },
                { emoji: '🔔', ingles: 'bell', espanol: 'campana' },
                { emoji: '🔕', ingles: 'bell', espanol: 'campana' },
                { emoji: '📢', ingles: 'loudspeaker', espanol: 'altavoz' },
                { emoji: '📣', ingles: 'megaphone', espanol: 'megáfono' },
                { emoji: '📯', ingles: 'horn', espanol: 'trompeta' },
                { emoji: '🔔', ingles: 'bell', espanol: 'campana' },
                { emoji: '🔕', ingles: 'bell', espanol: 'campana' },
                { emoji: '🔍', ingles: 'magnifying glass', espanol: 'lupa' },
                { emoji: '🔎', ingles: 'magnifying glass', espanol: 'lupa' },
                { emoji: '🔬', ingles: 'microscope', espanol: 'microscopio' },
                { emoji: '🔭', ingles: 'telescope', espanol: 'telescopio' },
                { emoji: '📡', ingles: 'satellite', espanol: 'satélite' },
                // ...more emojis...
            ],
            juegoIniciado: false,
            emojiActual: {},
            respuestaIngles: '',
            respuestaEspanol: '',
            respuestaCorrecta: false,
            resultados: [],
            juegoTerminado: false,
            historialResultados: [],
            timer: 0,
            timerInterval: null,
            mostrarFeedback: false,
            mostrarHistorial: false
        }
    },
    created() {
        const savedHistorial = localStorage.getItem('historialResultadosEmojis')
        if (savedHistorial) {
            this.historialResultados = JSON.parse(savedHistorial)
        }
    },
    computed: {
        formatTime() {
            const minutes = Math.floor(this.timer / 60)
            const seconds = this.timer % 60
            return `${minutes}:${seconds.toString().padStart(2, '0')}`
        },
        porcentajeAciertos() {
            const correctas = this.resultados.filter(r => r.correcta).length
            return ((correctas / this.resultados.length) * 100).toFixed(2)
        }
    },
    methods: {
        iniciarJuego() {
            this.juegoIniciado = true
            this.siguienteEmoji()
            this.respuestaIngles = ''
            this.respuestaEspanol = ''
            this.respuestaCorrecta = false
            this.juegoTerminado = false
            this.resultados = []
            this.timer = 0
            this.mostrarFeedback = false
            clearInterval(this.timerInterval)
            this.timerInterval = setInterval(() => {
                this.timer++
            }, 1000)
        },
        siguienteEmoji() {
            const indice = Math.floor(Math.random() * this.emojis.length)
            this.emojiActual = this.emojis[indice]
        },
        comprobarRespuesta() {
            if (this.respuestaIngles === '' || this.respuestaEspanol === '') {
                alert('Por favor, ingrese ambas respuestas.')
                return
            }
            this.respuestaCorrecta = this.respuestaIngles.toLowerCase() === this.emojiActual.ingles && this.respuestaEspanol.toLowerCase() === this.emojiActual.espanol
            this.resultados.push({
                emoji: this.emojiActual.emoji,
                ingles: this.respuestaIngles,
                espanol: this.respuestaEspanol,
                correcta: this.respuestaCorrecta
            })
            this.mostrarFeedback = true
            if (this.resultados.length >= 10) {
                this.terminarJuego()
            } else {
                this.siguienteEmoji()
                this.respuestaIngles = ''
                this.respuestaEspanol = ''
            }
        },
        terminarJuego() {
            this.juegoTerminado = true
            this.juegoIniciado = false
            clearInterval(this.timerInterval)
            this.historialResultados.push(...this.resultados.map(r => ({
                ...r,
                fecha: new Date().toLocaleString()
            })))
            localStorage.setItem('historialResultadosEmojis', JSON.stringify(this.historialResultados))
        },
        cerrarMensaje() {
            this.juegoTerminado = false
        },
        descargarHistorial() {
            const historialTXT = this.historialResultados.map(resultado =>
                `Fecha: ${resultado.fecha}, Emoji: ${resultado.emoji}, Inglés: ${resultado.ingles}, Español: ${resultado.espanol}, ` +
                `Correcta: ${resultado.correcta ? 'Sí' : 'No'}`
            ).join('\n')
            const blob = new Blob([historialTXT], { type: 'text/plain' })
            const link = document.createElement('a')
            link.href = URL.createObjectURL(blob)
            link.download = 'historial_resultados_emojis.txt'
            link.click()
        }
    }
}
</script>

<style scoped>
.container {
    text-align: center;
    min-height: 100vh;
    background: linear-gradient(to bottom,
            #f58880,
            #f0c576,
            #ffff00,
            #ecf1a4,
            #f8bedb,
            #8cb8eb);
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    box-sizing: border-box;
}

h2 {
    margin-bottom: 20px;
}

select,
button,
input {
    display: block;
    margin: 10px auto;
    padding: 10px;
    font-size: 16px;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    margin: 5px 0;
}

button {
    cursor: pointer;
}

button:hover {
    background-color: #ddd;
}

div>button {
    display: inline-block;
    margin: 10px 5px;
}

.button-group {
    text-align: center;
}

.start-button {
    box-shadow: 0 0 10px 2px rgba(1, 255, 35, 0.911);
}

h2 {
    background: linear-gradient(to left,
            #f58880,
            #f0c576,
            #ffff00,
            #ecf1a4,
            #f8bedb,
            #8cb8eb);
}

.winner-message,
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

.winner-message button,
.game-over-message button {
    margin-top: 10px;
    padding: 5px 10px;
    cursor: pointer;
}

.game-section .emoji {
    font-size: 7em;
    margin: 20px 0;
}

.game-section .response-input {
    font-size: 1.5em;
    padding: 10px;
    border: 2px solid #5e53fc;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(95, 255, 2, 0.514);
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

@media (max-width: 600px) {
    .container {
        padding: 10px;
    }

    h2 {
        font-size: 1.5em;
    }

    .game-section .emoji {
        font-size: 5em;
    }

    .game-section .response-input {
        font-size: 1.2em;
        padding: 8px;
    }

    select,
    button,
    input {
        font-size: 14px;
        padding: 8px;
    }

    .winner-message,
    .game-over-message {
        padding: 15px;
    }

    .winner-message button,
    .game-over-message button {
        padding: 5px 8px;
    }
}
</style>