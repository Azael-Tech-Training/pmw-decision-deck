<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  variant: {
    type: String,
    default: 'low-high'
  }
})

const isLowSide = ref(true)
const isGateOn = ref(false)
const isShortActive = ref(false)
const isFuseBlown = ref(false)
const isSwitchClosed = ref(false)

const V_SOURCE = 12.0
const R_FUSE = 0.02
const R_LOAD = 2.0
const R_SHORT = 0.03

const totalResistance = computed(() => {
  if (isFuseBlown.value) return Infinity
  if (isSwitchClosed.value || isGateOn.value) {
    if (isShortActive.value) return R_FUSE + R_SHORT
    return R_FUSE + R_LOAD
  }
  if (isShortActive.value) return R_FUSE + R_SHORT
  return R_FUSE + R_LOAD
})

const current = computed(() => {
  if (isFuseBlown.value) return 0
  return V_SOURCE / totalResistance.value
})

const fuseHeat = computed(() => current.value * current.value * R_FUSE)

watch(current, (val) => {
  if (val > 15 && !isFuseBlown.value) {
    triggerFuseBlow()
  }
})

function toggleSwitch() {
  if (isFuseBlown.value) return
  isSwitchClosed.value = !isSwitchClosed.value
  if (isSwitchClosed.value && isShortActive.value && !isFuseBlown.value) {
    triggerFuseBlow()
  }
}

function toggleShort() {
  if (isFuseBlown.value) return
  isShortActive.value = !isShortActive.value
  if (isShortActive.value && (isSwitchClosed.value || isGateOn.value) && !isFuseBlown.value) {
    if (!isLowSide.value || props.variant === 'short-circuit') {
      triggerFuseBlow()
    }
  }
}

function switchConfig(low) {
  isLowSide.value = low
  isGateOn.value = false
  isShortActive.value = false
  isFuseBlown.value = false
  isSwitchClosed.value = false
}

function triggerFuseBlow() {
  setTimeout(() => {
    isFuseBlown.value = true
  }, 400)
}

function replaceFuse() {
  isFuseBlown.value = false
  isSwitchClosed.value = false
  isShortActive.value = false
}

const wireClasses = computed(() => {
  if (isFuseBlown.value) {
    return { w1: 'wire', w2: 'wire', w3: 'wire grounded', w4: 'wire grounded', f1: 'transparent', f2: 'transparent', f3: 'transparent' }
  }
  
  if (isShortActive.value) {
    if (isSwitchClosed.value || isGateOn.value) {
      return { w1: 'wire energized', w2: 'wire grounded', w3: 'wire grounded', w4: 'wire grounded', f1: '#10b981', f2: 'transparent', f3: 'transparent' }
    }
    return { w1: 'wire energized', w2: 'wire grounded', w3: 'wire grounded', w4: 'wire grounded', f1: '#10b981', f2: 'transparent', f3: 'transparent' }
  }
  
  if (isSwitchClosed.value || isGateOn.value) {
    return { w1: 'wire energized', w2: 'wire energized', w3: 'wire energized', w4: 'wire grounded', f1: '#10b981', f2: '#10b981', f3: '#10b981' }
  }
  
  return { w1: 'wire energized', w2: 'wire energized', w3: 'wire energized', f1: 'transparent', f2: 'transparent', f3: 'transparent' }
})

const description = computed(() => {
  if (isFuseBlown.value) {
    return `<strong>¡FUSIBLE QUEMADO!</strong> La corriente se disparó a ${current.value.toFixed(0)}A, generando ${fuseHeat.value.toFixed(0)}W de calor. El elemento metálico se fundió en milisegundos. Reemplaza el fusible para continuar.`
  }
  
  if (isShortActive.value) {
    if (isSwitchClosed.value || isGateOn.value) {
      return `<strong>Corto a tierra activo:</strong> La corriente bypassa la carga completamente. ${current.value.toFixed(1)}A fluyendo a través del camino de corto.`
    }
    return `<strong>Corto a tierra (interruptor abierto):</strong> Incluso con el interruptor abierto, el corto crea un camino a tierra. La corriente fluye directamente de la batería a través del fusible.`
  }
  
  if (isSwitchClosed.value || isGateOn.value) {
    return `<strong>Circuito normal:</strong> La corriente fluye a través del fusible y la carga de ${R_LOAD}Ω. La resistencia de la carga limita la corriente a ${current.value.toFixed(1)}A de forma segura.`
  }
  
  return '<strong>Circuito abierto:</strong> No existe camino completo a tierra. Cero corriente fluye. El ventilador está apagado y el fusible está frío.'
})
</script>

