<template>
  <div class="app-container">
    <header class="app-header">
      <div class="logo-area">
        <svg class="sun-icon animate-spin-slow" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2" />
          <path d="M12 1V3M12 21V23M4.22 4.22L5.64 5.64M18.36 18.36L19.78 19.78M1 12H3M21 12H23M4.22 19.78L5.64 18.36M18.36 5.64L19.78 4.22" stroke="currentColor" stroke-width="2" stroke-linecap="round" />
        </svg>
        <h1>HeliosYield</h1>
      </div>
      <p class="subtitle">Solar ROI & Break-Even Calculator</p>
    </header>

    <main class="calculator-grid">
      <!-- INPUT SECTION -->
      <section class="card glass input-panel">
        <div class="section-title-group">
          <span class="number-badge">1</span>
          <h2>System Parameters</h2>
        </div>

        <div class="form-group">
          <label for="city">Region / Solar Yield Location</label>
          <div class="select-wrapper">
            <select id="city" v-model="selectedCity">
              <option v-for="city in cities" :key="city.name" :value="city">
                {{ city.name }} (~{{ city.yield }} kWh/kWp)
              </option>
            </select>
          </div>
        </div>

        <div class="form-group">
          <label>Solar Panel Type (2026 Prices)</label>
          <div class="panel-type-grid">
            <div 
              v-for="panel in panelTypes" 
              :key="panel.id"
              class="panel-card"
              :class="{ active: selectedPanelId === panel.id }"
              @click="selectedPanelId = panel.id"
            >
              <div class="panel-card-header">
                <span class="panel-badge" :class="panel.class">{{ panel.badge }}</span>
                <span class="panel-price">€{{ panel.price }}</span>
              </div>
              <p class="panel-name">{{ panel.name }}</p>
              <p class="panel-power">{{ panel.wattage }}W Peak Power</p>
            </div>
          </div>
        </div>

        <div class="form-group">
          <div class="label-val-row">
            <label for="numPanels">Number of Panels</label>
            <span class="value-display">{{ numPanels }} panels</span>
          </div>
          <input type="range" id="numPanels" v-model.number="numPanels" min="5" max="150" step="5" class="range-slider">
          <div class="slider-ticks">
            <span>5</span>
            <span>75</span>
            <span>150</span>
          </div>
        </div>

        <div class="form-group">
          <div class="label-val-row">
            <label for="elecPrice">Current Electricity Price</label>
            <span class="value-display">€{{ elecPrice.toFixed(2) }} / kWh</span>
          </div>
          <input type="range" id="elecPrice" v-model.number="elecPrice" min="0.10" max="0.60" step="0.01" class="range-slider">
          <div class="slider-ticks">
            <span>€0.10</span>
            <span>€0.35</span>
            <span>€0.60</span>
          </div>
        </div>

        <div class="form-group">
          <div class="label-val-row">
            <label for="consumption">Annual Electricity Consumption</label>
            <span class="value-display">{{ consumption.toLocaleString() }} kWh</span>
          </div>
          <input type="range" id="consumption" v-model.number="consumption" min="1000" max="50000" step="500" class="range-slider">
          <div class="slider-ticks">
            <span>1k</span>
            <span>25k</span>
            <span>50k</span>
          </div>
        </div>

        <div class="form-group">
          <div class="label-val-row">
            <label for="installCost">Installation & Inverter Cost</label>
            <span class="value-display">€{{ installCost.toLocaleString() }}</span>
          </div>
          <input type="range" id="installCost" v-model.number="installCost" min="1000" max="30000" step="500" class="range-slider">
          <div class="slider-ticks">
            <span>€1k</span>
            <span>€15k</span>
            <span>€30k</span>
          </div>
        </div>

        <div class="form-row">
          <div class="form-group half">
            <label for="maintCost">Annual Maintenance</label>
            <div class="input-prefix-wrapper">
              <span class="prefix">€</span>
              <input type="number" id="maintCost" v-model.number="maintCost" min="0">
            </div>
          </div>
          <div class="form-group half">
            <label for="longevity">Lifespan (Years)</label>
            <input type="number" id="longevity" v-model.number="longevity" min="5" max="40">
          </div>
        </div>
      </section>

      <!-- RESULTS SECTION -->
      <section class="results-panel">
        <div class="card glass summary-cards-container">
          <div class="section-title-group">
            <span class="number-badge secondary">2</span>
            <h2>ROI Forecast</h2>
          </div>

          <div class="metrics-grid">
            <div class="metric-box">
              <span class="metric-label">Total Upfront Investment</span>
              <span class="metric-val">€{{ totalUpfrontCost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</span>
              <span class="metric-sub">Panels: €{{ totalPanelCost.toLocaleString() }} + Install: €{{ installCost.toLocaleString() }}</span>
            </div>

            <div class="metric-box">
              <span class="metric-label">Annual Generation</span>
              <span class="metric-val text-primary-color">{{ Math.round(annualGeneration).toLocaleString() }} <span class="unit">kWh</span></span>
              <span class="metric-sub">System Size: {{ systemKwp.toFixed(2) }} kWp</span>
            </div>

            <div class="metric-box" :class="{ warning: netSavings <= 0 }">
              <span class="metric-label">Annual Net Savings</span>
              <span class="metric-val text-secondary-color" v-if="netSavings > 0">
                +€{{ netSavings.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}
              </span>
              <span class="metric-val text-danger" v-else>
                €{{ netSavings.toLocaleString() }}
              </span>
              <span class="metric-sub" v-if="netSavings > 0">
                Self-consumption capped at {{ Math.round(Math.min(100, (consumption / annualGeneration) * 100)) }}%
              </span>
              <span class="metric-sub" v-else>
                Maintenance costs exceed savings
              </span>
            </div>
          </div>

          <!-- BREAK EVEN SHIELD -->
          <div class="break-even-shield" :class="breakEvenStatusClass">
            <div class="shield-icon">
              <svg v-if="netSavings <= 0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <svg v-else-if="breakEvenYears > longevity" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10" />
                <line x1="12" y1="8" x2="12" y2="12" />
                <line x1="12" y1="16" x2="12.01" y2="16" />
              </svg>
              <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14" stroke-linecap="round" stroke-linejoin="round"/>
                <polyline points="22 4 12 14.01 9 11.01" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
            </div>
            <div class="shield-text">
              <h3>{{ breakEvenTitle }}</h3>
              <p>{{ breakEvenDescription }}</p>
            </div>
          </div>
        </div>

        <!-- VISUAL CHART CARD -->
        <div class="card glass chart-card" v-if="netSavings > 0">
          <h3>Cumulative Cash Flow Projection</h3>
          <p class="chart-desc">Upfront cost recovery timeline</p>
          
          <div class="svg-container">
            <svg class="roi-chart" viewBox="0 0 500 240" ref="chartSvg">
              <!-- Grid lines -->
              <line x1="40" y1="20" x2="480" y2="20" class="grid-line" />
              <line x1="40" y1="70" x2="480" y2="70" class="grid-line" />
              <line x1="40" y1="120" x2="480" y2="120" class="grid-line" />
              <line x1="40" y1="170" x2="480" y2="170" class="grid-line" />
              <line x1="40" y1="210" x2="480" y2="210" class="grid-line border" />

              <!-- Savings line (Green) -->
              <path :d="savingsPath" fill="none" class="chart-path savings-line" stroke-width="3" />

              <!-- Costs line (Red) -->
              <path :d="costsPath" fill="none" class="chart-path costs-line" stroke-width="2.5" stroke-dasharray="4 2" />

              <!-- Intersection point (Break-even dot) -->
              <g class="intersection-dot-group" v-if="breakEvenYears <= longevity" :style="{ transform: `translate(${breakEvenCoords.x}px, ${breakEvenCoords.y}px)` }">
                <circle cx="0" cy="0" r="8" class="pulse-ring" />
                <circle cx="0" cy="0" r="5" class="center-dot" />
              </g>

              <!-- Chart Interactive Nodes (Dots for each 5-year step) -->
              <circle 
                v-for="node in chartNodes" 
                :key="node.year" 
                :cx="node.x" 
                :cy="node.y" 
                r="4.5" 
                class="node-dot"
                :class="node.type"
                @mouseenter="hoveredNode = node"
                @mouseleave="hoveredNode = null"
              />

              <!-- Tooltip overlay -->
              <foreignObject v-if="hoveredNode" :x="tooltipX" :y="tooltipY" width="120" height="70" class="chart-tooltip-wrapper">
                <div class="chart-tooltip">
                  <span class="tooltip-year">Year {{ hoveredNode.year }}</span>
                  <span class="tooltip-val" :class="hoveredNode.type">
                    {{ hoveredNode.type === 'savings' ? 'Saved: ' : 'Spent: ' }}€{{ Math.round(hoveredNode.val).toLocaleString() }}
                  </span>
                </div>
              </foreignObject>

              <!-- X-Axis Labels -->
              <text x="40" y="230" class="axis-label">0y</text>
              <text x="150" y="230" class="axis-label">{{ Math.round(longevity * 0.25) }}y</text>
              <text x="260" y="230" class="axis-label">{{ Math.round(longevity * 0.5) }}y</text>
              <text x="370" y="230" class="axis-label">{{ Math.round(longevity * 0.75) }}y</text>
              <text x="480" y="230" class="axis-label text-right">{{ longevity }}y</text>
            </svg>
          </div>
          <div class="legend">
            <span class="legend-item"><span class="legend-line costs"></span>Cumulative Cost</span>
            <span class="legend-item"><span class="legend-line savings"></span>Cumulative Net Savings</span>
            <span class="legend-item" v-if="breakEvenYears <= longevity"><span class="legend-dot"></span>Break-Even Point</span>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// Region data
const cities = [
  { name: 'Leipzig', yield: 900 },
  { name: 'Munich (München)', yield: 1050 },
  { name: 'Berlin', yield: 930 },
  { name: 'Hamburg', yield: 840 },
  { name: 'Freiburg', yield: 1100 }
];
const selectedCity = ref(cities[0]);

// Panel choices
const panelTypes = [
  { id: 'topcon', name: 'High-Efficiency TOPCon', wattage: 400, price: 60, badge: 'TOPCon', class: 'badge-amber' },
  { id: 'black', name: 'Full Black Design', wattage: 400, price: 66, badge: 'Black', class: 'badge-slate' },
  { id: 'perc', name: 'Mainstream PERC', wattage: 400, price: 52, badge: 'PERC', class: 'badge-teal' }
];
const selectedPanelId = ref(panelTypes[0].id);
const selectedPanel = computed(() => panelTypes.find(p => p.id === selectedPanelId.value));

// Input variables
const numPanels = ref(50);
const elecPrice = ref(0.35);
const consumption = ref(25000);
const installCost = ref(12000);
const maintCost = ref(300);
const longevity = ref(25);

// Reactive node hover
const hoveredNode = ref(null);

// Calculations
const totalPanelCost = computed(() => selectedPanel.value.price * numPanels.value);
const totalUpfrontCost = computed(() => totalPanelCost.value + installCost.value);
const systemKwp = computed(() => (numPanels.value * selectedPanel.value.wattage) / 1000);
const annualGeneration = computed(() => systemKwp.value * selectedCity.value.yield);
const usableGeneration = computed(() => Math.min(annualGeneration.value, consumption.value));
const grossSavings = computed(() => usableGeneration.value * elecPrice.value);
const netSavings = computed(() => grossSavings.value - maintCost.value);

const breakEvenYears = computed(() => {
  if (netSavings.value <= 0) return Infinity;
  return totalUpfrontCost.value / netSavings.value;
});

// Break-even status text & styles
const breakEvenStatusClass = computed(() => {
  if (netSavings.value <= 0) return 'status-negative';
  if (breakEvenYears.value > longevity.value) return 'status-warning';
  return 'status-positive';
});

const breakEvenTitle = computed(() => {
  if (netSavings.value <= 0) return 'Non-Viable';
  if (breakEvenYears.value > longevity.value) return 'Late Break-Even';
  return `${breakEvenYears.value.toFixed(1)} Years Break-Even`;
});

const breakEvenDescription = computed(() => {
  if (netSavings.value <= 0) {
    return 'Your maintenance cost exceeds electricity savings. The system will never pay for itself.';
  }
  if (breakEvenYears.value > longevity.value) {
    return `The system takes ${breakEvenYears.value.toFixed(1)} years to break even, which exceeds the panels' ${longevity.value}-year lifespan.`;
  }
  return `Your system will recover all upfront costs in ${breakEvenYears.value.toFixed(1)} years. Afterwards, your solar production is pure profit.`;
});

// Graph Plotting Constants
const paddingX = 40;
const paddingY = 20;
const width = 440;
const height = 190; // relative coordinates mapping to viewBox (0, 0, 500, 240)

// Max Value for Y scaling in Chart
const maxVal = computed(() => {
  const maxSavings = grossSavings.value * longevity.value;
  const maxCosts = totalUpfrontCost.value + (maintCost.value * longevity.value);
  return Math.max(maxSavings, maxCosts, 10000) * 1.15;
});

// Transform X year into SVG Coordinate
function getX(year) {
  return paddingX + (year / longevity.value) * width;
}

// Transform Y monetary value into SVG Coordinate
function getY(val) {
  const percent = val / maxVal.value;
  // SVG coordinates: Y starts at top (0), goes down. High value -> Low Y.
  return paddingY + height * (1 - percent);
}

// SVG Path for Cumulative Net Savings
const savingsPath = computed(() => {
  let path = `M ${getX(0)} ${getY(0)}`;
  for (let year = 1; year <= longevity.value; year++) {
    const accumSavings = year * netSavings.value;
    path += ` L ${getX(year)} ${getY(accumSavings)}`;
  }
  return path;
});

// SVG Path for Cumulative Total Cost
const costsPath = computed(() => {
  let path = `M ${getX(0)} ${getY(totalUpfrontCost.value)}`;
  for (let year = 1; year <= longevity.value; year++) {
    const accumCosts = totalUpfrontCost.value + (year * maintCost.value);
    path += ` L ${getX(year)} ${getY(accumCosts)}`;
  }
  return path;
});

// Coordinates of the Break-Even intersection
const breakEvenCoords = computed(() => {
  if (breakEvenYears.value > longevity.value) return { x: 0, y: 0 };
  const costAtBreakEven = totalUpfrontCost.value + (breakEvenYears.value * maintCost.value);
  return {
    x: getX(breakEvenYears.value),
    y: getY(costAtBreakEven)
  };
});

// Nodes for the interactive graph (sample at 0%, 25%, 50%, 75%, 100% of lifespan)
const chartNodes = computed(() => {
  const nodes = [];
  const yearsToSample = [
    0,
    Math.round(longevity.value * 0.25),
    Math.round(longevity.value * 0.5),
    Math.round(longevity.value * 0.75),
    longevity.value
  ];

  yearsToSample.forEach(year => {
    // Costs node
    const costVal = totalUpfrontCost.value + (year * maintCost.value);
    nodes.push({
      year,
      x: getX(year),
      y: getY(costVal),
      val: costVal,
      type: 'costs'
    });
    // Savings node
    const savingsVal = year * netSavings.value;
    nodes.push({
      year,
      x: getX(year),
      y: getY(savingsVal),
      val: savingsVal,
      type: 'savings'
    });
  });

  return nodes;
});

// Tooltip placement logic
const tooltipX = computed(() => {
  if (!hoveredNode.value) return 0;
  return hoveredNode.value.x > 380 ? hoveredNode.value.x - 130 : hoveredNode.value.x + 10;
});

const tooltipY = computed(() => {
  if (!hoveredNode.value) return 0;
  return hoveredNode.value.y > 180 ? hoveredNode.value.y - 75 : hoveredNode.value.y - 15;
});
</script>

<style scoped>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 3rem 1.5rem;
}

.app-header {
  text-align: center;
  margin-bottom: 3.5rem;
}

.logo-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  margin-bottom: 0.5rem;
}

.logo-area h1 {
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: -0.05em;
  background: linear-gradient(135deg, #f59e0b 0%, #ffedd5 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.sun-icon {
  width: 2.75rem;
  height: 2.75rem;
  color: var(--primary);
  filter: drop-shadow(0 0 12px rgba(245, 158, 11, 0.4));
}

@keyframes spin-slow {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
.animate-spin-slow {
  animation: spin-slow 24s linear infinite;
}

.subtitle {
  color: var(--text-secondary);
  font-size: 1.1rem;
  font-weight: 500;
  letter-spacing: 0.05em;
}

.calculator-grid {
  display: grid;
  grid-template-columns: 1.1fr 1.2fr;
  gap: 2rem;
  align-items: start;
}

.card {
  padding: 2.25rem;
}

.section-title-group {
  display: flex;
  align-items: center;
  gap: 0.85rem;
  margin-bottom: 2rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  padding-bottom: 1.25rem;
}

.number-badge {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2rem;
  height: 2rem;
  border-radius: 8px;
  background: var(--primary-glow);
  color: var(--primary);
  font-weight: 700;
  font-size: 0.95rem;
  border: 1px solid rgba(245, 158, 11, 0.2);
}

.number-badge.secondary {
  background: var(--secondary-glow);
  color: var(--secondary);
  border: 1px solid rgba(16, 185, 129, 0.2);
}

.section-title-group h2 {
  font-size: 1.35rem;
  font-weight: 700;
  letter-spacing: -0.02em;
}

.form-group {
  margin-bottom: 1.85rem;
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
  font-size: 0.88rem;
  font-weight: 600;
  color: var(--text-secondary);
  margin-bottom: 0.65rem;
}

.label-val-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.35rem;
}

.value-display {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--primary);
}

/* Beautiful custom selectors */
.select-wrapper {
  position: relative;
}

.select-wrapper::after {
  content: '↓';
  font-size: 0.75rem;
  color: var(--text-secondary);
  position: absolute;
  right: 1.15rem;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}

select, input[type="number"] {
  width: 100%;
  padding: 0.85rem 1.15rem;
  background: rgba(0, 0, 0, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
  color: var(--text-primary);
  font-size: 0.95rem;
  font-family: inherit;
  transition: all var(--transition-fast);
  appearance: none;
}

select:focus, input[type="number"]:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px var(--primary-glow);
}

/* Range slider styling */
.range-slider {
  -webkit-appearance: none;
  width: 100%;
  height: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  outline: none;
  margin: 0.6rem 0 0.4rem;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 1.15rem;
  height: 1.15rem;
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
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 500;
}

/* Panel selection grid */
.panel-type-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0.75rem;
}

