<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

const isManual = ref(true)
const isButtonPressed = ref(false)
const isAutoOn = ref(false)
const dutyCycle = ref(50)
const fanSpeed = ref(0)
let fanRotation = 0
let animationId = null
let autoTimeout = null
const fanBlades = ref(null)

onMounted(() => {
  animate()
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
  if (autoTimeout) clearTimeout(autoTimeout)
})

const timeOn = computed(() => dutyCycle.value * 10)
const timeOff = computed(() => (100 - dutyCycle.value) * 10)

function toggleMode() {
  isManual.value = !isManual.value
  if (!isManual.value) {
    startAutoLoop()
  } else {
    stopAutoLoop()
  }
}

function startAutoLoop() {
  const duty = dutyCycle.value
  isAutoOn.value = true
  
  autoTimeout = setTimeout(() => {
    isAutoOn.value = false
    autoTimeout = setTimeout(() => {
      if (!isManual.value) startAutoLoop()
    }, timeOff.value)
  }, timeOn.value)
}

function stopAutoLoop() {
  if (autoTimeout) {
    clearTimeout(autoTimeout)
    autoTimeout = null
  }
  isAutoOn.value = false
}

function animate() {
  const signalOn = isManual.value ? isButtonPressed.value : isAutoOn.value
  
  if (signalOn) {
    fanSpeed.value = Math.min(fanSpeed.value + 15 * 0.016, 15)
  } else {
    fanSpeed.value = Math.max(fanSpeed.value - 1.5 * 0.016, 0)
  }
  
  fanRotation = (fanRotation + fanSpeed.value * 60 * 0.016) % 360
  
  if (fanBlades.value) {
    fanBlades.value.style.transform = `rotate(${fanRotation}deg)`
  }
  
  animationId = requestAnimationFrame(animate)
}

function onMouseDown() {
  isButtonPressed.value = true
}

function onMouseUp() {
  isButtonPressed.value = false
}

function onTouchStart() {
  isButtonPressed.value = true
}

function onTouchEnd() {
  isButtonPressed.value = false
}
</script>

<template>
  <div class="time-modulator">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      Modulación de Tiempo - {{ isManual ? 'Modo Manual' : 'Modo ECU Automático' }}
    </div>

    <div class="sim-content">
      <div class="mode-toggle">
        <button :class="{ active: isManual }" @click="isManual = true; stopAutoLoop()">
          Modo Manual
        </button>
        <button :class="{ active: !isManual }" @click="toggleMode">
          Modo ECU Automático
        </button>
      </div>

      <div class="display-grid">
        <div class="controls-panel">
          <!-- Manual Mode -->
          <div v-if="isManual" class="manual-panel">
            <p class="instruction">Mantén presionado el botón para enviar pulsos de 12V</p>
            <button 
              class="tap-button"
              :class="{ pressed: isButtonPressed }"
              @mousedown="onMouseDown"
              @mouseup="onMouseUp"
              @mouseleave="onMouseUp"
              @touchstart="onTouchStart"
              @touchend="onTouchEnd"
            >
              {{ isButtonPressed ? '12V - ENCENDIDO' : '0V - APAGADO' }}
            </button>
            <div class="signal-indicator" :class="{ active: isButtonPressed }">
              Señal: {{ isButtonPressed ? '12V' : '0V' }}
            </div>
          </div>

          <!-- Auto Mode -->
          <div v-else class="auto-panel">
            <div class="duty-control">
              <label>Ciclo de Trabajo ECU</label>
              <div class="slider-row">
                <input type="range" v-model.number="dutyCycle" min="10" max="90" class="duty-slider">
                <span class="duty-value">{{ dutyCycle }}%</span>
              </div>
            </div>
            
            <div class="time-bar-container">
              <div class="time-bar-label">
                <span class="time-on-label">ON: {{ timeOn }}ms</span>
                <span class="time-off-label">OFF: {{ timeOff }}ms</span>
              </div>
              <div class="time-bar">
                <div class="time-on" :style="{ width: dutyCycle + '%' }"></div>
                <div class="time-off" :style="{ width: (100 - dutyCycle) + '%' }"></div>
              </div>
            </div>

            <div class="signal-indicator" :class="{ active: isAutoOn }">
              Señal: {{ isAutoOn ? '12V' : '0V' }}
            </div>
          </div>
        </div>

        <div class="visual-panel">
          <div class="fan-container">
            <svg class="fan-svg" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="45" fill="none" stroke="#1e3a5f" stroke-width="2"/>
              <g ref="fanBlades" class="fan-blades">
                <path d="M50 50 Q55 30 50 10 Q45 30 50 50" :fill="fanSpeed > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q70 55 90 50 Q70 45 50 50" :fill="fanSpeed > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q55 70 50 90 Q45 70 50 50" :fill="fanSpeed > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q30 55 10 50 Q30 45 50 50" :fill="fanSpeed > 0 ? '#3b82f6' : '#1e3a5f'"/>
              </g>
              <circle cx="50" cy="50" r="8" fill="#0b0f19"/>
            </svg>
          </div>
          
          <div class="indicator-bulb" :class="{ active: isManual ? isButtonPressed : isAutoOn }"></div>
          
          <div class="voltage-label">
            {{ (isManual ? isButtonPressed : isAutoOn) ? '12V' : '0V' }}
          </div>
        </div>
      </div>

      <div class="stats-row">
        <div class="stat">
          <span class="stat-label">Velocidad Motor:</span>
          <span class="stat-value">{{ (fanSpeed / 15 * 100).toFixed(0) }}%</span>
        </div>
        <div class="stat">
          <span class="stat-label">Potencia Promedio:</span>
          <span class="stat-value">{{ isManual ? (isButtonPressed ? '100%' : '0%') : dutyCycle + '%' }}</span>
        </div>
      </div>

      <div class="info-box">
        <strong>Concepto clave:</strong> El MOSFET solo puede estar 100% encendido o 100% apagado. El motor promedia físicamente debido a su inercia mecánica, creando la ilusión de un control "analógico".
      </div>
    </div>
  </div>
