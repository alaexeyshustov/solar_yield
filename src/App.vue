<template>
  <div class="app-container">
    <header class="app-header">
      <div class="logo-area">
        <svg class="sun-icon animate-spin-slow" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="12" cy="12" r="5" stroke="currentColor" stroke-width="2" />
          <path d="M12 1V3M12 21V23M4.22 4.22L5.64 5.64M18.36 18.36L19.78 19.78M1 12H3M21 12H23M4.22 19.78L5.64 18.36M18.36 5.64L19.78 4.22" stroke="currentColor" stroke-width="2" stroke-linecap="round" />
        </svg>
        <h1>HeliosYield Pro</h1>
      </div>
      <p class="subtitle">German Solar ROI & Technical Analysis (2026 Standards)</p>
    </header>

    <!-- MAIN TABS SECTION -->
    <div class="main-tabs">
      <button 
        class="tab-trigger" 
        :class="{ active: currentMainTab === 'calculator' }" 
        @click="currentMainTab = 'calculator'"
      >
        📊 Interactive ROI Calculator
      </button>
      <button 
        class="tab-trigger" 
        :class="{ active: currentMainTab === 'doc' }" 
        @click="currentMainTab = 'doc'"
      >
        📖 Market Research & LBO Guidelines
      </button>
    </div>

    <main class="calculator-grid" v-if="currentMainTab === 'calculator'">
      <!-- INPUT SECTION COMPONENT -->
      <Form
        v-model:selectedPanelId="selectedPanelId"
        v-model:selectedModel="selectedModel"
        v-model:roofArea="roofArea"
        v-model:firewallType="firewallType"
        v-model:numPanels="numPanels"
        v-model:elecPrice="elecPrice"
        v-model:consumption="consumption"
        v-model:installCost="installCost"
        v-model:longevity="longevity"
        v-model:includeBattery="includeBattery"
        v-model:batteryCapacity="batteryCapacity"
        v-model:tariffType="tariffType"
        :maxFittingPanels="maxFittingPanels"
        :recommendedBatterySize="recommendedBatterySize"
        :panelTypes="panelTypes"
        :selectedPanel="selectedPanel"
      />

      <!-- RESULTS SECTION COMPONENTS -->
      <div class="results-panel">
        <!-- Fiscal status header -->
        <div class="fiscal-header-badges">
          <div class="fiscal-badge vat">
            <span class="f-dot"></span>
            <span>0% VAT § 12 Abs. 3 UStG</span>
          </div>
          <div class="fiscal-badge tax-exempt" v-if="systemKwp <= 30">
            <span class="f-dot"></span>
            <span>Income Tax Exempt § 3 Nr. 72 EStG</span>
          </div>
        </div>

        <Calculator
          :totalSubsidies="totalSubsidies"
          :solarSubsidy="solarSubsidy"
          :batterySubsidy="batterySubsidy"
          :netUpfrontCost="netUpfrontCost"
          :grossUpfrontCost="grossUpfrontCost"
          :annualGeneration="annualGeneration"
          :systemKwp="systemKwp"
          :netSavingsYear1="netSavingsYear1"
          :selfConsumptionRate="selfConsumptionRate"
          :annualOPEX="annualOPEX"
          :breakEvenYears="breakEvenYears"
          :longevity="longevity"
          :breakEvenStatusClass="breakEvenStatusClass"
          :breakEvenTitle="breakEvenTitle"
          :breakEvenDescription="breakEvenDescription"
        />

        <Chart
          v-if="netSavingsYear1 > 0"
          :lifecycleData="lifecycleData"
          :longevity="longevity"
          :includeBattery="includeBattery"
          :netUpfrontCost="netUpfrontCost"
          :breakEvenYears="breakEvenYears"
          :selectedPanel="selectedPanel"
          :annualOPEX="annualOPEX"
        />
      </div>
    </main>

    <!-- DOCUMENTATION VIEW TAB -->
    <Doc v-if="currentMainTab === 'doc'" />

    <!-- REFERENCE DATA SECTION -->
    <section class="card glass reference-section animate-fade-in">
      <div class="reference-header" @click="showReference = !showReference">
        <div class="reference-title-group">
          <svg class="book-icon" :class="{ rotate: showReference }" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2zM22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z" />
          </svg>
          <h2>2026 German PV Market Reference Data</h2>
        </div>
        <button class="toggle-btn">{{ showReference ? 'Hide Data' : 'Show Data' }}</button>
      </div>

      <div v-if="showReference" class="reference-content animate-fade-in">
        <div class="tab-selectors">
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'panels' }" 
            @click="activeTab = 'panels'"
          >
            Solar Modules Comparison (Section 3.1)
          </button>
          <button 
            class="tab-btn" 
            :class="{ active: activeTab === 'battery' }" 
            @click="activeTab = 'battery'"
          >
            Battery Sizing Guidelines (Section 6.3)
          </button>
        </div>

        <!-- Panels Tab -->
        <div v-if="activeTab === 'panels'" class="table-container animate-fade-in">
          <table class="market-table">
            <thead>
              <tr>
                <th>Manufacturer & Model</th>
                <th>Tech Standard</th>
                <th>Capacity (W)</th>
                <th>Efficiency (%)</th>
                <th>Annual Degradation</th>
                <th>Warranties (Prod / Perf)</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="p in marketPanels" 
                :key="p.model"
                :class="{ highlighted: p.model.toLowerCase().includes(selectedPanel.id) }"
              >
                <td><strong>{{ p.model }}</strong></td>
                <td><span class="tech-tag">{{ p.tech }}</span></td>
                <td>{{ p.power }} W</td>
                <td>{{ p.efficiency }}%</td>
                <td class="text-rose">{{ p.degradation }}%</td>
                <td>{{ p.warranty }}</td>
              </tr>
            </tbody>
          </table>
          <p class="table-note">* Row highlighted in gold indicates your currently active selected panel type.</p>
        </div>

        <!-- Battery Tab -->
        <div v-if="activeTab === 'battery'" class="table-container animate-fade-in">
          <table class="market-table">
            <thead>
              <tr>
                <th>Panels Count (440W)</th>
                <th>System Size</th>
                <th>Target Household Profile</th>
                <th>Battery Size (Standard)</th>
                <th>Battery Size (Dynamic Tariff)</th>
              </tr>
            </thead>
            <tbody>
              <tr 
                v-for="b in batteryGuidelines" 
                :key="b.panels"
                :class="{ highlighted: activePanels >= b.minPanels && activePanels <= b.maxPanels }"
              >
                <td><strong>{{ b.panels }}</strong></td>
                <td>{{ b.capacity }}</td>
                <td>{{ b.profile }}</td>
                <td><span class="bat-badge std">{{ b.batteryStd }}</span></td>
                <td><span class="bat-badge dyn">{{ b.batteryDyn }}</span></td>
              </tr>
            </tbody>
          </table>
          <p class="table-note">* Row highlights dynamically based on your current installation size ({{ activePanels }} panels).</p>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import Form from './components/Form.vue';
