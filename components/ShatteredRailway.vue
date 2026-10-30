<script setup lang="ts">
/**
 * The opening pitch. step 0: the railway as one complete model, a graph of distinct concepts.
 * step 1: computers came and we dropped it; the pieces scatter. step 2: we picked up the pieces we liked,
 * glued them together our own way and called them use cases (note how many silos carry their own track segment).
 * step 3: DIM puts the complete picture back together.
 */
import { computed } from 'vue'

const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const s = computed(() => Math.max(0, Math.min(3, props.step)))

// the concepts and relations of the "why railway data needs a knowledge graph" drawing, at its layout
const concepts: [string, number, number][] = [
  ['VerticalSlope', 565, 85], ['Maintenance centre', 810, 92], ['Road', 965, 143], ['Government', 1143, 40],
  ['PPIP', 1428, 40], ['HorizontalCurve', 367, 193], ['Track segment', 711, 300], ['Investment project', 1287, 193],
  ['LevelCrossing', 1010, 319], ['Grid operator', 1500, 262], ['SpeedSection', 181, 405], ['Train', 494, 483],
  ['Client', 705, 464], ['Signal', 840, 485], ['Power distribution shelter', 1227, 405], ['Substation', 1510, 470],
  ['PlatformEdge', 160, 599], ['Switch heating', 1123, 577], ['Client ', 1298, 553], ['Switch', 936, 646],
  ['Siding', 1272, 692], ['Station', 110, 781], ['AccountingSection', 460, 754], ['SectionInsulator', 750, 876],
]
const rel: [string, string, string][] = [
  ['Government', 'Investment project', 'pays for'], ['PPIP', 'Investment project', 'pays for'],
  ['Investment project', 'LevelCrossing', 'plans renewal on'], ['Investment project', 'Power distribution shelter', 'plans renewal on'],
  ['Maintenance centre', 'Track segment', 'has work order on'], ['Maintenance centre', 'LevelCrossing', 'has work order on'],
  ['Road', 'LevelCrossing', 'crosses'], ['VerticalSlope', 'Track segment', 'gives gradient to'],
  ['HorizontalCurve', 'Track segment', 'curves'], ['SpeedSection', 'HorizontalCurve', 'reduced speed because'],
  ['Train', 'HorizontalCurve', 'max curvature & cant'], ['Train', 'SpeedSection', 'respects'],
  ['Train', 'Track segment', 'is now on'], ['Signal', 'Track segment', 'is on'],
  ['Signal', 'LevelCrossing', 'protects'], ['Signal', 'Switch', 'protects'],
  ['Power distribution shelter', 'Signal', 'powers'], ['Power distribution shelter', 'Switch heating', 'powers'],
  ['Substation', 'Power distribution shelter', 'powers'], ['Grid operator', 'Substation', 'charges grid access of'],
  ['Client', 'Train', 'pays for'], ['Train', 'PlatformEdge', 'planned to stop at'], ['PlatformEdge', 'Station', 'part of'],
  ['Train', 'AccountingSection', 'benefits'], ['Signal', 'AccountingSection', 'generates costs on'],
  ['Switch', 'AccountingSection', 'generates costs on'], ['Switch heating', 'Switch', 'part of'],
  ['Switch', 'Siding', 'gives access to'], ['Client ', 'Siding', 'pays for'], ['Substation', 'Siding', 'powers'],
  ['SectionInsulator', 'Switch', 'is on diverging track of'], ['SectionInsulator', 'Substation', 'separates sections from'],
]
const rnd = (i: number, salt: number) => {
  const v = Math.sin(i * 91.7 + salt * 47.3) * 43758.5453
  return v - Math.floor(v)
}
const W = 940, H = 360, CX = W / 2, CY = H / 2 + 6, DX = 852, DY = 322
const nodes = concepts.map(([l, rx, ry], i) => {
  const x = 50 + (rx - 110) * (840 / 1400)
  const y = 16 + (ry - 40) * (330 / 836)
  const dx = x - CX, dy = y - CY
  const d = Math.hypot(dx, dy) || 1
  const push = 70 + rnd(i, 3) * 120
  return {
    i, l: l.trim(), id: l, x, y,
    sx: x + (dx / d) * push + (rnd(i, 4) - 0.5) * 60,
    sy: y + (dy / d) * push * 0.6 + 70 + rnd(i, 5) * 90,
    sr: (rnd(i, 6) - 0.5) * 120,
  }
})
const byId = (id: string) => nodes.find(n => n.id === id)!
const edges: [number, number, string][] = rel.map(([a, b, l]) => [byId(a).i, byId(b).i, l])

