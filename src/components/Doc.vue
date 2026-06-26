<template>
  <div class="doc-component glass animate-fade-in">
    <!-- Search and Filter Bar -->
    <div class="search-bar-container">
      <div class="search-input-wrapper">
        <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8" />
          <line x1="21" y1="21" x2="16.65" y2="16.65" />
        </svg>
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="Search research findings (e.g. LBO, subsidy, degradation, VAT)..."
          class="search-input"
        >
        <button 
          v-if="searchQuery" 
          @click="searchQuery = ''" 
          class="clear-search-btn"
        >
          ✕
        </button>
      </div>
      <div class="search-status" v-if="searchQuery">
        Found <strong>{{ totalMatchesCount }}</strong> matches across 
        <strong>{{ matchingSectionsCount }}</strong> sections.
      </div>
    </div>

    <!-- Main Layout -->
    <div class="doc-layout">
      <!-- Left Sidebar: Sections Nav -->
      <aside class="doc-sidebar">
        <div 
          v-for="(section, idx) in docSections" 
          :key="idx"
          class="sidebar-item"
          :class="{ active: activeSectionIdx === idx, 'has-matches': getSectionMatches(section) > 0 }"
          @click="activeSectionIdx = idx"
        >
          <span class="section-num">{{ idx + 1 }}</span>
          <span class="section-title">{{ section.shortTitle }}</span>
          <span v-if="searchQuery && getSectionMatches(section) > 0" class="match-badge">
            {{ getSectionMatches(section) }}
          </span>
        </div>
      </aside>

      <!-- Right Main Content Area -->
      <article class="doc-content">
        <div class="active-section-header">
          <h2>{{ docSections[activeSectionIdx].title }}</h2>
        </div>

        <div class="section-body">
          <div 
            v-for="(para, pIdx) in docSections[activeSectionIdx].paragraphs" 
            :key="pIdx"
            class="para-wrapper"
          >
            <!-- Check if paragraph is table representation -->
            <div v-if="para.type === 'table'" class="doc-table-container">
              <table class="market-table">
                <thead>
                  <tr>
                    <th v-for="col in para.headers" :key="col">{{ col }}</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, rIdx) in para.rows" :key="rIdx">
                    <td v-for="(val, cIdx) in row" :key="cIdx" v-html="highlightText(val)"></td>
                  </tr>
                </tbody>
              </table>
            </div>

            <!-- Check if paragraph is list representation -->
            <ul v-else-if="para.type === 'list'" class="doc-list">
              <li 
                v-for="(item, iIdx) in para.items" 
                :key="iIdx"
                v-html="highlightText(item)"
              ></li>
            </ul>

            <!-- Default Text Paragraph -->
            <p v-else v-html="highlightText(para.text)"></p>
          </div>
        </div>
      </article>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const searchQuery = ref('');
const activeSectionIdx = ref(0);