import Calculator from './components/Calculator.vue';
import Chart from './components/Chart.vue';
import Doc from './components/Doc.vue';

const currentMainTab = ref('calculator');

// Modern N-Type silicon modules
const panelTypes = [
  { 
    id: 'aiko', 
    name: 'Aiko Solar Neostar 3S+', 
    tech: 'N-Type ABC', 
    wattage: 470, 
    price: 180, 
    degradation: 0.35, 
    warranty: 30, 
    structure: 'Glass-Glass', 
    badge: 'Premium ABC', 
    class: 'badge-amber' 
  },
  { 
    id: 'luxor', 
    name: 'Luxor Eco Line HJT', 
    tech: 'N-Type HJT', 
    wattage: 450, 
    price: 160, 
    degradation: 0.25, 
    warranty: 30, 
    structure: 'Glass-Glass', 
    badge: 'Premium HJT', 
    class: 'badge-teal' 
  },
  { 
    id: 'jinko', 
    name: 'Jinko Solar Tiger Neo', 
    tech: 'N-Type TOPCon', 
    wattage: 440, 
    price: 60, 
    degradation: 0.40, 
    warranty: 25, 
    structure: 'Glass-Foil', 
    badge: 'Mainstream', 
    class: 'badge-slate' 
  },
  { 
    id: 'trina', 
    name: 'Trina Solar Vertex S+', 
    tech: 'N-Type TOPCon', 
    wattage: 445, 
    price: 62, 
    degradation: 0.40, 
    warranty: 25, 
    structure: 'Glass-Glass', 
    badge: 'Mainstream', 
    class: 'badge-teal' 
  },
  { 
    id: 'meyer', 
    name: 'Meyer Burger Glass', 
    tech: 'HJT', 
    wattage: 380, 
    price: 150, 
    degradation: 0.25, 
    warranty: 30, 
    structure: 'Glass-Glass', 
    badge: 'Ultra-Premium', 
    class: 'badge-amber' 
  }
];

