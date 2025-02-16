<template>
  <div class="memory-game">
    <div class="game-header">
      <h2>Juego de Memoria</h2>
      <div class="stats">
        <p>Tiempo: {{ formatTime }}</p>
        <p>Movimientos: {{ moves }}</p>
      </div>
      <div class="controls">
        <button @click="startGame">Nuevo Juego</button>
        <button @click="endGame">Terminar</button>
        <button @click="downloadReport">Descargar Reporte</button>
        <button @click="mostrarHistorial = !mostrarHistorial">Ver Historial</button>
      </div>
    </div>

    <div class="cards-grid">
      <div v-for="(card, index) in cards" :key="index" class="card"
        :class="{ flipped: card.isFlipped, matched: card.isMatched }" @click="flipCard(index)">
        <div class="card-inner">
          <div class="card-front">{{ index + 1 }}</div>
          <div class="card-back">{{ card.emoji }}</div>
        </div>
      </div>
    </div>

    <div v-if="gameWon" class="winner-message">
      ¡Felicitaciones! Completaste el juego en {{ formatTime }}
      <button @click="closeMessage">Cerrar</button>
    </div>
    <div v-if="gameOverMessage" class="game-over-message">
      {{ gameOverMessage }}
      <button @click="closeMessage">Cerrar</button>
    </div>
    <div v-if="mostrarHistorial" class="historial">
      <h3>Historial de Partidas</h3>
      <ul>
        <li v-for="partida in partidas" :key="partida.date">
          Fecha: {{ partida.date }}, Movimientos: {{ partida.moves }}, Tiempo: {{ partida.time }}, Ganado: {{ partida.won ? 'Sí' : 'No' }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'JuegoMemoria',
  data() {
    return {
      emojis: [
        // Animales
        '🐶', '🐱', '🐼', '🐘',
        // Vehículos
        '🚗', '🚲', '🚁', '🚂',
        // Vehículos 2
        '🚎', '🚒', '🚦', '🚓',
        // Emoticons
        '🚍', '🚀', '🚜', '🚉'
      ],
      cards: [],
      flippedCards: [],
      moves: 0,
      timer: 0,
      gameActive: false,
      timerInterval: null,
      gameWon: false,
      gameOverMessage: '',
      showAllCardsTimeout: null,
      partidas: [],
      mostrarHistorial: false
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
      this.resetGame()
      this.shuffleCards()
      this.showAllCards()
    },
    resetGame() {
      this.cards = []
      this.flippedCards = []
      this.moves = 0
      this.timer = 0
      this.gameWon = false
      this.gameOverMessage = ''
      clearInterval(this.timerInterval)
      clearTimeout(this.showAllCardsTimeout)
    },
    shuffleCards() {
      const doubled = [...this.emojis, ...this.emojis]
      doubled.sort(() => Math.random() - 0.5)
      this.cards = doubled.map(emoji => ({
        emoji,
        isFlipped: false,
        isMatched: false
      }))
    },
    showAllCards() {
      this.cards.forEach(card => card.isFlipped = true)
      this.showAllCardsTimeout = setTimeout(() => {
        this.cards.forEach(card => card.isFlipped = false)
        this.gameActive = true
        this.startTimer()
      }, 5000)
    },
    flipCard(index) {
      if (!this.gameActive) return
      if (this.flippedCards.length === 2) return
      if (this.cards[index].isFlipped || this.cards[index].isMatched) return

      this.cards[index].isFlipped = true
      this.flippedCards.push(index)

      if (this.flippedCards.length === 2) {
        this.moves++
        this.checkMatch()
      }
    },
    checkMatch() {
      const [first, second] = this.flippedCards
      if (this.cards[first].emoji === this.cards[second].emoji) {
        this.cards[first].isMatched = true
        this.cards[second].isMatched = true
        this.flippedCards = []
        this.checkWin()
      } else {
        setTimeout(() => {
          this.cards[first].isFlipped = false
          this.cards[second].isFlipped = false
          this.flippedCards = []
        }, 1000)
      }
    },
    checkWin() {
      if (this.cards.every(card => card.isMatched)) {
        this.gameWon = true
        this.endGame()
      }
    },
    startTimer() {
      this.timerInterval = setInterval(() => {
        this.timer++
      }, 1000)
    },
    endGame() {
      this.gameActive = false
      clearInterval(this.timerInterval)
      if (!this.gameWon) {
        this.gameOverMessage = `Juego terminado. Realizaste ${this.moves} movimientos en ${this.formatTime}`
      }
      this.saveGame()
    },
    saveGame() {
      const partida = {
        moves: this.moves,
        time: this.formatTime,
        won: this.gameWon,
        date: new Date().toLocaleString()
      }
      this.partidas.push(partida)
      localStorage.setItem('partidas', JSON.stringify(this.partidas))
    },
    loadGames() {
      const partidas = localStorage.getItem('partidas')
      if (partidas) {
        this.partidas = JSON.parse(partidas)
      }
    },
    downloadReport() {
      const data = this.partidas.map(partida =>
        `Fecha: ${partida.date}, Movimientos: ${partida.moves}, Tiempo: ${partida.time}, Ganado: ${partida.won ? 'Sí' : 'No'}`
      ).join('\n')
      const blob = new Blob([data], { type: 'text/plain' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = 'reporte_partidas.txt'
      a.click()
      URL.revokeObjectURL(url)
    },
    closeMessage() {
      this.gameWon = false
      this.gameOverMessage = ''
    }
  },
  mounted() {
    this.loadGames()
  }
}
</script>

<style scoped>
.memory-game {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.game-header {
  text-align: center;
  margin-bottom: 20px;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin: 10px 0;
}

.controls {
  margin: 10px 0;
}

.controls button {
  margin: 0 10px;
  padding: 8px 16px;
  cursor: pointer;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 10px;
}

.card {
  aspect-ratio: 1;
  perspective: 1000px;
  cursor: pointer;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  text-align: center;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.card.flipped .card-inner {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5em;
  border: 2px solid #ccc;
  border-radius: 8px;
  background: #fff;
}

.card-back {
  transform: rotateY(180deg);
}

.card.matched .card-inner {
  background-color: #e8f5e9;
}

.winner-message,
.game-over-message {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(177, 177, 177, 0.9);
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgb(255, 0, 0);
  text-align: center;
  font-size: 1.5em;
}

.winner-message button,
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

h2 {
  background: linear-gradient(to left,
      #f58880,
      #f0c576,
      #ffff00,
      #ecf1a4,
      #f8bedb,
      #8cb8eb);
}
</style>