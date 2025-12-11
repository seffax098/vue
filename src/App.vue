<template>
  <div id="app">
    <header class="app-header">
      <h1>Генератор цветовых палитр</h1>
    </header>

    <main class="main-content">
      <ColorPaletteGenerator />
    </main>

    <footer class="app-footer">
      <p>Vue 3 • Генератор палитр • Контрольная №5</p>
    </footer>
  </div>
</template>

<script>
import ColorPaletteGenerator from './components/ColorPaletteGenerator.vue'

export default {
  name: 'App',
  components: {
    ColorPaletteGenerator
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Шапка */
.app-header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 2rem;
  text-align: center;
}

.app-header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.3rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1.05rem;
}

/* Контент */
.main-content {
  flex: 1;
  padding: 2rem 1rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* Подвал */
.app-footer {
  background-color: #333;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
  font-size: 0.9rem;
}
</style>
