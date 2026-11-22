<script setup>
import { computed } from 'vue'
// Node = [value, 'R' | 'B', left, right] (null = empty). Strings like 'T1' draw as subtree triangles.
// hl (orange ring) / bad (red-violation ring) / ok (green ring), matched by value. nil = draw black NIL leaves.
const props = defineProps({
  t: { default: null }, hl: { type: Array, default: () => [] }, bad: { type: Array, default: () => [] },
  ok: { type: Array, default: () => [] }, nil: Boolean, w: { type: Number, default: 400 }
})
const L = computed(() => {
  const nodes = []; let c = 0, maxD = 0
  const go = (t, d, par) => {
    if (!t) { if (props.nil && par) { nodes.push({ nil: true, d, par, x: c++ }); maxD = Math.max(maxD, d) } return }
    const me = { v: t[0], c: t[1], d, par, sub: typeof t[0] === 'string' && /^T\d/.test(t[0]) }
    nodes.push(me); maxD = Math.max(maxD, d)
    go(t[2], d + 1, me); me.x = c++; go(t[3], d + 1, me)
  }
  go(props.t, 0, null)
  const dx = Math.min(54, props.w / Math.max(c, 1)), off = (props.w - c * dx) / 2
  nodes.forEach(n => { n.X = (n.x + 0.5) * dx + off; n.Y = 26 + n.d * 58 })
  return { nodes, h: (maxD + 1) * 58 + 14 }
})
const ring = v => props.bad.includes(v) ? 'rb' : props.hl.includes(v) ? 'rh' : props.ok.includes(v) ? 'ro' : ''
</script>
<template>
  <svg :width="w" :height="L.h" :viewBox="`0 0 ${w} ${L.h}`">
    <line v-for="(n, i) in L.nodes.filter(n => n.par)" :key="'e' + i" :x1="n.par.X" :y1="n.par.Y" :x2="n.X" :y2="n.Y" stroke="#64748b" stroke-width="1.5" />
    <g v-for="(n, i) in L.nodes" :key="i" class="g">
      <rect v-if="n.nil" :x="n.X - 7" :y="n.Y - 7" width="14" height="14" class="B" />
      <template v-else>
        <circle v-if="ring(n.v)" :cx="n.X" :cy="n.Y" r="25" :class="'ring ' + ring(n.v)" />
        <polygon v-if="n.sub" :points="`${n.X},${n.Y - 18} ${n.X - 24},${n.Y + 20} ${n.X + 24},${n.Y + 20}`" class="S" />
        <circle v-else :cx="n.X" :cy="n.Y" r="19" :class="n.c" />
        <text :x="n.X" :y="n.Y + (n.sub ? 14 : 5)" text-anchor="middle" class="t">{{ n.v }}</text>
      </template>
    </g>
  </svg>
</template>
<style scoped>
.g * { transition: all .45s }
.R { fill: #dc2626; stroke: #fecaca; stroke-width: 2 } .B { fill: #020617; stroke: #cbd5e1; stroke-width: 2 }
.S { fill: #1e293b; stroke: #94a3b8; stroke-width: 2 }
.ring { fill: none; stroke-width: 3 } .rh { stroke: #fbbf24 } .ro { stroke: #4ade80 } .rb { stroke: #e879f9; stroke-dasharray: 5 3 }
.t { fill: #f8fafc; font: bold 14px 'Space Mono', monospace }
</style>
