<template>
  <img alt="Vue logo" src="./assets/logon3.png" class="logoedit">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

<style scoped>
.logoedit {
  width: 300px; /* Ajusta el ancho según tu necesidad */
  height: auto; /* Mantén la proporción original */
}
</style>



<script>

import HelloWorld from './components/appTodo.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    
  }
}
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background: linear-gradient(135deg, rgba(244, 244, 244, 0.9), rgba(244, 244, 244, 0.9)); /* Degradado de color moderno */
  color: #ffffff; /* Color del texto para mayor contraste */
  padding-top: 60px;
  min-height: 100vh; /* Asegura que el contenedor ocupe toda la altura visible */
  position: relative; /* Necesario para efectos de overlay */
}

#app::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: inherit; /* Hereda el fondo del contenedor */
  filter: blur(5px); /* Efecto de desenfoque */
  z-index: -1; /* Coloca el desenfoque detrás del contenido */
}

</style>
