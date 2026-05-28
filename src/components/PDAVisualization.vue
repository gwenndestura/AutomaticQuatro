<script setup>
import { computed, ref, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  problemId: { type: Number, required: true },
  testString: { type: String, default: '' },
  simKey:     { type: Number, default: 0 }
})

const REGEX_MAP = {
  1: '(bab+bbb)a*b*(a*+b*)(ba)*(aba)(bab+aba)*bb(a+b)*(bab+aba)(a+b)*',
  2: '(1+0)*1*0*(101+01+000)(1+0)*(101+00)*(111+00+101)(1+0)*'
}

const OW = 36, OH = 14   // oblong half-dims
const DW = 28, DH = 20   // diamond half-dims

// ── PDA graph data ──────────────────────────────────────────────────────────

const PDA_CONFIGS = {
  // ── Problem 2  (R states, binary alphabet) ──────────────────────────────
  2: {
  nodes: [
    { id: 'START',  label: 'START',  type: 'start',  x:   60, y: 360 },
    { id: 'ACCEPT', label: 'ACCEPT', type: 'accept', x: 1660, y: 360 },

    // Reject nodes
    { id: 'rjP0',  label: 'REJECT', type: 'reject', x:  220, y: 120 },
    { id: 'rjP1',  label: 'REJECT', type: 'reject', x:  380, y: 120 },
    { id: 'rjP2',  label: 'REJECT', type: 'reject', x:  380, y: 620 },
    { id: 'rjP3',  label: 'REJECT', type: 'reject', x:  540, y: 120 },
    { id: 'rjP4',  label: 'REJECT', type: 'reject', x:  540, y: 620 },
    { id: 'rjP5',  label: 'REJECT', type: 'reject', x:  700, y: 120 },
    { id: 'rjP6',  label: 'REJECT', type: 'reject', x:  860, y: 120 },
    { id: 'rjP7',  label: 'REJECT', type: 'reject', x:  860, y: 620 },
    { id: 'rjP8',  label: 'REJECT', type: 'reject', x: 1020, y: 120 },
    { id: 'rjP9',  label: 'REJECT', type: 'reject', x: 1020, y: 620 },

    // PDA states
    { id: 'p0',  label: 'READ', type: 'state', x:  220, y: 360 },

    { id: 'p1',  label: 'READ', type: 'state', x:  380, y: 240 },
    { id: 'p2',  label: 'READ', type: 'state', x:  380, y: 480 },

    { id: 'p3',  label: 'READ', type: 'state', x:  540, y: 240 },
    { id: 'p4',  label: 'READ', type: 'state', x:  540, y: 480 },

    { id: 'p5',  label: 'READ', type: 'state', x:  700, y: 360 },

    { id: 'p6',  label: 'READ', type: 'state', x:  860, y: 240 },
    { id: 'p7',  label: 'READ', type: 'state', x:  860, y: 480 },

    { id: 'p8',  label: 'READ', type: 'state', x: 1020, y: 240 },
    { id: 'p9',  label: 'READ', type: 'state', x: 1020, y: 480 },

    { id: 'p10',   label: 'READ',   type: 'state',  x: 1340, y: 360 },
  ],

  links: [
    { src: 'START', tgt: 'p0' },

    // p0
    { src: 'p0',  tgt: 'p1',   label: '0' },
    { src: 'p0',  tgt: 'p2',   label: '1' },
    { src: 'p0',  tgt: 'rjP0', label: 'Δ' },

    // p1
    { src: 'p1',  tgt: 'p4',   label: '0' },
    { src: 'p1',  tgt: 'p5',   label: '1' },
    { src: 'p1',  tgt: 'rjP1', label: 'Δ' },

    // p2
    { src: 'p2',  tgt: 'p3',   label: '0' },
    { src: 'p2',  tgt: 'p2',   label: '1' },
    { src: 'p2',  tgt: 'rjP2', label: 'Δ' },

    // p3
    { src: 'p3',  tgt: 'p1',   label: '0' },
    { src: 'p3',  tgt: 'p5',   label: '1' },
    { src: 'p3',  tgt: 'rjP3', label: 'Δ' },

    // p4
    { src: 'p4',  tgt: 'p5',   label: '0' },
    { src: 'p4',  tgt: 'p2',   label: '1' },
    { src: 'p4',  tgt: 'rjP4', label: 'Δ' },

    // p5
    { src: 'p5',  tgt: 'p6',   label: '0' },
    { src: 'p5',  tgt: 'p7',   label: '1' },
    { src: 'p5',  tgt: 'rjP5', label: 'Δ' },

    // p6
    { src: 'p6',  tgt: 'p10',  label: '0' },
    { src: 'p6',  tgt: 'p7',   label: '1' },
    { src: 'p6',  tgt: 'rjP6', label: 'Δ' },

    // p7
    { src: 'p7',  tgt: 'p8',   label: '0' },
    { src: 'p7',  tgt: 'p9',   label: '1' },
    { src: 'p7',  tgt: 'rjP7', label: 'Δ' },

    // p8
    { src: 'p8',  tgt: 'p6',   label: '0' },
    { src: 'p8',  tgt: 'p10',  label: '1' },
    { src: 'p8',  tgt: 'rjP8', label: 'Δ' },

    // p9
    { src: 'p9',  tgt: 'p6',   label: '0' },
    { src: 'p9',  tgt: 'p10',  label: '1' },
    { src: 'p9',  tgt: 'rjP9', label: 'Δ' },

    // p10
    { src: 'p10', tgt: 'p10',   label: '0,1' },
    { src: 'p10', tgt: 'ACCEPT', label: 'Δ' },
  ]
},

  // ── Problem 1  (S states, a/b alphabet) ─────────────────────────────────
  1: {
    nodes: [
      { id: 'START',  label: 'START',  type: 'start',  x:   60, y: 360 },
      { id: 'ACCEPT', label: 'ACCEPT', type: 'accept', x: 2140, y: 480 },

      // Reject nodes
      { id: 'rjQ0',  label: 'REJECT', type: 'reject', x:  200, y: 140 },
      { id: 'rjQ1',  label: 'REJECT', type: 'reject', x:  360, y: 140 },
      { id: 'rjQ2',  label: 'REJECT', type: 'reject', x:  520, y:  80 },
      { id: 'rjQ3',  label: 'REJECT', type: 'reject', x:  520, y: 640 },
      { id: 'rjQ4',  label: 'REJECT', type: 'reject', x:  680, y: 140 },
      { id: 'rjQ5',  label: 'REJECT', type: 'reject', x:  840, y: 100 },
      { id: 'rjQ6',  label: 'REJECT', type: 'reject', x:  840, y: 620 },
      { id: 'rjQ7',  label: 'REJECT', type: 'reject', x: 1000, y: 140 },
      { id: 'rjQ8',  label: 'REJECT', type: 'reject', x: 1160, y: 140 },
      { id: 'rjQ9',  label: 'REJECT', type: 'reject', x: 1320, y: 100 },
      { id: 'rjQ10', label: 'REJECT', type: 'reject', x: 1480, y: 100 },
      { id: 'rjQ11', label: 'REJECT', type: 'reject', x: 1320, y: 620 },
      { id: 'rjQ12', label: 'REJECT', type: 'reject', x: 1480, y: 620 },
      { id: 'rjQ13', label: 'REJECT', type: 'reject', x: 1640, y: 100 },
      { id: 'rjQ14', label: 'REJECT', type: 'reject', x: 1640, y: 640 },
      { id: 'rjQ15', label: 'REJECT', type: 'reject', x: 1800, y: 140 },
      { id: 'rjQ16', label: 'REJECT', type: 'reject', x: 1960, y: 100 },

      // State nodes (diamonds)
      { id: 'q0',  label: 'READ',  type: 'state', x:  200, y: 360 },
      { id: 'q1',  label: 'READ',  type: 'state', x:  360, y: 360 },
      { id: 'q2',  label: 'READ',  type: 'state', x:  520, y: 200 },
      { id: 'q3',  label: 'READ',  type: 'state', x:  520, y: 520 },
      { id: 'q4',  label: 'READ',  type: 'state', x:  680, y: 360 },
      { id: 'q5',  label: 'READ',  type: 'state', x:  840, y: 240 },
      { id: 'q6',  label: 'READ',  type: 'state', x:  840, y: 480 },
      { id: 'q7',  label: 'READ',  type: 'state', x: 1000, y: 360 },
      { id: 'q8',  label: 'READ',  type: 'state', x: 1160, y: 360 },
      { id: 'q9',  label: 'READ',  type: 'state', x: 1320, y: 240 },
      { id: 'q10', label: 'READ', type: 'state', x: 1480, y: 240 },
      { id: 'q11', label: 'READ', type: 'state', x: 1320, y: 480 },
      { id: 'q12', label: 'READ', type: 'state', x: 1480, y: 480 },
      { id: 'q13', label: 'READ', type: 'state', x: 1640, y: 240 },
      { id: 'q14', label: 'READ', type: 'state', x: 1640, y: 480 },
      { id: 'q15', label: 'READ', type: 'state', x: 1800, y: 360 },
      { id: 'q16', label: 'READ', type: 'state', x: 1960, y: 240 },
      { id: 'q17', label: 'READ', type: 'state', x: 1960, y: 480 },
    ],
    links: [
      { src: 'START', tgt: 'q0' },

      { src: 'q0',  tgt: 'q1',    label: 'b' },
      { src: 'q0',  tgt: 'rjQ0',  label: 'a,Δ' },

      { src: 'q1',  tgt: 'q2',    label: 'a' },
      { src: 'q1',  tgt: 'q3',    label: 'b' },
      { src: 'q1',  tgt: 'rjQ1',  label: 'Δ' },

      { src: 'q2',  tgt: 'q4',    label: 'b' },
      { src: 'q2',  tgt: 'rjQ2',  label: 'a,Δ' },

      { src: 'q3',  tgt: 'q4',    label: 'b' },
      { src: 'q3',  tgt: 'rjQ3',  label: 'a,Δ' },

      { src: 'q4',  tgt: 'q5',    label: 'a' },
      { src: 'q4',  tgt: 'q6',    label: 'b' },
      { src: 'q4',  tgt: 'rjQ4',  label: 'Δ' },

      { src: 'q5',  tgt: 'q5',    label: 'a' },
      { src: 'q5',  tgt: 'q7',    label: 'b' },
      { src: 'q5',  tgt: 'rjQ5',  label: 'Δ' },

      { src: 'q6',  tgt: 'q5',    label: 'a' },
      { src: 'q6',  tgt: 'q6',    label: 'b' },
      { src: 'q6',  tgt: 'rjQ6',  label: 'Δ' },

      { src: 'q7',  tgt: 'q8',    label: 'a' },
      { src: 'q7',  tgt: 'q4',    label: 'b' },
      { src: 'q7',  tgt: 'rjQ7',  label: 'Δ' },

      { src: 'q8',  tgt: 'q9',    label: 'a' },
      { src: 'q8',  tgt: 'q11',   label: 'b' },
      { src: 'q8',  tgt: 'rjQ8',  label: 'Δ' },

      { src: 'q9',  tgt: 'q10',   label: 'b' },
      { src: 'q9',  tgt: 'rjQ9',  label: 'a,Δ' },

      { src: 'q10', tgt: 'q8',    label: 'a' },
      { src: 'q10', tgt: 'rjQ10', label: 'b,Δ' },

      { src: 'q11', tgt: 'q8',    label: 'a' },
      { src: 'q11', tgt: 'q12',   label: 'b' },
      { src: 'q11', tgt: 'rjQ11', label: 'Δ' },

      { src: 'q12', tgt: 'q13',   label: 'a' },
      { src: 'q12', tgt: 'q14',   label: 'b' },
      { src: 'q12', tgt: 'rjQ12', label: 'Δ' },

      { src: 'q13', tgt: 'q13',   label: 'a' },
      { src: 'q13', tgt: 'q15',   label: 'b' },
      { src: 'q13', tgt: 'rjQ13', label: 'Δ' },

      { src: 'q14', tgt: 'q16',   label: 'a' },
      { src: 'q14', tgt: 'q14',   label: 'b' },
      { src: 'q14', tgt: 'rjQ14', label: 'Δ' },

      { src: 'q15', tgt: 'q17',   label: 'a' },
      { src: 'q15', tgt: 'q14',   label: 'b' },
      { src: 'q15', tgt: 'rjQ15', label: 'Δ' },

      { src: 'q16', tgt: 'q13',   label: 'a' },
      { src: 'q16', tgt: 'q17',   label: 'b' },
      { src: 'q16', tgt: 'rjQ16', label: 'Δ' },

      { src: 'q17', tgt: 'q17',   label: 'a,b' },
      { src: 'q17', tgt: 'ACCEPT', label: 'Δ' },
    ]
  }
}

