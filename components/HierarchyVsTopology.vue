<script setup lang="ts">
/**
 * An asset hierarchy (RDS, railML 2, SAP / Maximo) next to a topology. The tree answers "what belongs
 * to what"; only the topology answers where a train can go. A train runs over the topology all the time.
 * step 0: both models · step 1: the questions with their answers.
 */
import { computed } from 'vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const q = computed(() => props.step >= 1)
const questions = [
  { t: 'Which station does this switch belong to?', tree: true, topo: true },
  { t: 'Can a train get from track 1 to track 3?', tree: false, topo: true },
  { t: 'Which signal protects this switch?', tree: false, topo: true },
  { t: 'How far is it from the signal to the platform?', tree: false, topo: true },
]
const tree = [
  { id: 'line', x: 160, y: 22, l: 'Line Oslo–Eidsvoll' },
  { id: 'st', x: 160, y: 80, l: 'Station Kløfta', p: 'line' },
  { id: 't1', x: 60, y: 140, l: 'Track 1', p: 'st' },
  { id: 't2', x: 160, y: 140, l: 'Track 2', p: 'st' },
  { id: 't3', x: 260, y: 140, l: 'Track 3', p: 'st' },
  { id: 'sw', x: 110, y: 198, l: 'Switch 12', p: 't1' },
  { id: 'sg', x: 210, y: 198, l: 'Signal N1', p: 't2' },
]
const byId = Object.fromEntries(tree.map(n => [n.id, n]))
</script>

<template>
  <div class="hvt">
    <div class="col glass">
      <div class="kicker">Asset hierarchy</div>
      <div class="sub">RDS · railML 2 · SAP / Maximo</div>
      <svg viewBox="0 0 320 220" class="svg">
        <line v-for="n in tree.filter(n => n.p)" :key="`e${n.id}`" :x1="byId[n.p!].x" :y1="byId[n.p!].y + 10" :x2="n.x" :y2="n.y - 10" class="te" />
        <g v-for="n in tree" :key="n.id">
          <rect :x="n.x - 46" :y="n.y - 11" width="92" height="22" rx="6" class="tn" />
          <text :x="n.x" :y="n.y + 4" text-anchor="middle" class="tl">{{ n.l }}</text>
        </g>
      </svg>
    </div>

    <div class="col glass topo">
      <div class="kicker">Topology</div>
      <div class="sub">net elements · net relations · positions</div>
      <svg viewBox="0 0 320 220" class="svg">
        <path id="hvt-run" d="M 10 120 L 90 120 C 120 120 120 60 150 60 L 230 60 C 260 60 260 120 290 120 L 312 120" class="ghost" />
        <path d="M 10 120 L 312 120" class="ne" />
        <path d="M 90 120 C 120 120 120 60 150 60 L 230 60 C 260 60 260 120 290 120" class="ne" />
        <path d="M 90 120 C 120 120 120 180 150 180 L 230 180 C 260 180 260 120 290 120" class="ne" />
        <circle cx="90" cy="120" r="5" class="nr" /><circle cx="290" cy="120" r="5" class="nr" />
        <text x="190" y="52" text-anchor="middle" class="tk">track 1</text>
        <text x="190" y="112" text-anchor="middle" class="tk">track 2</text>
        <text x="190" y="196" text-anchor="middle" class="tk">track 3</text>
        <text x="90" y="142" text-anchor="middle" class="sw">switch 12</text>
        <line x1="60" y1="120" x2="60" y2="102" class="mast" /><circle cx="60" cy="98" r="4.5" class="sig" />
        <text x="60" y="88" text-anchor="middle" class="sgl">N1</text>
        <rect x="160" y="66" width="60" height="6" rx="2" class="plat" />
        <circle r="6" class="runner">
          <animateMotion dur="4s" repeatCount="indefinite" rotate="auto"><mpath href="#hvt-run" /></animateMotion>
        </circle>
      </svg>
    </div>

    <div class="qs" :class="{ on: q }">
      <div v-for="(x, i) in questions" :key="x.t" class="qrow" :style="{ transitionDelay: `${i * 180}ms` }">
        <span class="mk" :class="x.tree ? 'ok' : 'no'">{{ x.tree ? '✓' : '✗' }}</span>
        <span class="qt">{{ x.t }}</span>
        <span class="mk" :class="x.topo ? 'ok' : 'no'">{{ x.topo ? '✓' : '✗' }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.hvt { display: grid; grid-template-columns: 1fr 1fr; gap: 16px 22px; }
.col { padding: 10px 14px 4px; }
.col.topo { box-shadow: 0 0 0 1px var(--c-sky), 0 0 24px rgba(100, 200, 250, .2); }
.sub { font-size: 11.5px; color: var(--c-ink-mod); }
.svg { width: 100%; height: 170px; display: block; }
.te { stroke: var(--c-ink-subtle); stroke-width: 1.4; }
.tn { fill: var(--c-surface-2, #1F2B4D); stroke: var(--c-stroke); }
.tl { font-size: 10.5px; fill: var(--c-ink); }
.ne { fill: none; stroke: var(--c-ink); stroke-width: 3.5; stroke-linecap: round; opacity: .85; }
.ghost { fill: none; stroke: none; }
.nr { fill: var(--c-bg); stroke: var(--c-amber); stroke-width: 2.5; }
.tk { font-size: 10.5px; fill: var(--c-ink-mod); }
.sw { font-size: 10px; fill: var(--c-amber); }
.mast { stroke: var(--c-ink-mod); stroke-width: 1.5; }
.sig { fill: var(--c-red); }
.sgl { font-size: 10px; fill: var(--c-red); font-weight: 700; }
.plat { fill: var(--c-amber); opacity: .8; }
.runner { fill: var(--c-sky); filter: drop-shadow(0 0 6px var(--c-sky)); }
.qs { grid-column: 1 / span 2; display: flex; flex-direction: column; gap: 4px; }
.qrow { display: grid; grid-template-columns: 1fr 440px 1fr; align-items: center; font-size: 14px; opacity: 0; transform: translateY(6px); transition: all .4s ease; }
.qs.on .qrow { opacity: 1; transform: none; }
.qt { text-align: center; padding: 0 12px; }
.mk { font-size: 18px; font-weight: 700; text-align: center; }
.mk.ok { color: var(--c-green); } .mk.no { color: var(--c-red); }
</style>
