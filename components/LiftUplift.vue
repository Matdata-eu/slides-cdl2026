<script setup lang="ts">
/**
 * Before and after the kg-gen refactor. Before: one CONSTRUCT per file straight into railML/RTM, then
 * ordered UPDATEs. After: lift every source mechanically into its own namespace, then uplift it with
 * SHACL shapes that validate and carry the rule. step 0: before · step 1: after.
 */
import { computed } from 'vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const after = computed(() => props.step >= 1)
</script>

<template>
  <div class="lu">
    <div class="lane before" :class="{ dim: after }">
      <div class="tag">Before</div>
      <div class="chip">71 source files</div>
      <div class="arrow" />
      <div class="chip bad">71 CONSTRUCTs<small>source → railML / RTM in one step · 63% copy-paste</small></div>
      <div class="arrow" />
      <div class="chip bad">61 ordered UPDATEs<small>delete and rewrite each other</small></div>
      <div class="arrow" />
      <div class="chip goal">DIM<small>bnd:</small></div>
    </div>

    <div class="lane after" :class="{ on: after }">
      <div class="tag">After</div>
      <div class="chip">Source tables<small>Trasé · DIM DB · …</small></div>
      <div class="arrow flow" />
      <div class="chip lift">Lift<small>one generic query per table: every column becomes a source property</small></div>
      <div class="arrow flow" />
      <div class="chip src">Source graphs<small><code>trase:</code> · <code>dimdb:</code> · each in its own named graph</small></div>
      <div class="arrow flow" />
      <div class="chip rule">Uplift shapes<small>validate the source, then a SHACL rule builds DIM</small></div>
      <div class="arrow flow" />
      <div class="chip goal">DIM<small>bnd: + prov:wasDerivedFrom</small></div>
    </div>

    <div class="wins" :class="{ on: after }">
      <div class="win glass"><b>Source concepts first.</b> The graph speaks Trasé before it speaks railML. Nothing is lost or guessed in the lift.</div>
      <div class="win glass"><b>One rule, one place.</b> Validity, designators, geometry: written once in a shared shape, not 45 times.</div>
      <div class="win glass"><b>Lineage for free.</b> Every DIM entity points to the source version it came from, and the rule that made it.</div>
    </div>
  </div>
</template>

<style scoped>
.lu { display: flex; flex-direction: column; gap: 14px; }
.lane { display: flex; align-items: stretch; gap: 0; position: relative; padding: 12px 12px 12px 74px; border-radius: 14px; border: 1px solid var(--c-stroke); background: rgba(23, 33, 61, .6); transition: all .6s ease; }
.lane.before.dim { opacity: .45; filter: saturate(.4); transform: scale(.97); }
.lane.after { opacity: 0; transform: translateY(18px); border-color: var(--c-green); box-shadow: 0 0 26px rgba(53, 222, 109, .18); }
.lane.after.on { opacity: 1; transform: none; }
.tag { position: absolute; left: 12px; top: 50%; transform: translateY(-50%); font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 13px; letter-spacing: .12em; text-transform: uppercase; }
.before .tag { color: var(--c-red); } .after .tag { color: var(--c-green); }
.chip { flex: 1; min-width: 0; text-align: center; font-size: 13px; font-weight: 600; line-height: 1.2; padding: 8px 6px; border-radius: 10px; background: var(--c-surface); border: 1px solid var(--c-stroke); display: flex; flex-direction: column; justify-content: center; }
.chip small { font-weight: 400; font-size: 10.5px; color: var(--c-ink-mod); margin-top: 3px; }
.chip.bad { border-color: var(--c-red); background: var(--c-red-bg); }
.chip.lift { border-color: var(--c-sky); background: rgba(0, 83, 166, .25); }
.chip.src { border-color: var(--c-sky); background: rgba(100, 200, 250, .12); }
.chip.rule { border-color: var(--c-purple); background: rgba(123, 63, 196, .2); }
.chip.goal { flex: .7; border-color: var(--c-green); background: var(--c-green-bg); }
.arrow { width: 26px; flex: none; position: relative; }
.arrow::before { content: ''; position: absolute; left: 3px; right: 6px; top: 50%; height: 2px; background: var(--c-ink-subtle); }
.arrow::after { content: ''; position: absolute; right: 1px; top: calc(50% - 4px); border: 5px solid transparent; border-left: 7px solid var(--c-ink-subtle); border-right: 0; }
.arrow.flow::before { background: repeating-linear-gradient(90deg, var(--c-green) 0 5px, transparent 5px 9px); background-size: 9px 2px; animation: dash .6s linear infinite; }
.arrow.flow::after { border-left-color: var(--c-green); }
@keyframes dash { to { background-position: 9px 0 } }
.wins { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; opacity: 0; transform: translateY(10px); transition: all .5s ease .4s; }
.wins.on { opacity: 1; transform: none; }
.win { font-size: 12.5px; line-height: 1.4; padding: 9px 12px; border-top: 3px solid var(--c-green); }
.win b { color: var(--c-green); }
</style>
