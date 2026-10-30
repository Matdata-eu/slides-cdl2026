<script setup lang="ts">
import { useNav } from '@slidev/client'
const { currentLayout } = useNav()
</script>

<template>
  <div v-if="!['cover', 'end', 'full'].includes(currentLayout)" class="abs-tr mt-4 mr-6 md-logo">
    <img src="/assets/matdata.svg" class="h-7" alt="Matdata" />
  </div>
</template>

<style>
.md-logo { opacity: .9; }
</style>
