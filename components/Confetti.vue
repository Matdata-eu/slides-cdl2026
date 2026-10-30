<script setup lang="ts">
// A one-shot confetti burst on a canvas laid over its parent. Fires when `fire` turns true.
import { onUnmounted, ref, watch } from 'vue'
const props = withDefaults(defineProps<{ fire: boolean, x?: number, y?: number, count?: number }>(), { x: 0.5, y: 0.5, count: 160 })
const canvas = ref<HTMLCanvasElement>()
const colours = ['#64C8FA', '#FAC414', '#35DE6D', '#C49BFF', '#FF8FB8', '#FFFFFF', '#FFA040']
let raf = 0
type P = { x: number, y: number, vx: number, vy: number, r: number, a: number, va: number, c: string, w: number, h: number, life: number }
function burst() {
  const el = canvas.value
  if (!el) return
  const ctx = el.getContext('2d')!
  const rect = el.getBoundingClientRect()
  const dpr = window.devicePixelRatio || 1
  el.width = rect.width * dpr
  el.height = rect.height * dpr
  ctx.scale(dpr, dpr)
  const ox = rect.width * props.x
  const oy = rect.height * props.y
  const ps: P[] = Array.from({ length: props.count }, () => {
    const ang = -Math.PI / 2 + (Math.random() - 0.5) * Math.PI * 1.1
    const sp = 4 + Math.random() * 7
    return { x: ox, y: oy, vx: Math.cos(ang) * sp, vy: Math.sin(ang) * sp, r: 0, a: Math.random() * 6, va: (Math.random() - 0.5) * 0.4,
      c: colours[Math.floor(Math.random() * colours.length)], w: 4 + Math.random() * 5, h: 3 + Math.random() * 4, life: 1 }
  })
  cancelAnimationFrame(raf)
  const tick = () => {
    ctx.clearRect(0, 0, rect.width, rect.height)
    let alive = 0
    for (const p of ps) {
      p.vy += 0.18; p.vx *= 0.99; p.vy *= 0.99
      p.x += p.vx; p.y += p.vy; p.a += p.va; p.life -= 0.006
      if (p.life <= 0 || p.y > rect.height + 20) continue
      alive++
      ctx.save(); ctx.globalAlpha = Math.max(p.life, 0); ctx.translate(p.x, p.y); ctx.rotate(p.a)
      ctx.fillStyle = p.c; ctx.fillRect(-p.w / 2, -p.h / 2, p.w, p.h * Math.abs(Math.cos(p.a * 2))); ctx.restore()
    }
    if (alive) raf = requestAnimationFrame(tick)
  }
  raf = requestAnimationFrame(tick)
}
watch(() => props.fire, (f) => { if (f) burst() })
onUnmounted(() => cancelAnimationFrame(raf))
</script>

<template>
  <canvas ref="canvas" class="confetti" />
</template>

<style scoped>
.confetti { position: absolute; inset: -40px; width: calc(100% + 80px); height: calc(100% + 80px); pointer-events: none; z-index: 30; }
</style>
