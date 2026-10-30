<script setup lang="ts">
// A number that counts up from zero whenever its slide is shown (static in exports).
import { onUnmounted, ref, watch } from 'vue'
import { useIsSlideActive, useNav } from '@slidev/client'
const props = withDefaults(defineProps<{ to: number, duration?: number, delay?: number, decimals?: number, prefix?: string, suffix?: string }>(),
  { duration: 1600, delay: 0, decimals: 0, prefix: '', suffix: '' })
const { isPrintMode } = useNav()
const isActive = useIsSlideActive()
const value = ref(props.to)
let raf = 0
let timer: ReturnType<typeof setTimeout> | undefined
function stop() { cancelAnimationFrame(raf); if (timer) clearTimeout(timer) }
watch(isActive, (active) => {
  stop()
  if (!active || isPrintMode.value) { value.value = props.to; return }
  value.value = 0
  timer = setTimeout(() => {
    const t0 = performance.now()
    const tick = (t: number) => {
      const p = Math.min((t - t0) / props.duration, 1)
      value.value = props.to * (1 - (1 - p) ** 3)
      if (p < 1) raf = requestAnimationFrame(tick)
    }
    raf = requestAnimationFrame(tick)
  }, props.delay)
}, { immediate: true })
onUnmounted(stop)
const fmt = (v: number) => v.toLocaleString('en-GB', { minimumFractionDigits: props.decimals, maximumFractionDigits: props.decimals })
</script>

<template>
  <span class="countup">{{ prefix }}{{ fmt(value) }}{{ suffix }}</span>
</template>

<style scoped>
.countup { font-variant-numeric: tabular-nums; }
</style>
