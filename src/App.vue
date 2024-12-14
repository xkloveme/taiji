<script setup>
import Tj3d from './components/tj3d.vue'

</script>

<template>
  <div class="app-container">
    <Tj3d class="taiji-overlay"/>
  </div>
</template>

<style>
.app-container {
  width: 100vw;
  height: 100vh;
      pointer-events: none;
  position: relative;
  overflow: hidden;
}

.fluid-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: transparent;
}

canvas {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
}

.taiji-overlay {
  position: relative;
  z-index: 1;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  overflow: hidden;
}
</style>