const pda = computed(() => PDA_CONFIGS[props.problemId])

const nodeMap = computed(() => {
  const m = {}
  pda.value.nodes.forEach(n => { m[n.id] = n })
  return m
})

const getOblongEdge = (cx, cy, tx, ty) => {
  const dx = tx - cx, dy = ty - cy
  const dist = Math.sqrt(dx*dx + dy*dy)
  if (dist === 0) return { x: cx, y: cy }
  const nx = dx/dist, ny = dy/dist
  const t = 1 / Math.sqrt((nx/OW)**2 + (ny/OH)**2)
  return { x: cx + t*nx, y: cy + t*ny }
}

const getDiamondEdge = (cx, cy, tx, ty) => {
  const dx = tx - cx, dy = ty - cy
  const dist = Math.sqrt(dx*dx + dy*dy)
  if (dist === 0) return { x: cx, y: cy }
  const cosA = Math.abs(dx/dist), sinA = Math.abs(dy/dist)
  const r = (DW * DH) / (DH * cosA + DW * sinA)
  return { x: cx + r*(dx/dist), y: cy + r*(dy/dist) }
}

const getEdge = (node, tx, ty) =>
  node.type === 'state'
    ? getDiamondEdge(node.x, node.y, tx, ty)
    : getOblongEdge(node.x, node.y, tx, ty)

