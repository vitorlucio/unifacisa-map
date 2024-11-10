<template>
  <div id="app">
    <!-- Adicione uma classe à imagem -->
    <img class="logo" alt="Vue logo" src="./assets/logo-teatro.jpg">
    <MapComponent msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import MapComponent from './components/MapComponent.vue'

export default {
  name: 'App',
  components: {
    MapComponent
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
  width: 140px; /* Ajusta o tamanho da imagem */
  height: auto; /* Mantém a proporção da imagem */
  display: block; /* Faz a imagem se comportar como bloco */
  margin: 0 auto; /* Centraliza horizontalmente */
}
</style>
