<template>
  <div id="app">
    <h1>Modern Snake</h1>
    <h2 class="subtitle">现代贪吃蛇</h2>
    <SnakeGame />
    <div class="open-source-info">
      项目已开源，请访问<a href="https://github.com/Criel14/modern-snake-app" target="_blank" rel="noopener noreferrer">仓库</a>
    </div>
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
  height: 100vh;
  overflow: hidden;
}

html {
  height: 100%;
  overflow: hidden;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px 0;
  height: 100vh;
  box-sizing: border-box;
  text-align: center;
  overflow: hidden;
  position: relative;
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

.open-source-info {
  position: absolute;
  bottom: 5%;
  left: 0;
  right: 0;
  text-align: center;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.4);
  padding: 10px;
  z-index: 10;
}

.open-source-info a {
  color: rgba(78, 204, 163, 0.8);
  text-decoration: none;
  font-weight: bold;
  transition: color 0.2s, text-shadow 0.2s;
}

.open-source-info a:hover {
  color: #5fddaf;
  text-shadow: 0 0 5px rgba(78, 204, 163, 0.4);
}
</style>
