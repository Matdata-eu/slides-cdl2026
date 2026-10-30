<script setup lang="ts">
// The cover's track: the train rolls in from the left and stops where the footer track starts on slide 2.
</script>

<template>
  <div class="ct">
    <svg viewBox="0 0 980 42" class="svg">
      <line x1="0" y1="31" x2="980" y2="31" class="sleepers" />
      <line x1="0" y1="31" x2="980" y2="31" class="rail" />
      <g class="train">
        <g transform="translate(-58 21)">
          <rect x="0" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
          <rect x="18" y="1" width="16" height="7.5" rx="1.6" class="wagon" />
          <path d="M36 1 h14 q6 0 8 7.5 h-22 z" class="loco" />
          <rect x="49" y="2.4" width="4.5" height="2.6" rx=".6" class="win" />
          <rect x="2" y="2.6" width="12" height="2" rx=".5" class="win" />
          <rect x="20" y="2.6" width="12" height="2" rx=".5" class="win" />
          <line x1="0" y1="6.3" x2="58" y2="6.3" class="stripe" />
          <circle v-for="w in [3, 13, 21, 31, 40, 52]" :key="w" :cx="w" cy="9" r="1.3" class="wheel" />
          <circle cx="58.5" cy="6" r="1.2" class="lamp" />
        </g>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.ct { position: absolute; left: 0; right: 0; bottom: 0; height: 42px; z-index: 5; }
.svg { width: 100%; height: 100%; display: block; overflow: visible; }
.sleepers { stroke: rgba(255, 255, 255, .5); stroke-width: 5; stroke-dasharray: 1 4; opacity: .35; }
.rail { stroke: #64C8FA; stroke-width: 1.4; opacity: .7; }
.train { animation: roll 2.6s cubic-bezier(.2, .7, .3, 1) .4s both; }
@keyframes roll { from { transform: translateX(-80px) } to { transform: translateX(70px) } }
.wagon { fill: #E8EEF8; stroke: #1E2869; stroke-width: .5; }
.loco { fill: #fff; stroke: #1E2869; stroke-width: .5; }
.win { fill: #1E2869; }
.stripe { stroke: #C8102E; stroke-width: .9; }
.wheel { fill: #1E2869; }
.lamp { fill: #FFE58A; filter: drop-shadow(0 0 3px #FFE58A); }
</style>
