<template>
  <section class="card glass input-panel">
    <div class="section-title-group">
      <span class="number-badge">1</span>
      <h2>Spatial & Hardware Parameters</h2>
    </div>

    <!-- Location & Yield (Static Leipzig) -->
    <div class="form-row">
      <div class="form-group half">
        <label>Region / Solar Yield Location</label>
        <div class="static-val-container">
          Leipzig (Saxony)
        </div>
        <span class="input-hint">Standard Leipzig yield: 900 kWh/kWp annually</span>
      </div>

      <div class="form-group half">
        <label for="model">Operational Model</label>
        <div class="select-wrapper">
          <select id="model" :value="selectedModel" @change="$emit('update:selectedModel', $event.target.value)">
            <option value="self">Self-Consumption</option>
            <option value="tenant">Tenant Electricity (Mieterstrom)</option>
            <option value="feed">Full Feed-in (Volleinspeisung)</option>
          </select>
        </div>
        <span class="input-hint" v-if="selectedModel === 'self'">Maximize self-consumption</span>
        <span class="input-hint" v-if="selectedModel === 'tenant'">Surcharge + 10% tariff discount</span>
        <span class="input-hint" v-if="selectedModel === 'feed'">100% feed-in state-guaranteed</span>
      </div>
    </div>

    <!-- Roof & Fire Safety borders -->
    <div class="form-row">
      <div class="form-group half">
        <label for="roofArea">Roof Area (sqm)</label>
        <div class="input-prefix-wrapper suffix">
          <input type="number" id="roofArea" :value="roofArea" @input="$emit('update:roofArea', Number($event.target.value))" min="10" max="1000">
          <span class="suffix">m²</span>
        </div>
      </div>

      <div class="form-group half">
        <label for="firewall">Firewall Boundaries (Saxony LBO)</label>
        <div class="select-wrapper">
          <select id="firewall" :value="firewallType" @change="$emit('update:firewallType', $event.target.value)">
            <option value="none">None (Stand-alone House)</option>
            <option value="shared">Shared Borders (Terraced/MFH)</option>
          </select>
        </div>
      </div>
    </div>

    <!-- Panel selection -->
    <div class="form-group">
      <label>Solar Panel Selection (N-Type Architectures)</label>
      <div class="panel-type-grid">
        <div 
          v-for="panel in panelTypes" 
          :key="panel.id"
          class="panel-card"
          :class="{ active: selectedPanelId === panel.id }"
          @click="$emit('update:selectedPanelId', panel.id)"
        >
          <div class="panel-card-header">
            <span class="panel-badge" :class="panel.class">{{ panel.badge }}</span>
            <span class="panel-type-text">{{ panel.tech }}</span>
          </div>
          <p class="panel-name">{{ panel.name }}</p>
          <div class="panel-specs">
            <span>{{ panel.wattage }}W</span>
            <span>•</span>
            <span>{{ panel.structure }}</span>
            <span>•</span>
            <span>Deg: -{{ panel.degradation }}%</span>
          </div>
          <div class="panel-price-row">
            <span class="panel-price">€{{ panel.price }}</span>
            <span class="panel-warranty">{{ panel.warranty }}y warranty</span>
          </div>
        </div>
      </div>
      <span class="input-hint text-warning-glow" v-if="selectedPanel.structure === 'Glass-Foil' && firewallType === 'shared'">
        ⚠️ Glass-Foil panels require larger setbacks (1.25m) from firewalls in Saxony.
      </span>
      <span class="input-hint text-success-glow" v-if="selectedPanel.structure === 'Glass-Glass' && firewallType === 'shared'">
        ✅ Glass-Glass (non-combustible) permits reduced setback (0.30m) under Saxony LBO.
      </span>
    </div>

    <!-- Panel count with physical constraints -->
    <div class="form-group">
      <div class="label-val-row">
        <label for="numPanels">Number of Panels</label>
        <span class="value-display" :class="{ 'text-danger': numPanels > maxFittingPanels }">
          {{ numPanels }} / {{ maxFittingPanels }} max fit
        </span>
      </div>
      <input type="range" id="numPanels" :value="numPanels" @input="$emit('update:numPanels', Number($event.target.value))" min="5" :max="Math.max(10, maxFittingPanels)" step="1" class="range-slider">
      <div class="slider-ticks">
        <span>5</span>
        <span>{{ Math.round(maxFittingPanels * 0.5) }}</span>
        <span>{{ maxFittingPanels }} (Max Fit)</span>
      </div>
    </div>

    <!-- Household parameters -->
    <div class="form-row" v-if="selectedModel !== 'feed'">
      <div class="form-group half">
        <div class="label-val-row">
          <label for="elecPrice">Base Electricity Price</label>
          <span class="value-display">€{{ elecPrice.toFixed(2) }}</span>
        </div>
        <input type="range" id="elecPrice" :value="elecPrice" @input="$emit('update:elecPrice', Number($event.target.value))" min="0.20" max="0.50" step="0.01" class="range-slider">
        <div class="slider-ticks">
          <span>€0.20</span>
          <span>€0.35</span>
          <span>€0.50</span>
        </div>
      </div>

      <div class="form-group half">
        <div class="label-val-row">
          <label for="consumption">Annual Consumption</label>
          <span class="value-display">{{ (consumption / 1000).toFixed(1) }}k kWh</span>
        </div>
        <input type="range" id="consumption" :value="consumption" @input="$emit('update:consumption', Number($event.target.value))" min="1500" max="45000" step="500" class="range-slider">
        <div class="slider-ticks">
          <span>1.5k</span>
          <span>23k</span>
          <span>45k</span>
        </div>
      </div>
    </div>

    <!-- Battery Storage options -->
    <div class="form-group border-top">
      <div class="toggle-row">
        <label class="toggle-container">
          <input type="checkbox" :checked="includeBattery" @change="$emit('update:includeBattery', $event.target.checked)">
          <span class="checkmark"></span>
          Include Battery Storage Integration
        </label>
        <span class="badge-status" :class="includeBattery ? 'active' : 'inactive'">
          {{ includeBattery ? 'Enabled' : 'Disabled' }}
        </span>
      </div>

      <div v-if="includeBattery" class="battery-sub-panel animate-fade-in">
        <div class="form-row">
          <div class="form-group half">
            <div class="label-val-row">
              <label for="batteryCapacity">Battery Capacity</label>
              <span class="value-display">{{ batteryCapacity.toFixed(1) }} kWh</span>
            </div>
            <input type="range" id="batteryCapacity" :value="batteryCapacity" @input="$emit('update:batteryCapacity', Number($event.target.value))" min="2" max="30" step="0.5" class="range-slider">
            <div class="slider-ticks">
              <span>2 kWh</span>
              <span>16 kWh</span>
              <span>30 kWh</span>
            </div>
            <span class="input-hint text-amber">Recommended size: {{ recommendedBatterySize }}</span>
          </div>

          <div class="form-group half" v-if="selectedModel !== 'feed'">
            <label for="tariff">Electricity Tariff Model</label>
            <div class="select-wrapper">
              <select id="tariff" :value="tariffType" @change="$emit('update:tariffType', $event.target.value)">
                <option value="standard">Standard Fixed Tariff</option>
                <option value="dynamic">Dynamic Arbitrage (Tibber/Awattar)</option>
              </select>
            </div>
            <span class="input-hint" v-if="tariffType === 'dynamic'">
              🔋 Dynamic tariff adds +€120 winter grid arbitrage.
            </span>
          </div>
        </div>
      </div>
    </div>

    <!-- Other Costs -->
    <div class="form-row">
      <div class="form-group half">
        <label for="installCost">Installation & Grid Fee</label>
        <div class="input-prefix-wrapper">
          <span class="prefix">€</span>
          <input type="number" id="installCost" :value="installCost" @input="$emit('update:installCost', Number($event.target.value))" min="500">
        </div>
      </div>

      <div class="form-group half">
        <label for="longevity">System Lifespan (Years)</label>
        <input type="number" id="longevity" :value="longevity" @input="$emit('update:longevity', Number($event.target.value))" min="10" max="40">
      </div>
    </div>
  </section>
