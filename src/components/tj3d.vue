<template>
  <div class="taiji-container">
    <div class="control-wrapper">
      <div class="lock-button" @click="toggleConfig">
        <svg v-if="showConfig" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <line x1="18" y1="6" x2="6" y2="18"></line>
          <line x1="6" y1="6" x2="18" y2="18"></line>
        </svg>
        <svg v-else xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="3"></circle>
          <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"></path>
        </svg>
      </div>
      
      <div class="realm-control" :class="{ show: showConfig }">
        <div class="realm-tabs">
          <div v-for="(realm, index) in realms" 
               :key="realm.name"
               :class="['realm-tab', { active: currentRealmIndex === index }]"
               @click="changeRealm(index)">
            <span class="realm-name">{{ realm.name }}</span>
            <span class="realm-speed">{{ realm.speed.toFixed(1) }}x</span>
          </div>
        </div>

        <div class="speed-control">
          <div class="current-speed">{{ rotationSpeed.toFixed(1) }}x</div>
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
    </div>
    
    <div class="taiji" 
         ref="taijiRef"
         :style="[transformStyle]" 
         :class="{ paused: isPaused, dragging: isDraggingTaiji }"
         @mousedown="startTaijiDrag"
         @mousemove="onTaijiDrag"
         @mouseup="stopTaijiDrag"
         @mouseleave="stopTaijiDrag">
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
const isLocked = ref(true)
const isDraggingTaiji = ref(false)
const lastRotation = ref(0)
const initialRotation = ref(0)
const dragStartTime = ref(0)
const taijiRef = ref(null)
const currentRotation = ref(0)
const angularVelocity = ref(0)
const lastTime = ref(0)
const dragStartAngle = ref(0)
const dragPrevAngle = ref(0)
const dragVelocity = ref(0)
const autoRotating = ref(false)
const rotationDirection = ref(1)
const showConfig = ref(false)

const transformStyle = computed(() => {
  if (isDraggingTaiji.value) {
    return {
      transform: `rotate(${currentRotation.value}deg)`,
      transition: 'none'
    }
  } else if (autoRotating.value) {
    const duration = 3 / rotationSpeed.value
    return {
      transform: `rotate(${currentRotation.value}deg)`,
      transition: `transform ${duration}s linear`
    }
  } else {
    const duration = 3 / rotationSpeed.value
    return {
      animation: `rotate ${duration}s linear infinite`
    }
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
  rotationSpeed.value = parseFloat((percentage * 180).toFixed(1))
  updateRealmFromSpeed(rotationSpeed.value)
  
  if (!isDraggingTaiji.value && !autoRotating.value) {
    startNormalRotation()
  }
}

const changeRealm = (index) => {
  currentRealmIndex.value = index
  rotationSpeed.value = parseFloat(realms[index].speed.toFixed(1))
  
  if (!isDraggingTaiji.value && !autoRotating.value) {
    startNormalRotation()
  }
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

const toggleLock = () => {
  isLocked.value = !isLocked.value
}

const toggleConfig = () => {
  showConfig.value = !showConfig.value
  if (!showConfig.value && !isDraggingTaiji.value && !autoRotating.value) {
    startNormalRotation()
  }
}

const startTaijiDrag = (e) => {
  isDraggingTaiji.value = true
  autoRotating.value = false
  const rect = taijiRef.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2
  
  dragStartAngle.value = Math.atan2(e.clientY - centerY, e.clientX - centerX) * (180 / Math.PI)
  dragPrevAngle.value = dragStartAngle.value
  lastRotation.value = currentRotation.value
  rotationSpeed.value = 0
}

const onTaijiDrag = (e) => {
  if (!isDraggingTaiji.value) return
  
  const rect = taijiRef.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2
  
  const currentAngle = Math.atan2(e.clientY - centerY, e.clientX - centerX) * (180 / Math.PI)
  let deltaDrag = currentAngle - dragPrevAngle.value
  
  if (deltaDrag > 180) deltaDrag -= 360
  if (deltaDrag < -180) deltaDrag += 360
  
  currentRotation.value = (lastRotation.value + deltaDrag) % 360
  dragVelocity.value = deltaDrag
  rotationDirection.value = Math.sign(deltaDrag) || 1
  dragPrevAngle.value = currentAngle
}

const stopTaijiDrag = () => {
  if (!isDraggingTaiji.value) return
  isDraggingTaiji.value = false
  
  const initialSpeed = Math.abs(dragVelocity.value * 2)
  rotationSpeed.value = parseFloat(Math.min(Math.max(initialSpeed, 1), 180).toFixed(1))
  
  autoRotating.value = true
  const startTime = performance.now()
  
  const animate = () => {
    if (!autoRotating.value) return
    
    const currentTime = performance.now()
    const elapsed = currentTime - startTime
    
    rotationSpeed.value = parseFloat(Math.max(
      1,
      initialSpeed * Math.exp(-elapsed / 2000)
    ).toFixed(1))
    
    currentRotation.value += rotationSpeed.value * rotationDirection.value
    
    updateRealmFromSpeed(rotationSpeed.value)
    
    if (rotationSpeed.value <= 1.1) {
      rotationSpeed.value = 1
      updateRealmFromSpeed(1)
      startNormalRotation()
      return
    }
    
    requestAnimationFrame(animate)
  }
  
  requestAnimationFrame(animate)
}

const startNormalRotation = () => {
  isDraggingTaiji.value = false
  autoRotating.value = false
  currentRotation.value = 0
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
  animation: rotate 3s linear infinite;
  animation-duration: 3s; /* 基础动画时间 */
  transition: animation-duration 0.1s linear;
  cursor: grab;
  pointer-events: auto;
  transform-origin: center;
  will-change: transform;
  transition: none;
  animation: none;
  user-select: none;
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
  top: -100%;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  padding: 20px;
  border-radius: 15px;
  opacity: 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
  min-width: 300px;
}

.realm-control.show {
  top: 20px;
  opacity: 1;
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

/* 动画果 */
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

.taiji.dragging {
  cursor: grabbing;
  animation: none !important;
  transition: none !important;
}

.hover-area {
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  padding: 40px;
  z-index: 100;
}

.control-wrapper {
  position: fixed;
  z-index: 100;
}

.lock-button {
  position: fixed;
  top: 20px;
  right: 20px;
  width: 40px;
  height: 40px;
  opacity: 0;
  pointer-events: auto;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(5px);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #fff;
  font-size: 16px;
  transition: all 0.3s ease;
}

.lock-button:hover {
  opacity: 1;
  transform: scale(1.1);
  background: rgba(0, 0, 0, 0.7);
}
</style>