const selectedPanelId = ref(panelTypes[0].id);
const selectedPanel = computed(() => panelTypes.find(p => p.id === selectedPanelId.value));

// Input variables
const selectedModel = ref('self');
const roofArea = ref(100);
const firewallType = ref('shared');
const numPanels = ref(40);
const elecPrice = ref(0.35);
const consumption = ref(25000);
const installCost = ref(12000);
const longevity = ref(25);

// Battery parameters
const includeBattery = ref(true);
const batteryCapacity = ref(12.0);
const tariffType = ref('dynamic');

// Reference Accordion states
const showReference = ref(false);
const activeTab = ref('panels');

// Market data for reference tables (Section 3.1)
const marketPanels = [
  { model: 'Aiko Solar Neostar 3S+', tech: 'N-Type ABC', power: '455 - 485', efficiency: '23.8 - 24.3', degradation: '0.35', warranty: '25-30 years / 88.5% performance at year 30' },
  { model: 'Luxor Eco Line HJT', tech: 'N-Type HJT', power: '450', efficiency: '23.4 - 23.6', degradation: '0.25', warranty: '30 years / 93.0% performance at year 30' },
  { model: 'Astronergy Astro N7s', tech: 'N-Type TOPCon', power: '440 - 630', efficiency: '23.3', degradation: '0.40', warranty: '25 years / Linear degradation' },
  { model: 'Jinko Solar Tiger Neo', tech: 'N-Type TOPCon', power: '415 - 445', efficiency: '23.0 - 23.2', degradation: '0.40', warranty: '25 years / Linear degradation' },
  { model: 'Hyundai Energy CE-BF', tech: 'HJT / TOPCon', power: '430', efficiency: '23.0', degradation: '0.26', warranty: '30 years / Linear degradation' },
  { model: 'JA Solar DeepBlue 4.0 Pro', tech: 'N-Type TOPCon', power: '460', efficiency: '22.8 - 23.0', degradation: '0.40', warranty: '25 years / Linear degradation' },
  { model: 'LONGi Hi-MO X10', tech: 'N-Type TOPCon', power: '420 - 445', efficiency: '22.8 - 23.0', degradation: '0.40', warranty: '25 years / Linear degradation' },
  { model: 'Canadian Solar TOPBiHiKu6', tech: 'N-Type TOPCon', power: '435', efficiency: '22.8', degradation: '0.40', warranty: '25 years / Bifacial yield +20%' },
  { model: 'Trina Solar Vertex S+', tech: 'N-Type TOPCon', power: '440 - 455', efficiency: '22.5', degradation: '0.40', warranty: '25 years / 87.4% performance at year 25' },
  { model: 'Meyer Burger Glass', tech: 'HJT', power: '370 - 395', efficiency: '20.7 - 20.9', degradation: '0.25', warranty: '30 years / 94.2% performance at year 25' }
];

// Battery guidelines reference (Section 6.3)
const batteryGuidelines = [
  { panels: '10 Panels', minPanels: 0, maxPanels: 14, capacity: '~4.4 kWp', profile: '1-2 Person Apartment (1.5k-2.5k kWh/yr)', batteryStd: '4.0 – 5.0 kWh', batteryDyn: '5.0 – 8.0 kWh' },
  { panels: '20 Panels', minPanels: 15, maxPanels: 24, capacity: '~8.8 kWp', profile: '3-4 Person House (3.5k-5.0k kWh/yr)', batteryStd: '8.0 – 10.0 kWh', batteryDyn: '10.0 – 13.0 kWh' },
  { panels: '30 Panels', minPanels: 25, maxPanels: 34, capacity: '~13.2 kWp', profile: '5+ Persons or Heat Pump (6.0k-10.0k kWh/yr)', batteryStd: '12.0 – 15.0 kWh', batteryDyn: '15.0 – 20.0 kWh' },
  { panels: '40+ Panels', minPanels: 35, maxPanels: 999, capacity: '~17.6+ kWp', profile: 'Large Estate + EV + Heat Pump (>10.0k kWh/yr)', batteryStd: '16.0 – 20.0 kWh', batteryDyn: '20.0 – 30.0 kWh' }
];

