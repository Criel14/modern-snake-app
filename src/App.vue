<template>
  <div id="app">
    <h1>Modern Snake</h1>
    <h2 class="subtitle">现代贪吃蛇</h2>
    <SnakeGame />
  </div>
</template>

<script>
import SnakeGame from './components/SnakeGame.vue'

export default {
  name: 'App',
  components: {
    SnakeGame
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  color: #fff;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  min-height: 100vh;
  overflow-y: auto;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px 0 40px;
  min-height: 100vh;
  text-align: center;
}

h1 {
  margin-top: 10px;
  margin-bottom: 0;
  font-weight: 700;
  font-size: 3rem;
  color: #4ecca3;
  text-shadow: 0 0 10px rgba(78, 204, 163, 0.5);
}

.subtitle {
  margin-top: 5px;
  margin-bottom: 20px;
  font-weight: 300;
  font-size: 1.5rem;
  color: #4ecca3;
  opacity: 0.8;
}
</style>
