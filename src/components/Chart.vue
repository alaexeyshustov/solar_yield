<template>
  <div class="card glass chart-card">
    <div class="chart-header">
      <div>
        <h3>Cumulative Financial Lifecycle Projection</h3>
        <p class="chart-desc">Factoring in {{ selectedPanel.degradation }}% annual panel degradation & hardware replacements</p>
      </div>
      <div class="replacements-indicators" v-if="includeBattery">
        <span class="indicator-tag battery">🔋 Battery replacement (Yr 12)</span>
        <span class="indicator-tag inverter">⚡ Inverter replacement (Yr 15)</span>
      </div>
    </div>
    
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
        <path :d="costsPath" fill="none" class="chart-path costs-line" stroke-width="2.5" />

        <!-- Intersection point (Break-even dot) -->
        <g class="intersection-dot-group" v-if="breakEvenYears <= longevity" :style="{ transform: `translate(${breakEvenCoords.x}px, ${breakEvenCoords.y}px)` }">
          <circle cx="0" cy="0" r="8" class="pulse-ring" />
          <circle cx="0" cy="0" r="5" class="center-dot" />
        </g>

        <!-- Chart Interactive Nodes (Dots for each 5-year step) -->
        <circle 
          v-for="node in chartNodes" 
          :key="node.year + '-' + node.type" 
          :cx="node.x" 
          :cy="node.y" 
          r="4.5" 
          class="node-dot"
          :class="node.type"
          @mouseenter="hoveredNode = node"
          @mouseleave="hoveredNode = null"
        />

        <!-- Tooltip overlay -->
        <foreignObject v-if="hoveredNode" :x="tooltipX" :y="tooltipY" width="135" height="75" class="chart-tooltip-wrapper">
          <div class="chart-tooltip">
            <span class="tooltip-year">Year {{ hoveredNode.year }}</span>
            <span class="tooltip-val" :class="hoveredNode.type">
              {{ hoveredNode.type === 'savings' ? 'Saved: ' : 'Cost: ' }}€{{ Math.round(hoveredNode.val).toLocaleString() }}
            </span>
            <span class="tooltip-detail" v-if="hoveredNode.year === 12 && hoveredNode.type === 'costs' && includeBattery">
              (Incl. Battery replacement)
            </span>
            <span class="tooltip-detail" v-if="hoveredNode.year === 15 && hoveredNode.type === 'costs'">
              (Incl. Inverter replacement)
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
      <span class="legend-item"><span class="legend-line costs"></span>Cumulative Cost (OPEX + Replacements)</span>
      <span class="legend-item"><span class="legend-line savings"></span>Cumulative Net Savings (Degraded)</span>
      <span class="legend-item" v-if="breakEvenYears <= longevity"><span class="legend-dot"></span>Break-Even Point</span>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  lifecycleData: Array,
  longevity: Number,
  includeBattery: Boolean,
  netUpfrontCost: Number,
  breakEvenYears: Number,
  selectedPanel: Object,
  annualOPEX: Number
});

const hoveredNode = ref(null);

// Graph Plotting Constants
const paddingX = 40;
const paddingY = 20;
const width = 440;
const height = 190;

// Max Value for Y scaling in Chart
const maxVal = computed(() => {
  const maxSavings = props.lifecycleData[props.longevity]?.savings || 10000;
  const maxCosts = props.lifecycleData[props.longevity]?.cost || 10000;
  return Math.max(maxSavings, maxCosts) * 1.15;
});

// Transform X year into SVG Coordinate
function getX(year) {
  return paddingX + (year / props.longevity) * width;
}

// Transform Y monetary value into SVG Coordinate
function getY(val) {
  const percent = val / maxVal.value;
  return paddingY + height * (1 - percent);
}

// SVG Path for Cumulative Net Savings
const savingsPath = computed(() => {
  let path = `M ${getX(0)} ${getY(0)}`;
  props.lifecycleData.forEach(d => {
    if (d.year > 0) path += ` L ${getX(d.year)} ${getY(d.savings)}`;
  });
  return path;
});

// SVG Path for Cumulative Total Cost
const costsPath = computed(() => {
  let path = `M ${getX(0)} ${getY(props.netUpfrontCost)}`;
  props.lifecycleData.forEach(d => {
    if (d.year > 0) path += ` L ${getX(d.year)} ${getY(d.cost)}`;
  });
  return path;
});

