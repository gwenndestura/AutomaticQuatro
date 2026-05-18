<script setup>
import { computed } from 'vue'

const props = defineProps({
    problemId: { type: Number, required: true }
})

const REGEX_MAP = {
  1: '(bab+bbb)a*b*(a*+b*)(ba)*(aba)(bab+aba)*bb(a+b)*(bab+aba)(a+b)*',
  2: '(1+0)*1*0*(101+01+000)(1+0)*(101+00)*(111+00+101)(1+0)*'
}

const NW = 34  // oblong half-width
const NH = 15  // oblong half-height

const PDA_CONFIGS = {
  1: {
    start: 'q0',
    accept: ['q9'],
    nodes: [
      { id: 'q0',  label: 'q₀', type: 'start',  x: 80,   y: 260 },
      { id: 'qT1', label: 'T₁', type: 'trap',   x: 80,   y: 410 },
      { id: 'q1',  label: 'q₁', type: 'state',  x: 260,  y: 260 },
      { id: 'q2',  label: 'q₂', type: 'state',  x: 420,  y: 155 },
      { id: 'qT2', label: 'T₂', type: 'trap',   x: 420,  y: 20  },
      { id: 'q3',  label: 'q₃', type: 'state',  x: 420,  y: 370 },
      { id: 'qT3', label: 'T₃', type: 'trap',   x: 420,  y: 505 },
      { id: 'q4',  label: 'q₄', type: 'state',  x: 590,  y: 260 },
      { id: 'q5',  label: 'q₅', type: 'state',  x: 750,  y: 155 },
      { id: 'q6',  label: 'q₆', type: 'state',  x: 910,  y: 260 },
      { id: 'q7',  label: 'q₇', type: 'state',  x: 910,  y: 390 },
      { id: 'q8',  label: 'q₈', type: 'state',  x: 1070, y: 155 },
      { id: 'q9',  label: 'q₉', type: 'accept', x: 1230, y: 260 },
    ],
    links: [
      { src: 'q0',  tgt: 'q1',  label: 'b, Δ/Δ' },
      { src: 'q0',  tgt: 'qT1', label: 'a, Δ/Δ' },
      { src: 'qT1', tgt: 'qT1', label: 'a,b, Δ/Δ' },
      { src: 'q1',  tgt: 'q2',  label: 'a, Δ/Δ' },
      { src: 'q1',  tgt: 'q3',  label: 'b, Δ/Δ' },
      { src: 'q2',  tgt: 'q4',  label: 'b, Δ/Δ' },
      { src: 'q2',  tgt: 'qT2', label: 'a, Δ/Δ' },
      { src: 'qT2', tgt: 'qT2', label: 'a,b, Δ/Δ' },
      { src: 'q3',  tgt: 'q4',  label: 'b, Δ/Δ' },
      { src: 'q3',  tgt: 'qT3', label: 'a, Δ/Δ' },
      { src: 'qT3', tgt: 'qT3', label: 'a,b, Δ/Δ' },
      { src: 'q4',  tgt: 'q5',  label: 'a, Δ/Δ' },
      { src: 'q4',  tgt: 'q4',  label: 'b, Δ/Δ' },
      { src: 'q5',  tgt: 'q5',  label: 'a, Δ/Δ' },
      { src: 'q5',  tgt: 'q6',  label: 'b, Δ/Δ' },
      { src: 'q6',  tgt: 'q7',  label: 'a, Δ/Δ',  curve: 10001 },
      { src: 'q6',  tgt: 'q4',  label: 'b, Δ/Δ',  curve: 1.4, sweep: 1 },
      { src: 'q7',  tgt: 'q5',  label: 'a, Δ/Δ',  curve: 1.3, sweep: 0 },
      { src: 'q7',  tgt: 'q8',  label: 'b, Δ/Δ' },
      { src: 'q8',  tgt: 'q5',  label: 'a, Δ/Δ',  curve: 1.4, sweep: 0 },
      { src: 'q8',  tgt: 'q9',  label: 'b, Δ/Δ' },
      { src: 'q9',  tgt: 'q9',  label: 'a,b, Δ/Δ' },
    ]
  },
  2: {
    start: 'p0',
    accept: ['p10'],
    nodes: [
      { id: 'p0',  label: 'p₀',  type: 'start',  x: 80,   y: 250 },
      { id: 'p1',  label: 'p₁',  type: 'state',  x: 240,  y: 165 },
      { id: 'p2',  label: 'p₂',  type: 'state',  x: 240,  y: 335 },
      { id: 'p3',  label: 'p₃',  type: 'state',  x: 400,  y: 165 },
      { id: 'p4',  label: 'p₄',  type: 'state',  x: 400,  y: 335 },
      { id: 'p5',  label: 'p₅',  type: 'state',  x: 560,  y: 250 },
      { id: 'p6',  label: 'p₆',  type: 'state',  x: 720,  y: 165 },
      { id: 'p7',  label: 'p₇',  type: 'state',  x: 720,  y: 335 },
      { id: 'p8',  label: 'p₈',  type: 'state',  x: 880,  y: 165 },
      { id: 'p9',  label: 'p₉',  type: 'state',  x: 910,  y: 385 },
      { id: 'p10', label: 'p₁₀', type: 'accept', x: 1060, y: 250 },
    ],
    links: [
      { src: 'p0',  tgt: 'p1',  label: '0, Δ/Δ' },
      { src: 'p0',  tgt: 'p2',  label: '1, Δ/Δ' },
      { src: 'p1',  tgt: 'p4',  label: '0, Δ/Δ' },
      { src: 'p1',  tgt: 'p5',  label: '1, Δ/Δ', curve: 1.6, sweep: 0 },
      { src: 'p2',  tgt: 'p3',  label: '0, Δ/Δ' },
      { src: 'p2',  tgt: 'p2',  label: '1, Δ/Δ' },
      { src: 'p3',  tgt: 'p5',  label: '1, Δ/Δ', curve: 1.6, sweep: 1 },
      { src: 'p3',  tgt: 'p1',  label: '0, Δ/Δ', curve: 1.4, sweep: 1 },
      { src: 'p4',  tgt: 'p5',  label: '0, Δ/Δ' },
      { src: 'p4',  tgt: 'p2',  label: '1, Δ/Δ', curve: 1.4, sweep: 0 },
      { src: 'p5',  tgt: 'p6',  label: '0, Δ/Δ' },
      { src: 'p5',  tgt: 'p7',  label: '1, Δ/Δ' },
      { src: 'p6',  tgt: 'p10', label: '0, Δ/Δ', curve: 1.6, sweep: 0 },
      { src: 'p6',  tgt: 'p7',  label: '1, Δ/Δ', curve: 10001 },
      { src: 'p7',  tgt: 'p8',  label: '0, Δ/Δ' },
      { src: 'p7',  tgt: 'p9',  label: '1, Δ/Δ' },
      { src: 'p8',  tgt: 'p10', label: '1, Δ/Δ', curve: 1.6, sweep: 1 },
      { src: 'p8',  tgt: 'p6',  label: '0, Δ/Δ', curve: 1.4, sweep: 1 },
      { src: 'p9',  tgt: 'p10', label: '1, Δ/Δ' },
      { src: 'p9',  tgt: 'p6',  label: '0, Δ/Δ', curve: 1.4, sweep: 0 },
      { src: 'p10', tgt: 'p10', label: '0,1, Δ/Δ' },
    ]
  }
}