/* 
  Physical Space Layout & Setback Math (Section 4)
  - Standard panel dimensions: 1.76m x 1.13m = 1.99 sqm
  - Max theoretical density: roofArea / 2.0
  - Saxon LBO setback:
    - Glass-Glass (non-combustible): 0.30m from borders (effectively minor space loss, ~10% loss)
    - Glass-Foil (combustible): 1.25m from borders (significant space loss, ~30% loss)
    - None firewall: negligible loss (~5% loss for maintenance corridors)
*/
const maxFittingPanels = computed(() => {
  const baseCount = Math.floor(roofArea.value / 2.0);
  if (firewallType.value === 'none') {
    return Math.max(1, Math.floor(baseCount * 0.90)); // 10% space loss for walkways
  }
  
  if (selectedPanel.value.structure === 'Glass-Glass') {
    return Math.max(1, Math.floor(baseCount * 0.85)); // 15% space loss with 0.3m setback
  } else {
    return Math.max(1, Math.floor(baseCount * 0.65)); // 35% space loss with 1.25m setback
  }
});

// Auto-clamp panel count to fitting limits
const activePanels = computed(() => {
  return Math.min(numPanels.value, maxFittingPanels.value);
});

// Battery sizing recommendations (Section 6)
const recommendedBatterySize = computed(() => {
  const minRatio = 0.8;
  const maxRatio = 1.5;
  const targetKwp = systemKwp.value;
  const baseRec = `${(targetKwp * minRatio).toFixed(1)} - ${(targetKwp * maxRatio).toFixed(1)} kWh`;
  if (tariffType.value === 'dynamic') {
    return `${baseRec} (+ 3kWh dynamic headroom)`;
  }
  return baseRec;
});

// System Calculations
const systemKwp = computed(() => (activePanels.value * selectedPanel.value.wattage) / 1000);
const totalPanelCost = computed(() => selectedPanel.value.price * activePanels.value);
const grossUpfrontCost = computed(() => totalPanelCost.value + installCost.value);

// Subsidies (Section 8.3 - Leipzig municipal rules)
const solarSubsidy = computed(() => {
  return Math.min(systemKwp.value * 300, 1200);
});

const batterySubsidy = computed(() => {
  if (!includeBattery.value) return 0;
  return Math.min(batteryCapacity.value * 200, 800);
});

const totalSubsidies = computed(() => solarSubsidy.value + batterySubsidy.value);
const netUpfrontCost = computed(() => Math.max(0, grossUpfrontCost.value - totalSubsidies.value));

// Generation & Self-consumption physics (Section 5 & 6)
const annualGeneration = computed(() => systemKwp.value * 900);

const selfConsumptionRate = computed(() => {
  if (selectedModel.value === 'feed') return 0;
  return includeBattery.value ? 0.70 : 0.30;
});

const usableSelfConsumedEnergy = computed(() => {
  return Math.min(annualGeneration.value * selfConsumptionRate.value, consumption.value);
});

const exportedEnergy = computed(() => {
  return Math.max(0, annualGeneration.value - usableSelfConsumedEnergy.value);
});

// Recurrent OPEX Cost Calculations (Section 7)
const annualOPEX = computed(() => {
  const mainInspection = 200;
  const cleaning = activePanels.value * 2.0;
  const insurance = 100;
  const monitoring = 60;
  return mainInspection + cleaning + insurance + monitoring;
});