// Document Structure split into sections based on docs/research.md
const docSections = [
  {
    shortTitle: "Macroeconomic Context",
    title: "1. Macroeconomic Context: The Cost of Energy in Germany",
    paragraphs: [
      { text: "The fundamental economic driver for photovoltaic adoption in Germany is the stark disparity between the wholesale cost of electricity generation and the retail price levied on consumers. Analyzing this spread is essential for calculating the return on investment for any solar infrastructure." },
      { text: "The European electricity market has experienced extreme volatility since 2022. According to data from Eurostat covering the second half of 2025 and entering 2026, household consumers in the European Union face wildly divergent energy costs. For medium-sized household consumers, defined as utilizing between 2,500 and 5,000 kilowatt-hours (kWh) annually, the EU weighted average price stands at €0.2896 per kWh. Germany consistently ranks among the most expensive markets on the continent. In the second half of 2025, German retail prices averaged €0.3869 per kWh, placing the nation second only to Ireland (€0.4042 per kWh) and significantly ahead of nations like Sweden (€0.0970 per kWh) and Finland (€0.0748 per kWh)." },
      { text: "While wholesale prices on the European Energy Exchange (EEX) have cooled from their 2022 and 2023 peaks—stabilizing around USD 95 per megawatt-hour (MWh) or approximately €0.135 per kWh in 2026—this reduction has only partially translated to the consumer level. Retail contracts in Germany currently exhibit a bifurcation: consumers securing new contracts in the competitive market can achieve rates between €0.26 and €0.32 per kWh, whereas those remaining in basic supply (Grundversorgung) face tariffs approaching €0.40 per kWh. This stubbornly high retail baseline is maintained by fixed grid fees (Netzentgelte), municipal concession levies, and overarching transmission costs, despite the phase-out of the traditional EEG surcharge." },
      { text: "Conversely, the wholesale market in 2026 is increasingly characterized by negative pricing events. Driven by the massive influx of inflexible renewable generation during peak daylight hours, the proportion of hours with negative electricity prices in Germany has reached approximately 6% of the year. This phenomenon fundamentally alters the economics of solar arrays, shifting the financial imperative away from merely feeding power into the grid toward maximizing self-consumption and deploying smart battery storage for load shifting." }
    ]
  },
  {
    shortTitle: "Capital Expenditure (MFH)",
    title: "2. Capital Expenditure: Multi-Apartment Building Economics",
    paragraphs: [
      { text: "Deploying photovoltaic infrastructure on a multi-apartment building (Mehrfamilienhaus or MFH) introduces distinct scaling advantages. Unlike single-family homes, multi-apartment installations benefit from shared substructures, centralized high-capacity inverters, and diluted fixed costs such as scaffolding and grid connection logistics." },
      { text: "The capital expenditure required for a multi-apartment building is highly dependent on the total kilowatt-peak (kWp) capacity of the system. In 2026, the net investment costs for MFH projects are structured around significant economies of scale. For a small multi-apartment building comprising four to six residential units, a recommended system size of 15 to 20 kWp requires a total net investment ranging from €18,000 to €25,000, translating to a specific cost of €1,200 to €1,300 per kWp." },
      { text: "Scaling up to a medium-sized building housing eight to ten units necessitates a system capacity of 25 to 35 kWp. A typical 30 kWp installation in this category demands a capital outlay between €30,000 and €45,000, which reduces the specific cost to between €1,150 and €1,250 per kWp. For large residential complexes with twelve or more units requiring 40 to 60 kWp of capacity, the aggregate investment scales from €45,000 to €70,000, pushing the specific cost down to a highly efficient €1,100 to €1,200 per kWp." },
      { text: "To accurately assess the installation cost on a per-panel basis for a multi-apartment building, one must differentiate between the raw hardware procurement cost and the fully burdened, installed cost. The raw hardware cost of solar modules has plummeted to historic lows due to massive global production capacities. In 2026, a standard N-Type module with a capacity of roughly 440 watts costs between €60 and €140 in the retail market, while premium bifacial or Heterojunction modules range from €150 to €240 per unit." },
      { text: "However, the fully burdened cost per panel—which incorporates the module, its proportional share of the commercial-grade hybrid inverter, the aluminum mounting substructure, DC/AC cabling, scaffolding, electrical labor, and grid integration—is substantially higher. Taking the example of a 30 kWp system on a medium multi-apartment building costing €45,000, the system would require approximately 68 panels rated at 440 watts each. Dividing the total turnkey capital expenditure by the number of panels reveals a fully installed cost of approximately €441 to €661 per panel." },
      { text: "Landlords operating MFH solar systems have three primary regulatory pathways to monetize their generated power, each offering different amortization horizons:" },
      {
        type: "list",
        items: [
          "Tenant Electricity model (Mieterstrom): Landlords sell generated solar electricity directly to tenants at a price capped at least 10% below the local basic supply tariff. Yields 8% to 12% returns with amortization in 8 to 11 years. Operators receive a state-backed tenant electricity surcharge (Mieterstromzuschlag) ranging from 1.6 to 2.6 cents per kWh.",
          "Energy Sharing model (Gemeinschaftliche Gebäudeversorgung): Allows tenants to communally utilize solar power without the landlord assuming legal duties of an energy utility supplier. Lacks the surcharge but offers minimal bureaucratic overhead.",
          "Full Feed-in (Volleinspeisung): 100% of generated electricity is exported to the grid.commissioned in early 2026, the guaranteed state tariff is 12.35 cents per kWh for the first 10 kWp, and 10.35 cents per kWh for the remaining capacity. Amortization takes 12 to 15 years."
        ]
      }
    ]
  },
  {
    shortTitle: "Technological Landscape",
    title: "3. The Technological Landscape: Leading Solar Modules in 2026",
    paragraphs: [
      { text: "The technological baseline of the German solar market has advanced significantly. The industry has largely abandoned older P-Type Passivated Emitter and Rear Cell (PERC) technology, which suffered from higher degradation. The 2026 market is completely dominated by N-Type silicon architectures, specifically Tunnel Oxide Passivated Contact (TOPCon), Heterojunction (HJT), and All-Back-Contact (ABC) cells. These technologies suppress charge carrier recombination, limit light-induced degradation, and push module efficiencies well beyond the 22% threshold." },
      {
        type: "table",
        headers: ["Manufacturer & Model", "Technology", "Power Capacity (W)", "Efficiency (%)", "Annual Degradation", "Warranties"],
        rows: [
          ["Aiko Solar Neostar 3S+", "N-Type ABC", "455 – 485", "23.8 – 24.3%", "0.35%", "25-30 years / 88.5% at year 30"],
          ["Luxor Eco Line HJT", "N-Type HJT", "450", "23.4 – 23.6%", "0.25%", "30 years / 93.0% at year 30"],
          ["Astronergy Astro N7s", "N-Type TOPCon", "440 – 630", "23.3%", "0.40%", "25 years / Linear degradation"],
          ["Jinko Solar Tiger Neo", "N-Type TOPCon", "415 – 445", "23.0 – 23.2%", "0.40%", "25 years / Linear degradation"],
          ["Hyundai Energy CE-BF", "HJT / TOPCon", "430", "23.0%", "0.26%", "30 years / Linear degradation"],
          ["JA Solar DeepBlue 4.0 Pro", "N-Type TOPCon", "460", "22.8 – 23.0%", "0.40%", "25 years / Linear degradation"],
          ["LONGi Hi-MO X10", "N-Type TOPCon", "420 – 445", "22.8 – 23.0%", "0.40%", "25 years / Linear degradation"],
          ["Canadian Solar TOPBiHiKu6", "N-Type TOPCon", "435", "22.8%", "0.40%", "25 years / Bifacial yield +20%"],
          ["Trina Solar Vertex S+", "N-Type TOPCon", "440 – 455", "22.5%", "0.40%", "25 years / 87.4% at year 25"],
          ["Meyer Burger Glass", "HJT", "370 – 395", "20.7 – 20.9%", "0.25%", "30 years / 94.2% at year 25"]
        ]
      },
      { text: "The technical superiority of N-Type panels is most evident in their longevity. passivating layers restrict annual degradation to between 0.25% and 0.40%, compared to legacy PERC. N-type manufacturers now guarantee that modules retain between 88.5% and 93% of their initial day-one power capacity even after 30 years." }
    ]
  },
  {
    shortTitle: "Spatial Engineering",
    title: "4. Spatial Engineering: Optimizing a 100 Square Meter Roof",
    paragraphs: [
      { text: "Determining the exact number of solar panels that can be installed on a 100 square meter (sqm) roof requires synthesizing physical module dimensions, maintenance corridors, and strict regional fire safety regulations." },
      { text: "A modern high-efficiency module, such as the Aiko Neostar or Trina Vertex, standardizes around dimensions of 1.76 meters by 1.13 meters, yielding a physical footprint of approximately 1.99 to 2.0 square meters per panel. In a purely theoretical vacuum, a 100 sqm surface could accommodate precisely 50 panels. However, practical engineering dictates significant constraints." },
      { text: "In Germany, solar installations are governed by the respective state building codes (Landesbauordnungen or LBO), which mandate setbacks from neighboring property lines and firewalls to prevent flame propagation. In Saxony (Sachsen), the required distance is heavily dependent on the combustibility of the module materials. If a module incorporates combustible materials (like polymer backsheets), standard firewall distance is 1.25 meters. If it utilizes non-combustible components, primarily double-glass (Glas-Glas) modules, the setback requirement shrinks to 0.30 meters (or 0 meters if roof-parallel firewall present)." },
      {
        type: "table",
        headers: ["Federal State", "Combustible setback", "Non-Combustible (Glass-Glass) setback", "Conditions"],
        rows: [
          ["Saxony (Sachsen)", "1.25 meters", "0.30 meters", "0m if firewall present"],
          ["North Rhine-Westphalia", "0.00 meters", "0.00 meters", "Abolished in 2024"],
          ["Berlin / Bremen", "1.25 meters", "0.50 meters", "0m if firewall protrudes 30cm"],
          ["Bavaria (Bayern)", "1.25 meters", "0.50 meters", "0m for Class 1 & 2 buildings"],
          ["Hesse (Hessen)", "1.25 meters", "0.00 meters", "Complete setback waiver"]
        ]
      },
      { text: "Given structural setbacks, maintenance walkways (0.6 meters), and localized shading, a 100 sqm roof yields a realistic capacity of approximately 20 kWp. With 440W panels, this maps to between 40 and 42 active panels." }
    ]
  },
  {
    shortTitle: "Meteorological Viability",
    title: "5. Meteorological Viability: Seasonal Efficiency and Leipzig Climatology",
    paragraphs: [
      { text: "The functional efficiency of a solar array is linked to the local meteorological profile. Leipzig, located in Saxony, averages around 1,717 hours of unoccluded sunshine annually. However, climate shifts have increased solar yields: 2024 DWD records showed 1,925 hours of sunshine in Leipzig with a temperature anomaly of +3.55 Kelvin." },
      { text: "Solar generation in Germany is heavily seasonal. Between 70% and 90% of a system's total annual yield is generated between April and September. In summer months, irradiance reaches 150-220 kWh/sqm/month. In winter, this drops to 40-80 kWh/sqm/month due to the low solar zenith angle, cloud cover, and fog." },
      { text: "Modern N-Type panels have low temperature coefficients of -0.24% to -0.29% per °C. Operating at 0°C on a clear winter day, they experience voltage spikes that partially offset Photon scarcity by converting diffuse winter light at peak thermodynamic efficiency." },
      { text: "Diffuse radiation (light scattered by clouds) accounts for roughly 50% of the annual irradiance in Germany. Advanced TOPCon and HJT panels maintain 94% to 96.5% relative efficiency in weak light (200 W/m²), preventing voltage collapses and generating minor power continuously even in overcast conditions." }
    ]
  },
  {
    shortTitle: "Battery Storage",
    title: "6. Energy Storage Integration: Battery Sizing Ratios",
    paragraphs: [
      { text: "Without storage, a typical household self-consumes only 25% to 35% of its generated solar energy. Integrating battery storage increases this rate to between 65% and 75%." },
      { text: "The standard battery sizing heuristic is 0.8 to 1.5 kWh of usable storage capacity per 1 kWp of installed PV capacity. For a house consuming 4,500 kWh annually (12.3 kWh/day), assuming half is consumed after sundown, a 6.15 kWh battery is perfectly sized for standard solar self-consumption." },
      { text: "The proliferation of dynamic electricity tariffs (e.g. Tibber, Awattar) has shifted the battery paradigm. Systems can charge from the grid overnight when prices are negative or near-zero, and discharge during expensive peak hours. Projections recommend adding 2 to 5 kWh of 'headroom' to capture these dynamic winter pricing spreads." },
      {
        type: "table",
        headers: ["Number of Panels", "System Capacity", "Target Profile", "Battery (Standard)", "Battery (Dynamic)"],
        rows: [
          ["10 Panels", "~4.4 kWp", "1-2 Person (1.5k-2.5k kWh/yr)", "4.0 – 5.0 kWh", "5.0 – 8.0 kWh"],
          ["20 Panels", "~8.8 kWp", "3-4 Person (3.5k-5.0k kWh/yr)", "8.0 – 10.0 kWh", "10.0 – 13.0 kWh"],
          ["30 Panels", "~13.2 kWp", "5+ Persons or Heat Pump", "12.0 – 15.0 kWh", "15.0 – 20.0 kWh"],
          ["40+ Panels", "~17.6+ kWp", "Large Estate + EV + Heat Pump", "16.0 – 20.0 kWh", "20.0 – 30.0 kWh"]
        ]
      }
    ]
  },
  {
    shortTitle: "Lifecycle Operations",
    title: "7. Lifecycle Operations: Maintenance and Recurrent Costs",
    paragraphs: [
      { text: "While solar arrays are highly autonomous, preserving efficiency over a 25-to-30-year lifecycle requires disciplined OPEX. TÜV Rheinland recommends a visual inspection annually, with a comprehensive technical assessment every 2-4 years. Residential maintenance contracts cost between €150 and €250 annually." },
      { text: "Physical cleaning is recommended every 5 years to remove dirt, pollen, and droppings, which can create resistance hot-spots. Cleaning costs approximately €2.50 per square meter, or a €50-100 flat fee. All-risk insurance ranges from €50 to €150 annually." },
      { text: "Lithium Iron Phosphate (LiFePO4) storage batteries endure 5,000 to 10,000 cycles, providing a 10-15 year lifespan. The central inverter acts as the system's thermal workhorse and has a lifespan of 12-15 years. Owners must budget for a central hybrid inverter replacement at Year 12 to 15, costing between €1,500 and €4,000." }
    ]
  },
  {
    shortTitle: "Fiscal & Subsidies",
    title: "8. The Fiscal Framework: Taxation and Subsidies in 2026",
    paragraphs: [
      { text: "The legislative environment in Germany removes nearly all bureaucratic friction for decentralized energy producers. § 12 Abs. 3 UStG applies a 0% VAT rate (Nullsteuersatz) on the delivery, installation, and commissioning of solar components, including scaffolding, modules, inverters, and battery units." },
      { text: "Revenues from PV systems commissioned after 2022 are exempt from income tax under § 3 Nr. 72 EStG. Limits are: up to 30 kWp gross capacity for single-family homes, and up to 15 kWp per unit for multi-apartment buildings (portfolio cap of 100 kWp per taxpayer). Operators are exempt from trade tax and from filing an income-surplus calculation (Einnahmen-Überschuss-Rechnung)." },
      { text: "Local subsidies further accelerate profitability. The City of Leipzig offers grants of €300 per kWp of installed PV capacity (max €1,200) and an additional €200 per kWh of battery storage capacity (max €800). These grants stack with Sächsische Aufbaubank (SAB) state funds and low-interest KfW Program 270 loans." }
    ]
  }
];

