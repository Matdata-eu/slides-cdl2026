<script setup lang="ts">
import { computed } from 'vue'
import { useNav } from '@slidev/client'

/**
 * The train footer. A track runs along the bottom of every slide and the train's position follows the
 * slide number (and creeps forward with the clicks of a slide). Behind it the track becomes a knowledge
 * graph: every slide leaves a node, some leave a triple hanging off the track, stations (slides with a
 * `station:` frontmatter) get a passing loop and a label, signals turn green once passed, a slide with
 * `overpass: true` gets a bridge over the track and the last slide is the buffer stop.
 */
const { currentPage, total, clicks, clicksTotal, slides, currentLayout } = useNav()

const Y = 31
const X0 = 70
const X1 = 940
const n = computed(() => Math.max(total.value, 2))
const xOf = (page: number) => X0 + ((page - 1) / (n.value - 1)) * (X1 - X0)

// deterministic pseudo-random per slide, so the graph looks the same on every visit
const rnd = (i: number, salt: number) => {
  const s = Math.sin(i * 127.1 + salt * 311.7) * 43758.5453
  return s - Math.floor(s)
}
const colours = ['var(--c-amber)', 'var(--c-green)', 'var(--c-purple)', 'var(--c-pink)', 'var(--c-teal)']

const stops = computed(() => slides.value.slice(0, n.value).map((route, idx) => {
  const page = idx + 1
  const fm = (route.meta?.slide?.frontmatter ?? {}) as Record<string, unknown>
  const station = typeof fm.station === 'string' ? fm.station : undefined
  const triple = !station && rnd(page, 1) > 0.45
  return {
    page,
    x: xOf(page),
    station,
    overpass: fm.overpass === true,
    signal: !station && !fm.overpass && page % 4 === 2,
    triple: triple ? {
      dx: (rnd(page, 2) - 0.5) * 18,
      dy: 9 + rnd(page, 3) * 9,
      colour: colours[Math.floor(rnd(page, 4) * colours.length)],
      second: rnd(page, 5) > 0.6,
    } : undefined,
  }
}))

const progress = computed(() => {
  const step = clicksTotal.value > 0 ? Math.min(clicks.value / clicksTotal.value, 1) * 0.6 : 0
  return Math.min(currentPage.value + step, n.value)
})
const trainX = computed(() => xOf(progress.value))
const passed = (page: number) => page <= currentPage.value
const hidden = computed(() => currentLayout.value === 'cover')
const onClip = computed(() => currentLayout.value === 'full')
</script>

<template>
  <div v-if="!hidden" class="tt" :class="{ clip: onClip }">
    <svg viewBox="0 0 980 42" class="tt-svg">
      <defs>
        <linearGradient id="tt-done" x1="0" x2="1">
          <stop offset="0" stop-color="var(--c-sky)" stop-opacity=".35" />
          <stop offset="1" stop-color="var(--c-sky)" stop-opacity="1" />
        </linearGradient>
        <filter id="tt-glow" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="1.6" result="b" />
          <feMerge><feMergeNode in="b" /><feMergeNode in="SourceGraphic" /></feMerge>
        </filter>
      </defs>

      <!-- sleepers and the track still ahead -->
      <line :x1="X0" :y1="Y" :x2="X1 + 14" :y2="Y" class="sleepers" />
      <line :x1="X0" :y1="Y" :x2="X1 + 14" :y2="Y" class="ahead" />
      <!-- the track the train has built -->
      <rect :x="X0" :y="Y - 1" height="2" class="done" fill="url(#tt-done)" filter="url(#tt-glow)" :style="{ width: `${trainX - X0}px` }" />

      <g v-for="s in stops" :key="s.page">
        <!-- a triple hanging off the track: node → literal (→ another node) -->
        <g v-if="s.triple" class="kg" :class="{ on: passed(s.page) }">
          <line :x1="s.x" :y1="Y" :x2="s.x + s.triple.dx" :y2="Y - s.triple.dy" class="kg-edge" />
          <circle :cx="s.x + s.triple.dx" :cy="Y - s.triple.dy" r="2" :fill="s.triple.colour" />
          <template v-if="s.triple.second">
            <line :x1="s.x + s.triple.dx" :y1="Y - s.triple.dy" :x2="s.x + s.triple.dx + 9" :y2="Y - s.triple.dy - 5" class="kg-edge" />
            <rect :x="s.x + s.triple.dx + 8" :y="Y - s.triple.dy - 7.5" width="5" height="5" rx="1" :fill="s.triple.colour" opacity=".8" />
          </template>
        </g>

        <!-- station: passing loop (two switches) and a label -->
        <g v-if="s.station" class="st" :class="{ on: passed(s.page) }">
          <path :d="`M ${s.x - 16} ${Y} C ${s.x - 10} ${Y} ${s.x - 10} ${Y - 7} ${s.x - 4} ${Y - 7} L ${s.x + 4} ${Y - 7} C ${s.x + 10} ${Y - 7} ${s.x + 10} ${Y} ${s.x + 16} ${Y}`" class="loop" />
          <rect :x="s.x - 7" :y="Y + 3" width="14" height="2.4" rx="1" class="platform" />
          <text :x="s.x" :y="Y - 15" text-anchor="middle" class="st-label">{{ s.station }}</text>
        </g>

        <!-- signal: red ahead of the train, green behind it -->
        <g v-if="s.signal">
          <line :x1="s.x" :y1="Y - 1" :x2="s.x" :y2="Y - 9" class="mast" />
          <circle :cx="s.x" :cy="Y - 10.5" r="1.9" :class="passed(s.page) ? 'sig-go' : 'sig-stop'" />
        </g>

        <!-- overpass -->
        <g v-if="s.overpass" class="bridge" :class="{ on: passed(s.page) }">
          <path :d="`M ${s.x - 13} ${Y + 7} Q ${s.x} ${Y - 15} ${s.x + 13} ${Y + 7}`" class="arch" />
          <line :x1="s.x - 15" :y1="Y - 6.5" :x2="s.x + 15" :y2="Y - 6.5" class="deck" />
        </g>

        <!-- every slide is a node in the topology -->
        <circle :cx="s.x" :cy="Y" :r="s.station ? 3.4 : 2.1" class="node" :class="{ on: passed(s.page), st: s.station }" />
      </g>

      <!-- buffer stop -->
      <path :d="`M ${X1 + 14} ${Y - 6} L ${X1 + 14} ${Y + 4} M ${X1 + 11} ${Y - 6} L ${X1 + 17} ${Y - 6}`" class="buffer" />

      <!-- the train -->
      <g class="train" :style="{ transform: `translate(${trainX}px, ${Y}px)` }">
        <g transform="translate(-58 -10)">
          <rect x="0" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
          <rect x="18" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
          <path d="M36 1 h14 q6 0 8 7.5 h-22 z" class="loco" />
          <rect x="49" y="2.4" width="4.5" height="2.6" rx=".6" class="window" />
          <rect x="2" y="2.6" width="12" height="2" rx=".5" class="window" />
          <rect x="20" y="2.6" width="12" height="2" rx=".5" class="window" />
          <line x1="0" y1="6.3" x2="58" y2="6.3" class="stripe" />
          <circle v-for="w in [3, 13, 21, 31, 40, 52]" :key="w" :cx="w" cy="9" r="1.3" class="wheel" />
          <circle cx="58.5" cy="6" r="1.2" class="lamp" />
        </g>
      </g>

      <text x="972" :y="Y + 9" text-anchor="end" class="pg">{{ currentPage }}/{{ total }}</text>
    </svg>
  </div>