.panel-card {
  padding: 1rem 0.85rem;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.06);
  cursor: pointer;
  transition: all var(--transition-normal);
}

.panel-card:hover {
  background: rgba(255, 255, 255, 0.04);
  border-color: rgba(255, 255, 255, 0.12);
  transform: translateY(-2px);
}

.panel-card.active {
  background: rgba(245, 158, 11, 0.05);
  border-color: var(--primary);
  box-shadow: 0 4px 20px rgba(245, 158, 11, 0.1);
}

.panel-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.6rem;
}

.panel-badge {
  font-size: 0.65rem;
  font-weight: 700;
  padding: 0.15rem 0.45rem;
  border-radius: 5px;
  text-transform: uppercase;
  letter-spacing: 0.02em;
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

.panel-price {
  font-size: 0.88rem;
  font-weight: 700;
  color: var(--text-primary);
}

.panel-name {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-secondary);
  margin-bottom: 0.15rem;
  line-height: 1.25;
}

.panel-power {
  font-size: 0.72rem;
  color: var(--text-muted);
}

/* Input numbers wrapping */
.input-prefix-wrapper {
  position: relative;
}

.input-prefix-wrapper .prefix {
  position: absolute;
  left: 1.15rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-secondary);
  font-weight: 500;
  font-size: 0.95rem;
}

