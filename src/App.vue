<script setup>
import { computed, ref } from 'vue'

const periods = [
  { label: 'Last 12 months', value: '12' },
  { label: 'Last 24 months', value: '24' },
  { label: 'FY 2024', value: 'fy24' },
]
const regions = ['All regions', 'North Basin', 'Central Ward', 'East Mesa', 'South Valley']
const period = ref('12')
const region = ref('All regions')
const proposalOpen = ref(false)

const periodData = {
  12: {
    total: '$48.6M', awarded: '82%', applications: '38', utilization: '71%',
    spend: [5.4, 6.2, 5.8, 7.1, 6.6, 7.8, 8.3, 9.1],
    impact: [42, 48, 51, 49, 58, 64, 69, 76],
    labels: ['OCT', 'NOV', 'DEC', 'JAN', 'FEB', 'MAR', 'APR', 'MAY'],
    commitment: '$14.2M', approved: '$39.9M',
  },
  24: {
    total: '$91.4M', awarded: '78%', applications: '64', utilization: '68%',
    spend: [3.8, 4.6, 5.1, 5.5, 6.1, 6.7, 7.4, 8.2],
    impact: [28, 34, 39, 43, 45, 53, 62, 76],
    labels: ['JUN', 'SEP', 'DEC', 'MAR', 'JUN', 'SEP', 'DEC', 'MAR'],
    commitment: '$25.7M', approved: '$71.3M',
  },
  fy24: {
    total: '$42.1M', awarded: '86%', applications: '31', utilization: '74%',
    spend: [4.9, 5.8, 6.2, 6.5, 7.0, 7.6, 8.5, 9.0],
    impact: [39, 44, 50, 56, 58, 65, 71, 76],
    labels: ['JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC', 'JAN', 'FEB'],
    commitment: '$12.1M', approved: '$36.2M',
  },
}
const data = computed(() => periodData[period.value])
const regionFactor = computed(() => region.value === 'All regions' ? 1 : 0.24)
const shownTotal = computed(() => region.value === 'All regions' ? data.value.total : '$' + (parseFloat(data.value.total.slice(1)) * regionFactor.value).toFixed(1) + 'M')
const shownImpact = computed(() => data.value.impact.map((value, index) => Math.max(18, Math.round(value * (region.value === 'All regions' ? 1 : 0.8) + (index % 2)))))
const hoveredSpend = ref(null)
const hoveredClimate = ref(null)
const hoveredImpact = ref(null)
const hoveredProjection = ref(null)
const climateValues = [31, 38, 35, 48, 44, 63, 55, 77, 68, 86, 82, 94]
const climateLabels = ['JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC', 'JAN', 'FEB', 'MAR', 'APR', 'MAY']
const projectionValues = [2.4, 3.1, 2.8, 4.2, 3.8, 4.9]
const projectionLabels = ['JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV']
const fundingPoints = computed(() => data.value.spend.map((committed, index) => ({
  x: index * 114,
  committed,
  awarded: +(committed * 0.88).toFixed(1),
  committedY: Math.max(12, 250 - committed * 18),
  awardedY: Math.max(12, 250 - committed * 0.88 * 18),
})))

const fundingRows = [
  { name: 'Drought resilience & water reuse', agency: 'Water Resources', amount: '$12.8M', status: 'On track', progress: 78, tone: 'good' },
  { name: 'Neighborhood cooling network', agency: 'Public Health', amount: '$8.4M', status: 'At risk', progress: 43, tone: 'warn' },
  { name: 'Workforce re-entry pathways', agency: 'Community Services', amount: '$6.2M', status: 'On track', progress: 67, tone: 'good' },
  { name: 'Mesa solar microgrids', agency: 'Energy Office', amount: '$4.9M', status: 'Review', progress: 26, tone: 'alert' },
]
const gaps = [
  { label: 'Cooling center access', value: '−18%', note: 'South Valley · 3.2 mi avg.', tone: 'alert' },
  { label: 'Water assistance uptake', value: '−11%', note: 'East Mesa · 54% eligible', tone: 'warn' },
  { label: 'Re-entry service reach', value: '+6%', note: 'Central Ward · improving', tone: 'good' },
]

function selectPeriod(value) { period.value = value }
</script>

