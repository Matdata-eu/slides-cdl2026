<script setup lang="ts">
/**
 * Pipeline stages as bars that grow to their run time, one after another, when the slide is shown.
 * `max` sets the scale in minutes, so two TimeBars on one slide can share it.
 */
import { onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'
type Row = { label: string, min: number, tone?: string, note?: string, time?: string }
const props = withDefaults(defineProps<{ rows: Row[], max?: number, delay?: number, stagger?: number }>(), { delay: 300, stagger: 350 })
const max = props.max ?? Math.max(...props.rows.map(r => r.min))
const fmt = (m: number) => m >= 60 ? `${Math.floor(m / 60)} h${m % 60 ? ` ${Math.round(m % 60)}` : ''}` : `${m < 1 ? m.toFixed(1) : Math.round(m * 10) / 10} min`

const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const grown = ref(props.rows.length)
const timers: ReturnType<typeof setTimeout>[] = []
watch(isActive, (active) => {
  timers.splice(0).forEach(clearTimeout)
  if (!active || isPrintMode.value) { grown.value = props.rows.length; return }
  grown.value = 0
  props.rows.forEach((_, i) => timers.push(setTimeout(() => { grown.value = i + 1 }, props.delay + i * props.stagger)))
}, { immediate: true })
onUnmounted(() => timers.forEach(clearTimeout))
</script>

<template>
  <div class="tb">
    <div v-for="(r, i) in rows" :key="r.label" class="row">
      <div class="lab">{{ r.label }}<span v-if="r.note" class="note">{{ r.note }}</span></div>
      <div class="track">
        <div class="bar" :class="r.tone ?? 'sky'" :style="{ width: i < grown ? `${Math.max((r.min / max) * 100, 0.8)}%` : '0%' }" />
        <span class="t" :class="{ on: i < grown }">{{ r.time ?? fmt(r.min) }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.tb { display: flex; flex-direction: column; gap: 7px; }
.row { display: grid; grid-template-columns: 210px 1fr; align-items: center; gap: 10px; }
.lab { font-size: 13px; text-align: right; line-height: 1.15; }
.note { display: block; font-size: 10.5px; color: var(--c-ink-mod); }
.track { position: relative; height: 20px; display: flex; align-items: center; }
.bar { height: 100%; border-radius: 4px; transition: width 1s cubic-bezier(.3, .9, .3, 1); }
.t { margin-left: 8px; font-size: 13px; font-weight: 700; white-space: nowrap; opacity: 0; transition: opacity .3s ease .7s; font-variant-numeric: tabular-nums; }
.t.on { opacity: 1; }
.bar.sky { background: linear-gradient(90deg, #0053A6, #64C8FA); box-shadow: 0 0 12px rgba(100, 200, 250, .35); }
.bar.green { background: linear-gradient(90deg, #11603A, #35DE6D); box-shadow: 0 0 12px rgba(53, 222, 109, .35); }
.bar.amber { background: linear-gradient(90deg, #7A5700, #FAC414); box-shadow: 0 0 12px rgba(250, 196, 20, .3); }
.bar.red { background: linear-gradient(90deg, #8A0F22, #FF7A7A); box-shadow: 0 0 14px rgba(255, 122, 122, .4); }
.bar.purple { background: linear-gradient(90deg, #4B2380, #C49BFF); }
.bar.gray { background: linear-gradient(90deg, #394150, #7F858E); }
</style>