const clusters = [
  { name: 'Maintenance', tone: '#FAC414', items: ['Track segment', 'Maintenance centre', 'LevelCrossing', 'Switch', 'Switch heating', 'Signal'] },
  { name: 'Timetabling', tone: '#B494DB', items: ['Track segment', 'Train', 'Station', 'PlatformEdge', 'SpeedSection', 'HorizontalCurve'] },
  { name: 'Finance', tone: '#64C8FA', items: ['Track segment', 'AccountingSection', 'Client', 'Investment project', 'Government', 'PPIP'] },
  { name: 'Energy', tone: '#35DE6D', items: ['Track segment', 'Substation', 'Grid operator', 'Power distribution shelter', 'SectionInsulator', 'Siding'] },
  { name: 'Projects', tone: '#EB518F', items: ['Track segment', 'Investment project', 'LevelCrossing', 'Road', 'VerticalSlope', 'Station'] },
]
const CW = 176, CG = 9, CX0 = (W - (clusters.length * CW + (clusters.length - 1) * CG)) / 2
const boxX = (c: number) => CX0 + c * (CW + CG)
// pieces: the first piece of a concept is the original, the others are the copies each silo made
const seen = new Set<string>()
const pieces = clusters.flatMap((c, ci) => c.items.map((l, k) => {
  const n = byId(l)
  const primary = !seen.has(l)
  seen.add(l)
  const j = ci * 10 + k
  return { key: `${ci}-${l}`, n, primary, tone: c.tone, cx: boxX(ci) + CW / 2 + (rnd(j, 7) - 0.5) * 26, cy: 72 + k * 40, cr: (rnd(j, 8) - 0.5) * 14 }
}))
// concepts no silo picked up stay on the floor
const orphans = nodes.filter(n => !pieces.some(p => p.n.i === n.i))

function place(p: { n: typeof nodes[number], cx?: number, cy?: number, cr?: number }) {
  if (s.value === 1) return `translate(${p.n.sx}px, ${p.n.sy}px) rotate(${p.n.sr}deg)`
  if (s.value === 2 && p.cx !== undefined) return `translate(${p.cx}px, ${p.cy}px) rotate(${p.cr}deg)`
  if (s.value === 2) return `translate(${p.n.sx}px, ${H + 40}px) rotate(${p.n.sr}deg)`
  return `translate(${p.n.x}px, ${p.n.y}px) rotate(0deg)`
}
const pw = (l: string) => l.length * 6 + 18
const captions = [
  ['The railway is the most fantastic domain there is.', 'No other industry packs this many distinct concepts and niche specialities into one system.'],
  ['Then computers came along.', 'We took this beautiful, complete model, dropped it on the floor, and it broke into a million pieces.'],
  ['We picked up the pieces that suited us…', '…glued them together our own way, and called them “use cases”. Count the track segments.'],
  ['DIM restores the complete picture.', 'One model of the railway, every concept in its place, every use case a view on it.'],
]
</script>