const linkData = computed(() => {
  return pda.value.links.map(lnk => {
    const sn = nodeMap.value[lnk.src]
    const tn = nodeMap.value[lnk.tgt]
    if (!sn || !tn) return { ...lnk, d: '', lx: 0, ly: 0, isSelf: false }

    if (lnk.src === lnk.tgt) {
      const cx = sn.x, cy = sn.y
      const lr = 18, ex = 10, ey = (sn.type === 'state' ? DH : OH)
      const d = `M${cx-ex},${cy-ey} A ${lr} ${lr} 0 1 1 ${cx+ex},${cy-ey}`
      return { ...lnk, d, lx: cx, ly: cy - ey - 22, isSelf: true }
    }

    const se = getEdge(sn, tn.x, tn.y)
    const te = getEdge(tn, sn.x, sn.y)
    const dx = te.x - se.x, dy = te.y - se.y
    const dist = Math.sqrt(dx*dx + dy*dy)

    if (lnk.curve && lnk.curve > 1000) {
      const d = `M${se.x},${se.y} L${te.x},${te.y}`
      return { ...lnk, d, lx: (se.x+te.x)/2, ly: (se.y+te.y)/2 - 8, isSelf: false }
    }

    const snNum = parseInt(lnk.src.replace(/\D/g,'')) || 0
    const tnNum = parseInt(lnk.tgt.replace(/\D/g,'')) || 0
    const sweep = lnk.sweep !== undefined ? lnk.sweep : (snNum < tnNum ? 1 : 0)
    const dr    = dist * (lnk.curve || 1.3)
    const d     = `M${se.x},${se.y} A ${dr} ${dr} 0 0 ${sweep} ${te.x},${te.y}`

    const mx = (se.x+te.x)/2, my = (se.y+te.y)/2
    const perpX = -dy/dist, perpY = dx/dist
    const sag   = dist * 0.09
    const sign  = sweep === 1 ? -1 : 1
    return { ...lnk, d, lx: mx+sign*perpX*sag, ly: my+sign*perpY*sag, isSelf: false }
  })
})