const pda = computed(() => PDA_CONFIGS[props.problemId])

const nodeMap = computed(() => {
    const m = {}
    pda.value.nodes.forEach(n => { m[n.id] = n })
    return m
})

// Get point on oblong edge (treating as ellipse) in direction of target
const getEdge = (cx, cy, tx, ty) => {
    const dx = tx - cx, dy = ty - cy
    const dist = Math.sqrt(dx * dx + dy * dy)
    if (dist === 0) return { x: cx, y: cy }
    const nx = dx / dist, ny = dy / dist
    const t = 1 / Math.sqrt((nx / NW) ** 2 + (ny / NH) ** 2)
    return { x: cx + t * nx, y: cy + t * ny }
}

const linkData = computed(() => {
    return pda.value.links.map((lnk) => {
        const sn = nodeMap.value[lnk.src]
        const tn = nodeMap.value[lnk.tgt]
        if (!sn || !tn) return { ...lnk, d: '', lx: 0, ly: 0, isSelf: false }

        // Self-loop
        if (lnk.src === lnk.tgt) {
            const cx = sn.x, cy = sn.y
            const lr = 18
            const ex = 10, ey = NH
            const d = `M${cx - ex},${cy - ey} A ${lr} ${lr} 0 1 1 ${cx + ex},${cy - ey}`
            return { ...lnk, d, lx: cx, ly: cy - NH - 44, isSelf: true }
        }

        const se = getEdge(sn.x, sn.y, tn.x, tn.y)
        const te = getEdge(tn.x, tn.y, sn.x, sn.y)
        const dx = te.x - se.x, dy = te.y - se.y
        let dist = Math.sqrt(dx * dx + dy * dy)

        // Straight line
        if (lnk.curve && lnk.curve > 1000) {
            const d = `M${se.x},${se.y} L${te.x},${te.y}`
            return { ...lnk, d, lx: (se.x + te.x) / 2, ly: (se.y + te.y) / 2 - 8, isSelf: false }
        }

        const snNum = parseInt(lnk.src.replace(/\D/g, '')) || 0
        const tnNum = parseInt(lnk.tgt.replace(/\D/g, '')) || 0
        const sweep = lnk.sweep !== undefined ? lnk.sweep : (snNum < tnNum ? 1 : 0)
        const dr = dist * (lnk.curve || 1.3)
        const d = `M${se.x},${se.y} A ${dr} ${dr} 0 0 ${sweep} ${te.x},${te.y}`

        // Approximate label midpoint: offset perpendicular to chord
        const mx = (se.x + te.x) / 2
        const my = (se.y + te.y) / 2
        const perpX = -dy / dist
        const perpY = dx / dist
        const sag = dist * 0.18
        const sign = sweep === 1 ? -1 : 1
        const lx = mx + sign * perpX * sag
        const ly = my + sign * perpY * sag

        return { ...lnk, d, lx, ly, isSelf: false }
    })
})

