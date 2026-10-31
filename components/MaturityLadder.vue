<script setup lang="ts">
/**
 * Proof of concept, pilot or production? Four steps; a train climbs from PoC to Production, confetti goes off
 * at Production, then the three facts appear one by one. Foundation is the next stop (dashed, pulsing).
 * Runs by itself when the slide is shown, no clicks. Exports show the end state.
 */
import { computed, onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'
import Confetti from './Confetti.vue'

const steps = [
  { name: 'Proof of concept', sub: '“It can be done”' },
  { name: 'Pilot', sub: 'One team, one use case' },
  { name: 'Production', sub: 'Others depend on it' },
  { name: 'Foundation', sub: 'Other systems are built on it' },
]
const facts = [
  { k: 'ERTMS', t: 'The ETCS roll-out <b>starts from DIM</b> to engineer every line.' },
  { k: 'Traffic systems', t: 'DIM is the chosen <b>successor</b> for infrastructure data to traffic management, planning and timetabling.' },
  { k: 'Next', t: 'From building the graph out of sources to <b>managing it</b> as the master of the topology.' },
]
const W = 210, GAP = 14, X0 = 8, BASE = 262, RISE = 44, TOP0 = 196
const box = (i: number) => ({ x: X0 + i * (W + GAP), y: TOP0 - i * RISE, w: W, h: BASE - (TOP0 - i * RISE) })

// the train's path over the step tops, from PoC to the middle of Production
const pts: [number, number][] = []
for (let i = 0; i <= 2; i++) {
  const b = box(i)
  pts.push([b.x + (i === 0 ? 100 : 0), b.y], [b.x + (i === 2 ? W / 2 + 46 : W), b.y])
}
const segLen = pts.slice(1).map((p, i) => Math.hypot(p[0] - pts[i][0], p[1] - pts[i][1]))
const total = segLen.reduce((a, b) => a + b, 0)
function at(t: number) {
  let d = t * total
  for (let i = 0; i < segLen.length; i++) {
    if (d <= segLen[i] || i === segLen.length - 1) {
      const f = Math.min(d / segLen[i], 1)
      const [ax, ay] = pts[i]
      const [bx, by] = pts[i + 1]
      return { x: ax + (bx - ax) * f, y: ay + (by - ay) * f, rot: Math.atan2(by - ay, bx - ax) * 180 / Math.PI }
    }
    d -= segLen[i]
  }
  return { x: pts[pts.length - 1][0], y: pts[pts.length - 1][1], rot: 0 }
}

const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const t = ref(1)
const phase = ref(3) // 0 idle · 1 climbing · 2 arrived · 3 facts
const shownFacts = ref(facts.length)
let raf = 0
const timers: ReturnType<typeof setTimeout>[] = []
function stop() { cancelAnimationFrame(raf); timers.splice(0).forEach(clearTimeout) }
watch(isActive, (active) => {
  stop()
  if (!active || isPrintMode.value) { t.value = 1; phase.value = 3; shownFacts.value = facts.length; return }
  t.value = 0; phase.value = 0; shownFacts.value = 0
  timers.push(setTimeout(() => {
    phase.value = 1
    const t0 = performance.now()
    const dur = 2600
    const tick = (now: number) => {
      const p = Math.min((now - t0) / dur, 1)
      t.value = p < 0.5 ? 2 * p * p : 1 - (-2 * p + 2) ** 2 / 2
      if (p < 1) { raf = requestAnimationFrame(tick); return }
      phase.value = 2
      facts.forEach((_, i) => timers.push(setTimeout(() => { shownFacts.value = i + 1; phase.value = 3 }, 900 + i * 700)))
    }
    raf = requestAnimationFrame(tick)
  }, 700))
}, { immediate: true })
onUnmounted(stop)
const train = computed(() => at(t.value))
const arrived = computed(() => phase.value >= 2)
</script>

<template>
  <div class="ml">
    <div class="stage">
      <svg viewBox="0 0 900 266" class="svg">
        <defs>
          <linearGradient id="ml-prod" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0" stop-color="#35DE6D" stop-opacity=".45" />
            <stop offset="1" stop-color="#35DE6D" stop-opacity=".08" />
          </linearGradient>
        </defs>
        <g v-for="(s, i) in steps" :key="s.name" class="step" :class="[`s${i}`, { prod: i === 2 && arrived, fdn: i === 3 }]" :style="{ animationDelay: `${i * 120}ms` }">
          <rect :x="box(i).x" :y="box(i).y" :width="box(i).w" :height="box(i).h" rx="10" class="blk" />
          <line :x1="box(i).x + 6" :y1="box(i).y" :x2="box(i).x + W - 6" :y2="box(i).y" class="rail" />
          <text :x="box(i).x + W / 2" :y="box(i).y + 30" text-anchor="middle" class="nm">{{ s.name }}{{ i === 2 && arrived ? ' ✓' : '' }}</text>
          <text :x="box(i).x + W / 2" :y="box(i).y + 48" text-anchor="middle" class="sb">{{ s.sub }}</text>
        </g>
        <!-- ramps between steps -->
        <line v-for="i in [0, 1]" :key="i" :x1="box(i).x + W" :y1="box(i).y" :x2="box(i + 1).x" :y2="box(i + 1).y" class="ramp" />
        <!-- the train -->
        <g :transform="`translate(${train.x} ${train.y}) rotate(${train.rot})`">
          <g transform="translate(-94 -16) scale(1.6)">
            <rect x="0" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
            <rect x="18" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
            <path d="M36 1 h14 q6 0 8 7.5 h-22 z" class="loco" />
            <rect x="49" y="2.4" width="4.5" height="2.6" rx=".6" class="win" />
            <rect x="2" y="2.6" width="12" height="2" rx=".5" class="win" />
            <rect x="20" y="2.6" width="12" height="2" rx=".5" class="win" />
            <line x1="0" y1="6.3" x2="58" y2="6.3" class="stripe" />
            <circle v-for="w in [3, 13, 21, 31, 40, 52]" :key="w" :cx="w" cy="9" r="1.3" class="wheel" />
            <circle cx="58.5" cy="6" r="1.4" class="lamp" />
          </g>
        </g>
      </svg>
      <div class="stamp" :class="{ on: arrived }">IN PRODUCTION</div>
      <Confetti :fire="arrived && !isPrintMode" :x="(box(2).x + W / 2) / 900" :y="0.35" />
    </div>

    <div class="facts">
      <div v-for="(f, i) in facts" :key="f.k" class="fact glass" :class="{ on: i < shownFacts }">
        <div class="kicker">{{ f.k }}</div>
        <div class="ft" v-html="f.t" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.ml { position: relative; }
.stage { position: relative; }
.svg { width: 100%; height: 266px; display: block; overflow: visible; }
.step { animation: rise .6s cubic-bezier(.3, 1.4, .5, 1) both; }
@keyframes rise { from { opacity: 0; transform: translateY(30px) } to { opacity: 1; transform: none } }
.blk { fill: var(--c-surface); stroke: var(--c-stroke); stroke-width: 1.2; transition: all .5s ease; }
.rail { stroke: var(--c-ink-subtle); stroke-width: 2.5; stroke-linecap: round; }
.ramp { stroke: var(--c-ink-subtle); stroke-width: 2.5; stroke-dasharray: 4 3; }
.nm { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 18px; fill: var(--c-ink); }
.sb { font-size: 12.5px; fill: var(--c-ink-mod); }
.s0 .blk, .s1 .blk { opacity: .7; }
.prod .blk { fill: url(#ml-prod); stroke: #35DE6D; stroke-width: 2.5; filter: drop-shadow(0 0 14px rgba(53, 222, 109, .55)); }
.prod .nm { fill: #35DE6D; }
.prod .rail { stroke: #35DE6D; }
.fdn .blk { fill: transparent; stroke: var(--c-sky); stroke-dasharray: 7 5; animation: pulse 2.2s ease-in-out infinite; }
.fdn .nm { fill: var(--c-sky); }
@keyframes pulse { 50% { filter: drop-shadow(0 0 10px rgba(100, 200, 250, .7)) } }
.wagon { fill: #E8EEF8; stroke: #1E2869; stroke-width: .5; }
.loco { fill: #fff; stroke: #1E2869; stroke-width: .5; }
.win { fill: #1E2869; }
.stripe { stroke: #C8102E; stroke-width: .9; }
.wheel { fill: #1E2869; }
.lamp { fill: #FFE58A; filter: drop-shadow(0 0 3px #FFE58A); }
.stamp { position: absolute; left: 47%; top: 0; transform: rotate(-8deg) scale(2.2); opacity: 0;
  font-family: 'Poppins', sans-serif; font-weight: 700; letter-spacing: .14em; font-size: 15px; color: #35DE6D;
  border: 3px solid #35DE6D; border-radius: 8px; padding: 3px 12px; transition: all .35s cubic-bezier(.3, 1.6, .5, 1); }
.stamp.on { opacity: 1; transform: rotate(-8deg) scale(1); box-shadow: 0 0 22px rgba(53, 222, 109, .45); }
.facts { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; margin-top: 14px; }
.fact { padding: 10px 14px; opacity: 0; transform: translateY(14px); transition: all .5s cubic-bezier(.3, 1.4, .5, 1); }
.fact.on { opacity: 1; transform: none; }
.ft { font-size: 13.5px; line-height: 1.4; margin-top: 4px; }
.ft :deep(b) { color: var(--c-sky); }
</style>