const nodeColor = (n) => {
  if (n.type === 'accept') return '#22c55e'
  if (n.type === 'start')  return '#3b82f6'
  if (n.type === 'reject') return '#ef4444'
  return '#eab308'
}

const viewBox = computed(() => {
  const ns = pda.value.nodes
  const xs = ns.map(n => n.x), ys = ns.map(n => n.y)
  const pad = 70
  return `${Math.min(...xs)-OW-pad} ${Math.min(...ys)-DH-pad} ${Math.max(...xs)-Math.min(...xs)+OW*2+pad*2} ${Math.max(...ys)-Math.min(...ys)+OH*2+pad*2}`
})

const startX = computed(() => {
  const sn = pda.value.nodes.find(n => n.type === 'start')
  return sn ? sn.x - OW : 0
})
const startY = computed(() => pda.value.nodes.find(n => n.type === 'start')?.y ?? 0)

// ── Animation ────────────────────────────────────────────────────────────────

const animPath  = ref([])
const animIndex = ref(-1)
let   animTimer = null

const activeNodeId  = computed(() => animPath.value[animIndex.value]?.nodeId ?? null)
const activeLinkIdx = computed(() => animPath.value[animIndex.value]?.linkIdx ?? -1)
const animDone      = computed(() => animPath.value.length > 0 && animIndex.value >= animPath.value.length - 1)
const animAccepted  = computed(() => animDone.value && animPath.value.at(-1)?.nodeId === 'ACCEPT')
const animCharIdx   = computed(() => animPath.value[animIndex.value]?.charIdx ?? -1)
const animCurrentChar = computed(() => {
  const c = animPath.value[animIndex.value]?.char
  return c && c !== 'Δ' ? c : null
})

const tape = computed(() => {
  if (!props.testString) return []
  return props.testString.split('').map((ch, i) => {
    if (animIndex.value <= 0) return { ch, status: 'pending' }
    const idx = animCharIdx.value
    if (idx >= props.testString.length) return { ch, status: 'done' }
    if (i < idx)  return { ch, status: 'done' }
    if (i === idx) return { ch, status: 'active' }
    return { ch, status: 'pending' }
  })
})

const buildPath = (input) => {
  const transMap = {}
  pda.value.links.forEach((lnk, idx) => {
    if (!transMap[lnk.src]) transMap[lnk.src] = []
    transMap[lnk.src].push({ ...lnk, idx })
  })

  const isRej = (id) => id.startsWith('rj') || nodeMap.value[id]?.type === 'reject'

  // ── NFA DFS: find an accepting path ─────────────────────────────────────
  const memo  = new Map()
  const stack = new Set()

  const findAccept = (node, pos) => {
    if (node === 'ACCEPT') return pos === input.length ? [] : null
    if (isRej(node)) return null

    const key = `${node}:${pos}`
    if (memo.has(key))  return memo.get(key)
    if (stack.has(key)) return null
    stack.add(key)

    const char  = pos < input.length ? input[pos] : null
    const avail = (transMap[node] || []).filter(t => !isRej(t.tgt))
    let result  = null

    for (const t of avail) {
      if (result) break
      const parts   = (t.label ?? '').split(',').map(s => s.trim())
      const charHit = char !== null && parts.includes(char)
      const epsHit  = parts.includes('Δ') || parts.includes('')

      if (charHit) {
        const sub = findAccept(t.tgt, pos + 1)
        if (sub !== null) result = [{ tIdx: t.idx, consume: true }, ...sub]
      }
      if (!result && epsHit) {
        const sub = findAccept(t.tgt, pos)
        if (sub !== null) result = [{ tIdx: t.idx, consume: false }, ...sub]
      }
    }

    stack.delete(key)
    memo.set(key, result)
    return result
  }

  const decisions = findAccept('START', 0)

  if (decisions !== null) {
    const path = []
    let node = 'START', pos = 0
    path.push({ nodeId: node, linkIdx: -1, charIdx: pos, char: input[pos] ?? 'Δ' })
    for (const dec of decisions) {
      path[path.length - 1] = { ...path[path.length - 1], linkIdx: dec.tIdx }
      node = pda.value.links[dec.tIdx].tgt
      if (dec.consume) pos++
      path.push({ nodeId: node, linkIdx: -1, charIdx: pos, char: pos < input.length ? input[pos] : 'Δ' })
    }
    return path
  }

  // ── Fallback: deterministic rejection path for animation ─────────────────
  const path = []
  let node = 'START', pos = 0
  const max = (input.length + 1) * 10 + 40

  for (let step = 0; step < max; step++) {
    const char = pos < input.length ? input[pos] : null
    path.push({ nodeId: node, linkIdx: -1, charIdx: pos, char: char ?? 'Δ' })
    if (node === 'ACCEPT' || isRej(node)) break

    const avail = transMap[node] || []
    let matched = null

    if (char !== null) {
      for (const t of avail) {
        if ((t.label ?? '').split(',').map(s => s.trim()).includes(char)) { matched = t; break }
      }
    }
    if (!matched) {
      for (const t of avail) {
        const parts = (t.label ?? '').split(',').map(s => s.trim())
        if (parts.includes('Δ') || parts.includes('')) {
          if (t.tgt === 'ACCEPT' && char !== null) continue
          matched = t; break
        }
      }
    }

    if (!matched) break
    path[path.length - 1] = { ...path[path.length - 1], linkIdx: matched.idx }
    if (char !== null && (matched.label ?? '').split(',').map(s => s.trim()).includes(char)) pos++
    node = matched.tgt
  }
  return path
}