const nodeColor = (node) => {
    if (node.type === 'accept') return '#4caf50'
    if (node.type === 'start')  return '#2563eb'
    if (node.type === 'trap')   return '#ef4444'
    return '#f59e0b'
}

const viewBox = computed(() => {
    const ns = pda.value.nodes
    const xs = ns.map(n => n.x)
    const ys = ns.map(n => n.y)
    const pad = 70
    const minX = Math.min(...xs) - NW - pad
    const maxX = Math.max(...xs) + NW + pad
    const minY = Math.min(...ys) - NH - pad
    const maxY = Math.max(...ys) + NH + pad
    return `${minX} ${minY} ${maxX - minX} ${maxY - minY}`
})
</script>

<template>
  <div class="pda-wrap">

    <!-- Header -->
    <div class="pda-header">
      <div class="header-left">
        <span class="badge">PDA</span>
        <span class="title">Problem {{ problemId }}</span>
      </div>
      <div class="header-right">
        <svg width="34" height="18" style="vertical-align:middle;flex-shrink:0">
          <rect x="1" y="1" width="32" height="16" rx="8" fill="#ff9800"/>
        </svg>
        <span class="leg">State</span>

        <svg width="18" height="18" style="vertical-align:middle;flex-shrink:0;margin-left:10px">
          <polygon points="9,0 18,9 9,18 0,9" fill="#64748b"/>
        </svg>
        <span class="leg">Read / Pop</span>

        <svg width="22" height="14" style="vertical-align:middle;flex-shrink:0;margin-left:10px">
          <rect x="1" y="1" width="20" height="12" rx="2" fill="#64748b"/>
        </svg>
        <span class="leg">Push</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap" v-if="REGEX_MAP[problemId]">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ REGEX_MAP[problemId] }}</code>
    </div>

    <!-- Notation hint -->
    <div class="notation-card">
      <span class="note-icon">◇</span>
      <span class="note-text">Transition format: <code>input, pop → push</code> — Δ = blank stack symbol</span>
      <span class="note-icon" style="margin-left:12px">□</span>
      <span class="note-text">Double border = accept state &nbsp;|&nbsp; Red = trap (dead) state</span>
    </div>

    <!-- Diagram -->
    <div class="pda-viz-card">
      <div class="viz-head">
        <span>Pushdown Automata Diagram</span>
      </div>
      <div class="image-viewport">
        <svg :viewBox="viewBox" style="width:100%;max-height:540px;overflow:visible" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <marker id="pda-arr" viewBox="0 -5 10 10" refX="10" refY="0"
              markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#334155"/>
            </marker>
            <marker id="pda-arr-s" viewBox="0 -5 10 10" refX="9" refY="0"
              markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#334155"/>
            </marker>
          </defs>

          <!-- Start arrow -->
          <line
            :x1="(nodeMap[pda.start]?.x ?? 0) - NW - 38"
            :y1="nodeMap[pda.start]?.y ?? 0"
            :x2="(nodeMap[pda.start]?.x ?? 0) - NW"
            :y2="nodeMap[pda.start]?.y ?? 0"
            stroke="#334155" stroke-width="2"
            marker-end="url(#pda-arr)"
          />

          <!-- Edges -->
          <g v-for="(lnk, i) in linkData" :key="'e'+i">
            <path
              :d="lnk.d"
              fill="none"
              stroke="#334155"
              stroke-width="1.8"
              :marker-end="lnk.isSelf ? 'url(#pda-arr-s)' : 'url(#pda-arr)'"
            />
            <text
              :x="lnk.lx"
              :y="lnk.ly"
              font-size="11"
              fill="#e63946"
              font-weight="bold"
              text-anchor="middle"
              font-family="Courier New, monospace"
              style="paint-order:stroke;stroke:#ffffff;stroke-width:4px;stroke-linejoin:round"
            >{{ lnk.label }}</text>
          </g>

          <!-- Nodes -->
          <g v-for="node in pda.nodes" :key="node.id">
            <!-- Oblong body -->
            <rect
              :x="node.x - NW"
              :y="node.y - NH"
              :width="NW * 2"
              :height="NH * 2"
              :rx="NH"
              :ry="NH"
              :fill="nodeColor(node)"
              stroke="white"
              stroke-width="2"
            />
            <!-- Double border for accept -->
            <rect
              v-if="pda.accept.includes(node.id)"
              :x="node.x - NW + 4"
              :y="node.y - NH + 4"
              :width="(NW - 4) * 2"
              :height="(NH - 4) * 2"
              :rx="NH - 4"
              :ry="NH - 4"
              fill="none"
              stroke="white"
              stroke-width="1.5"
            />
            <!-- State label -->
            <text
              :x="node.x"
              :y="node.y + 4.5"
              font-size="13"
              fill="white"
              font-weight="bold"
              text-anchor="middle"
              font-family="Inter, Segoe UI, sans-serif"
              pointer-events="none"
            >{{ node.label }}</text>
          </g>
        </svg>
      </div>
    </div>

  </div>
