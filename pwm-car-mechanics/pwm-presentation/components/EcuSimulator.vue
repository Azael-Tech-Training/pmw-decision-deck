<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

const temp = ref(85)
const FIXED_FREQ = 150
let animationId = null
let fanRotation = 0
let currentSpeed = 0
const fanBlades = ref(null)

onMounted(() => {
  animate()
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
})

const dutyCycle = computed(() => {
  if (temp.value < 85) return 0
  if (temp.value > 110) return 100
  return Math.round(((temp.value - 85) / 25) * 100)
})

const tracePath = computed(() => {
  const duty = dutyCycle.value / 100
  const w = 400
  const h = 120
  const highY = 20
  const lowY = 100
  const cycles = 4
  const cycleWidth = w / cycles
  
  if (duty === 0) return `M 0 ${lowY} L ${w} ${lowY}`
  if (duty === 1) return `M 0 ${highY} L ${w} ${highY}`
  
  let d = `M 0 ${lowY}`
  for (let i = 0; i < cycles; i++) {
    const x = i * cycleWidth
    const onWidth = cycleWidth * duty
    d += ` L ${x} ${lowY} L ${x} ${highY} L ${x + onWidth} ${highY} L ${x + onWidth} ${lowY}`
  }
  return d
})

function animate() {
  const duty = dutyCycle.value / 100
  const targetSpeed = duty * 12
  currentSpeed += (targetSpeed - currentSpeed) * 0.1
  fanRotation = (fanRotation + currentSpeed) % 360
  
  if (fanBlades.value) {
    fanBlades.value.style.transform = `rotate(${fanRotation}deg)`
  }
  
  animationId = requestAnimationFrame(animate)
}

function getFrequencyDisplay() {
  const duty = dutyCycle.value
  if (duty === 0 || duty === 100) return '0 Hz'
  return `${FIXED_FREQ} Hz`
}
</script>

<template>
  <div class="ecu-simulator">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      Simulador ECU - Control en Loops Cerrados
    </div>

    <div class="sim-content">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">Temp. Refrigerante</div>
          <div class="stat-val">{{ temp }}°C</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Temp. Objetivo</div>
          <div class="stat-val">90°C</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Ciclo de Trabajo</div>
          <div class="stat-val accent">{{ dutyCycle }}%</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Frecuencia</div>
          <div class="stat-val">{{ getFrequencyDisplay() }}</div>
        </div>
      </div>

      <div class="display-grid">
        <div class="scope-box">
          <div class="scope-label">Señal ECU (12V)</div>
          <svg class="scope-screen" viewBox="0 0 400 120">
            <!-- Grid -->
            <g class="scope-grid">
              <line v-for="i in 10" :key="'v'+i" :x1="i*40" y1="0" :x2="i*40" y2="120" stroke="#0a220b" stroke-width="0.5"/>
              <line v-for="i in 6" :key="'h'+i" x1="0" :y1="i*20" x2="400" :y2="i*20" stroke="#0a220b" stroke-width="0.5"/>
            </g>
            <!-- Labels -->
            <text x="5" y="15" fill="#6b7280" font-size="8">12V</text>
            <text x="5" y="115" fill="#6b7280" font-size="8">0V</text>
            <!-- Trace -->
            <path :d="tracePath" class="scope-trace" fill="none"/>
          </svg>
        </div>

        <div class="fan-box">
          <div class="fan-label">Ventilador Radiador</div>
          <div class="fan-visual">
            <svg class="fan-svg" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="45" fill="none" stroke="#1e3a5f" stroke-width="2"/>
              <g ref="fanBlades" class="fan-blades">
                <path d="M50 50 Q55 30 50 10 Q45 30 50 50" :fill="dutyCycle > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q70 55 90 50 Q70 45 50 50" :fill="dutyCycle > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q55 70 50 90 Q45 70 50 50" :fill="dutyCycle > 0 ? '#3b82f6' : '#1e3a5f'"/>
                <path d="M50 50 Q30 55 10 50 Q30 45 50 50" :fill="dutyCycle > 0 ? '#3b82f6' : '#1e3a5f'"/>
              </g>
              <circle cx="50" cy="50" r="8" fill="#0b0f19"/>
            </svg>
          </div>
        </div>
      </div>

      <div class="control-panel">
        <label>Temperatura del Refrigerante</label>
        <div class="slider-row">
          <span class="temp-low">75°C</span>
          <input type="range" v-model.number="temp" min="75" max="115" class="temp-slider">
          <span class="temp-high">115°C</span>
        </div>
        <div class="temp-zones">
          <span class="zone cold">Frío</span>
          <span class="zone optimal">Óptimo (90°C)</span>
          <span class="zone hot">Sobrecalentado</span>
        </div>
      </div>

      <div class="info-box">
        <strong>Concepto clave:</strong> El ECU mantiene la frecuencia constante ({{ FIXED_FREQ }} Hz) y solo ajusta el ciclo de trabajo ({{ dutyCycle }}%) para controlar la velocidad del ventilador según la temperatura del refrigerante.
      </div>
    </div>
  </div>
</template>

<style scoped>
.ecu-simulator {
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

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.75rem;
  margin-bottom: 1.25rem;
}

.stat-card {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 0.75rem;
  text-align: center;
}

.stat-label {
  font-size: 0.7rem;
  color: var(--text-muted);
  text-transform: uppercase;
  margin-bottom: 0.25rem;
}

.stat-val {
  font-family: var(--font-mono);
  font-size: 1.1rem;
  color: var(--text-primary);
  font-weight: 500;
}

.stat-val.accent {
  color: var(--accent);
}

.display-grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 1.25rem;
  margin-bottom: 1.25rem;
}

.scope-box, .fan-box {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 0.75rem;
}

.scope-label, .fan-label {
  font-size: 0.75rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.scope-screen {
  width: 100%;
  height: 120px;
  background: #05070a;
  border-radius: 6px;
  border: 1px solid #1a2332;
}

.scope-trace {
  stroke: #10b981;
  stroke-width: 2.5;
  filter: drop-shadow(0 0 4px rgba(16, 185, 129, 0.6));
}

.fan-visual {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 120px;
}

.fan-svg {
  width: 100px;
  height: 100px;
}

.fan-blades {
  transform-origin: 50px 50px;
  transition: fill 0.3s ease;
}

.control-panel {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.control-panel label {
  display: block;
  color: var(--text-secondary);
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
}

.slider-row {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.temp-low, .temp-high {
  font-family: var(--font-mono);
  font-size: 0.8rem;
  color: var(--text-muted);
  min-width: 40px;
}

.temp-low { color: #3b82f6; }
.temp-high { color: #ef4444; }

.temp-slider {
  flex: 1;
  -webkit-appearance: none;
  height: 8px;
  border-radius: 4px;
  background: linear-gradient(to right, #3b82f6, #10b981 40%, #f59e0b 70%, #ef4444);
  outline: none;
}

.temp-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}

.temp-zones {
  display: flex;
  justify-content: space-between;
  margin-top: 0.5rem;
  font-size: 0.7rem;
}

.zone { color: var(--text-muted); }
.zone.cold { color: #3b82f6; }
.zone.optimal { color: #10b981; }
.zone.hot { color: #ef4444; }

.info-box {
  background: rgba(59, 130, 246, 0.1);
  border: 1px solid rgba(59, 130, 246, 0.3);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  font-size: 0.85rem;
  color: #93c5fd;
  line-height: 1.5;
}
</style>