// Year 1 Financial Yields
const netSavingsYear1 = computed(() => {
  let returns = 0;

  if (selectedModel.value === 'feed') {
    const first10Kwp = Math.min(systemKwp.value, 10);
    const first10Returns = first10Kwp * 900 * 0.1235;
    const remainingKwp = Math.max(0, systemKwp.value - 10);
    const remainingReturns = remainingKwp * 900 * 0.1035;
    returns = first10Returns + remainingReturns;
  } else {
    if (selectedModel.value === 'tenant') {
      const tenantPrice = elecPrice.value * 0.90;
      returns += usableSelfConsumedEnergy.value * (tenantPrice + 0.021);
    } else {
      returns += usableSelfConsumedEnergy.value * elecPrice.value;
    }

    returns += exportedEnergy.value * 0.082;

    if (includeBattery.value && tariffType.value === 'dynamic') {
      returns += 120;
    }
  }

  return returns - annualOPEX.value;
});

// Mathematical multi-year lifecycle calculation
const lifecycleData = computed(() => {
  const years = [];
  let cumulativeSavings = 0;
  
  for (let year = 0; year <= longevity.value; year++) {
    let costThisYear = 0;
    
    if (year === 0) {
      years.push({
        year: 0,
        cost: netUpfrontCost.value,
        savings: 0
      });
      continue;
    }

    costThisYear += annualOPEX.value;

    if (year === 12 && includeBattery.value) {
      costThisYear += 3500;
    }
    if (year === 15) {
      costThisYear += 2500;
    }

    const annualDegradationRate = selectedPanel.value.degradation / 100;
    const degradationMultiplier = Math.max(0.1, 1 - (year * annualDegradationRate));
    const rawSavingsThisYear = (netSavingsYear1.value + annualOPEX.value) * degradationMultiplier;

    cumulativeSavings += rawSavingsThisYear;

    const previousCost = years[year - 1].cost;
    const totalCostAccum = previousCost + costThisYear;

    years.push({
      year,
      cost: year === 0 ? netUpfrontCost.value : totalCostAccum,
      savings: cumulativeSavings
    });
  }

  return years;
});

// Detect precise fractional break-even year
const breakEvenYears = computed(() => {
  if (netSavingsYear1.value <= 0) return Infinity;

  const data = lifecycleData.value;
  let beYear = Infinity;

  for (let i = 1; i < data.length; i++) {
    const prev = data[i - 1];
    const curr = data[i];

    if (prev.savings < prev.cost && curr.savings >= curr.cost) {
      const costSpread = curr.cost - prev.cost;
      const savingsSpread = curr.savings - prev.savings;
      const progressToGoal = prev.cost - prev.savings;
      
      const fraction = progressToGoal / (savingsSpread - costSpread);
      beYear = prev.year + fraction;
      break;
    }
  }

  return beYear;
});

// Break-even status text & styles
const breakEvenStatusClass = computed(() => {
  if (netSavingsYear1.value <= 0) return 'status-negative';
  if (breakEvenYears.value > longevity.value) return 'status-warning';
  return 'status-positive';
});

const breakEvenTitle = computed(() => {
  if (netSavingsYear1.value <= 0) return 'Non-Viable';
  if (breakEvenYears.value > longevity.value) return 'Late Break-Even';
  return `${breakEvenYears.value.toFixed(1)} Years Break-Even`;
});

const breakEvenDescription = computed(() => {
  if (netSavingsYear1.value <= 0) {
    return 'Your maintenance cost exceeds electricity savings. The system will never pay for itself.';
  }
  if (breakEvenYears.value > longevity.value) {
    return `The system takes ${breakEvenYears.value.toFixed(1)} years to break even, which exceeds the panels' ${longevity.value}-year lifespan.`;
  }
  return `Your system will recover all upfront and recurrent costs in ${breakEvenYears.value.toFixed(1)} years. Afterwards, your solar production is pure profit.`;
});
</script>

<style scoped>
.main-tabs {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2.5rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 1rem;
}

.tab-trigger {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  color: var(--text-secondary);
  padding: 0.75rem 1.5rem;
  border-radius: 10px;
  font-size: 0.95rem;
  font-weight: 700;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.tab-trigger:hover {
  background: rgba(255, 255, 255, 0.05);
  color: var(--text-primary);
  border-color: rgba(255, 255, 255, 0.1);
}

.tab-trigger.active {
  background: var(--primary-glow);
  border-color: var(--primary);
  color: var(--primary);
  box-shadow: 0 0 15px rgba(245, 158, 11, 0.15);
}

.app-container {
  max-width: 1300px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem;
}

.app-header {
  text-align: center;
  margin-bottom: 2.5rem;
}

.logo-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  margin-bottom: 0.35rem;
}

