<script setup lang="ts">
/**
 * Contents of the value tour as a metro line out of the DIM hub: the line draws itself, the stops light up
 * one by one, and a train runs along it. Every stop is one of the demos that follow.
 */
import { onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'

const stops = [
  { l: 'DIM portal', d: 'railML files, datasets', x: 190, y: 120 },
  { l: 'Key figures', d: 'computed from the graph', x: 290, y: 220 },
  { l: 'RINF status', d: 'what we fill, what is missing', x: 400, y: 120 },
  { l: 'Search builder', d: 'point-and-click SPARQL', x: 510, y: 220 },
  { l: 'Data catalog', d: 'what is in each dataset', x: 620, y: 120 },
  { l: 'TPS mapping', d: 'linked to timetable planning', x: 730, y: 220 },
  { l: 'Validation portal', d: 'SHACL, every run', x: 840, y: 120 },
  { l: 'REST API', d: 'JSON-LD for developers', x: 840, y: 290 },
]
const HUB = { x: 70, y: 170 }
const path = `M ${HUB.x} ${HUB.y} ` + stops.map(p => `L ${p.x} ${p.y}`).join(' ')
const tones = ['#64C8FA', '#2384EB', '#B494DB', '#8D66BA', '#EB518F', '#FE6263', '#FAC414', '#35DE6D']

const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const lit = ref(stops.length)
const timers: ReturnType<typeof setTimeout>[] = []
watch(isActive, (active) => {
  timers.splice(0).forEach(clearTimeout)
  if (!active || isPrintMode.value) { lit.value = stops.length; return }
  lit.value = 0
  stops.forEach((_, i) => timers.push(setTimeout(() => { lit.value = i + 1 }, 500 + i * 330)))
}, { immediate: true })
onUnmounted(() => timers.forEach(clearTimeout))
</script>

<template>
  <div class="vt">
    <svg viewBox="0 0 940 340" class="svg">
      <defs>
        <linearGradient id="vt-line" x1="0" x2="1">
          <stop offset="0" stop-color="#64C8FA" />
          <stop offset=".55" stop-color="#8D66BA" />
          <stop offset="1" stop-color="#EB518F" />
        </linearGradient>
      </defs>
      <path :d="path" class="bed" />
      <path id="vt-path" :d="path" class="line" :class="{ on: isActive || isPrintMode }" pathLength="100" />
      <circle r="7" class="runner">
        <animateMotion dur="7s" repeatCount="indefinite" begin="3s"><mpath href="#vt-path" /></animateMotion>
      </circle>

      <g class="hub">
        <circle :cx="HUB.x" :cy="HUB.y" r="40" class="hub-c" />
        <text :x="HUB.x" :y="HUB.y + 8" text-anchor="middle" class="hub-t">DIM</text>
      </g>

      <g v-for="(p, i) in stops" :key="p.l" class="stop" :class="{ on: i < lit }">
        <circle :cx="p.x" :cy="p.y" r="13" class="st" :stroke="tones[i]" />
        <text :x="p.x" :y="p.y + 4.5" text-anchor="middle" class="num" :fill="tones[i]">{{ i + 1 }}</text>
        <text :x="p.x" :y="p.y + (p.y < 170 ? -34 : 34)" text-anchor="middle" class="lab">{{ p.l }}</text>
        <text :x="p.x" :y="p.y + (p.y < 170 ? -19 : 49)" text-anchor="middle" class="desc">{{ p.d }}</text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.svg { width: 100%; height: 340px; display: block; overflow: visible; }
.bed { fill: none; stroke: rgba(255, 255, 255, .07); stroke-width: 16; stroke-linejoin: round; stroke-linecap: round; }
.line { fill: none; stroke: url(#vt-line); stroke-width: 7; stroke-linejoin: round; stroke-linecap: round;
  stroke-dasharray: 100; stroke-dashoffset: 100; filter: drop-shadow(0 0 8px rgba(141, 102, 186, .6)); }
.line.on { animation: draw 2.8s ease-out .3s forwards; }
@keyframes draw { to { stroke-dashoffset: 0 } }
.runner { fill: #fff; filter: drop-shadow(0 0 8px #fff); }
.hub-c { fill: #1E2869; stroke: #64C8FA; stroke-width: 3; filter: drop-shadow(0 0 18px rgba(100, 200, 250, .6)); }
.hub-t { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 22px; fill: #fff; }
.stop { opacity: .15; transition: opacity .4s ease; }
.stop.on { opacity: 1; }
.stop.on .st { animation: ping .6s cubic-bezier(.3, 1.6, .5, 1); }
@keyframes ping { from { r: 4 } }
.st { fill: #0B0C1C; stroke-width: 4; }
.num { font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 13px; }
.lab { font-family: 'Poppins', sans-serif; font-weight: 600; font-size: 15px; fill: #fff; }
.desc { font-size: 12px; fill: var(--c-ink-mod); }
</style>