const doReset = () => {
  clearInterval(animTimer); animTimer = null
  animIndex.value = -1
  animPath.value  = []
}

const runAuto = () => {
  doReset()
  if (props.testString === null || props.testString === undefined) return

  const path = buildPath(props.testString)
  if (!path.length) return

  animPath.value  = path
  animIndex.value = 0
  let idx = 0

  animTimer = setInterval(() => {
    if (idx < path.length - 1) { idx++; animIndex.value = idx }
    else { clearInterval(animTimer); animTimer = null }
  }, 800)
}

watch(() => props.simKey,     () => { if (props.testString != null) runAuto() })
watch(() => props.testString, (v)  => { v != null ? runAuto() : doReset() })
watch(() => props.problemId,  ()   => doReset())
onMounted(() => { if (props.testString) runAuto() })
onUnmounted(() => clearInterval(animTimer))

// ── Node / link visual helpers ───────────────────────────────────────────────

const glowColor = computed(() => {
  if (!animDone.value) return '#f59e0b'
  return animAccepted.value ? '#22c55e' : '#ef4444'
})

const nodeStroke      = (n) => activeNodeId.value === n.id ? glowColor.value : 'white'
const nodeStrokeWidth = (n) => activeNodeId.value === n.id ? '4' : '2'
const nodeFilter      = (n) =>
  activeNodeId.value === n.id ? `drop-shadow(0 0 9px ${glowColor.value})` : 'none'

const linkStroke      = (i) => activeLinkIdx.value === i ? '#f59e0b' : '#475569'
const linkStrokeWidth = (i) => activeLinkIdx.value === i ? '3'       : '1.7'
const linkMarker      = (lnk, i) => {
  const active = activeLinkIdx.value === i
  return lnk.isSelf ? (active ? 'url(#pda-arr-active-s)' : 'url(#pda-arr-s)')
                    : (active ? 'url(#pda-arr-active)'   : 'url(#pda-arr)')
}
const labelFill = (i) => activeLinkIdx.value === i ? '#1d4ed8' : '#1d4ed8'

// ── Zoom / pan ────────────────────────────────────────────────
const pdaZoom    = ref(1)
const pdaPanX    = ref(0)
const pdaPanY    = ref(0)
const pdaDrag    = ref(false)
const pdaDragSt  = ref({ x: 0, y: 0, px: 0, py: 0 })
const PDA_ZMIN   = 0.1
const PDA_ZMAX   = 5

const pdaZoomIn    = () => { pdaZoom.value = Math.min(PDA_ZMAX, +(pdaZoom.value * 1.2).toFixed(3)) }
const pdaZoomOut   = () => { pdaZoom.value = Math.max(PDA_ZMIN, +(pdaZoom.value / 1.2).toFixed(3)) }
const pdaResetZoom = () => { pdaZoom.value = 1; pdaPanX.value = 0; pdaPanY.value = 0 }

const pdaOnWheel = (e) => {
  const f = e.deltaY < 0 ? 1.1 : 1 / 1.1
  pdaZoom.value = Math.min(PDA_ZMAX, Math.max(PDA_ZMIN, +(pdaZoom.value * f).toFixed(3)))
}
const pdaOnDragStart = (e) => {
  pdaDrag.value   = true
  pdaDragSt.value = { x: e.clientX, y: e.clientY, px: pdaPanX.value, py: pdaPanY.value }
}
const pdaOnDrag = (e) => {
  if (!pdaDrag.value) return
  pdaPanX.value = pdaDragSt.value.px + (e.clientX - pdaDragSt.value.x)
  pdaPanY.value = pdaDragSt.value.py + (e.clientY - pdaDragSt.value.y)
}
const pdaOnDragEnd = () => { pdaDrag.value = false }
</script>