</template>

<script setup>
defineProps({
  selectedPanelId: String,
  selectedModel: String,
  roofArea: Number,
  firewallType: String,
  numPanels: Number,
  elecPrice: Number,
  consumption: Number,
  installCost: Number,
  longevity: Number,
  includeBattery: Boolean,
  batteryCapacity: Number,
  tariffType: String,
  maxFittingPanels: Number,
  recommendedBatterySize: String,
  panelTypes: Array,
  selectedPanel: Object
});

defineEmits([
  'update:selectedPanelId',
  'update:selectedModel',
  'update:roofArea',
  'update:firewallType',
  'update:numPanels',
  'update:elecPrice',
  'update:consumption',
  'update:installCost',
  'update:longevity',
  'update:includeBattery',
  'update:batteryCapacity',
  'update:tariffType'
]);
</script>

<style scoped>
.input-panel {
  padding: 2rem;
}

.section-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1.75rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 1rem;
}

.number-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1.85rem;
  height: 1.85rem;
  border-radius: 8px;
  background: var(--primary-glow);
  color: var(--primary);
  font-weight: 700;
  font-size: 0.9rem;
  border: 1px solid rgba(245, 158, 11, 0.25);
}

.section-title-group h2 {
  font-size: 1.25rem;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group.border-top {
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: 1.5rem;
}

.form-group.half {
  width: 48%;
}

.form-row {
  display: flex;
  justify-content: space-between;
}

label {
  display: block;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
}

.label-val-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.2rem;
}

.value-display {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--primary);
}

.input-hint {
  display: block;
  font-size: 0.72rem;
  color: var(--text-muted);
  margin-top: 0.35rem;
}

.text-warning-glow {
  color: #fda4af;
}

.text-success-glow {
  color: #a7f3d0;
}

/* Static and custom selectors */
.static-val-container {
  padding: 0.75rem 1rem;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  font-weight: 700;
  color: var(--primary);
  font-size: 0.9rem;
}

.select-wrapper {
  position: relative;
}