// Coordinates of the Break-Even intersection
const breakEvenCoords = computed(() => {
  if (props.breakEvenYears > props.longevity) return { x: 0, y: 0 };
  
  const yearFloor = Math.floor(props.breakEvenYears);
  const yearCeil = Math.ceil(props.breakEvenYears);
  const fraction = props.breakEvenYears - yearFloor;

  const data = props.lifecycleData;
  const costAtFloor = data[yearFloor].cost;
  const costAtCeil = data[yearCeil].cost;
  const interpCost = costAtFloor + fraction * (costAtCeil - costAtFloor);

  return {
    x: getX(props.breakEvenYears),
    y: getY(interpCost)
  };
});

// Interactive nodes for chart
const chartNodes = computed(() => {
  const nodes = [];
  const yearsToSample = [
    0,
    Math.round(props.longevity * 0.25),
    Math.round(props.longevity * 0.5),
    Math.round(props.longevity * 0.75),
    props.longevity
  ];

  if (props.includeBattery && !yearsToSample.includes(12)) yearsToSample.push(12);
  if (!yearsToSample.includes(15)) yearsToSample.push(15);
  yearsToSample.sort((a, b) => a - b);

  yearsToSample.forEach(year => {
    if (year > props.longevity) return;
    const item = props.lifecycleData[year];
    if (!item) return;

    nodes.push({
      year,
      x: getX(year),
      y: getY(item.cost),
      val: item.cost,
      type: 'costs'
    });
    nodes.push({
      year,
      x: getX(year),
      y: getY(item.savings),
      val: item.savings,
      type: 'savings'
    });
  });

  return nodes;
});

// Tooltip placement logic
const tooltipX = computed(() => {
  if (!hoveredNode.value) return 0;
  return hoveredNode.value.x > 360 ? hoveredNode.value.x - 145 : hoveredNode.value.x + 10;
});

const tooltipY = computed(() => {
  if (!hoveredNode.value) return 0;
  return hoveredNode.value.y > 180 ? hoveredNode.value.y - 80 : hoveredNode.value.y - 15;
});
</script>

<style scoped>
.chart-card {
  display: flex;
  flex-direction: column;
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1.25rem;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.chart-header h3 {
  font-size: 1.1rem;
  font-weight: 700;
}

.chart-desc {
  font-size: 0.78rem;
  color: var(--text-muted);
  margin-bottom: 1.5rem;
}

.replacements-indicators {
  display: flex;
  gap: 0.5rem;
}

.indicator-tag {
  font-size: 0.65rem;
  font-weight: 700;
  padding: 0.2rem 0.45rem;
  border-radius: 4px;
  text-transform: uppercase;
}

.indicator-tag.battery {
  background: rgba(239, 68, 68, 0.08);
  color: #fca5a5;
  border: 1px solid rgba(239, 68, 68, 0.15);
}

.indicator-tag.inverter {
  background: rgba(245, 158, 11, 0.08);
  color: #fde047;
  border: 1px solid rgba(245, 158, 11, 0.15);
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
  stroke: rgba(255, 255, 255, 0.03);
  stroke-width: 1;
}
.grid-line.border {
  stroke: rgba(255, 255, 255, 0.12);
  stroke-width: 1.5;
}

.savings-line {
  stroke: var(--secondary);
  filter: drop-shadow(0 0 6px rgba(16, 185, 129, 0.25));
}

.costs-line {
  stroke: var(--danger);
  stroke-dasharray: 3 3;
}

/* Break even dot */
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
  0% { r: 4; opacity: 1; }
  100% { r: 11; opacity: 0; }
}

/* node dots */
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

/* Tooltip overlay */
.chart-tooltip-wrapper {
  overflow: visible;
  pointer-events: none;
}

.chart-tooltip {
  background: #090d16;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 8px;
  padding: 0.45rem 0.65rem;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.6);
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
  pointer-events: none;
  animation: fade-in 0.12s ease-out;
}

.tooltip-year {
  font-size: 0.65rem;
  font-weight: 700;
  color: var(--text-muted);
  text-transform: uppercase;
}

.tooltip-val {
  font-size: 0.78rem;
  font-weight: 800;
}

.tooltip-val.savings {
  color: var(--secondary);
}

.tooltip-val.costs {
  color: var(--danger);
}

.tooltip-detail {
  font-size: 0.58rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.axis-label {
  fill: var(--text-muted);
  font-size: 0.72rem;
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
</style>