</template>

<style scoped>
.tt { position: absolute; left: 0; right: 0; bottom: 0; height: 42px; pointer-events: none; z-index: 20; }
.tt.clip { background: linear-gradient(to top, rgba(10, 16, 34, .92), rgba(10, 16, 34, .55) 70%, transparent); }
.tt-svg { width: 100%; height: 100%; display: block; overflow: visible; }
.sleepers { stroke: var(--c-ink-subtle); stroke-width: 5; stroke-dasharray: 1 4; opacity: .35; }
.ahead { stroke: var(--c-ink-subtle); stroke-width: 1.2; stroke-dasharray: 3 3; opacity: .55; }
.done { transition: width 1.1s cubic-bezier(.45, .05, .25, 1); }
.node { fill: var(--c-bg); stroke: var(--c-ink-subtle); stroke-width: 1; transition: all .4s ease .7s; }
.node.on { fill: var(--c-sky); stroke: var(--c-sky); }
.node.st.on { fill: #fff; stroke: var(--c-sky); stroke-width: 1.6; }
.kg { opacity: 0; transform-box: fill-box; transition: opacity .5s ease .9s; }
.kg.on { opacity: .9; }
.kg-edge { stroke: var(--c-ink-mod); stroke-width: .6; }
.st .loop { fill: none; stroke: var(--c-ink-subtle); stroke-width: 1; stroke-dasharray: 2 2; transition: all .5s ease .7s; }
.st.on .loop { stroke: var(--c-sky); stroke-dasharray: none; stroke-width: 1.4; }
.st .platform { fill: var(--c-ink-subtle); opacity: .5; }
.st.on .platform { fill: var(--c-amber); opacity: 1; }
.st-label { font-size: 7.5px; font-weight: 700; letter-spacing: .06em; text-transform: uppercase; fill: var(--c-ink-subtle); transition: fill .5s ease .7s; }
.st.on .st-label { fill: var(--c-ink); }
.mast { stroke: var(--c-ink-mod); stroke-width: .8; }
.sig-stop { fill: var(--c-red); }
.sig-go { fill: var(--c-green); filter: drop-shadow(0 0 2px var(--c-green)); transition: fill .3s ease .8s; }
.bridge .arch { fill: none; stroke: var(--c-ink-subtle); stroke-width: 1.4; }
.bridge .deck { stroke: var(--c-ink-subtle); stroke-width: 2.2; stroke-linecap: round; }
.bridge.on .arch, .bridge.on .deck { stroke: var(--c-orange); }
.buffer { stroke: var(--c-red); stroke-width: 1.6; fill: none; }
.train { transition: transform 1.1s cubic-bezier(.45, .05, .25, 1); }
.wagon { fill: #E8EEF8; stroke: #1E2869; stroke-width: .5; }
.loco { fill: #fff; stroke: #1E2869; stroke-width: .5; }
.window { fill: #1E2869; opacity: .85; }
.stripe { stroke: var(--c-red); stroke-width: .9; }
.wheel { fill: #1E2869; }
.lamp { fill: #FFE58A; filter: drop-shadow(0 0 3px #FFE58A); }
.pg { font-size: 7.5px; fill: var(--c-ink-subtle); letter-spacing: .05em; }
</style>
