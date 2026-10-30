<script setup lang="ts">
/**
 * Who relies on DIM: the graph in the middle, its consumers around it. The links draw out of the hub
 * one by one with data flowing along them; the dashed one is the consumer that is moving onto DIM.
 * Runs by itself when the slide is shown.
 */
import { onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'

const consumers = [
  { k: 'ERTMS / ETCS', t: 'Starting point to engineer <b>ETCS</b> on every line.', x: 0, y: 8, tone: 'sky' },
  { k: 'Traffic planners', t: '<b>Run-time calculation</b> and <b>timetable simulation</b> on the modelled network.', x: 640, y: 8, tone: 'amber' },
  { k: 'RINF · EU register', t: 'Generated from the graph. First submission at <b>micro, topology level</b>.', x: 0, y: 214, tone: 'green' },
  { k: 'Railway undertakings', t: '<b>Platform lengths</b>: which trains fit, where passengers should board.', x: 640, y: 214, tone: 'purple' },
  { k: 'Traffic management & timetabling', t: 'Next: DIM replaces the old supply of infrastructure data.', x: 320, y: 262, tone: 'next' },
]
const HX = 450, HY = 150
const anchor = (c: typeof consumers[number]) => ({ x: c.x + 110, y: c.y + 38 })

const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const shown = ref(consumers.length)
const timers: ReturnType<typeof setTimeout>[] = []
watch(isActive, (active) => {
  timers.splice(0).forEach(clearTimeout)
  if (!active || isPrintMode.value) { shown.value = consumers.length; return }
  shown.value = 0
  consumers.forEach((_, i) => timers.push(setTimeout(() => { shown.value = i + 1 }, 500 + i * 650)))
}, { immediate: true })
onUnmounted(() => timers.forEach(clearTimeout))
</script>

<template>
  <div class="hub-wrap">
    <svg viewBox="0 0 900 340" class="links">
      <path v-for="(c, i) in consumers" :key="c.k"
        :d="`M ${HX} ${HY} Q ${(HX + anchor(c).x) / 2} ${HY + (anchor(c).y - HY) * 0.15} ${anchor(c).x} ${anchor(c).y}`"
        class="link" :class="[c.tone, { on: i < shown }]" pathLength="100" />
      <path v-for="(c, i) in consumers" :key="`f${c.k}`"
        :d="`M ${HX} ${HY} Q ${(HX + anchor(c).x) / 2} ${HY + (anchor(c).y - HY) * 0.15} ${anchor(c).x} ${anchor(c).y}`"
        class="flow" :class="[c.tone, { on: i < shown }]" />
    </svg>

    <div class="core">
      <div class="ring r1" /><div class="ring r2" />
      <div class="dim">DIM</div>
      <div class="dim-sub">one knowledge graph</div>
    </div>

    <div v-for="(c, i) in consumers" :key="c.k" class="card glass" :class="[c.tone, { on: i < shown }]" :style="{ left: `${c.x}px`, top: `${c.y}px` }">
      <div class="kicker">{{ c.k }}</div>
      <div class="ct" v-html="c.t" />
    </div>
  </div>
</template>

<style scoped>
.hub-wrap { position: relative; width: 900px; height: 340px; margin: 0 auto; }
.links { position: absolute; inset: 0; width: 100%; height: 100%; overflow: visible; }
.link { fill: none; stroke-width: 2; stroke-dasharray: 100; stroke-dashoffset: 100; transition: stroke-dashoffset .7s ease; opacity: .55; }
.link.on { stroke-dashoffset: 0; }
.flow { fill: none; stroke-width: 3; stroke-dasharray: 2 14; stroke-linecap: round; opacity: 0; transition: opacity .3s ease .6s; animation: flow 1.2s linear infinite; }
.flow.on { opacity: 1; }
@keyframes flow { to { stroke-dashoffset: -16 } }
.link.sky, .flow.sky { stroke: var(--c-sky); } .link.amber, .flow.amber { stroke: var(--c-amber); }
.link.green, .flow.green { stroke: var(--c-green); } .link.purple, .flow.purple { stroke: var(--c-purple); }
.link.next, .flow.next { stroke: var(--c-ink-mod); } .link.next { stroke-dasharray: 5 5; }
.core { position: absolute; left: 450px; top: 150px; transform: translate(-50%, -50%); width: 150px; height: 150px; border-radius: 50%;
  background: radial-gradient(circle at 35% 30%, #2A5BB8, #1E2869 70%); display: flex; flex-direction: column; align-items: center; justify-content: center;
  box-shadow: 0 0 0 3px var(--c-sky), 0 0 40px rgba(100, 200, 250, .45); z-index: 2; }
.dim { font-family: 'Poppins', sans-serif; font-size: 40px; font-weight: 700; color: #fff; line-height: 1; }
.dim-sub { font-size: 11px; color: #C4EFFF; margin-top: 4px; }
.ring { position: absolute; inset: -10px; border-radius: 50%; border: 1.5px solid var(--c-sky); opacity: 0; animation: ripple 3s ease-out infinite; }
.ring.r2 { animation-delay: 1.5s; }
@keyframes ripple { 0% { transform: scale(.9); opacity: .6 } 100% { transform: scale(1.5); opacity: 0 } }
.card { position: absolute; width: 230px; background: var(--c-surface) !important; padding: 9px 13px; opacity: 0; transform: scale(.85); transition: all .45s cubic-bezier(.3, 1.5, .5, 1); z-index: 3; }
.card.on { opacity: 1; transform: none; }
.card.sky { border-top: 3px solid var(--c-sky); } .card.amber { border-top: 3px solid var(--c-amber); }
.card.green { border-top: 3px solid var(--c-green); } .card.purple { border-top: 3px solid var(--c-purple); }
.card.next { border: 1.5px dashed var(--c-ink-mod); width: 260px; }
.card.amber .kicker { color: var(--c-amber); } .card.green .kicker { color: var(--c-green); }
.card.purple .kicker { color: var(--c-purple); } .card.next .kicker { color: var(--c-ink-mod); }
.ct { font-size: 12.5px; line-height: 1.38; margin-top: 3px; }
.ct :deep(b) { color: #fff; }
</style>
