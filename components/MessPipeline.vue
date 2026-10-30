<script setup lang="ts">
/**
 * The mess that grew around "source straight into the target": the clean pipeline first, then per step
 * the fix-up layers drop in (with a shake) and the wiring behind the pipeline gets more tangled.
 * step 1 SQL patches · 2 GeoSPARQL gaps (spatial SQL + Python) · 3 post-processing UPDATEs.
 */
import { computed } from 'vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const s = computed(() => Math.max(0, Math.min(3, props.step)))

// tangled wiring behind the boxes; more of it on every step
const wires = [
  { d: 'M 190 70 C 260 -10 330 210 420 120 S 600 30 640 150', at: 1 },
  { d: 'M 200 180 C 300 260 360 40 470 150 S 700 260 760 120', at: 2 },
  { d: 'M 360 40 C 420 200 520 -20 600 180 S 760 40 820 200', at: 2 },
  { d: 'M 120 140 C 250 60 380 250 540 100 S 760 160 880 60', at: 3 },
  { d: 'M 600 30 C 520 120 700 220 560 230 S 380 120 300 210', at: 3 },
  { d: 'M 660 210 C 760 140 600 40 720 30 S 860 120 800 220', at: 3 },
]
const cards = [
  { at: 1, tone: 'amber', h: 'Data patches', t: '~100 in SQL: fixes in the source take months, and must land before anything reads the data.' },
  { at: 2, tone: 'red', h: 'GeoSPARQL gaps', t: 'No linear referencing, no routing, not even <code>ST_X</code>. Engines skip parts of the spec, so geometry moves to SQL and Python.' },
  { at: 3, tone: 'purple', h: 'Post-processing', t: '61 ordered UPDATEs that delete and rewrite each other: an imperative program in SPARQL, with no lineage.' },
]
</script>

<template>
  <div class="mp">
    <svg viewBox="0 0 900 250" class="wires">
      <path v-for="(w, i) in wires" :key="i" :d="w.d" class="wire" :class="{ on: s >= w.at }" pathLength="100" />
    </svg>

    <div class="grid">
      <div class="cell r1 c2"><div class="box amber" :class="{ on: s >= 1 }">~100 SQL data patches</div></div>
      <div class="cell r1 c4"><div class="box red" :class="{ on: s >= 2 }">6 Python steps<small>geometry · tracks · sequences</small></div></div>

      <div class="cell r2 c1"><div class="box main on">Sources<small>GeoPackage · XML · JSON</small></div></div>
      <div class="cell r2 c2"><div class="box main on">Source Ingestor</div></div>
      <div class="cell r2 c3"><div class="box main sa on">SPARQL Anything<small>71 CONSTRUCTs</small></div></div>
      <div class="cell r2 c4"><div class="box main on">Jena Fuseki</div></div>
      <div class="cell r2 c5"><div class="box main on">railML · RINF · API</div></div>

      <div class="cell r3 c2"><div class="box red" :class="{ on: s >= 2 }">Spatial SQL views<small>SpatiaLite</small></div></div>
      <div class="cell r3 c4"><div class="box purple" :class="{ on: s >= 3 }">61 post-processing<small>SPARQL UPDATEs</small></div></div>
    </div>

    <div class="cards">
      <div v-for="c in cards" :key="c.h" class="card glass" :class="[c.tone, { on: s >= c.at }]">
        <b>{{ c.h }}</b> <span v-html="c.t" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.mp { position: relative; }
.wires { position: absolute; left: 0; top: 0; width: 100%; height: 250px; pointer-events: none; }
.wire { fill: none; stroke: var(--c-red); stroke-width: 1.6; stroke-dasharray: 100; stroke-dashoffset: 100; opacity: .45; transition: stroke-dashoffset 1.2s ease; }
.wire.on { stroke-dashoffset: 0; }
.grid { position: relative; display: grid; grid-template-columns: repeat(5, 1fr); grid-template-rows: 70px 90px 70px; gap: 6px 16px; height: 250px; align-items: center; }
.r1 { grid-row: 1; } .r2 { grid-row: 2; } .r3 { grid-row: 3; }
.c1 { grid-column: 1; } .c2 { grid-column: 2; } .c3 { grid-column: 3; } .c4 { grid-column: 4; } .c5 { grid-column: 5; }
.box { text-align: center; font-size: 13px; line-height: 1.25; padding: 9px 8px; border-radius: 10px; border: 1px solid var(--c-stroke);
  background: var(--c-surface); opacity: 0; transform: translateY(-14px) scale(.9); }
.box small { display: block; font-size: 10.5px; color: var(--c-ink-mod); margin-top: 2px; }
.box.on { opacity: 1; transform: none; animation: drop .55s cubic-bezier(.3, 1.6, .5, 1), shake .5s ease .45s; }
.box.main.on { animation: none; }
.box.main { font-weight: 600; }
.box.sa { background: linear-gradient(160deg, #0053A6, #1E2869); border-color: var(--c-sky); box-shadow: 0 0 18px rgba(100, 200, 250, .35); }
.box.sa small { color: #C4EFFF; }
.box.amber { border-color: var(--c-amber); box-shadow: 0 0 16px rgba(250, 196, 20, .3); background: var(--c-amber-bg); }
.box.red { border-color: var(--c-red); box-shadow: 0 0 16px rgba(255, 122, 122, .35); background: var(--c-red-bg); }
.box.purple { border-color: var(--c-purple); box-shadow: 0 0 16px rgba(196, 155, 255, .35); background: rgba(123, 63, 196, .18); }
@keyframes drop { from { opacity: 0; transform: translateY(-30px) scale(.8) } to { opacity: 1; transform: none } }
@keyframes shake { 20%, 60% { transform: translateX(-4px) rotate(-1deg) } 40%, 80% { transform: translateX(4px) rotate(1deg) } }
.cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-top: 10px; }
.card { font-size: 12.5px; line-height: 1.38; padding: 9px 12px; opacity: 0; transform: translateY(10px); transition: all .45s ease .25s; }
.card.on { opacity: 1; transform: none; }
.card.amber { border-left: 4px solid var(--c-amber); } .card.amber b { color: var(--c-amber); }
.card.red { border-left: 4px solid var(--c-red); } .card.red b { color: var(--c-red); }
.card.purple { border-left: 4px solid var(--c-purple); } .card.purple b { color: var(--c-purple); }
</style>
