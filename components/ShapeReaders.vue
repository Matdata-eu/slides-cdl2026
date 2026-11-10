<script setup lang="ts">
/**
 * SHACL in DIM: one property shape that every reader of the graph uses.
 * step 0: before, shape sets written apart · step 1: one shape, five readers light up in turn.
 */
import { computed } from 'vue'
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
const after = computed(() => props.step >= 1)
</script>

<template>
  <div class="sr">
    <div class="lane before" :class="{ dim: after }">
      <div class="tag">Before</div>
      <div class="chip">Validation shapes</div>
      <div class="chip">Search shapes</div>
      <div class="chip">4 shape sets in 2 repositories<small>each describing the same classes for itself</small></div>
      <div class="arrow" />
      <div class="chip bad">Written apart, they drifted<small>17 properties under two names · 4 real conflicts</small></div>
    </div>

    <div class="after" :class="{ on: after }">
      <div class="shape glass">
        <div class="lbl">After · one shape, one repository</div>
        <pre><span class="k">shapes:Track-Type</span> a sh:PropertyShape ;
  sh:path railml3:hasType ;
  sh:name "has type"@en, "har type"@nb ;
  sh:datatype xsd:string ;
  sh:in ( "mainTrack" "secondaryTrack"
          "sidingTrack" "connectingTrack" ) ;
  sh:minCount 1 ; sh:maxCount 1 ;
  bno:key "subType" .</pre>
      </div>

      <div class="uses">
        <div class="use val" style="--d: .2s">
          <div class="h">✅ Validation <span>checks the data</span></div>
          <div class="bad-row">✖ track 7 · "tramTrack" is not an allowed type</div>
        </div>
        <div class="use srch" style="--d: .5s">
          <div class="h">🔎 Search <span>builds the query form</span></div>
          <div class="query"><span class="pill">Track</span><span class="rel">has type</span><span class="pill sel">mainTrack ▾</span></div>
        </div>
        <div class="use view" style="--d: .8s">
          <div class="h">🪪 Web page <span>labels the value</span></div>
          <div class="row"><span>has type</span><b>mainTrack</b></div>
        </div>
        <div class="use map" style="--d: 1.1s">
          <div class="h">🗺️ Map <span>offers the filter</span></div>
          <div class="facets">☑ mainTrack · ☐ secondaryTrack · ☐ sidingTrack · …</div>
        </div>
        <div class="use api" style="--d: 1.4s">
          <div class="h">🔌 REST API <span>names the field</span></div>
          <div class="mono">GET /tracks?subType=mainTrack → { "subType": "mainTrack" }</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.sr { display: flex; flex-direction: column; gap: 14px; }
.lane { display: flex; align-items: stretch; gap: 8px; position: relative; padding: 10px 12px 10px 74px; border-radius: 14px; border: 1px solid var(--c-stroke); background: rgba(23, 33, 61, .6); transition: all .6s ease; }
.lane.before.dim { opacity: .45; filter: saturate(.4); transform: scale(.97); }
.tag { position: absolute; left: 12px; top: 50%; transform: translateY(-50%); font-family: 'Poppins', sans-serif; font-weight: 700; font-size: 13px; letter-spacing: .12em; text-transform: uppercase; color: var(--c-red); }
.chip { flex: 1; min-width: 0; text-align: center; font-size: 12.5px; font-weight: 600; line-height: 1.2; padding: 7px 6px; border-radius: 10px; background: var(--c-surface); border: 1px solid var(--c-stroke); display: flex; flex-direction: column; justify-content: center; }
.chip small { font-weight: 400; font-size: 10.5px; color: var(--c-ink-mod); margin-top: 3px; }
.chip.bad { flex: 2; border-color: var(--c-red); background: var(--c-red-bg); }
.arrow { width: 22px; flex: none; position: relative; }
.arrow::before { content: ''; position: absolute; left: 2px; right: 6px; top: 50%; height: 2px; background: var(--c-ink-subtle); }
.arrow::after { content: ''; position: absolute; right: 1px; top: calc(50% - 4px); border: 5px solid transparent; border-left: 7px solid var(--c-ink-subtle); border-right: 0; }

.after { display: grid; grid-template-columns: 1fr 1.3fr; gap: 16px; align-items: start; opacity: 0; transform: translateY(18px); transition: all .6s ease; }
.after.on { opacity: 1; transform: none; }
.shape { padding: 10px 14px; border: 1px solid var(--c-green); box-shadow: 0 0 26px rgba(53, 222, 109, .18); }
.lbl { font-size: 11px; letter-spacing: .1em; text-transform: uppercase; color: var(--c-green); font-weight: 700; margin-bottom: 6px; }
pre { margin: 0; font-family: ui-monospace, monospace; font-size: 12.5px; line-height: 1.55; color: var(--c-ink); white-space: pre; }
.k { font-weight: 700; color: var(--c-sky); }

.uses { display: grid; gap: 7px; font-size: 13px; }
.use { border: 1px solid var(--c-stroke); border-left: 5px solid; border-radius: 10px; padding: 5px 12px; background: var(--c-surface); opacity: 0; transform: translateX(12px); transition: all .45s ease; }
.after.on .use { opacity: 1; transform: none; transition-delay: var(--d); }
.val { border-left-color: var(--c-red); }
.srch { border-left-color: var(--c-blue); }
.view { border-left-color: var(--c-teal); }
.map { border-left-color: var(--c-green); }
.api { border-left-color: var(--c-purple); }
.h { font-weight: 700; margin-bottom: 2px; }
.h span { font-weight: 400; color: var(--c-ink-mod); margin-left: 6px; }
.bad-row { background: var(--c-red-bg); color: var(--c-red-ink); border-radius: 6px; padding: 1px 6px; font-size: 11.5px; }
.query { display: flex; align-items: center; gap: 8px; font-size: 12px; }
.pill { border: 1px solid var(--c-blue); border-radius: 14px; padding: 0 10px; color: var(--c-blue); }
.pill.sel { background: var(--c-blue-bg); }
.rel { color: var(--c-ink-mod); font-style: italic; }
.row { display: grid; grid-template-columns: 7rem 1fr; font-size: 12px; }
.row span { color: var(--c-ink-mod); }
.facets { font-size: 12px; }
.mono { font-family: ui-monospace, monospace; font-size: 11px; }
</style>
