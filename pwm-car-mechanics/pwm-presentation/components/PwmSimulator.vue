<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const dutyCycle = ref(50)
const frequency = ref(20)
const canvas = ref(null)
const fanBlades = ref(null)
let animationId = null
let fanAngle = 0
let lastTime = 0

onMounted(() => {
  setupCanvas()
  lastTime = performance.now()
  animate()
})

onUnmounted(() => {
  if (animationId) cancelAnimationFrame(animationId)
})

function setupCanvas() {
  const c = canvas.value
  if (!c) return
  const rect = c.getBoundingClientRect()
  const dpr = window.devicePixelRatio || 1
  c.width = rect.width * dpr
  c.height = 150 * dpr
  c.style.height = '150px'
  const ctx = c.getContext('2d')
  ctx.scale(dpr, dpr)
}

function animate(timestamp) {
  const deltaTime = timestamp ? (timestamp - lastTime) / 1000 : 0.016
  lastTime = timestamp || performance.now()
  
  drawScope()
  animateFan(deltaTime)
  animationId = requestAnimationFrame(animate)
}

function drawScope() {
  const c = canvas.value
  if (!c) return
  const ctx = c.getContext('2d')
  const dpr = window.devicePixelRatio || 1
  const w = c.width / dpr
  const h = c.height / dpr
  
  ctx.clearRect(0, 0, w, h)
  
  // Grid
  ctx.strokeStyle = '#0a220b'
  ctx.lineWidth = 1
  ctx.beginPath()
  for (let i = 1; i < 10; i++) {
    const x = (w / 10) * i
    ctx.moveTo(x, 0)
    ctx.lineTo(x, h)
  }
  for (let i = 1; i < 6; i++) {
    const y = (h / 6) * i
    ctx.moveTo(0, y)
    ctx.lineTo(w, y)
  }
  ctx.stroke()
  
  // PWM wave - time-based drawing like original
  const duty = dutyCycle.value / 100
  const freq = frequency.value
  const yLow = h * 0.8
  const yHigh = h * 0.2
  const screenTimeSpan = 0.05 // 50ms display window
  const periodSec = 1 / freq
  
  ctx.strokeStyle = '#10b981'
  ctx.shadowBlur = 6
  ctx.shadowColor = '#10b981'
  ctx.lineWidth = 3.5
  ctx.lineCap = 'round'
  ctx.lineJoin = 'miter'
  ctx.beginPath()
  
  let lastY = null
  for (let x = 0; x < w; x++) {
    const t = (x / w) * screenTimeSpan
    const phase = t / periodSec
    const cycleFraction = phase % 1.0
    const val = cycleFraction < duty ? 1 : 0
    const y = val === 1 ? yHigh : yLow

    if (x === 0) {
      ctx.moveTo(x, y)
    } else {
      if (lastY !== null && lastY !== y) {
        ctx.lineTo(x, lastY)
      }
      ctx.lineTo(x, y)
    }
    lastY = y
  }
  ctx.stroke()
  ctx.shadowBlur = 0
}

function animateFan(deltaTime) {
  const duty = dutyCycle.value / 100
  const freq = frequency.value
  let pulseFactor = 1.0
  
  if (freq < 15) {
    const wave = Math.sin(2 * Math.PI * freq * (performance.now() / 1000))
    pulseFactor = (wave > (1 - 2 * duty)) ? 1.2 : 0.2
    if (duty === 0) pulseFactor = 0
  }
  
  const speed = duty * 720 * pulseFactor
  fanAngle = (fanAngle + speed * deltaTime) % 360
  
  if (fanBlades.value) {
    fanBlades.value.setAttribute('transform', `rotate(${fanAngle} 30 30)`)
  }
}

function getAvgVoltage() {
  return (12 * dutyCycle.value / 100).toFixed(1)
}
</script>