.logo-area h1 {
  font-size: 2.25rem;
  font-weight: 800;
  letter-spacing: -0.04em;
  background: linear-gradient(135deg, #f59e0b 0%, #ffedd5 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.sun-icon {
  width: 2.5rem;
  height: 2.5rem;
  color: var(--primary);
  filter: drop-shadow(0 0 12px rgba(245, 158, 11, 0.4));
}

@keyframes spin-slow {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
.animate-spin-slow {
  animation: spin-slow 30s linear infinite;
}

.subtitle {
  color: var(--text-secondary);
  font-size: 1.05rem;
  font-weight: 500;
}

/* Fiscal Headers */
.fiscal-header-badges {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  margin-bottom: 0.75rem;
}

.fiscal-badge {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.35rem 0.75rem;
  border-radius: 99px;
  font-size: 0.72rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.02em;
}

.fiscal-badge.vat {
  background: rgba(16, 185, 129, 0.1);
  border: 1px solid rgba(16, 185, 129, 0.2);
  color: var(--secondary);
}

.fiscal-badge.vat .f-dot {
  background: var(--secondary);
}

.fiscal-badge.tax-exempt {
  background: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.2);
  color: var(--primary);
}

.fiscal-badge.tax-exempt .f-dot {
  background: var(--primary);
}

.f-dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
}

/* Layout Grid */
.calculator-grid {
  display: grid;
  grid-template-columns: 1.15fr 1.15fr;
  gap: 2rem;
  align-items: start;
}

.results-panel {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* REFERENCE SECTION */
.reference-section {
  margin-top: 2rem;
  padding: 1.5rem 2rem;
}

.reference-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: pointer;
}

.reference-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.book-icon {
  width: 1.5rem;
  height: 1.5rem;
  color: var(--primary);
  transition: transform var(--transition-normal);
}

.book-icon.rotate {
  transform: rotate(180deg);
}

.toggle-btn {
  background: var(--btn-secondary-bg);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: var(--text-primary);
  padding: 0.45rem 1rem;
  border-radius: 8px;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.toggle-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.15);
}

.reference-content {
  margin-top: 1.5rem;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: 1.5rem;
}

.tab-selectors {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 1.25rem;
}

.tab-btn {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.05);
  color: var(--text-secondary);
  padding: 0.5rem 1.15rem;
  border-radius: 8px;
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-fast);
}

.tab-btn:hover {
  background: rgba(255, 255, 255, 0.06);
  color: var(--text-primary);
}

.tab-btn.active {
  background: var(--primary-glow);
  border-color: var(--primary);
  color: var(--primary);
}

.table-container {
  overflow-x: auto;
}

.market-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 0.82rem;
}

.market-table th {
  color: var(--text-muted);
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.72rem;
  letter-spacing: 0.04em;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.market-table td {
  padding: 0.85rem 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.03);
  color: var(--text-primary);
  vertical-align: middle;
}

.market-table tr.highlighted {
  background: rgba(245, 158, 11, 0.04);
}

.market-table tr.highlighted td {
  border-top: 1px solid rgba(245, 158, 11, 0.15);
  border-bottom: 1px solid rgba(245, 158, 11, 0.15);
}

.tech-tag {
  font-size: 0.68rem;
  font-weight: 700;
  padding: 0.15rem 0.45rem;
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--text-secondary);
}

.text-rose {
  color: #fda4af;
}

.bat-badge {
  font-size: 0.72rem;
  font-weight: 700;
  padding: 0.2rem 0.5rem;
  border-radius: 6px;
}

.bat-badge.std {
  background: rgba(255, 255, 255, 0.05);
  color: var(--text-primary);
}

.bat-badge.dyn {
  background: var(--secondary-glow);
  color: var(--secondary);
}

.table-note {
  font-size: 0.72rem;
  color: var(--text-muted);
  margin-top: 0.75rem;
}

/* Animations */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 0.25s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

/* Responsive Layout */
@media (max-width: 1024px) {
  .calculator-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .tab-selectors {
    flex-direction: column;
  }
}
</style>
