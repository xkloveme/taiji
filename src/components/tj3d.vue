<template>
  <div class="taiji-container">
    <div class="realm-control">
      <!-- 境界选择器 -->
      <div class="realm-tabs">
        <div v-for="(realm, index) in realms" 
             :key="realm.name"
             :class="['realm-tab', { active: currentRealmIndex === index }]"
             @click="changeRealm(index)">
          <span class="realm-name">{{ realm.name }}</span>
          <span class="realm-speed">{{ realm.speed }}x</span>
        </div>
      </div>

      <!-- 速度控制条 -->
      <div class="speed-control">
        <div class="current-speed">{{ rotationSpeed }}x</div>
        <div class="speed-bar" 
             @mousedown="startDrag"
             @mousemove="onDrag"
             @mouseup="stopDrag"
             @mouseleave="stopDrag"
             ref="speedBar">
          <div class="speed-progress" :style="progressStyle"></div>
          <div class="speed-handle" :style="handleStyle"></div>
        </div>
      </div>
    </div>
    
    <div class="taiji" :style="rotationStyle" :class="{ paused: isPaused }">
      <div class="taiji-inner">
        <div class="white-circle">
          <div class="black-dot"></div>
        </div>
        <div class="black-circle">
          <div class="white-dot"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const realms = [
  { name: '无极', speed: 1 },
  { name: '太极', speed: 15 },
  { name: '两仪', speed: 30 },
  { name: '三才', speed: 36 },
  { name: '四象', speed: 46 },
  { name: '五行', speed: 56 },
  { name: '六合', speed: 60 },
  { name: '七星', speed: 90 },
  { name: '八卦', speed: 120 },
  { name: '九宫', speed: 150 },
  { name: '十方', speed: 180 }
]

const currentRealmIndex = ref(0)
const rotationSpeed = ref(1)
const isPaused = ref(false)
const isDragging = ref(false)
const speedBar = ref(null)

const rotationStyle = computed(() => {
  const duration = 3 / rotationSpeed.value
  return {
    'animation-duration': `${duration}s`
  }
})

const progressStyle = computed(() => {
  const progress = (rotationSpeed.value / 180) * 100
  return {
    width: `${progress}%`
  }
})

const handleStyle = computed(() => {
  return {
    left: `${(rotationSpeed.value / 180) * 100}%`
  }
})

const togglePause = () => {
  isPaused.value = !isPaused.value
}

const startDrag = (e) => {
  isDragging.value = true
  updateSpeedFromPosition(e)
}

const onDrag = (e) => {
  if (isDragging.value) {
    updateSpeedFromPosition(e)
  }
}

const stopDrag = () => {
  isDragging.value = false
}

const updateSpeedFromPosition = (e) => {
  if (!speedBar.value) return
  const rect = speedBar.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const percentage = Math.max(0, Math.min(1, x / rect.width))
  rotationSpeed.value = Math.round(percentage * 180)
  updateRealmFromSpeed(rotationSpeed.value)
}

const changeRealm = (index) => {
  currentRealmIndex.value = index
  rotationSpeed.value = realms[index].speed
}

const updateRealmFromSpeed = (speed) => {
  const realmIndex = realms.findIndex((realm, index) => {
    const nextRealm = realms[index + 1]
    return speed >= realm.speed && (!nextRealm || speed < nextRealm.speed)
  })
  if (realmIndex !== -1) {
    currentRealmIndex.value = realmIndex
  }
}
</script>

<style scoped>
.current-speed {
  min-width: 60px;
  color: #fff;
  font-size: 16px;
  font-weight: bold;
  text-align: center;
  padding: 4px 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  margin-right: 10px;
}

.speed-control {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  margin-top: 10px;
}

.speed-bar {
  position: relative;
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
  cursor: pointer;
}

.speed-progress {
  position: absolute;
  height: 100%;
  background: #fff;
  border-radius: 2px;
  pointer-events: none;
}

.speed-handle {
  position: absolute;
  top: 50%;
  width: 12px;
  height: 12px;
  background: #fff;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  cursor: pointer;
  transition: transform 0.2s;
}

.speed-handle:hover {
  transform: translate(-50%, -50%) scale(1.2);
}

.taiji-container {
  width: 100%;
  height: 100vh;
  background: transparent;
  display: flex;
  justify-content: center;
  align-items: center;
      pointer-events: none;
}

.taiji {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  background: #fff;
  position: relative;
  box-shadow: 0 0 30px rgba(255, 255, 255, 0.1);
  animation: rotate linear infinite;
  animation-duration: 3s; /* 基础动画时间 */
  transition: animation-duration 0.1s linear;
}

.taiji::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 50%;
  height: 100%;
  border-radius: 300px 0 0 300px;
  background: #000;
}

.taiji-inner {
  width: 150px;
  height: 300px;
  position: absolute;
  left: 75px;
}

.white-circle {
  width: 150px;
  height: 150px;
  position: absolute;
  top: 0;
  left: 0;
  border-radius: 50%;
  background: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
}

.black-circle {
  width: 150px;
  height: 150px;
  position: absolute;
  bottom: 0;
  left: 0;
  border-radius: 50%;
  background: #000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.black-dot {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #000;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

.white-dot {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.realm-control {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.8);
  padding: 20px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  backdrop-filter: blur(10px);
  z-index: 100;
      pointer-events: auto;
}

.realm-tabs {
  display: grid;
  grid-template-columns: repeat(11, 1fr);
  gap: 5px;
}

.realm-tab {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #fff;
}

.realm-tab:hover {
  background: rgba(255, 255, 255, 0.2);
}

.realm-tab.active {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.05);
}

.realm-name {
  font-size: 14px;
  margin-bottom: 4px;
}

.realm-speed {
  font-size: 12px;
  opacity: 0.8;
}

/* 动画效果 */
.realm-tab {
  position: relative;
  overflow: hidden;
}

.realm-tab::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transform: translateX(-100%);
  transition: transform 0.3s;
}

.realm-tab:hover::after {
  transform: translateX(100%);
}

/* 适配暗色主题 */
@media (prefers-color-scheme: dark) {
  .realm-control {
    background: rgba(255, 255, 255, 0.1);
  }
}

/* 动画过渡效果 */
.taiji {
  transition: animation-duration 0.3s ease-out;
}
</style>