<template>
  <div class="pda-wrap">

    <!-- Header -->
    <div class="pda-header">
      <div class="header-left">
        <div class="header-tag">
          <span class="tag-type">PDA</span>
          <span class="tag-prob">Problem {{ problemId }}</span>
        </div>
      </div>
      <div class="header-right">
        <svg width="28" height="20" style="flex-shrink:0;vertical-align:middle">
          <polygon points="14,0 28,10 14,20 0,10" fill="#eab308"/>
        </svg>
        <span class="leg">READ</span>
        <svg width="36" height="18" style="flex-shrink:0;vertical-align:middle;margin-left:10px">
          <rect x="1" y="1" width="34" height="16" rx="8" fill="#3b82f6"/>
        </svg>
        <span class="leg">START</span>
        <svg width="36" height="18" style="flex-shrink:0;vertical-align:middle;margin-left:10px">
          <rect x="1" y="1" width="34" height="16" rx="8" fill="#ef4444"/>
        </svg>
        <span class="leg">REJECT</span>
        <svg width="36" height="18" style="flex-shrink:0;vertical-align:middle;margin-left:10px">
          <rect x="1" y="1" width="34" height="16" rx="8" fill="#22c55e"/>
        </svg>
        <span class="leg">ACCEPT</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ REGEX_MAP[problemId] }}</code>
    </div>


    <!-- Simulation status (visible when a string is being traced) -->
    <div v-if="animPath.length > 0" class="sim-panel" :class="animDone ? (animAccepted ? 'sim-ok' : 'sim-fail') : 'sim-running'">

      <!-- Left: state + reading -->
      <div class="sim-left">
        <div class="sim-state-wrap">
          <span class="sim-micro-label">State</span>
          <span :class="['sim-state', animDone ? (animAccepted ? 'ok' : 'fail') : 'running']">
            {{ activeNodeId }}
          </span>
        </div>
        <div class="sim-divider-v"></div>
        <div class="sim-reading-wrap">
          <span class="sim-micro-label">Reading</span>
          <span v-if="animCurrentChar != null" class="sim-char">{{ animCurrentChar }}</span>
          <span v-else-if="animIndex > 0" class="sim-char delta">Δ</span>
          <span v-else class="sim-char muted">—</span>
        </div>
      </div>

      <!-- Center: tape -->
      <div class="sim-tape-wrap" v-if="tape.length > 0">
        <div class="sim-tape no-scrollbar-x">
          <div v-for="(cell, i) in tape" :key="i" :class="['sim-cell', cell.status]">
            <span class="sim-cell-ch">{{ cell.ch }}</span>
            <span v-if="cell.status === 'active'" class="sim-cell-head">▲</span>
          </div>
        </div>
      </div>

      <!-- Right: result -->
      <div class="sim-right">
        <transition name="pop">
          <div v-if="animDone" :class="['sim-result', animAccepted ? 'result-ok' : 'result-fail']">
            <span class="result-icon">{{ animAccepted ? '✓' : '✕' }}</span>
            <span class="result-text">{{ animAccepted ? 'Accepted' : 'Rejected' }}</span>
          </div>
          <div v-else class="sim-result result-running">
            <span class="result-icon spin">◌</span>
            <span class="result-text">Tracing…</span>
          </div>
        </transition>
      </div>

    </div>

    <!-- Diagram -->
    <div class="pda-viz-card">
      <div class="viz-head-row">
        <div class="viz-head">Pushdown Automata Diagram</div>
        <div class="zoom-controls">
          <button class="zoom-btn" title="Zoom in"   @click="pdaZoomIn">+</button>
          <button class="zoom-btn" title="Zoom out"  @click="pdaZoomOut">−</button>
          <button class="zoom-btn" title="Reset view" @click="pdaResetZoom">⊙</button>
        </div>
      </div>
      <div
        class="image-viewport"
        @wheel.prevent="pdaOnWheel"
        @mousedown.prevent="pdaOnDragStart"
        @mousemove="pdaOnDrag"
        @mouseup="pdaOnDragEnd"
        @mouseleave="pdaOnDragEnd"
      >
        <div :style="{ transform: `translate(${pdaPanX}px,${pdaPanY}px) scale(${pdaZoom})`, transformOrigin: '0 0', display: 'inline-block', width: '100%' }">
        <svg :viewBox="viewBox" style="width:100%;overflow:visible;min-height:300px" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <marker id="pda-arr" viewBox="0 -5 10 10" refX="10" refY="0" markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#475569"/>
            </marker>
            <marker id="pda-arr-s" viewBox="0 -5 10 10" refX="9" refY="0" markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#475569"/>
            </marker>
            <marker id="pda-arr-active" viewBox="0 -5 10 10" refX="10" refY="0" markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#f59e0b"/>
            </marker>
            <marker id="pda-arr-active-s" viewBox="0 -5 10 10" refX="9" refY="0" markerWidth="6" markerHeight="6" orient="auto">
              <path d="M0,-5L10,0L0,5" fill="#f59e0b"/>
            </marker>
          </defs>

          <!-- Entry arrow -->
          <line
            :x1="startX - 36" :y1="startY"
            :x2="startX"       :y2="startY"
            stroke="#475569" stroke-width="2" marker-end="url(#pda-arr)"
          />

          <!-- Edges -->
          <g v-for="(lnk, i) in linkData" :key="'e'+i">
            <path :d="lnk.d" fill="none"
              :stroke="linkStroke(i)"
              :stroke-width="linkStrokeWidth(i)"
              :marker-end="linkMarker(lnk, i)" />
            <text v-if="lnk.label"
              :x="lnk.lx" :y="lnk.ly"
              font-size="11" :fill="labelFill(i)" font-weight="700"
              text-anchor="middle" font-family="Courier New, monospace"
              style="paint-order:stroke;stroke:#fff;stroke-width:4px;stroke-linejoin:round"
            >{{ lnk.label }}</text>
          </g>

          <!-- Nodes -->
          <g v-for="node in pda.nodes" :key="node.id">
            <polygon
              v-if="node.type === 'state'"
              :points="`${node.x},${node.y-DH} ${node.x+DW},${node.y} ${node.x},${node.y+DH} ${node.x-DW},${node.y}`"
              :fill="nodeColor(node)"
              :stroke="nodeStroke(node)"
              :stroke-width="nodeStrokeWidth(node)"
              :style="{ filter: nodeFilter(node) }"
            />
            <rect
              v-else
              :x="node.x-OW" :y="node.y-OH"
              :width="OW*2" :height="OH*2"
              :rx="OH" :ry="OH"
              :fill="nodeColor(node)"
              :stroke="nodeStroke(node)"
              :stroke-width="nodeStrokeWidth(node)"
              :style="{ filter: nodeFilter(node) }"
            />
            <rect
              v-if="node.type === 'accept'"
              :x="node.x-OW+4" :y="node.y-OH+4"
              :width="(OW-4)*2" :height="(OH-4)*2"
              :rx="OH-4" :ry="OH-4"
              fill="none" stroke="white" stroke-width="1.5"
            />
            <text
              :x="node.x" :y="node.y+4.5"
              font-size="11" fill="white" font-weight="700"
              text-anchor="middle" font-family="Inter, Segoe UI, sans-serif"
              pointer-events="none"
            >{{ node.label }}</text>
          </g>
        </svg>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.pda-wrap {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-width: 1140px;
  margin: 16px auto;
  padding: 1.2rem;
  background: #ffffff;
  border-radius: 14px;
  box-shadow: 0 4px 24px rgba(0,0,0,0.07), 0 1px 4px rgba(0,0,0,0.04);
  font-family: 'Inter','Segoe UI',sans-serif;
  border: 1px solid #e8ecf0;
}

