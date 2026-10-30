<script setup lang="ts">
/**
 * railML generation before and after the refactor: splitting the KG into subnetwork graphs versus one
 * complete KG in Oxigraph where an area location sets the scope. step 0: before · step 1: after.
 */
import { computed } from 'vue'
import TimeBars from './TimeBars.vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const after = computed(() => props.step >= 1)
const MAX = 105
</script>

<template>
  <div class="rb">
    <div class="side before" :class="{ dim: after }">
      <div class="tag">Before</div>
      <div class="flow">
        <div class="chip">Complete KG<small>Fuseki</small></div>
        <div class="arrow" />
        <div class="chip bad">Split<small>15 sequential queries, one graph per subnetwork</small></div>
        <div class="arrow" />
        <div class="chip bad">rdflib in memory<small>one file per subnetwork; the complete network ran out of memory</small></div>
        <div class="arrow" />
        <div class="chip">railML<small>only for the fixed subnetworks</small></div>
      </div>
      <TimeBars :rows="[
        { label: 'Splitting', min: 90, tone: 'red' },
        { label: 'railML generation', min: 15, tone: 'amber' },
      ]" :max="MAX" />
    </div>

    <div class="side after" :class="{ on: after }">
      <div class="tag">After</div>
      <div class="flow">
        <div class="chip">Complete KG</div>
        <div class="arrow flow" />
        <div class="chip ok">Oxigraph<small>whole network in memory, loaded in 36 s</small></div>
        <div class="arrow flow" />
        <div class="chip ok">Scope = area location<small>a subnetwork, or any area you draw</small></div>
        <div class="arrow flow" />
        <div class="chip ok">railML 3.2 + 3.3</div>
      </div>
      <TimeBars v-if="after" :rows="[
        { label: 'Load into Oxigraph', min: 0.6, tone: 'green', time: '36 s' },
        { label: 'railML, complete network', min: 4.6, tone: 'green', note: 'both versions' },
      ]" :max="MAX" />
      <div class="saved" :class="{ on: after }">≈ 1.5 h saved on every run</div>
    </div>
  </div>
</template>

<style scoped>
.rb { display: flex; flex-direction: column; gap: 12px; }
.side { position: relative; padding: 10px 14px 12px 82px; border-radius: 14px; border: 1px solid var(--c-stroke); background: rgba(23, 33, 61, .6); transition: all .6s ease; display: flex; flex-direction: column; gap: 10px; }
.side.before.dim { opacity: .5; filter: saturate(.5); }
.side.after { opacity: 0; transform: translateY(18px); border-color: var(--c-green); box-shadow: 0 0 26px rgba(53, 222, 109, .18); min-height: 140px; }
.side.after.on { opacity: 1; transform: none; }
.tag { position: absolute; left: 14px; top: 18px; font-family: 'Space Grotesk', sans-serif; font-weight: 700; font-size: 13px; letter-spacing: .12em; text-transform: uppercase; }
.before .tag { color: var(--c-red); } .after .tag { color: var(--c-green); }
.flow { display: flex; align-items: stretch; }
.chip { flex: 1; min-width: 0; text-align: center; font-size: 13px; font-weight: 600; line-height: 1.2; padding: 7px 6px; border-radius: 10px; background: var(--c-surface); border: 1px solid var(--c-stroke); display: flex; flex-direction: column; justify-content: center; }
.chip small { font-weight: 400; font-size: 10.5px; color: var(--c-ink-mod); margin-top: 3px; }
.chip.bad { border-color: var(--c-red); background: var(--c-red-bg); }
.chip.ok { border-color: var(--c-green); background: var(--c-green-bg); }
.arrow { width: 24px; flex: none; position: relative; }
.arrow::before { content: ''; position: absolute; left: 3px; right: 6px; top: 50%; height: 2px; background: var(--c-ink-subtle); }
.arrow::after { content: ''; position: absolute; right: 1px; top: calc(50% - 4px); border: 5px solid transparent; border-left: 7px solid var(--c-ink-subtle); border-right: 0; }
.arrow.flow::before { background: repeating-linear-gradient(90deg, var(--c-green) 0 5px, transparent 5px 9px); background-size: 9px 2px; animation: dash .6s linear infinite; }
.arrow.flow::after { border-left-color: var(--c-green); }
@keyframes dash { to { background-position: 9px 0 } }
.saved { position: absolute; right: 16px; bottom: 12px; font-family: 'Space Grotesk', sans-serif; font-weight: 700; font-size: 20px; color: var(--c-green);
  text-shadow: 0 0 16px rgba(53, 222, 109, .5); opacity: 0; transform: scale(1.6); transition: all .45s cubic-bezier(.3, 1.6, .5, 1) 1.4s; }
.saved.on { opacity: 1; transform: none; }
</style>