// Helper to escape regex special characters
function escapeRegExp(string) {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Global highlight text helper
function highlightText(text) {
  if (!text) return '';
  const query = searchQuery.value.trim();
  if (!query) return text;
  
  const regex = new RegExp(`(${escapeRegExp(query)})`, 'gi');
  return text.replace(regex, '<mark class="highlight-mark">$1</mark>');
}

// Count matches inside a specific section
function getSectionMatches(section) {
  const query = searchQuery.value.trim().toLowerCase();
  if (!query) return 0;
  
  let matches = 0;
  section.paragraphs.forEach(p => {
    if (p.type === 'table') {
      p.headers.forEach(h => {
        if (h.toLowerCase().includes(query)) matches++;
      });
      p.rows.forEach(row => {
        row.forEach(cell => {
          if (cell.toLowerCase().includes(query)) matches++;
        });
      });
    } else if (p.type === 'list') {
      p.items.forEach(item => {
        const occurrences = (item.toLowerCase().match(new RegExp(escapeRegExp(query), 'g')) || []).length;
        matches += occurrences;
      });
    } else if (p.text) {
      const occurrences = (p.text.toLowerCase().match(new RegExp(escapeRegExp(query), 'g')) || []).length;
      matches += occurrences;
    }
  });
  
  return matches;
}

// Computed totals for search results
const totalMatchesCount = computed(() => {
  let total = 0;
  docSections.forEach(s => {
    total += getSectionMatches(s);
  });
  return total;
});

const matchingSectionsCount = computed(() => {
  let count = 0;
  docSections.forEach(s => {
    if (getSectionMatches(s) > 0) count++;
  });
  return count;
});
</script>

<style scoped>
.doc-component {
  display: flex;
  flex-direction: column;
  padding: 2rem;
  min-height: 550px;
}

.search-bar-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 2rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 1.5rem;
  flex-wrap: wrap;
}