<template>
  <div class="pwm-simulator">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      Simulador PWM
    </div>
    
    <div class="sim-content">
      <div class="controls-row">
        <div class="control-group">
          <label>Ciclo de Trabajo</label>
          <div class="slider-row">
            <input type="range" v-model.number="dutyCycle" min="0" max="100" class="slider">
            <span class="value-badge">{{ dutyCycle }}%</span>
          </div>
        </div>
        <div class="control-group">
          <label>Frecuencia</label>
          <div class="slider-row">
            <input type="range" v-model.number="frequency" min="1" max="100" class="slider">
            <span class="value-badge">{{ frequency }} Hz</span>
          </div>
        </div>
      </div>

      <div class="output-grid">
        <div class="scope-box">
          <div class="scope-label">Osciloscopio</div>
          <canvas ref="canvas" class="scope-canvas"></canvas>
          <div class="scope-info">
            <span>V promedio: {{ getAvgVoltage() }}V</span>
            <span>Escala: 12V</span>
          </div>
        </div>
        
        <div class="demo-box">
          <div class="demo-label">Demostración</div>
          <div class="demo-icons">
            <div class="icon-group">
              <svg class="fan-icon" viewBox="0 0 60 60">
                <circle cx="30" cy="30" r="28" fill="none" stroke="#374151" stroke-width="2"/>
                <g ref="fanBlades">
                  <path d="M30 30 Q35 15 30 5 Q25 15 30 30" fill="#3b82f6"/>
                  <path d="M30 30 Q45 35 55 30 Q45 25 30 30" fill="#3b82f6"/>
                  <path d="M30 30 Q35 45 30 55 Q25 45 30 30" fill="#3b82f6"/>
                  <path d="M30 30 Q15 35 5 30 Q15 25 30 30" fill="#3b82f6"/>
                </g>
                <circle cx="30" cy="30" r="4" fill="#1e3a5f"/>
              </svg>
              <span class="icon-label">Ventilador</span>
            </div>
            <div class="icon-group">
              <svg class="bulb-icon" viewBox="0 0 50 60">
                <path d="M25 5 C15 5 8 15 8 25 C8 33 13 38 15 42 L35 42 C37 38 42 33 42 25 C42 15 35 5 25 5Z" 
                      :fill="`rgba(250, 204, 21, ${dutyCycle/100})`"
                      :filter="dutyCycle > 0 ? 'drop-shadow(0 0 8px rgba(250, 204, 21, 0.6))' : 'none'"/>
                <rect x="18" y="42" width="14" height="8" fill="#6b7280"/>
                <rect x="20" y="50" width="10" height="4" fill="#4b5563"/>
              </svg>
              <span class="icon-label">Luz</span>
            </div>
          </div>
          <div class="stats-row">
            <span>Potencia: {{ dutyCycle }}%</span>
            <span>Velocidad: {{ (dutyCycle * 0.01).toFixed(2) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pwm-simulator {
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

.controls-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
  margin-bottom: 1.25rem;
}

.control-group label {
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

.slider {
  flex: 1;
  -webkit-appearance: none;
  height: 6px;
  background: var(--bg-tertiary);
  border-radius: 3px;
  outline: none;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  background: var(--accent);
  border-radius: 50%;
  cursor: pointer;
}

.value-badge {
  background: var(--bg-tertiary);
  color: var(--accent);
  padding: 0.3rem 0.6rem;
  border-radius: 6px;
  font-family: var(--font-mono);
  font-size: 0.85rem;
  font-weight: 500;
  min-width: 60px;
  text-align: center;
}

.output-grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 1.25rem;
}

.scope-box, .demo-box {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 0.75rem;
}

.scope-label, .demo-label {
  font-size: 0.75rem;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.scope-canvas {
  width: 100%;
  height: 150px;
  background: #05070a;
  border-radius: 6px;
  border: 1px solid #1a2332;
}

.scope-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.7rem;
  color: var(--text-muted);
  margin-top: 0.5rem;
  font-family: var(--font-mono);
}

.demo-icons {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin: 1rem 0;
}

.icon-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.fan-icon, .bulb-icon {
  width: 60px;
  height: 60px;
}

.icon-label {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.stats-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: var(--text-muted);
  font-family: var(--font-mono);
}
</style>
