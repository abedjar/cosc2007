<script setup>
import { computed } from 'vue'
// t = nested [value, left, right] (null = empty). Strings like 'T1' draw as subtree triangles.
// bf shown = height(left) - height(right). hl = orange, bad = red, ok = green (by value).
const props = defineProps({
  t: { default: null }, hl: { type: Array, default: () => [] }, bad: { type: Array, default: () => [] },
  ok: { type: Array, default: () => [] }, bf: { type: Boolean, default: true }, w: { type: Number, default: 400 }
})
const L = computed(() => {
  const nodes = []; let c = 0, maxD = 0
  const go = (t, d, par) => {
    if (!t) return 0
    const me = { v: t[0], d, par, x: 0, sub: typeof t[0] === 'string' && /^T\d/.test(t[0]) }
    nodes.push(me)
    const hl = go(t[1], d + 1, me); me.x = c++; const hr = go(t[2], d + 1, me)
    me.bf = hl - hr; maxD = Math.max(maxD, d); return 1 + Math.max(hl, hr)
  }
  go(props.t, 0, null)
  const dx = Math.min(54, props.w / Math.max(c, 1)), off = (props.w - c * dx) / 2
  nodes.forEach(n => { n.X = (n.x + 0.5) * dx + off; n.Y = 24 + n.d * 58 })
  return { nodes, h: (maxD + 1) * 58 + 12 }
})
const cls = v => props.bad.includes(v) ? 'bad' : props.ok.includes(v) ? 'ok' : props.hl.includes(v) ? 'hl' : 'n'
</script>
<template>
  <svg :width="w" :height="L.h" :viewBox="`0 0 ${w} ${L.h}`">
    <line v-for="(n, i) in L.nodes.filter(n => n.par)" :key="'e' + i" :x1="n.par.X" :y1="n.par.Y" :x2="n.X" :y2="n.Y" stroke="#64748b" stroke-width="1.5" />
    <g v-for="(n, i) in L.nodes" :key="i" class="g">
      <polygon v-if="n.sub" :points="`${n.X},${n.Y - 18} ${n.X - 24},${n.Y + 20} ${n.X + 24},${n.Y + 20}`" :class="cls(n.v)" />
      <circle v-else :cx="n.X" :cy="n.Y" r="19" :class="cls(n.v)" />
      <text :x="n.X" :y="n.Y + (n.sub ? 14 : 5)" text-anchor="middle" class="t">{{ n.v }}</text>
      <text v-if="bf && !n.sub" :x="n.X + 21" :y="n.Y + 24" class="bf" :class="{ x: Math.abs(n.bf) > 1 }">{{ n.bf > 0 ? '+' + n.bf : n.bf }}</text>
    </g>
  </svg>
</template>
<style scoped>
.g * { transition: all .4s }
.n { fill: #1e3a5f; stroke: #60a5fa; stroke-width: 2 } .hl { fill: #7c4a03; stroke: #fb923c; stroke-width: 3 }
.ok { fill: #14532d; stroke: #4ade80; stroke-width: 2 } .bad { fill: #7f1d1d; stroke: #f87171; stroke-width: 3 }
.t { fill: #f1f5f9; font: bold 14px 'Space Mono', monospace }
.bf { fill: #94a3b8; font: 11px 'Space Mono', monospace } .bf.x { fill: #f87171; font-weight: bold }
</style>