.search-input-wrapper {
  position: relative;
  flex: 1;
  min-width: 280px;
  max-width: 600px;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  width: 1.15rem;
  height: 1.15rem;
  color: var(--text-muted);
}

.search-input {
  width: 100%;
  padding: 0.8rem 1rem 0.8rem 2.75rem;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 10px;
  color: var(--text-primary);
  font-size: 0.9rem;
  font-family: inherit;
  transition: all var(--transition-fast);
}

.search-input:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px var(--primary-glow);
}

.clear-search-btn {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  font-size: 0.9rem;
  transition: color var(--transition-fast);
}

.clear-search-btn:hover {
  color: var(--text-primary);
}

.search-status {
  font-size: 0.82rem;
  color: var(--text-secondary);
}

/* Layout */
.doc-layout {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 2rem;
  align-items: start;
}

.doc-sidebar {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  max-height: 550px;
  overflow-y: auto;
}

.sidebar-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.85rem 1rem;
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.01);
  border: 1px solid rgba(255, 255, 255, 0.03);
  cursor: pointer;
  transition: all var(--transition-normal);
  position: relative;
}

.sidebar-item:hover {
  background: rgba(255, 255, 255, 0.03);
  border-color: rgba(255, 255, 255, 0.08);
}

.sidebar-item.active {
  background: var(--primary-glow);
  border-color: var(--primary);
}