.pda-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 0.5rem;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f4f8;
}
.header-left { display:flex; align-items:center; }
.header-tag {
  display: inline-flex;
  align-items: stretch;
  border-radius: 9px;
  overflow: hidden;
  border: 1.5px solid #e2e8f0;
  box-shadow: 0 1px 4px rgba(0,0,0,0.06);
}
.tag-type {
  background: #16a34a;
  color: #fff;
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  padding: 7px 13px;
  display: flex;
  align-items: center;
}
.tag-prob {
  background: #f8fafc;
  color: #0f172a;
  font-size: 13.5px;
  font-weight: 700;
  padding: 7px 15px;
  display: flex;
  align-items: center;
  border-left: 1.5px solid #e2e8f0;
  letter-spacing: -0.01em;
}
.header-right { display:flex; align-items:center; gap:0.35rem; flex-wrap:wrap; }
.leg { font-size:11.5px; color:#6b7280; font-weight:500; }

.regex-wrap {
  display: flex;
  align-items: flex-start;
  gap: 0.7rem;
  background: #f8fafc;
  border: 1.5px solid #e2e8f0;
  border-left: 3px solid #22c55e;
  border-radius: 8px;
  padding: 0.65rem 0.9rem;
}
.regex-label { font-size:10px; font-weight:700; color:#94a3b8; letter-spacing:0.07em; text-transform:uppercase; padding-top:2px; white-space:nowrap; }
.regex-code { font-family:'Courier New',monospace; font-size:12.5px; color:#1e40af; word-break:break-all; line-height:1.65; }

/* ── Simulation panel ───────────────────────────────────────── */
.sim-panel {
  display: flex;
  align-items: center;
  gap: 0;
  border-radius: 12px;
  border: 1.5px solid #e8ecf0;
  background: #ffffff;
  overflow: hidden;
  min-height: 68px;
  transition: border-color 0.3s;
}
.sim-panel.sim-ok   { border-color: #86efac; }
.sim-panel.sim-fail { border-color: #fca5a5; }

/* Left: state + reading */
.sim-left {
  display: flex;
  align-items: center;
  gap: 0;
  flex-shrink: 0;
  background: #f8fafc;
  border-right: 1.5px solid #e8ecf0;
  height: 100%;
  min-height: 68px;
}
.sim-state-wrap, .sim-reading-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 10px 16px;
  gap: 4px;
}
.sim-micro-label {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #94a3b8;
}
.sim-state {
  font-size: 13px;
  font-weight: 800;
  padding: 3px 10px;
  border-radius: 6px;
  border: 1.5px solid transparent;
  letter-spacing: 0.01em;
  white-space: nowrap;
}
.sim-state.running { background: #eff6ff; color: #1d4ed8; border-color: #93c5fd; }
.sim-state.ok      { background: #f0fdf4; color: #15803d; border-color: #86efac; }
.sim-state.fail    { background: #fef2f2; color: #b91c1c; border-color: #fca5a5; }

.sim-char {
  font-family: 'Courier New', monospace;
  font-size: 15px;
  font-weight: 800;
  color: #d97706;
  background: #fffbeb;
  border: 1.5px solid #fcd34d;
  border-radius: 6px;
  padding: 2px 10px;
  min-width: 32px;
  text-align: center;
}
.sim-char.delta { color: #7c3aed; background: #f5f3ff; border-color: #c4b5fd; }
.sim-char.muted { color: #cbd5e1; background: #f8fafc; border-color: #e2e8f0; }

.sim-divider-v {
  width: 1px;
  align-self: stretch;
  background: #e8ecf0;
  margin: 10px 0;
}

/* Center: tape */
.sim-tape-wrap {
  flex: 1;
  min-width: 0;
  padding: 0 14px;
  overflow: hidden;
}
.sim-tape {
  display: flex;
  gap: 5px;
  overflow-x: auto;
  padding: 10px 0 12px;
  align-items: flex-end;
}
.sim-cell {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  flex-shrink: 0;
}
.sim-cell-ch {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: 8px;
  font-family: monospace;
  font-weight: 700;
  font-size: 15px;
  border: 1.5px solid #e2e8f0;
  background: #f8fafc;
  color: #94a3b8;
  transition: all 0.2s;
}
.sim-cell.done .sim-cell-ch   { background: #f0fdf4; color: #16a34a; border-color: #86efac; }
.sim-cell.active .sim-cell-ch { background: #fffbeb; color: #d97706; border-color: #fcd34d; box-shadow: 0 4px 12px rgba(217,119,6,0.22); transform: translateY(-3px); }
.sim-cell-head {
  font-size: 9px;
  color: #f59e0b;
  line-height: 1;
  margin-top: 1px;
}

/* Right: result */
.sim-right {
  flex-shrink: 0;
  padding: 0 16px;
  border-left: 1.5px solid #e8ecf0;
  height: 100%;
  min-height: 68px;
  display: flex;
  align-items: center;
  background: #f8fafc;
}
.sim-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  min-width: 72px;
}
.result-icon {
  font-size: 20px;
  font-weight: 900;
  line-height: 1;
}
.result-text {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.07em;
  text-transform: uppercase;
}
.result-ok   .result-icon { color: #16a34a; }
.result-ok   .result-text { color: #16a34a; }
.result-fail .result-icon { color: #dc2626; }
.result-fail .result-text { color: #dc2626; }
.result-running .result-icon { color: #94a3b8; }
.result-running .result-text { color: #94a3b8; }

@keyframes spin { to { transform: rotate(360deg); } }
.spin { display: inline-block; animation: spin 1.2s linear infinite; }

.pda-viz-card { border:1.5px solid #e8ecf0; border-radius:10px; overflow:hidden; background:#fff; }
.viz-head-row { display:flex; align-items:center; justify-content:space-between; padding:9px 12px 9px 16px; background:#f8fafc; border-bottom:1.5px solid #e8ecf0; }
.viz-head { font-size:10.5px; font-weight:700; color:#64748b; text-transform:uppercase; letter-spacing:0.08em; }
.image-viewport { padding:20px; background:#fafbfc; overflow:hidden; cursor:grab; user-select:none; }
.image-viewport:active { cursor:grabbing; }
.zoom-controls { display:flex; gap:4px; }
.zoom-btn {
  width:28px; height:28px;
  border:1.5px solid #e1e4e8; border-radius:6px;
  background:#fff; color:#57606a;
  font-size:14px; font-weight:700;
  cursor:pointer;
  display:inline-flex; align-items:center; justify-content:center;
  transition:all 0.13s;
  box-shadow:0 1px 4px rgba(0,0,0,0.08);
  line-height:1;
}
.zoom-btn:hover { background:#f0fdf4; border-color:#22c55e; color:#16a34a; }

.no-scrollbar-x { scrollbar-width:none; }
.no-scrollbar-x::-webkit-scrollbar { display:none; }

.pop-enter-active { animation:popIn 0.3s cubic-bezier(0.34,1.56,0.64,1); }
@keyframes popIn { from { transform:scale(0.9); opacity:0; } to { transform:scale(1); opacity:1; } }
</style>
