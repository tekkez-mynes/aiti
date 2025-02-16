<template>
  <div class="container">
    <h2>Juego de Sumas</h2>
    <select v-model="nivel">
      <option value="1">Nivel 1 (5-6 años)</option>
      <option value="2">Nivel 2 (7-8 años)</option>
      <option value="3">Nivel 3 (9-10 años)</option>
      <option value="4">Nivel 4 (Desafío)</option>
      <option value="5">Nivel 5 (Experto)</option>
    </select>
    <div class="button-group">
      <button @click="iniciarJuego" class="start-button">Iniciar Juego</button>
      <button @click="terminarJuego">Terminar Juego</button>
      <button @click="descargarResultados">Descargar Resultados</button>
      <button @click="descargarHistorial">Descargar Historial</button>
    </div>
    <div v-if="!juegoIniciado">
      <p>Haz click en "iniciar juego" para comenzar a jugar</p>
    </div>
    <div v-else class="game-section">
      <p class="operation">¿Cuánto es {{ num1 }} + {{ num2 }}?</p>
      <input type="number" v-model="respuesta" class="response-input" />
      <button @click="comprobarRespuesta">Comprobar</button>
      <p v-if="mostrarFeedback && respuestaCorrecta">¡Correcto!</p>
      <p v-if="mostrarFeedback && !respuestaCorrecta">Incorrecto</p>
      <p>Tiempo de juego: {{ formatTime }}</p>
      <br>
      <ul v-if="juegoTerminado">
        <li v-for="resultado in resultados" :key="resultado.pregunta">
          {{ resultado.pregunta }} = {{ resultado.respuesta }} ({{ resultado.correcta ? 'Correcto' : 'Incorrecto' }}) -
          {{ resultado.tiempo }}s
        </li>
        <li>Porcentaje de aciertos: {{ porcentajeAciertos }}%</li>
      </ul>
    </div>
    <div v-if="juegoTerminado" class="game-over-message">
      <p>Juego Terminado</p>
      <button @click="cerrarMensaje">Cerrar</button>
    </div>
    <div v-if="juegoGanado" class="winner-message">
      <p>¡Felicidades! Has ganado el juego.</p>
      <button @click="cerrarMensaje">Cerrar</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'JuegoSumas',
  data() {
    return {
      nivel: '1',
      juegoIniciado: false,
      num1: 0,
      num2: 0,
      respuesta: '',
      respuestaCorrecta: false,
      resultado: 0,
      resultados: [],
      juegoTerminado: false,
      juegoGanado: false,
      historialResultados: [],
      timer: 0,
      timerInterval: null,
      mostrarFeedback: false
    }
  },
  created() {
    const savedHistorial = localStorage.getItem('historialResultados')
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
      this.generarNumeros()
      this.respuesta = ''
      this.respuestaCorrecta = false
      this.juegoTerminado = false
      this.juegoGanado = false
      this.resultados = []
      this.timer = 0
      this.mostrarFeedback = false
      clearInterval(this.timerInterval)
      this.timerInterval = setInterval(() => {
        this.timer++
      }, 1000)
    },
    generarNumeros() {
      const rango = this.obtenerRango()
      this.num1 = Math.floor(Math.random() * rango) + 1
      this.num2 = Math.floor(Math.random() * rango) + 1
    },
    obtenerRango() {
      switch (this.nivel) {
        case '1': return 10
        case '2': return 20
        case '3': return 50
        case '4': return 100
        case '5': return 200
        default: return 10
      }
    },
    comprobarRespuesta() {
      if (this.respuesta === '') {
        alert('Por favor, ingrese una respuesta.')
        return
      }
      this.resultado = this.num1 + this.num2
      this.respuestaCorrecta = parseInt(this.respuesta) === this.resultado
      const tiempoRespuesta = this.timer
      this.resultados.push({
        pregunta: `${this.num1} + ${this.num2}`,
        respuesta: this.respuesta,
        correcta: this.respuestaCorrecta,
        tiempo: tiempoRespuesta,
        nivel: this.nivel
      })
      this.mostrarFeedback = true
      if (this.resultados.length >= 10) {
        this.terminarJuego()
        this.juegoGanado = this.resultados.every(r => r.correcta)
      } else {
        this.generarNumeros()
        this.respuesta = ''
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
      localStorage.setItem('historialResultados', JSON.stringify(this.historialResultados))
    },
    cerrarMensaje() {
      this.juegoTerminado = false
      this.juegoGanado = false
    },
    descargarResultados() {
      const fecha = new Date().toLocaleString()
      const tiempoJuego = this.formatTime
      const porcentajeAciertos = this.porcentajeAciertos
      const nivelDificultad = this.nivel
      const resultadosTXT = `Fecha: ${fecha}\nTiempo de juego: ${tiempoJuego}\nPorcentaje de aciertos: ${porcentajeAciertos}%\nNivel de dificultad: ${nivelDificultad}\n\n` +
        this.resultados.map(resultado => `${resultado.pregunta} = ${resultado.respuesta} (${resultado.correcta ? 'Correcto' : 'Incorrecto'})`).join('\n')
      const blob = new Blob([resultadosTXT], { type: 'text/plain' })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.download = 'resultados.txt'
      link.click()
    },
    descargarHistorial() {
      const historialTXT = this.historialResultados.map(resultado =>
        `Fecha: ${resultado.fecha}, Pregunta: ${resultado.pregunta}, Respuesta: ${resultado.respuesta}, ` +
        `Correcta: ${resultado.correcta ? 'Sí' : 'No'}, Tiempo: ${resultado.tiempo}s, Nivel: ${resultado.nivel}`
      ).join('\n')
      const blob = new Blob([historialTXT], { type: 'text/plain' })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.download = 'historial_resultados.txt'
      link.click()
    }
  }
}
</script>

<style scoped>
.container {
  margin: 0 auto;
  max-width: 800px;
  min-height: 70vh;
  background: linear-gradient(to bottom,
      #f58880,
      #f0c576,
      #ffff00,
      #ecf1a4,
      #f8bedb,
      #8cb8eb);
  padding: 20px;
  box-sizing: border-box;
}

h1 {
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

.game-section .operation {
  font-size: 2em;
  color: #5e53fc;
  text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
}

.game-section .response-input {
  font-size: 1.5em;
  padding: 10px;
  border: 2px solid #5e53fc;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(95, 255, 2, 0.514);
}

@media (max-width: 600px) {
  .container {
    padding: 10px;
  }

  h2 {
    font-size: 1.5em;
  }

  .game-section .operation {
    font-size: 1.5em;
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