.sidebar-item.has-matches:not(.active) {
  border-color: rgba(245, 158, 11, 0.3);
}

.section-num {
  font-size: 0.8rem;
  font-weight: 800;
  color: var(--text-muted);
  width: 1.25rem;
}

.sidebar-item.active .section-num {
  color: var(--primary);
}

.section-title {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-secondary);
  line-height: 1.3;
}

.sidebar-item.active .section-title {
  color: var(--text-primary);
}

.match-badge {
  position: absolute;
  right: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  background: var(--primary);
  color: #03001e;
  font-size: 0.65rem;
  font-weight: 800;
  min-width: 1rem;
  height: 1rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 0.25rem;
  box-shadow: 0 0 10px var(--primary-glow);
}

/* Right Content Area */
.doc-content {
  background: rgba(0, 0, 0, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.03);
  border-radius: 16px;
  padding: 2rem;
  min-height: 450px;
}

.active-section-header {
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 1rem;
  margin-bottom: 1.5rem;
}

.active-section-header h2 {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--text-primary);
  line-height: 1.4;
}

.section-body {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

p {
  font-size: 0.92rem;
  color: var(--text-secondary);
  line-height: 1.6;
}

.doc-list {
  padding-left: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.doc-list li {
  font-size: 0.92rem;
  color: var(--text-secondary);
  line-height: 1.6;
}

.doc-table-container {
  overflow-x: auto;
  margin: 0.5rem 0;
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 10px;
}

.market-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 0.8rem;
}

.market-table th {
  color: var(--text-muted);
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.7rem;
  letter-spacing: 0.04em;
  padding: 0.65rem 0.85rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  background: rgba(0, 0, 0, 0.2);
}

.market-table td {
  padding: 0.75rem 0.85rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.03);
  color: var(--text-secondary);
}

/* Custom highlight mark */
:deep(.highlight-mark) {
  background: rgba(245, 158, 11, 0.25);
  border-bottom: 2px solid var(--primary);
  color: var(--text-primary);
  padding: 0 0.15rem;
  border-radius: 2px;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 0.25s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

/* Responsive */
@media (max-width: 900px) {
  .doc-layout {
    grid-template-columns: 1fr;
  }
}
</style>
