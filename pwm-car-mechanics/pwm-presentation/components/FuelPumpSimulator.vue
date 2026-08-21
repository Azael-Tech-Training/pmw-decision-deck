<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const pedal = ref(0)
let animationId = null
let dot1 = 40, dot2 = 80, dot3 = 120
const fuelDot1 = ref(null)
const fuelDot2 = ref(null)
const fuelDot3 = ref(null)

onMounted(() => {
  animateDroplets()
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
})

const flow = computed(() => 0.5 + (pedal.value / 100) * 14.5)
const duty = computed(() => 25 + Math.round((pedal.value / 100) * 60))
const pressure = computed(() => 40)

const gaugeAngle = computed(() => {
  return (pressure.value - 40) * 2.25
})

const scopePath = computed(() => {
  const dutyVal = duty.value / 100
  const w = 400
  const h = 100
  const cycles = 5
  const cycleWidth = w / cycles
  const highY = 15
  const lowY = 85
  
  let d = `M 0 ${lowY}`
  for (let i = 0; i < cycles; i++) {
    const x = i * cycleWidth
    const onWidth = cycleWidth * dutyVal
    d += ` L ${x} ${lowY} L ${x} ${highY} L ${x + onWidth} ${highY} L ${x + onWidth} ${lowY}`
  }
  return d
})

function animateDroplets() {
  const speed = 0.5 + (pedal.value / 100) * 3
  
  dot1 += speed
  dot2 += speed
  dot3 += speed
  
  if (dot1 > 165) dot1 = 35
  if (dot2 > 165) dot2 = 35
  if (dot3 > 165) dot3 = 35
  
  if (fuelDot1.value) fuelDot1.value.setAttribute('cx', dot1)
  if (fuelDot2.value) fuelDot2.value.setAttribute('cx', dot2)
  if (fuelDot3.value) fuelDot3.value.setAttribute('cx', dot3)
  
  animationId = requestAnimationFrame(animateDroplets)
}
</script>