<template>
  <main class="app-shell">
    <header class="topbar">
      <div class="brand-lockup">
        <div class="sun-mark" aria-hidden="true"><span></span><span></span><span></span><span></span></div>
        <div>
          <p class="eyebrow">Office of Strategy &amp; Performance</p>
          <h1>Sun City <em>/</em> Funding Observatory</h1>
        </div>
      </div>
      <div class="topbar-meta">
        <span class="live-dot"></span>
        <span>Leadership view</span>
        <span class="divider"></span>
        <span>Updated 09:42 AM</span>
        <button class="icon-button" title="More options" aria-label="More options">•••</button>
      </div>
    </header>

    <section class="intro-row">
      <div>
        <p class="kicker">CITY FUNDING / 01</p>
        <h2>Where public dollars meet<br /><span>the climate moment.</span></h2>
        <p class="intro-copy">A live view of grants, proposals, and the outcomes they are creating across Sun City.</p>
      </div>
      <div class="controls">
        <label>View period
          <select v-model="period" aria-label="View period">
            <option v-for="item in periods" :key="item.value" :value="item.value">{{ item.label }}</option>
          </select>
        </label>
        <label>Region
          <select v-model="region" aria-label="Region">
            <option v-for="item in regions" :key="item">{{ item }}</option>
          </select>
        </label>
        <button class="primary-button" @click="proposalOpen = true"><span>+</span> New proposal</button>
      </div>
    </section>

    <section class="stat-strip" aria-label="Current city statistics">
      <article class="stat-card lead-stat"><p>Total funding in view <span class="info">i</span></p><strong>{{ shownTotal }}</strong><small><b class="up">↗ 14.8%</b> vs prior period</small></article>
      <article class="stat-card"><p>Funds awarded</p><strong>{{ data.awarded }}</strong><small class="muted">of committed funds</small></article>
      <article class="stat-card"><p>Active applications</p><strong>{{ data.applications }}</strong><small><b class="up">+ 7</b> since January</small></article>
      <article class="stat-card"><p>Service utilization</p><strong>{{ data.utilization }}</strong><small><b class="down">↓ 4.2%</b> needs attention</small></article>
    </section>

    <section class="dashboard-grid">
      <article class="panel spend-panel">
        <div class="panel-heading"><div><p class="kicker">FUNDING FLOW</p><h3>Committed vs. awarded</h3></div><span class="period-chip">{{ periods.find(item => item.value === period).label }}</span></div>
        <div class="chart-legend"><span><i class="legend-line coral"></i> Awarded</span><span><i class="legend-line ink"></i> Committed</span><span class="chart-total">{{ data.approved }} <small>awarded</small></span></div>
        <div class="line-chart" role="img" aria-label="Funding awarded trend chart">
          <div class="y-axis"><span>$10M</span><span>$7.5M</span><span>$5M</span><span>$2.5M</span><span>$0</span></div>
          <div class="chart-area"><div class="grid-lines"><i></i><i></i><i></i><i></i><i></i></div><svg viewBox="0 0 800 250" preserveAspectRatio="none"><polyline class="committed-path" points="0,200 114,181 228,190 342,152 456,168 570,137 684,120 800,89"/><polyline class="awarded-path" points="0,214 114,198 228,202 342,176 456,187 570,165 684,148 800,125"/><circle v-for="(point, index) in fundingPoints" :key="index" :cx="point.x" :cy="point.awardedY" r="7" class="hover-point" tabindex="0" @mouseenter="hoveredSpend = { label: data.labels[index], awarded: point.awarded, committed: point.committed }" @mouseleave="hoveredSpend = null" @focus="hoveredSpend = { label: data.labels[index], awarded: point.awarded, committed: point.committed }" @blur="hoveredSpend = null"/><circle cx="800" cy="125" r="5" class="end-dot"/></svg><div v-if="hoveredSpend" class="chart-tooltip" :style="{ left: `${(data.labels.indexOf(hoveredSpend.label) / (data.labels.length - 1)) * 100}%` }"><b>{{ hoveredSpend.label }}</b><span>Awarded {{ hoveredSpend.awarded }}M</span><span>Committed {{ hoveredSpend.committed }}M</span></div><div class="x-axis"><span v-for="label in data.labels" :key="label">{{ label }}</span></div></div>
        </div>
      </article>

      <article class="panel climate-panel">
        <div class="panel-heading"><div><p class="kicker">CLIMATE SIGNAL</p><h3>Heat &amp; water stress</h3></div><span class="alert-badge">2 signals</span></div>
        <div class="climate-main"><div class="ring-wrap"><div class="ring"><strong>74</strong><span>stress index</span></div></div><div class="climate-copy"><p>Higher than <b>89%</b> of comparable cities</p><span class="trend-label">↑ 9 points this year</span></div></div>
        <div class="mini-bars"><div v-for="(height, index) in climateValues" :key="index" class="bar" :class="{ hovered: hoveredClimate?.index === index }" :style="{ height: height + '%' }" tabindex="0" @mouseenter="hoveredClimate = { index, label: climateLabels[index], value: height }" @mouseleave="hoveredClimate = null" @focus="hoveredClimate = { index, label: climateLabels[index], value: height }" @blur="hoveredClimate = null"><span v-if="hoveredClimate?.index === index" class="bar-tooltip">{{ climateLabels[index] }} / {{ height }}</span></div></div><div v-if="hoveredClimate" class="chart-caption">{{ hoveredClimate.label }} climate stress: <b>{{ hoveredClimate.value }}</b> / 100</div><div class="mini-labels"><span>JUN</span><span>SEP</span><span>DEC</span><span>MAR</span></div>
      </article>

      <article class="panel impact-panel">
        <div class="panel-heading"><div><p class="kicker">GRANT IMPACT</p><h3>Outcomes over time</h3></div><span class="period-chip">Cumulative</span></div>
        <div class="impact-number"><strong>{{ shownImpact[shownImpact.length - 1] }}<sup>%</sup></strong><span>target outcomes reached</span></div>
        <div class="impact-chart"><svg viewBox="0 0 800 210" preserveAspectRatio="none"><defs><linearGradient id="impactFill" x1="0" x2="0" y1="0" y2="1"><stop offset="0" stop-color="#dd684e" stop-opacity=".25"/><stop offset="1" stop-color="#dd684e" stop-opacity="0"/></linearGradient></defs><polygon :points="'0,210 ' + shownImpact.map((v, i) => `${i * 114},${210 - v * 2.4}`).join(' ') + ' 800,210'" fill="url(#impactFill)"/><polyline :points="shownImpact.map((v, i) => `${i * 114},${210 - v * 2.4}`).join(' ')" fill="none" stroke="#d65f48" stroke-width="4" stroke-linecap="round" stroke-linejoin="round"/><circle v-for="(value, index) in shownImpact" :key="index" :cx="index * 114" :cy="210 - value * 2.4" r="8" class="impact-hover-point" tabindex="0" @mouseenter="hoveredImpact = { label: data.labels[index], value }" @mouseleave="hoveredImpact = null" @focus="hoveredImpact = { label: data.labels[index], value }" @blur="hoveredImpact = null"/></svg><div v-if="hoveredImpact" class="chart-tooltip impact-tooltip"><b>{{ hoveredImpact.label }}</b><span>{{ hoveredImpact.value }}% target outcomes reached</span></div></div><div class="impact-footer"><span>Baseline</span><span>Current</span><span>Target <b>80%</b></span></div>
      </article>

      <article class="panel status-panel">
        <div class="panel-heading"><div><p class="kicker">PORTFOLIO HEALTH</p><h3>Funding status</h3></div><button class="text-button">View all <span>↗</span></button></div>
        <div class="funding-table"><div class="table-header"><span>PROGRAM</span><span>AMOUNT</span><span>STATUS</span></div><div v-for="row in fundingRows" :key="row.name" class="funding-row"><div class="program-name"><strong>{{ row.name }}</strong><small>{{ row.agency }}</small><div class="progress"><i :class="row.tone" :style="{ width: row.progress + '%' }"></i></div></div><strong class="amount">{{ row.amount }}</strong><span class="status-pill" :class="row.tone">{{ row.status }}</span></div></div>
      </article>

      <article class="panel trend-panel">
        <div class="panel-heading"><div><p class="kicker">FORWARD LOOK</p><h3>Projected spending</h3></div><span class="period-chip">Next 6 months</span></div>
        <div class="projection"><strong>$22.4M</strong><span>planned deployment</span></div><div class="projection-bars"><div v-for="(month, index) in projectionLabels" :key="month" class="projection-col"><div class="projection-bar" :class="{ hovered: hoveredProjection?.index === index }" :style="{ height: [45, 58, 52, 74, 68, 89][index] + '%' }" tabindex="0" @mouseenter="hoveredProjection = { index, month, value: projectionValues[index] }" @mouseleave="hoveredProjection = null" @focus="hoveredProjection = { index, month, value: projectionValues[index] }" @blur="hoveredProjection = null"><span>${{ projectionValues[index] }}M</span></div><small>{{ month }}</small></div></div><div v-if="hoveredProjection" class="chart-caption projection-caption">{{ hoveredProjection.month }} planned deployment: <b>${{ hoveredProjection.value }}M</b></div>
      </article>

      <article class="panel gaps-panel">
        <div class="panel-heading"><div><p class="kicker">EQUITY LENS</p><h3>Service utilization gaps</h3></div><span class="alert-badge">3 to review</span></div>
        <div class="gap-list"><div v-for="gap in gaps" :key="gap.label" class="gap-row"><span class="gap-icon" :class="gap.tone">{{ gap.tone === 'good' ? '↗' : '!' }}</span><div><strong>{{ gap.label }}</strong><small>{{ gap.note }}</small></div><b :class="gap.tone">{{ gap.value }}</b></div></div>
        <div class="hypothesis"><span>!</span><p><b>Potential hypothesis</b><br />Distance to funded services may be suppressing uptake in South Valley.</p><button title="Explore hypothesis" aria-label="Explore hypothesis">↗</button></div>
      </article>
    </section>

    <footer class="footer"><span>Sun City / Office of Strategy &amp; Performance</span><span>Data refreshes every 15 minutes <i></i></span></footer>

    <div v-if="proposalOpen" class="modal-backdrop" @click.self="proposalOpen = false"><div class="modal"><button class="close-button" @click="proposalOpen = false" aria-label="Close">×</button><p class="kicker">NEW WORKFLOW</p><h2>Start a proposal</h2><p>Capture the first details and route the idea to the right program office.</p><label>Proposal name<input placeholder="e.g. South Valley shade corridors" /></label><label>Owning office<select><option>Water Resources</option><option>Community Services</option><option>Public Health</option></select></label><button class="primary-button full" @click="proposalOpen = false">Create draft <span>→</span></button></div></div>
  </main>
</template>
