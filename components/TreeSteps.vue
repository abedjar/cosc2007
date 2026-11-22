<script setup>
import { ref, computed, onUnmounted } from 'vue'
import AvlTree from './AvlTree.vue'
// frames: [{ t, hl, bad, ok, note }]  — step with the buttons or press Play.
const props = defineProps({ frames: { type: Array, required: true }, bf: { type: Boolean, default: true }, w: { type: Number, default: 400 }, ms: { type: Number, default: 2400 }, h: { type: Number, default: 230 } })
const i = ref(0), playing = ref(false); let timer = null
const f = computed(() => props.frames[i.value])
const stop = () => { playing.value = false; clearInterval(timer) }
const next = () => { if (i.value < props.frames.length - 1) i.value++; else stop() }
const prev = () => { stop(); if (i.value > 0) i.value-- }
const play = () => { if (playing.value) return stop(); if (i.value >= props.frames.length - 1) i.value = 0; playing.value = true; timer = setInterval(next, props.ms) }
const jump = k => { stop(); i.value = k }
onUnmounted(stop)
</script>
<template>
  <div class="ts">
    <div class="stage" :style="{ minHeight: h + 'px' }"><AvlTree :t="f.t" :hl="f.hl || []" :bad="f.bad || []" :ok="f.ok || []" :bf="bf" :w="w" /></div>
    <div class="note">{{ f.note }}</div>
    <div class="ctl">
      <button @click="prev">◀</button>
      <button @click="play">{{ playing ? '⏸' : '▶ play' }}</button>
      <button @click="next(); stop()">▶|</button>
      <span v-for="(_, k) in frames" :key="k" class="dot" :class="{ on: k === i }" @click="jump(k)" />
    </div>
  </div>
</template>
<style scoped>
.ts { text-align: center } .stage { display: flex; align-items: center; justify-content: center }
.note { min-height: 3.2em; font-size: 15px; color: #e2e8f0; margin: 4px 0 }
.ctl { display: flex; gap: 8px; justify-content: center; align-items: center }
button { background: #1e293b; border: 1px solid #475569; color: #e2e8f0; border-radius: 6px; padding: 2px 10px; font: 13px 'Space Mono', monospace; cursor: pointer }
.dot { width: 9px; height: 9px; border-radius: 50%; background: #475569; cursor: pointer } .dot.on { background: #fb923c }
</style>
