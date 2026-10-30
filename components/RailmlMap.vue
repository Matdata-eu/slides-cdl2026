<script setup lang="ts">
/**
 * Draw an area on the network, get a railML file: the outline draws itself, the tracks inside it light up,
 * and a file pops out. Runs once whenever the slide is shown; exports show the end state.
 */
import { onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'

const tracks = [
  'M 10 210 C 120 200 160 150 240 140 S 380 120 460 128 S 620 160 700 120 S 820 40 880 30',
  'M 240 140 C 270 90 300 60 360 40 S 460 20 520 10',
  'M 460 128 C 500 170 540 210 600 230 S 720 250 800 248',
  'M 120 202 C 140 240 180 260 240 262',
  'M 700 120 C 740 150 790 170 870 172',
]
const stations = [{ x: 240, y: 140 }, { x: 460, y: 128 }, { x: 700, y: 120 }, { x: 360, y: 40 }, { x: 600, y: 230 }]
const area = 'M 300 70 C 380 40 520 60 560 110 S 600 210 520 220 S 360 200 320 170 S 250 100 300 70 Z'

const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const phase = ref(3)
const timers: ReturnType<typeof setTimeout>[] = []
watch(isActive, (active) => {
  timers.splice(0).forEach(clearTimeout)
  if (!active || isPrintMode.value) { phase.value = 3; return }
  phase.value = 0
  timers.push(setTimeout(() => { phase.value = 1 }, 400), setTimeout(() => { phase.value = 2 }, 2400), setTimeout(() => { phase.value = 3 }, 3200))
}, { immediate: true })
onUnmounted(() => timers.forEach(clearTimeout))
</script>

<template>
  <div class="rmap glass">
    <svg viewBox="0 0 900 280" class="svg">
      <defs>
        <clipPath id="rmap-area"><path :d="area" /></clipPath>
      </defs>
      <path v-for="(d, i) in tracks" :key="i" :d="d" class="trk" />
      <g clip-path="url(#rmap-area)" class="inside" :class="{ on: phase >= 2 }">
        <path v-for="(d, i) in tracks" :key="`in${i}`" :d="d" class="trk hot" />
      </g>
      <circle v-for="(s, i) in stations" :key="i" :cx="s.x" :cy="s.y" r="6" class="st" />
      <path :d="area" class="area" :class="{ draw: phase >= 1, fill: phase >= 2 }" pathLength="100" />
    </svg>
    <div class="file" :class="{ on: phase >= 3 }">
      <div class="doc"><span>XML</span></div>
      <div><b>my-area.railml</b><br><small>railML 3.3 · only what is inside the area</small></div>
    </div>
  </div>
</template>

<style scoped>
.rmap { position: relative; padding: 6px; }
.svg { width: 100%; height: 230px; display: block; }
.trk { fill: none; stroke: var(--c-ink-subtle); stroke-width: 3; stroke-linecap: round; }
.trk.hot { stroke: var(--c-sky); stroke-width: 4; filter: drop-shadow(0 0 5px var(--c-sky)); }
.inside { opacity: 0; transition: opacity .6s ease; }
.inside.on { opacity: 1; }
.st { fill: var(--c-bg); stroke: var(--c-ink); stroke-width: 2.5; }
.area { fill: transparent; stroke: var(--c-amber); stroke-width: 2.5; stroke-dasharray: 100; stroke-dashoffset: 100; stroke-linejoin: round; transition: stroke-dashoffset 1.9s ease-in-out, fill .6s ease; }
.area.draw { stroke-dashoffset: 0; }
.area.fill { fill: rgba(250, 196, 20, .08); stroke-dasharray: 2 1; }
.file { position: absolute; right: 22px; bottom: 18px; display: flex; gap: 10px; align-items: center; padding: 8px 14px; border-radius: 12px; background: var(--c-surface-2, #1F2B4D);
  border: 1px solid var(--c-green); box-shadow: 0 0 22px rgba(53, 222, 109, .35); font-size: 13px; opacity: 0; transform: scale(.6) translateY(10px); transition: all .45s cubic-bezier(.3, 1.6, .5, 1); }
.file.on { opacity: 1; transform: none; }
.file small { color: var(--c-ink-mod); }
.doc { width: 30px; height: 38px; border-radius: 4px; background: #fff; color: #1E2869; font-size: 9px; font-weight: 700; display: flex; align-items: flex-end; justify-content: center; padding-bottom: 4px; clip-path: polygon(0 0, 70% 0, 100% 25%, 100% 100%, 0 100%); }
</style>