.input-prefix-wrapper input {
  padding-left: 2.25rem;
}

/* RESULTS & FORECASTS */
.results-panel {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.metrics-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1.15rem;
  margin-bottom: 1.75rem;
}

.metric-box {
  padding: 1.5rem;
  border-radius: 16px;
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.metric-label {
  display: block;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.45rem;
}

.metric-val {
  display: block;
  font-size: 2rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  line-height: 1;
  margin-bottom: 0.35rem;
}

.unit {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-secondary);
}

.metric-sub {
  display: block;
  font-size: 0.75rem;
  color: var(--text-muted);
  font-weight: 500;
}

.text-primary-color {
  color: var(--primary);
}

.text-secondary-color {
  color: var(--secondary);
  text-shadow: 0 0 15px rgba(16, 185, 129, 0.3);
}

.text-danger {
  color: var(--danger);
}

.metric-box.warning {
  border-color: rgba(239, 68, 68, 0.15);
  background: rgba(239, 68, 68, 0.02);
}

/* Shield indicators for break-even */
.break-even-shield {
  display: flex;
  gap: 1.15rem;
  padding: 1.25rem 1.5rem;
  border-radius: 16px;
  align-items: center;
}

.shield-icon {
  flex-shrink: 0;
  width: 2.75rem;
  height: 2.75rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.shield-icon svg {
  width: 1.5rem;
  height: 1.5rem;
}

.shield-text h3 {
  font-size: 1.15rem;
  font-weight: 700;
  margin-bottom: 0.15rem;
}

.shield-text p {
  font-size: 0.82rem;
  line-height: 1.35;
}

.status-positive {
  background: rgba(16, 185, 129, 0.08);
  border: 1px solid rgba(16, 185, 129, 0.25);
}
.status-positive .shield-icon {
  background: var(--secondary-glow);
  color: var(--secondary);
  box-shadow: 0 0 15px rgba(16, 185, 129, 0.2);
}
.status-positive .shield-text h3 {
  color: var(--secondary);
}
.status-positive .shield-text p {
  color: #cbd5e1;
}

.status-warning {
  background: rgba(245, 158, 11, 0.08);
  border: 1px solid rgba(245, 158, 11, 0.25);
}
.status-warning .shield-icon {
  background: var(--primary-glow);
  color: var(--primary);
  box-shadow: 0 0 15px rgba(245, 158, 11, 0.2);
}
.status-warning .shield-text h3 {
  color: var(--primary);
}
.status-warning .shield-text p {
  color: #cbd5e1;
}

.status-negative {
  background: rgba(239, 68, 68, 0.08);
  border: 1px solid rgba(239, 68, 68, 0.25);
}
.status-negative .shield-icon {
  background: rgba(239, 68, 68, 0.15);
  color: var(--danger);
  box-shadow: 0 0 15px rgba(239, 68, 68, 0.2);
}
.status-negative .shield-text h3 {
  color: var(--danger);
}
.status-negative .shield-text p {
  color: #fca5a5;
}

/* CHART CONTAINER */
.chart-card {
  display: flex;
  flex-direction: column;
}

.chart-card h3 {
  font-size: 1.15rem;
  font-weight: 700;
  margin-bottom: 0.15rem;
}

.chart-desc {
  font-size: 0.78rem;
  color: var(--text-muted);
  margin-bottom: 1.5rem;
}

.svg-container {
  position: relative;
  width: 100%;
}

.roi-chart {
  width: 100%;
  height: auto;
  overflow: visible;
}

.grid-line {
  stroke: rgba(255, 255, 255, 0.04);
  stroke-width: 1;
}
.grid-line.border {
  stroke: rgba(255, 255, 255, 0.15);
  stroke-width: 1.5;
}

.chart-path {
  stroke-linecap: round;
  stroke-linejoin: round;
}

.savings-line {
  stroke: var(--secondary);
  filter: drop-shadow(0 0 8px rgba(16, 185, 129, 0.3));
}

.costs-line {
  stroke: var(--danger);
  opacity: 0.75;
}

/* pulsing intersection dot */
.intersection-dot-group {
  transition: transform var(--transition-normal);
}

.pulse-ring {
  fill: var(--primary-glow);
  stroke: var(--primary);
  stroke-width: 1.5;
  animation: pulse-glow 2s infinite;
  transform-origin: center;
}

.center-dot {
  fill: var(--text-primary);
  stroke: var(--primary);
  stroke-width: 2.5;
}

@keyframes pulse-glow {
  0% { r: 5; opacity: 1; }
  100% { r: 12; opacity: 0; }
}

/* chart interactive node dots */
.node-dot {
  fill: #03001e;
  stroke-width: 2;
  cursor: pointer;
  transition: r var(--transition-fast), stroke-width var(--transition-fast);
}

.node-dot.costs {
  stroke: var(--danger);
}

.node-dot.savings {
  stroke: var(--secondary);
}

.node-dot:hover {
  r: 6.5;
  stroke-width: 3.5;
  fill: var(--text-primary);
}

/* Chart Tooltip */
.chart-tooltip-wrapper {
  overflow: visible;
  pointer-events: none;
}

.chart-tooltip {
  background: #0f172a;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 8px;
  padding: 0.5rem 0.75rem;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
  pointer-events: none;
  animation: fade-in 0.15s ease-out;
}

@keyframes fade-in {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}

.tooltip-year {
  font-size: 0.7rem;
  font-weight: 700;
  color: var(--text-secondary);
  text-transform: uppercase;
}

.tooltip-val {
  font-size: 0.8rem;
  font-weight: 800;
}

.tooltip-val.savings {
  color: var(--secondary);
}

.tooltip-val.costs {
  color: var(--danger);
}

.axis-label {
  fill: var(--text-muted);
  font-size: 0.75rem;
  font-weight: 600;
  text-anchor: middle;
}

.text-right {
  text-anchor: end;
}

/* Legend styling */
.legend {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  margin-top: 1.5rem;
  flex-wrap: wrap;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.78rem;
  font-weight: 600;
  color: var(--text-secondary);
}

.legend-line {
  width: 1.25rem;
  height: 3px;
  border-radius: 2px;
}

.legend-line.costs {
  background: var(--danger);
  border-bottom: 2px dashed rgba(255, 255, 255, 0.3);
}

.legend-line.savings {
  background: var(--secondary);
}

.legend-dot {
  width: 0.6rem;
  height: 0.6rem;
  border-radius: 50%;
  background: var(--text-primary);
  border: 2px solid var(--primary);
}

/* Responsive grid layout */
@media (max-width: 900px) {
  .calculator-grid {
    grid-template-columns: 1fr;
  }
  
  .app-container {
    padding: 2rem 1rem;
  }
}

@media (max-width: 500px) {
  .panel-type-grid {
    grid-template-columns: 1fr;
  }
  
  .form-row {
    flex-direction: column;
    gap: 1.25rem;
  }
  
  .form-group.half {
    width: 100%;
  }
}
</style>