<template>
  <div class="circuit-simulator">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      {{ variant === 'short-circuit' ? 'Simulador de Cortocircuitos' : 'Circuitos Low-Side vs High-Side' }}
    </div>

    <div class="sim-content">
      <div v-if="variant !== 'short-circuit'" class="config-toggle">
        <button :class="{ active: isLowSide }" @click="switchConfig(true)">Low-Side</button>
        <button :class="{ active: !isLowSide }" @click="switchConfig(false)">High-Side</button>
      </div>

      <div class="stats-grid" v-if="variant === 'short-circuit'">
        <div class="stat-card">
          <div class="stat-label">Resistencia Total</div>
          <div class="stat-val">{{ isFuseBlown ? '∞ Ω' : totalResistance.toFixed(2) + ' Ω' }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Corriente</div>
          <div class="stat-val" :class="{ 'high-alert': current > 15 }">{{ current.toFixed(1) }} A</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Calor en Fusible</div>
          <div class="stat-val" :class="{ 'high-alert': fuseHeat > 100 }">{{ fuseHeat.toFixed(1) }} W</div>
        </div>
      </div>

      <div class="circuit-card">
        <svg class="circuit-svg" viewBox="0 0 400 200">
          <!-- Battery -->
          <g transform="translate(30, 80)">
            <line x1="0" y1="0" x2="0" y2="40" stroke="#f3f4f6" stroke-width="2"/>
            <line x1="-15" y1="10" x2="15" y2="10" stroke="#f43f5e" stroke-width="4"/>
            <line x1="-8" y1="20" x2="8" y2="20" stroke="#3b82f6" stroke-width="2"/>
            <line x1="-15" y1="30" x2="15" y2="30" stroke="#f43f5e" stroke-width="4"/>
            <line x1="-8" y1="40" x2="8" y2="40" stroke="#3b82f6" stroke-width="2"/>
            <text x="20" y="8" fill="#f43f5e" font-size="10">12V</text>
          </g>
          <path d="M 30 140 L 30 160 M 20 160 L 40 160 M 24 165 L 36 165 M 28 170 L 32 170" stroke="#f3f4f6" stroke-width="2"/>

          <!-- Fuse -->
          <g transform="translate(80, 80)">
            <rect x="0" y="-10" width="30" height="20" rx="3" fill="#1e293b" stroke="#475569" stroke-width="2"/>
            <path v-if="!isFuseBlown" d="M 0 0 C 10 -10, 20 10, 30 0" fill="none" stroke="#f59e0b" stroke-width="2.5" class="fuse-element"/>
            <path v-else d="M 0 0 C 5 -10, 8 -3, 11 -8 M 18 5 L 30 0" fill="none" stroke="#ef4444" stroke-width="2.5"/>
            <text x="0" y="-15" fill="#9ca3af" font-size="9">Fusible (15A)</text>
            <!-- Spark -->
            <polygon v-if="isFuseBlown" points="15,-15 10,0 20,-5 15,10 25,0" class="spark-flash"/>
          </g>

          <!-- Load (Fan) -->
          <g transform="translate(180, 80)">
            <circle cx="20" cy="0" r="22" fill="#1e293b" stroke="#475569" stroke-width="2"/>
            <text x="20" y="4" fill="#f3f4f6" font-size="12" text-anchor="middle" font-weight="bold">{{ R_LOAD }} Ω</text>
            <text x="20" y="-28" fill="#9ca3af" font-size="9" text-anchor="middle">Ventilador</text>
          </g>

          <!-- Switch -->
          <g transform="translate(270, 80)">
            <circle cx="20" cy="0" r="18" fill="#131a2b" stroke="#4b5563" stroke-width="1.5" stroke-dasharray="3,3"/>
            <line x1="5" y1="0" x2="35" :y2="(isSwitchClosed || isGateOn) ? 0 : -12" :stroke="(isSwitchClosed || isGateOn) ? '#10b981' : '#f3f4f6'" stroke-width="3" stroke-linecap="round" :transform="(isSwitchClosed || isGateOn) ? 'rotate(18)' : 'rotate(0)'" style="transform-origin: 5px 0px"/>
            <circle cx="5" cy="0" r="3" fill="#ef4444"/>
            <circle cx="35" cy="0" r="3" fill="#3b82f6"/>
            <text x="20" y="-24" fill="#9ca3af" font-size="9" text-anchor="middle">Interruptor</text>
          </g>
          <path d="M 305 80 L 340 80 M 340 80 L 340 120 M 330 120 L 350 120 M 334 125 L 346 125 M 338 130 L 342 130" stroke="#f3f4f6" stroke-width="2"/>

          <!-- Fault Ground -->
          <g v-if="isShortActive" transform="translate(140, 80)" class="fault-ground">
            <path d="M 0 0 L 0 50 M -10 50 L 10 50 M -6 55 L 6 55 M -2 60 L 2 60" stroke="#ef4444" stroke-width="2" stroke-dasharray="3,3"/>
            <text x="15" y="48" fill="#ef4444" font-size="9">Corto a Tierra</text>
          </g>

          <!-- Wires -->
          <path d="M 30 80 L 30 40 L 80 40" :class="wireClasses.w1"/>
          <path d="M 110 40 L 180 40" :class="wireClasses.w2"/>
          <path d="M 222 40 L 275 40" :class="wireClasses.w3"/>
          <path d="M 305 40 L 340 40" :class="wireClasses.w4"/>

          <!-- Flow Animation -->
          <path d="M 30 80 L 30 40 L 80 40" class="wire flow-path" :style="{ stroke: wireClasses.f1, strokeDasharray: '6 4' }"/>
          <path d="M 110 40 L 180 40" class="wire flow-path" :style="{ stroke: wireClasses.f2, strokeDasharray: '6 4' }"/>
          <path d="M 222 40 L 275 40" class="wire flow-path" :style="{ stroke: wireClasses.f3, strokeDasharray: '6 4' }"/>
        </svg>
      </div>

      <div class="button-group">
        <button v-if="variant === 'short-circuit'" 
                :class="{ active: isSwitchClosed }" 
                :disabled="isFuseBlown"
                @click="toggleSwitch">
          {{ isSwitchClosed ? 'Abrir Interruptor' : 'Cerrar Interruptor' }}
        </button>
        <button v-else 
                :class="{ active: isGateOn }" 
                @click="isGateOn = !isGateOn">
          {{ isGateOn ? 'Desactivar Gate' : 'Activar Gate' }}
        </button>
        
        <button :class="{ 'danger-btn': isShortActive }" 
                :disabled="isFuseBlown"
                @click="toggleShort">
          {{ isShortActive ? 'Quitar Corto' : 'Inyectar Corto a Tierra' }}
        </button>

        <button v-if="isFuseBlown" class="replace-btn" @click="replaceFuse">
          Reemplazar Fusible
        </button>
      </div>

      <div class="description-box" :class="{ error: isFuseBlown }" v-html="description"></div>
    </div>
  </div>
</template>

<style scoped>
.circuit-simulator {
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

.config-toggle {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.config-toggle button {
  flex: 1;
  padding: 0.5rem;
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 0.2s;
}

.config-toggle button.active {
  background: var(--accent);
  border-color: var(--accent);
  color: white;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0.75rem;
  margin-bottom: 1rem;
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

.stat-val.high-alert {
  color: #f87171;
  animation: pulse 1s infinite alternate;
}

@keyframes pulse {
  from { transform: scale(1); }
  to { transform: scale(1.05); }
}

.circuit-card {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.circuit-svg {
  width: 100%;
  height: auto;
}

.wire {
  fill: none;
  stroke: #4b5563;
  stroke-width: 2;
  transition: stroke 0.3s ease;
}

.wire.energized {
  stroke: #10b981;
  filter: drop-shadow(0 0 3px rgba(16, 185, 129, 0.5));
}

.wire.grounded {
  stroke: #f59e0b;
}

.flow-path {
  stroke-width: 5;
  opacity: 0.7;
  animation: flow 0.4s linear infinite;
}

@keyframes flow {
  to { stroke-dashoffset: -10; }
}

.fault-ground {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fuse-element {
  filter: drop-shadow(0 0 2px rgba(245, 158, 11, 0.5));
}

.spark-flash {
  fill: #f59e0b;
  animation: spark 0.4s ease-out;
}

@keyframes spark {
  0% { opacity: 0; transform: scale(0.5); }
  50% { opacity: 1; transform: scale(2); }
  100% { opacity: 0; transform: scale(0.5); }
}

.button-group {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.button-group button {
  flex: 1;
  padding: 0.6rem;
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  color: var(--text-primary);
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.85rem;
}

.button-group button:hover:not(:disabled) {
  background: var(--border-color);
}

.button-group button.active {
  background: var(--accent);
  border-color: var(--accent);
}

.button-group button.danger-btn {
  background: #dc2626;
  border-color: #dc2626;
}

.button-group button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.button-group .replace-btn {
  background: var(--accent-success);
  border-color: var(--accent-success);
}

.description-box {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  padding: 0.75rem 1rem;
  font-size: 0.9rem;
  color: var(--text-secondary);
  line-height: 1.5;
}

.description-box.error {
  border-color: #ef4444;
  background: rgba(239, 68, 68, 0.1);
  color: #fca5a5;
}

.description-box :deep(strong) {
  color: var(--text-primary);
}
</style>
