<script setup>
import { ref } from 'vue'

const isFlowActive = ref(false)

function toggleFlow() {
  isFlowActive.value = !isFlowActive.value
}
</script>

<template>
  <div class="flow-comparison">
    <div class="sim-header">
      <span class="sim-badge">INTERACTIVO</span>
      Analogía: Manguera de Jardín vs Circuito Eléctrico
    </div>

    <div class="sim-content">
      <div class="split-layout">
        <!-- Water System -->
        <div class="panel">
          <div class="panel-title">
            <span class="panel-icon">💧</span>
            Sistema de Agua
            <span class="status" :class="{ active: isFlowActive }">
              {{ isFlowActive ? 'FLUYENDO' : 'DETENIDO' }}
            </span>
          </div>
          <svg class="panel-svg" viewBox="0 0 250 120">
            <!-- Spigot -->
            <rect x="10" y="40" width="30" height="40" fill="#374151" rx="3"/>
            <text x="14" y="65" fill="#9ca3af" font-size="8">LLAVE</text>
            
            <!-- Hose -->
            <line x1="40" y1="60" x2="100" y2="60" :stroke="isFlowActive ? '#38bdf8' : '#475569'" stroke-width="8" stroke-linecap="round"/>
            
            <!-- Water flow -->
            <g v-if="isFlowActive" class="water-flow">
              <circle cx="55" cy="60" r="3" fill="#38bdf8" opacity="0.8"/>
              <circle cx="75" cy="60" r="3" fill="#38bdf8" opacity="0.8"/>
              <circle cx="95" cy="60" r="3" fill="#38bdf8" opacity="0.8"/>
            </g>
            
            <!-- Sprinkler -->
            <circle cx="120" cy="60" r="15" fill="none" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="2"/>
            <g :class="{ spinning: isFlowActive }" style="transform-origin: 120px 60px">
              <line x1="120" y1="45" x2="120" y2="50" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="2"/>
              <line x1="135" y1="60" x2="130" y2="60" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="2"/>
              <line x1="120" y1="75" x2="120" y2="70" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="2"/>
              <line x1="105" y1="60" x2="110" y2="60" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="2"/>
            </g>
            
            <!-- Drain -->
            <rect x="180" y="50" width="40" height="20" fill="#1e3a5f" rx="3"/>
            <text x="188" y="64" fill="#6b7280" font-size="7">DRENAJE</text>
          </svg>
        </div>

        <!-- Electric System -->
        <div class="panel">
          <div class="panel-title">
            <span class="panel-icon">⚡</span>
            Circuito Eléctrico
            <span class="status" :class="{ active: isFlowActive }">
              {{ isFlowActive ? 'CORRIENTE' : 'SIN FLUJO' }}
            </span>
          </div>
          <svg class="panel-svg" viewBox="0 0 250 120">
            <!-- Battery -->
            <rect x="10" y="40" width="30" height="40" fill="#f59e0b" rx="3"/>
            <text x="14" y="65" fill="#0b0f19" font-size="8">12V</text>
            
            <!-- Wire to fan -->
            <line x1="40" y1="60" x2="100" y2="60" :stroke="isFlowActive ? '#10b981' : '#4b5563'" stroke-width="3"/>
            
            <!-- Fan -->
            <circle cx="120" cy="60" r="15" fill="none" :stroke="isFlowActive ? '#3b82f6' : '#6b7280'" stroke-width="2"/>
            <text x="112" y="64" :fill="isFlowActive ? '#3b82f6' : '#6b7280'" font-size="8">FAN</text>
            
            <!-- Wire to MOSFET -->
            <line x1="135" y1="60" x2="180" y2="60" :stroke="isFlowActive ? '#10b981' : '#4b5563'" stroke-width="3"/>
            
            <!-- MOSFET -->
            <rect x="180" y="45" width="30" height="30" fill="#1e3a5f" rx="3"/>
            <line x1="185" y1="60" x2="205" y2="60" :stroke="isFlowActive ? '#10b981' : '#6b7280'" stroke-width="3"/>
            <text x="184" y="78" fill="#6b7280" font-size="6">MOSFET</text>
            
            <!-- Ground -->
            <line x1="210" y1="60" x2="230" y2="60" stroke="#6b7280" stroke-width="2"/>
            <line x1="218" y1="60" x2="218" y2="80" stroke="#6b7280" stroke-width="2"/>
            <line x1="208" y1="80" x2="228" y2="80" stroke="#6b7280" stroke-width="2"/>
            <line x1="212" y1="85" x2="224" y2="85" stroke="#6b7280" stroke-width="2"/>
          </svg>
        </div>
      </div>

      <button class="toggle-btn" :class="{ active: isFlowActive }" @click="toggleFlow">
        {{ isFlowActive ? 'Detener Flujo' : 'Iniciar Flujo' }}
      </button>

      <div class="comparison-table">
        <div class="comp-row header">
          <div class="comp-cell">Agua</div>
          <div class="comp-cell">Eléctrico</div>
          <div class="comp-cell">Función</div>
        </div>
        <div class="comp-row">
          <div class="comp-cell">Llave de paso</div>
          <div class="comp-cell">MOSFET</div>
          <div class="comp-cell">Interruptor</div>
        </div>
        <div class="comp-row">
          <div class="comp-cell">Manguera</div>
          <div class="comp-cell">Cable</div>
          <div class="comp-cell">Conductor</div>
        </div>
        <div class="comp-row">
          <div class="comp-cell">Aspersor</div>
          <div class="comp-cell">Ventilador</div>
          <div class="comp-cell">Carga</div>
        </div>
        <div class="comp-row">
          <div class="comp-cell">Drenaje</div>
          <div class="comp-cell">Tierra</div>
          <div class="comp-cell">Regreso</div>
        </div>
      </div>

      <div class="info-box">
        <strong>Regla del Circuito en Serie:</strong> La misma corriente fluye por todos los componentes en serie. Abrir la válvula (MOSFET) en cualquier punto detiene todo el flujo.
      </div>
    </div>
  </div>