</template>

<style scoped>
.pda-wrap {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    max-width: 900px;
    margin: 20px auto;
    padding: 1.2rem;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
    font-family: 'Inter', 'Segoe UI', sans-serif;
}

.pda-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 0.5rem;
}
.header-left {
    display: flex;
    align-items: center;
    gap: 0.6rem;
}
.badge {
    background: #1e1e2e;
    color: #cdd6f4;
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0.08em;
    padding: 3px 8px;
    border-radius: 5px;
}
.title {
    font-size: 18px;
    font-weight: 600;
    color: #1e1e2e;
}
.header-right {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    flex-wrap: wrap;
}
.leg {
    font-size: 12px;
    color: #6b7280;
}

.regex-wrap {
    display: flex;
    align-items: flex-start;
    gap: 0.6rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 0.6rem 0.9rem;
}
.regex-label {
    font-size: 11px;
    font-weight: 600;
    color: #94a3b8;
    letter-spacing: 0.06em;
    text-transform: uppercase;
    padding-top: 2px;
    white-space: nowrap;
}
.regex-code {
    font-family: 'Courier New', monospace;
    font-size: 13px;
    color: #334155;
    word-break: break-all;
    line-height: 1.6;
}

.notation-card {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.35rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 0.5rem 0.9rem;
    font-size: 12px;
    color: #475569;
}
.note-icon {
    font-size: 13px;
    color: #64748b;
}
.note-text {
    font-size: 12px;
    color: #475569;
}
.notation-card code {
    font-family: 'Courier New', monospace;
    font-size: 12px;
    background: #e2e8f0;
    padding: 1px 5px;
    border-radius: 3px;
}

.pda-viz-card {
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    overflow: hidden;
    background: #fff;
}
.viz-head {
    padding: 8px 16px;
    background: #f8fafc;
    border-bottom: 1px solid #e2e8f0;
    font-size: 12px;
    font-weight: 600;
    color: #64748b;
    text-transform: uppercase;
}
.image-viewport {
    padding: 20px;
    display: flex;
    justify-content: center;
    background: #fafafa;
    overflow-x: auto;
}
</style>
