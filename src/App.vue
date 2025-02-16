<template>
  <div id="app">
    <img alt="Aiti logo" src="./assets/image4.jpg" class="logo">
    <HelloWorld msg="¡Apprende jugando con Aiti!" class="hello-world"/>
    <JuegoMemoria msg="Juego de Memoria" class="game"/>
    <JuegoSumas msg="Juego de Sumas" class="game"/>
    <JuegoEmojis msg="Juego de Emojis e idiomas" class="game"/>
    <ElAhorcado msg="Juego del Ahorcado" class="game"/>
    <!--<PintaBuses msg="Pinta Buses"/>-->
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import JuegoMemoria from './components/juegomemoria.vue';
import JuegoSumas from './components/juegosumas.vue';
import JuegoEmojis from './components/juegoingles.vue';
import ElAhorcado from './components/elahorcado.vue';
//import PintaBuses from './components/pintabuses.vue';

export default {
  name: 'App',
  components: {
    HelloWorld, JuegoMemoria, JuegoSumas, JuegoEmojis, ElAhorcado //PintaBuses
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.logo {
  width: 100%;
  max-width: 200px;
  height: auto;
  border-radius: 20%;
}

.hello-world, .game {
  background-color: lightcyan;
  font-size: 2em;
  margin: 10px 0;
}

@media (max-width: 600px) {
  .hello-world, .game {
    font-size: 1.5em;
    margin: 5px 0;
  }

  .logo {
    max-width: 150px;
  }
}
</style>