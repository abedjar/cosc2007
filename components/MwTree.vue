<script setup>
import { computed } from 'vue'
// Multiway tree. Node = [[keys], child1, child2, ...]. Leaf = [[10, 20]]. Empty node = [[]].
// hl (orange) / bad (red) / ok (green): a node takes that colour if it holds any listed key.
const props = defineProps({
  t: { default: null }, hl: { type: Array, default: () => [] }, bad: { type: Array, default: () => [] },
  ok: { type: Array, default: () => [] }, w: { type: Number, default: 460 }
})
const L = computed(() => {
  const nodes = []; let cx = 0, maxD = 0
  const go = (t, d, par, idx, nk) => {
    const keys = t[0], kids = t.slice(1)
    const me = { keys, d, par, idx, nk, w: Math.max(keys.length, 1) * 30 + 12, kids: [] }
    nodes.push(me); maxD = Math.max(maxD, d)
    if (!kids.length) { me.x = cx + me.w / 2; cx += me.w + 18 }
    else { kids.forEach((k, i) => me.kids.push(go(k, d + 1, me, i, kids.length))); me.x = (me.kids[0].x + me.kids[me.kids.length - 1].x) / 2 }
    me.y = 22 + d * 64
    return me
  }
  if (props.t) go(props.t, 0, null, 0, 1)
  const total = Math.max(cx - 18, 40), H = (maxD + 1) * 64 + 8, s = Math.min(1, props.w / total)
  return { nodes, total, H, s }
})
const cls = n => n.keys.length === 0 ? 'empty' : n.keys.some(k => props.bad.includes(k)) ? 'bad' : n.keys.some(k => props.ok.includes(k)) ? 'ok' : n.keys.some(k => props.hl.includes(k)) ? 'hl' : 'n'
const ax = n => n.par.x - n.par.w / 2 + n.par.w * (n.idx + 0.5) / n.nk
</script>
<template>
  <svg :width="L.total * L.s" :height="L.H * L.s" :viewBox="`0 0 ${L.total} ${L.H}`">
    <line v-for="(n, i) in L.nodes.filter(n => n.par)" :key="'e' + i" :x1="ax(n)" :y1="n.par.y + 14" :x2="n.x" :y2="n.y - 14" stroke="#64748b" stroke-width="1.5" />
    <g v-for="(n, i) in L.nodes" :key="i" class="g">
      <rect :x="n.x - (n.keys.length ? n.w : 42) / 2" :y="n.y - 14" :width="n.keys.length ? n.w : 42" height="28" rx="6" :class="cls(n)" />
      <text v-for="(k, j) in n.keys" :key="j" :x="n.x - n.w / 2 + 6 + 30 * j + 15" :y="n.y + 5" text-anchor="middle" class="t">{{ k }}</text>
    </g>
  </svg>
</template>
<style scoped>
.g * { transition: all .4s }
.n { fill: #1e3a5f; stroke: #60a5fa; stroke-width: 2 } .hl { fill: #7c4a03; stroke: #fb923c; stroke-width: 3 }
.ok { fill: #14532d; stroke: #4ade80; stroke-width: 2 } .bad { fill: #7f1d1d; stroke: #f87171; stroke-width: 3 }
.empty { fill: none; stroke: #f87171; stroke-width: 2; stroke-dasharray: 4 3 }
.t { fill: #f1f5f9; font: bold 14px 'Space Mono', monospace }
</style>
