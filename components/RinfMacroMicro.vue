<script setup lang="ts">
/**
 * RINF at macro level (operational points linked by sections of line) and, from step 1, at micro level:
 * the same network as tracks, switches, signals and platforms on a topology.
 */
import { computed } from 'vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const micro = computed(() => props.step >= 1)
const ops = [{ x: 90, n: 'Lillestrøm' }, { x: 450, n: 'Kløfta' }, { x: 810, n: 'Eidsvoll' }]
const Y = 110
</script>

<template>
  <div class="rm">
    <svg viewBox="0 0 900 190" class="svg">
      <!-- macro -->
      <g class="layer" :class="{ off: micro }">
        <line :x1="ops[0].x" :y1="Y" :x2="ops[1].x" :y2="Y" class="sol" />
        <line :x1="ops[1].x" :y1="Y" :x2="ops[2].x" :y2="Y" class="sol" />
        <text :x="(ops[0].x + ops[1].x) / 2" :y="Y - 14" text-anchor="middle" class="lbl">section of line</text>
        <text :x="(ops[1].x + ops[2].x) / 2" :y="Y - 14" text-anchor="middle" class="lbl">section of line</text>
        <g v-for="o in ops" :key="o.n">
          <circle :cx="o.x" :cy="Y" r="15" class="op" />
          <text :x="o.x" :y="Y + 40" text-anchor="middle" class="opn">{{ o.n }}</text>
        </g>
      </g>

      <!-- micro -->
      <g class="layer" :class="{ off: !micro }">
        <g v-for="o in ops" :key="`a${o.n}`">
          <rect :x="o.x - 80" :y="Y - 52" width="160" height="104" rx="14" class="area" />
          <text :x="o.x" :y="Y + 72" text-anchor="middle" class="opn">{{ o.n }}</text>
        </g>
        <!-- double track between stations -->
        <path :d="`M 10 ${Y - 14} H 890`" class="trk" />
        <path :d="`M 10 ${Y + 14} H 890`" class="trk" />
        <!-- station tracks with switches -->
        <g v-for="(o, i) in ops" :key="`s${o.n}`">
          <path :d="`M ${o.x - 64} ${Y - 14} C ${o.x - 44} ${Y - 14} ${o.x - 44} ${Y - 38} ${o.x - 24} ${Y - 38} H ${o.x + 24} C ${o.x + 44} ${Y - 38} ${o.x + 44} ${Y - 14} ${o.x + 64} ${Y - 14}`" class="trk" />
          <path v-if="i !== 1" :d="`M ${o.x - 64} ${Y + 14} C ${o.x - 44} ${Y + 14} ${o.x - 44} ${Y + 38} ${o.x - 24} ${Y + 38} H ${o.x + 24} C ${o.x + 44} ${Y + 38} ${o.x + 44} ${Y + 14} ${o.x + 64} ${Y + 14}`" class="trk" />
          <path :d="`M ${o.x - 30} ${Y - 14} L ${o.x - 10} ${Y + 14}`" class="trk thin" />
          <circle v-for="dx in [-64, 64]" :key="dx" :cx="o.x + dx" :cy="Y - 14" r="3.5" class="sw" />
          <template v-if="i !== 1">
            <circle v-for="dx in [-64, 64]" :key="`b${dx}`" :cx="o.x + dx" :cy="Y + 14" r="3.5" class="sw" />
          </template>
          <rect :x="o.x - 22" :y="Y - 31" width="44" height="6" rx="2" class="plat" />
          <rect :x="o.x - 22" :y="Y + 1" width="44" height="6" rx="2" class="plat" />
        </g>
        <!-- signals -->
        <g v-for="sx in [200, 330, 570, 700]" :key="sx">
          <line :x1="sx" :y1="Y - 14" :x2="sx" :y2="Y - 28" class="mast" />
          <circle :cx="sx" :cy="Y - 31" r="3.5" class="sig" />
        </g>
        <text x="450" y="18" text-anchor="middle" class="lbl">tracks · switches · signals · platforms, all positioned on the topology</text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.svg { width: 100%; height: 250px; display: block; }
.layer { transition: opacity .7s ease, transform .7s ease; transform-origin: 450px 110px; }
.layer.off { opacity: 0; transform: scale(.94); }
.sol { stroke: var(--c-sky); stroke-width: 6; stroke-linecap: round; }
.op { fill: var(--c-bg); stroke: var(--c-sky); stroke-width: 4; }
.opn { font-family: 'Poppins', sans-serif; font-size: 15px; font-weight: 600; fill: var(--c-ink); }
.lbl { font-size: 13px; fill: var(--c-ink-mod); }
.area { fill: rgba(100, 200, 250, .06); stroke: var(--c-sky); stroke-width: 1.4; stroke-dasharray: 6 4; }
.trk { fill: none; stroke: var(--c-ink); stroke-width: 2.6; opacity: .85; }
.trk.thin { stroke-width: 2; }
.sw { fill: var(--c-bg); stroke: var(--c-amber); stroke-width: 2; }
.plat { fill: var(--c-amber); opacity: .8; }
.mast { stroke: var(--c-ink-mod); stroke-width: 1.4; }
.sig { fill: var(--c-green); }
</style>