<template>
  <div class="sr">
    <div class="cap">
      <div class="h">{{ captions[s][0] }}</div>
      <div class="sub">{{ captions[s][1] }}</div>
    </div>
    <svg :viewBox="`0 0 ${W} ${H + 30}`" class="svg">
      <defs>
        <radialGradient id="sr-glow">
          <stop offset="0" stop-color="#64C8FA" stop-opacity=".35" />
          <stop offset=".6" stop-color="#8D66BA" stop-opacity=".12" />
          <stop offset="1" stop-color="#8D66BA" stop-opacity="0" />
        </radialGradient>
      </defs>
      <ellipse :cx="CX" :cy="CY" rx="470" ry="210" fill="url(#sr-glow)" class="halo" :class="{ on: s === 3 }" />

      <!-- the web of relations -->
      <line v-for="([a, b], k) in edges" :key="k" :x1="nodes[a].x" :y1="nodes[a].y" :x2="nodes[b].x" :y2="nodes[b].y"
        class="edge" :class="{ gone: s === 1 || s === 2, back: s === 3 }" :style="{ transitionDelay: s === 3 ? `${600 + rnd(k, 9) * 600}ms` : `${rnd(k, 9) * 200}ms` }" />
      <text v-for="([a, b, l], k) in edges" :key="`l${k}`" :x="(nodes[a].x + nodes[b].x) / 2" :y="(nodes[a].y + nodes[b].y) / 2 - 3"
        text-anchor="middle" class="elab" :class="{ gone: s === 1 || s === 2 }" :style="{ transitionDelay: s === 3 ? '1.2s' : '0s' }">{{ l }}</text>

      <!-- the silos -->
      <g v-for="(c, ci) in clusters" :key="c.name" class="silo" :class="{ on: s === 2 }" :style="{ transitionDelay: `${ci * 90}ms` }">
        <rect :x="boxX(ci)" y="22" :width="CW" :height="H - 16" rx="14" :stroke="c.tone" class="box" />
        <text :x="boxX(ci) + CW / 2" y="44" text-anchor="middle" :fill="c.tone" class="sname">{{ c.name }}</text>
        <line v-for="k in c.items.length - 1" :key="k" :x1="boxX(ci) + CW / 2" :y1="72 + (k - 1) * 34" :x2="boxX(ci) + CW / 2" :y2="72 + k * 34" class="glue" :stroke="c.tone" />
      </g>

      <!-- concepts nobody picked up -->
      <g v-for="n in orphans" :key="`o${n.i}`" class="piece" :style="{ transform: place({ n }), transitionDelay: `${rnd(n.i, 10) * 250}ms` }">
        <rect :x="-pw(n.l) / 2" y="-11" :width="pw(n.l)" height="22" rx="11" class="pill" :class="{ lost: s === 2 }" />
        <text y="4" text-anchor="middle" class="pl">{{ n.l }}</text>
      </g>

      <!-- pieces -->
      <g v-for="p in pieces" :key="p.key" class="piece" :class="{ copy: !p.primary, show: p.primary || s === 2 }"
        :style="{ transform: place(p), transitionDelay: `${rnd(p.n.i, 10) * 250}ms` }">
        <rect :x="-pw(p.n.l) / 2" y="-11" :width="pw(p.n.l)" height="22" rx="11" class="pill"
          :style="s === 2 ? { stroke: p.tone, fill: 'rgba(11,12,28,.95)' } : {}" :class="{ track: p.n.l === 'Track segment' && s === 2 }" />
        <text y="4" text-anchor="middle" class="pl">{{ p.n.l }}</text>
      </g>

      <g class="dim" :class="{ on: s === 3 }">
        <rect :x="DX - 70" :y="DY - 24" width="140" height="48" rx="24" class="dim-pill" />
        <text :x="DX" :y="DY + 9" text-anchor="middle" class="dim-t">DIM</text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.sr { position: relative; }
.cap { min-height: 64px; }
.cap .h { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 1.55rem; line-height: 1.2;
  background: linear-gradient(90deg, #fff, #D0AAFD); -webkit-background-clip: text; background-clip: text; color: transparent; }
.cap .sub { font-size: 15px; color: var(--c-ink-mod); margin-top: 2px; }
.svg { width: 100%; height: 360px; display: block; overflow: visible; }
.edge { stroke: #64C8FA; stroke-width: 1.1; opacity: .4; transition: opacity .5s ease; }
.edge.gone { opacity: 0; }
.elab { font-size: 8.5px; fill: #A6A8C1; opacity: .9; transition: opacity .5s ease; paint-order: stroke; stroke: #0A0B1A; stroke-width: 3px; }
.elab.gone { opacity: 0; }
.edge.back { opacity: .75; stroke: #B494DB; stroke-width: 1.4; }
.piece { transition: transform 1.1s cubic-bezier(.5, .05, .2, 1), opacity .5s ease; }
.piece.copy { opacity: 0; }
.piece.copy.show { opacity: 1; }
.pill { fill: #141833; stroke: #3A4170; stroke-width: 1.2; transition: all .5s ease; }
.pill.lost { opacity: .35; }
.pill.track { stroke-width: 2.4; filter: drop-shadow(0 0 6px rgba(254, 98, 99, .8)); stroke: #FE6263 !important; }
.pl { font-size: 11px; fill: #E9E6F5; font-weight: 600; pointer-events: none; }
.silo { opacity: 0; transition: opacity .6s ease; }
.silo.on { opacity: 1; transition-delay: .5s; }
.box { fill: rgba(255, 255, 255, .025); stroke-width: 1.4; stroke-dasharray: 5 4; }
.sname { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 14px; letter-spacing: .04em; }
.glue { stroke-width: 3; stroke-dasharray: 2 3; opacity: .7; }
.halo { opacity: 0; transition: opacity 1s ease 1s; }
.halo.on { opacity: 1; }
.dim { opacity: 0; transform: scale(.4); transform-origin: 852px 322px; transition: all .6s cubic-bezier(.3, 1.6, .5, 1); }
.dim.on { opacity: 1; transform: none; transition-delay: 1.3s; }
.dim-pill { fill: #1E2869; stroke: #64C8FA; stroke-width: 2.5; filter: drop-shadow(0 0 16px rgba(100, 200, 250, .7)); }
.dim-t { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 24px; fill: #fff; }
</style>