</template>

<style scoped>
.flow-comparison {
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

.split-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.panel {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 10px;
  padding: 0.75rem;
}

.panel-title {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.85rem;
  font-weight: 500;
  margin-bottom: 0.5rem;
}

.panel-icon {
  font-size: 1rem;
}

.status {
  margin-left: auto;
  font-size: 0.7rem;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  background: var(--bg-tertiary);
  color: var(--text-muted);
}

.status.active {
  background: rgba(16, 185, 129, 0.2);
  color: #10b981;
}

.panel-svg {
  width: 100%;
  height: 120px;
}

.water-flow circle {
  animation: flowRight 0.8s linear infinite;
}

@keyframes flowRight {
  from { opacity: 0; transform: translateX(-10px); }
  to { opacity: 0.8; transform: translateX(10px); }
}

.spinning {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.toggle-btn {
  width: 100%;
  padding: 0.75rem;
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  color: var(--text-primary);
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  margin-bottom: 1rem;
}

.toggle-btn:hover {
  background: var(--border-color);
}

.toggle-btn.active {
  background: var(--accent);
  border-color: var(--accent);
}

.comparison-table {
  background: var(--bg-primary);
  border: 1px solid var(--border-color);
  border-radius: 8px;
  overflow: hidden;
  margin-bottom: 1rem;
}

.comp-row {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  border-bottom: 1px solid var(--border-color);
}

.comp-row:last-child {
  border-bottom: none;
}

.comp-row.header {
  background: var(--bg-tertiary);
  font-weight: 500;
  font-size: 0.85rem;
}

.comp-cell {
  padding: 0.5rem 0.75rem;
  font-size: 0.8rem;
  color: var(--text-secondary);
  border-right: 1px solid var(--border-color);
}

.comp-cell:last-child {
  border-right: none;
}

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
