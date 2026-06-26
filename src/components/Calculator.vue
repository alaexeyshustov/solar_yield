<template>
  <div class="card glass summary-cards-container">
    <div class="section-title-group">
      <span class="number-badge secondary">2</span>
      <h2>Economic Analysis</h2>
    </div>

    <!-- Subsidy alert if Leipzig has subsidies -->
    <div class="subsidy-alert" v-if="totalSubsidies > 0">
      <div class="subsidy-gift-icon">🎁</div>
      <div class="subsidy-details">
        <h4>Leipzig Municipal Subsidy Applied</h4>
        <p>Capped grant of <strong>-€{{ totalSubsidies.toLocaleString() }}</strong> applied to net upfront cost (PV: €{{ Math.round(solarSubsidy) }}, Storage: €{{ Math.round(batterySubsidy) }}).</p>
      </div>
    </div>

    <!-- Dynamic Metrics Grid -->
    <div class="metrics-grid">
      <div class="metric-box">
        <span class="metric-label">Net Investment (Upfront)</span>
        <span class="metric-val">€{{ netUpfrontCost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</span>
        <span class="metric-sub">
          Gross: €{{ grossUpfrontCost.toLocaleString() }}
          <span v-if="totalSubsidies > 0"> (Subsidy: -€{{ totalSubsidies.toLocaleString() }})</span>
        </span>
      </div>

      <div class="metric-box">
        <span class="metric-label">Year 1 Solar Generation</span>
        <span class="metric-val text-primary-color">{{ Math.round(annualGeneration).toLocaleString() }} <span class="unit">kWh</span></span>
        <span class="metric-sub">
          Capacity: {{ systemKwp.toFixed(2) }} kWp
          <span class="spec-detail">• Diffuse light conversion: ~50%</span>
        </span>
      </div>

      <div class="metric-box" :class="{ warning: netSavingsYear1 <= 0 }">
        <span class="metric-label">Year 1 Net Returns</span>
        <span class="metric-val text-secondary-color" v-if="netSavingsYear1 > 0">
          +€{{ netSavingsYear1.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}
        </span>
        <span class="metric-val text-danger" v-else>
          €{{ netSavingsYear1.toLocaleString() }}
        </span>
        <span class="metric-sub" v-if="netSavingsYear1 > 0">
          Self-consumption rate: {{ Math.round(selfConsumptionRate * 100) }}% | OPEX: -€{{ Math.round(annualOPEX) }}/y
        </span>
        <span class="metric-sub text-danger" v-else>
          Annual OPEX exceeds generation value!
        </span>
      </div>
    </div>

    <!-- BREAK EVEN SHIELD -->
    <div class="break-even-shield" :class="breakEvenStatusClass">
      <div class="shield-icon">
        <svg v-if="netSavingsYear1 <= 0" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
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
</template>

<script setup>
defineProps({
  totalSubsidies: Number,
  solarSubsidy: Number,
  batterySubsidy: Number,
  netUpfrontCost: Number,
  grossUpfrontCost: Number,
  annualGeneration: Number,
  systemKwp: Number,
  netSavingsYear1: Number,
  selfConsumptionRate: Number,
  annualOPEX: Number,
  breakEvenYears: Number,
  longevity: Number,
  breakEvenStatusClass: String,
  breakEvenTitle: String,
  breakEvenDescription: String
});
</script>

<style scoped>
.section-title-group {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 1.75rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 1rem;
}

.number-badge.secondary {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1.85rem;
  height: 1.85rem;
  border-radius: 8px;
  background: var(--secondary-glow);
  color: var(--secondary);
  font-weight: 700;
  font-size: 0.9rem;
  border: 1px solid rgba(16, 185, 129, 0.25);
}

.section-title-group h2 {
  font-size: 1.25rem;
  font-weight: 700;
  letter-spacing: -0.01em;
}

.subsidy-alert {
  display: flex;
  gap: 0.85rem;
  padding: 0.85rem 1.15rem;
  border-radius: 12px;
  background: rgba(16, 185, 129, 0.05);
  border: 1px solid rgba(16, 185, 129, 0.15);
  margin-bottom: 1.25rem;
}

.subsidy-gift-icon {
  font-size: 1.5rem;
}

.subsidy-details h4 {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--secondary);
  margin-bottom: 0.15rem;
}

.subsidy-details p {
  font-size: 0.76rem;
  color: var(--text-secondary);
  line-height: 1.35;
}

.metrics-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.metric-box {
  padding: 1.25rem 1.5rem;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.04);
}

.metric-label {
  display: block;
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.04em;
  margin-bottom: 0.35rem;
}

.metric-val {
  display: block;
  font-size: 1.85rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  line-height: 1;
  margin-bottom: 0.3rem;
}

.unit {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--text-secondary);
}

.metric-sub {
  display: block;
  font-size: 0.72rem;
  color: var(--text-secondary);
  font-weight: 500;
}

.spec-detail {
  color: var(--text-muted);
  margin-left: 0.35rem;
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
  border-color: rgba(239, 68, 68, 0.1);
  background: rgba(239, 68, 68, 0.01);
}

/* Break Even Shield */
.break-even-shield {
  display: flex;
  gap: 1rem;
  padding: 1.15rem 1.35rem;
  border-radius: 14px;
  align-items: center;
}

.shield-icon {
  flex-shrink: 0;
  width: 2.5rem;
  height: 2.5rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.shield-icon svg {
  width: 1.35rem;
  height: 1.35rem;
}

.shield-text h3 {
  font-size: 1.1rem;
  font-weight: 700;
  margin-bottom: 0.15rem;
}

.shield-text p {
  font-size: 0.78rem;
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
</style>