</template>

<style scoped>
.time-modulator {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 16px;
  overflow: hidden;
}

.sim-header {
  background: var(--bg-tertiary);
  padding: 0.75rem 1.25rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  border-bottom: 1px solid var(--border-color);
}

.sim-badge {
  background: var(--accent);
  color: white;
  font-size: 0.7rem;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-weight: 600;
}

.sim-content {
  padding: 1.25rem;
}

.mode-toggle {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.25rem;
}

.mode-toggle button {
  flex: 1;
  padding: 0.6rem;
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s;
}

.mode-toggle button.active {
  background: var(--accent);
  border-color: var(--accent);
  color: white;
}

.display-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
  margin-bottom: 1rem;
}

.controls-panel, .visual-panel {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 1rem;
}

.instruction {
  font-size: 0.85rem;
  color: var(--text-secondary);
  margin-bottom: 1rem;
  text-align: center;
}

.tap-button {
  width: 100%;
  padding: 1.5rem;
  background: var(--bg-tertiary);
  border: 2px solid var(--border-color);
  border-radius: 12px;
  color: var(--text-primary);
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.1s;
  user-select: none;
}

.tap-button:active, .tap-button.pressed {
  background: var(--accent);
  border-color: var(--accent);
  transform: scale(0.98);
}

.duty-control {
  margin-bottom: 1rem;
}

.duty-control label {
  display: block;
  color: var(--text-secondary);
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.slider-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.duty-slider {
  flex: 1;
  -webkit-appearance: none;
  height: 6px;
  background: var(--bg-tertiary);
  border-radius: 3px;
  outline: none;
}

.duty-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  background: var(--accent);
  border-radius: 50%;
  cursor: pointer;
}

.duty-value {
  background: var(--bg-tertiary);
  color: var(--accent);
  padding: 0.3rem 0.6rem;
  border-radius: 6px;
  font-family: var(--font-mono);
  font-size: 0.85rem;
  min-width: 50px;
  text-align: center;
}

.time-bar-container {
  margin-bottom: 1rem;
}

.time-bar-label {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  margin-bottom: 0.3rem;
}

.time-on-label { color: #10b981; }
.time-off-label { color: #ef4444; }

.time-bar {
  display: flex;
  height: 24px;
  border-radius: 6px;
  overflow: hidden;
  background: var(--bg-tertiary);
}

.time-on {
  background: #10b981;
  transition: width 0.15s ease;
}

.time-off {
  background: #ef4444;
  transition: width 0.15s ease;
}

.signal-indicator {
  text-align: center;
  padding: 0.5rem;
  background: var(--bg-tertiary);
  border-radius: 6px;
  font-family: var(--font-mono);
  font-size: 0.85rem;
  color: var(--text-muted);
}

.signal-indicator.active {
  background: rgba(16, 185, 129, 0.2);
  color: #10b981;
}

.visual-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.fan-container {
  width: 120px;
  height: 120px;
}

.fan-svg {
  width: 100%;
  height: 100%;
}

.fan-blades {
  transform-origin: 50px 50px;
  transition: fill 0.3s ease;
}

.indicator-bulb {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #374151;
  transition: all 0.2s ease;
}

.indicator-bulb.active {
  background: #10b981;
  box-shadow: 0 0 15px rgba(16, 185, 129, 0.6);
}

.voltage-label {
  font-family: var(--font-mono);
  font-size: 1rem;
  color: var(--text-muted);
}

.stats-row {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.stat {
  flex: 1;
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 0.6rem 0.75rem;
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
}

.stat-label { color: var(--text-muted); }
.stat-value { 
  color: var(--accent); 
  font-family: var(--font-mono);
  font-weight: 500;
}

.info-box {
  background: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.3);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  font-size: 0.85rem;
  color: #fcd34d;
  line-height: 1.5;
}
</style>