<template>
  <div class="fuel-simulator">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      Simulador Bomba de Combustible PWM
    </div>

    <div class="sim-content">
      <div class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">Pedal Acelerador</div>
          <div class="stat-val">{{ pedal }}%</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Presión Riel</div>
          <div class="stat-val">{{ pressure }} psi</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Ciclo de Trabajo</div>
          <div class="stat-val accent">{{ duty }}%</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Consumo</div>
          <div class="stat-val">{{ flow.toFixed(1) }} GPH</div>
        </div>
      </div>

      <div class="display-grid">
        <div class="left-panel">
          <!-- Fuel Flow Animation -->
          <div class="flow-box">
            <div class="flow-label">Flujo de Combustible</div>
            <svg class="flow-svg" viewBox="0 0 200 60">
              <rect x="10" y="20" width="30" height="20" fill="#374151" rx="3"/>
              <text x="15" y="33" fill="#9ca3af" font-size="7">BOMBA</text>
              
              <line x1="40" y1="30" x2="170" y2="30" stroke="#475569" stroke-width="6" stroke-linecap="round"/>
              
              <circle ref="fuelDot1" :cx="dot1" cy="30" r="4" fill="#eab308"/>
              <circle ref="fuelDot2" :cx="dot2" cy="30" r="4" fill="#eab308"/>
              <circle ref="fuelDot3" :cx="dot3" cy="30" r="4" fill="#eab308"/>
              
              <rect x="170" y="15" width="25" height="30" fill="#374151" rx="3"/>
              <text x="173" y="33" fill="#9ca3af" font-size="6">RIEL</text>
            </svg>
          </div>

          <!-- Oscilloscope -->
          <div class="scope-box">
            <div class="scope-label">Salida FPDM (12V)</div>
            <svg class="scope-screen" viewBox="0 0 400 100">
              <g class="scope-grid">
                <line v-for="i in 10" :key="'v'+i" :x1="i*40" y1="0" :x2="i*40" y2="100" stroke="#0a220b" stroke-width="0.5"/>
                <line v-for="i in 5" :key="'h'+i" x1="0" :y1="i*20" x2="400" :y2="i*20" stroke="#0a220b" stroke-width="0.5"/>
              </g>
              <path :d="scopePath" class="scope-trace" fill="none"/>
            </svg>
          </div>
        </div>

        <div class="right-panel">
          <!-- Pressure Gauge -->
          <div class="gauge-box">
            <div class="gauge-label">Presión del Riel</div>
            <svg class="gauge-svg" viewBox="0 0 120 100">
              <!-- Gauge arc background -->
              <path d="M 20 80 A 45 45 0 0 1 100 80" fill="none" stroke="#1e3a5f" stroke-width="8" stroke-linecap="round"/>
              <!-- Gauge arc fill -->
              <path d="M 20 80 A 45 45 0 0 1 100 80" fill="none" stroke="#10b981" stroke-width="8" stroke-linecap="round"
                    :stroke-dasharray="`${(pressure / 80) * 141} 141`"/>
              <!-- Target marker -->
              <line x1="60" y1="35" x2="60" y2="42" stroke="#f59e0b" stroke-width="2"/>
              <!-- Needle -->
              <line x1="60" y1="80" x2="60" y2="40" stroke="#ef4444" stroke-width="3" stroke-linecap="round"
                    :transform="`rotate(${gaugeAngle} 60 80)`"/>
              <circle cx="60" cy="80" r="5" fill="#1e3a5f"/>
              <!-- Labels -->
              <text x="15" y="92" fill="#6b7280" font-size="8">0</text>
              <text x="100" y="92" fill="#6b7280" font-size="8">80</text>
              <text x="55" y="75" fill="#9ca3af" font-size="10">psi</text>
            </svg>
          </div>
        </div>
      </div>

      <div class="control-panel">
        <label>Posición del Pedal del Acelerador</label>
        <div class="slider-row">
          <span class="pedal-label">0%</span>
          <input type="range" v-model.number="pedal" min="0" max="100" class="pedal-slider">
          <span class="pedal-label">100%</span>
        </div>
      </div>

      <div class="info-box">
        <strong>Sistema sin retorno:</strong> El PCM controla el FPDM a 80 Hz, que a su vez conduce la bomba a 20 kHz. La presión del riel se mantiene en {{ pressure }} psi mediante retroalimentación cerrada.
      </div>
    </div>
  </div>
</template>

<style scoped>
.fuel-simulator {
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
  grid-template-columns: 1.2fr 1fr;
  gap: 1.25rem;
  margin-bottom: 1.25rem;
}

.left-panel, .right-panel {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.flow-box, .scope-box, .gauge-box {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 0.75rem;
}

.flow-label, .scope-label, .gauge-label {
  font-size: 0.75rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.flow-svg {
  width: 100%;
  height: 60px;
}

.scope-screen {
  width: 100%;
  height: 100px;
  background: #05070a;
  border-radius: 6px;
  border: 1px solid #1a2332;
}

.scope-trace {
  stroke: #eab308;
  stroke-width: 2.5;
  filter: drop-shadow(0 0 4px rgba(234, 179, 8, 0.6));
}

.gauge-svg {
  width: 100%;
  max-width: 150px;
  margin: 0 auto;
  display: block;
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

.pedal-label {
  font-family: var(--font-mono);
  font-size: 0.8rem;
  color: var(--text-muted);
  min-width: 35px;
}

.pedal-slider {
  flex: 1;
  -webkit-appearance: none;
  height: 8px;
  border-radius: 4px;
  background: linear-gradient(to right, #1e3a5f, #10b981 50%, #f59e0b);
  outline: none;
}

.pedal-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}

.info-box {
  background: rgba(16, 185, 129, 0.1);
  border: 1px solid rgba(16, 185, 129, 0.3);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  font-size: 0.85rem;
  color: #6ee7b7;
  line-height: 1.5;
}
</style>
