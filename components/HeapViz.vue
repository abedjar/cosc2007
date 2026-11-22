<script setup>
import { computed } from 'vue'
// a: entries in heap order (null = empty hole). hl = orange (being compared/moved),
// ok = green (settled/sorted), dim = grey (outside the heap). zero = 0-based labels.
const props = defineProps({
  a: { type: Array, default: () => [] },
  hl: { type: Array, default: () => [] },
  ok: { type: Array, default: () => [] },
  dim: { type: Array, default: () => [] },
  size: { type: Number, default: -1 },
  zero: Boolean, tree: { type: Boolean, default: true },
  arr: { type: Boolean, default: true }, w: { type: Number, default: 440 },
  idx: { type: Boolean, default: true }
})
const n = computed(() => props.size < 0 ? props.a.length : props.size)
const levels = computed(() => Math.max(1, Math.floor(Math.log2(Math.max(n.value, 1))) + 1))
const off = computed(() => props.zero ? 0 : 1)
const nodes = computed(() => props.a.slice(0, n.value).map((v, i) => {
  const k = i + 1, lv = Math.floor(Math.log2(k)), pos = k - 2 ** lv
  return { i, v, x: (pos + 0.5) / 2 ** lv * props.w, y: 24 + lv * 54, p: k > 1 ? Math.floor(k / 2) - 1 : -1 }
}))
const col = (i, v) => v == null ? 'hole' : props.ok.includes(i) ? 'ok' : props.hl.includes(i) ? 'hl' : props.dim.includes(i) ? 'dim' : 'n'
const cw = computed(() => Math.min(46, (props.w - 10) / (props.a.length + (props.zero ? 0 : 1))))
</script>
<template>
  <div class="hv">
    <svg v-if="tree" :width="w" :height="levels * 54 + 16" :viewBox="`0 0 ${w} ${levels * 54 + 16}`">
      <line v-for="o in nodes.filter(o => o.p >= 0)" :key="'e' + o.i" :x1="nodes[o.p].x" :y1="nodes[o.p].y" :x2="o.x" :y2="o.y" stroke="#64748b" stroke-width="1.5" />
      <g v-for="o in nodes" :key="o.i">
        <circle :cx="o.x" :cy="o.y" r="19" :class="col(o.i, o.v)" />
        <text :x="o.x" :y="o.y + 5" text-anchor="middle" class="t">{{ o.v ?? '' }}</text>
        <text v-if="idx" :x="o.x + 21" :y="o.y - 15" class="ix">{{ o.i + off }}</text>
      </g>
    </svg>
    <div v-if="arr" class="row">
      <div v-if="!zero" class="cell unused"><b>–</b><i>0</i></div>
      <div v-for="(v, i) in a" :key="i" class="cell" :class="i >= n ? 'dim' : col(i, v)" :style="{ width: cw + 'px' }">
        <b>{{ v ?? '' }}</b><i>{{ i + off }}</i>
      </div>
    </div>
  </div>
</template>
<style scoped>
.hv { display: inline-block; text-align: center }
.n { fill: #1e3a5f; stroke: #60a5fa; stroke-width: 2 } .hl { fill: #7c4a03; stroke: #fb923c; stroke-width: 3 }
.ok { fill: #14532d; stroke: #4ade80; stroke-width: 2 } .dim { fill: #1f2937; stroke: #475569; stroke-width: 2 }
.hole { fill: none; stroke: #94a3b8; stroke-width: 2; stroke-dasharray: 4 3 }
.t { fill: #f1f5f9; font: bold 14px 'Space Mono', monospace } .ix { fill: #94a3b8; font: 10px 'Space Mono', monospace }
.row { display: flex; justify-content: center; margin-top: 6px }
.cell { border: 2px solid #60a5fa; background: #1e3a5f; margin-right: -2px; padding: 2px 0; display: flex; flex-direction: column; font-family: 'Space Mono', monospace }
.cell b { font-size: 13px; min-height: 20px } .cell i { font-size: 10px; color: #94a3b8; font-style: normal; border-top: 1px solid #475569 }
.cell.hl { background: #7c4a03; border-color: #fb923c } .cell.ok { background: #14532d; border-color: #4ade80 }
.cell.dim, .cell.unused { background: #1f2937; border-color: #475569; width: 34px; opacity: .8 }
.cell.hole { background: transparent; border-style: dashed }
</style>