.select-wrapper::after {
  content: '▼';
  font-size: 0.65rem;
  color: var(--text-muted);
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}

select, input[type="number"] {
  width: 100%;
  padding: 0.75rem 1rem;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  color: var(--text-primary);
  font-size: 0.9rem;
  font-family: inherit;
  transition: all var(--transition-fast);
  appearance: none;
}

select:focus, input[type="number"]:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px var(--primary-glow);
}

/* Sliders */
.range-slider {
  -webkit-appearance: none;
  appearance: none;
  width: 100%;
  height: 5px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 8px;
  outline: none;
  margin: 0.4rem 0 0.2rem;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 1rem;
  height: 1rem;
  border-radius: 50%;
  background: var(--primary);
  cursor: pointer;
  border: 2px solid #03001e;
  box-shadow: 0 0 10px rgba(245, 158, 11, 0.5);
  transition: transform var(--transition-fast);
}

.range-slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
}

.slider-ticks {
  display: flex;
  justify-content: space-between;
  font-size: 0.72rem;
  color: var(--text-muted);
  font-weight: 500;
}

/* Panel selection grid */
.panel-type-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 0.75rem;
}

.panel-card {
  padding: 0.85rem;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  cursor: pointer;
  transition: all var(--transition-normal);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.panel-card:hover {
  background: rgba(255, 255, 255, 0.04);
  border-color: rgba(255, 255, 255, 0.1);
  transform: translateY(-1px);
}

.panel-card.active {
  background: rgba(245, 158, 11, 0.03);
  border-color: var(--primary);
  box-shadow: 0 4px 15px rgba(245, 158, 11, 0.08);
}

.panel-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.4rem;
}

.panel-badge {
  font-size: 0.6rem;
  font-weight: 700;
  padding: 0.1rem 0.35rem;
  border-radius: 4px;
  text-transform: uppercase;
}

.panel-type-text {
  font-size: 0.65rem;
  color: var(--text-muted);
  font-weight: 600;
}

.badge-amber {
  background: var(--primary-glow);
  color: var(--primary);
  border: 1px solid rgba(245, 158, 11, 0.2);
}

.badge-slate {
  background: rgba(148, 163, 184, 0.15);
  color: #cbd5e1;
  border: 1px solid rgba(148, 163, 184, 0.2);
}

.badge-teal {
  background: var(--secondary-glow);
  color: var(--secondary);
  border: 1px solid rgba(16, 185, 129, 0.2);
}

.panel-name {
  font-size: 0.82rem;
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1.2;
  margin-bottom: 0.4rem;
}

.panel-specs {
  display: flex;
  gap: 0.35rem;
  font-size: 0.68rem;
  color: var(--text-secondary);
  font-weight: 500;
  margin-bottom: 0.5rem;
}

.panel-price-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
  padding-top: 0.4rem;
}

.panel-price {
  font-size: 0.9rem;
  font-weight: 800;
  color: var(--text-primary);
}

.panel-warranty {
  font-size: 0.65rem;
  color: var(--text-muted);
}

/* Number inputs wrap */
.input-prefix-wrapper {
  position: relative;
}

.input-prefix-wrapper .prefix {
  position: absolute;
  left: 0.85rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  font-weight: 500;
  font-size: 0.9rem;
}

.input-prefix-wrapper input {
  padding-left: 1.85rem;
}

.input-prefix-wrapper.suffix input {
  padding-left: 0.85rem;
  padding-right: 2.25rem;
}

.input-prefix-wrapper .suffix {
  position: absolute;
  right: 0.85rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  font-weight: 500;
  font-size: 0.9rem;
}

/* Toggle Checkbox */
.toggle-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.toggle-container {
  display: flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0;
}

.toggle-container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

.checkmark {
  height: 18px;
  width: 18px;
  background-color: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  margin-right: 0.65rem;
  position: relative;
  transition: all var(--transition-fast);
}

.toggle-container:hover input ~ checkmark {
  background-color: rgba(255, 255, 255, 0.05);
}

.toggle-container input:checked ~ checkmark {
  background-color: var(--primary);
  border-color: var(--primary);
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.toggle-container input:checked ~ checkmark:after {
  display: block;
}

.toggle-container .checkmark:after {
  left: 6px;
  top: 2px;
  width: 4px;
  height: 8px;
  border: solid #03001e;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.badge-status {
  font-size: 0.65rem;
  font-weight: 700;
  padding: 0.2rem 0.5rem;
  border-radius: 6px;
  text-transform: uppercase;
}

.badge-status.active {
  background: var(--primary-glow);
  color: var(--primary);
  border: 1px solid rgba(245, 158, 11, 0.2);
}

.badge-status.inactive {
  background: rgba(255, 255, 255, 0.05);
  color: var(--text-muted);
}

.battery-sub-panel {
  background: rgba(0, 0, 0, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.03);
  border-radius: 12px;
  padding: 1rem;
  margin-top: 0.5rem;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 0.25s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@media (max-width: 640px) {
  .panel-type-grid {
    grid-template-columns: 1fr;
  }
  .form-row {
    flex-direction: column;
  }
  .form-group.half {
    width: 100%;
  }
}
</style>
