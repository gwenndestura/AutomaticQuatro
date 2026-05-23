<script setup>
import { computed, ref, watch, onUnmounted } from 'vue'

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
      { id: 'START', label: 'START',  type: 'start',  x: 200,  y:  60 },
      { id: 'ACCEPT',label: 'ACCEPT', type: 'accept', x:1100,  y: 860 },
      { id: 'rj1',  label: 'REJECT', type: 'reject', x:  50,  y: 160 },
      { id: 'rj2',  label: 'REJECT', type: 'reject', x: 380,  y:  60 },
      { id: 'rj3',  label: 'REJECT', type: 'reject', x:  50,  y: 300 },
      { id: 'rj4',  label: 'REJECT', type: 'reject', x: 560,  y: 300 },
      { id: 'rj5',  label: 'REJECT', type: 'reject', x:  50,  y: 440 },
      { id: 'rj6',  label: 'REJECT', type: 'reject', x: 360,  y: 530 },
      { id: 'rj7',  label: 'REJECT', type: 'reject', x: 620,  y: 360 },
      { id: 'rj8',  label: 'REJECT', type: 'reject', x: 280,  y: 580 },
      { id: 'rj9',  label: 'REJECT', type: 'reject', x: 780,  y: 360 },
      { id: 'rj10', label: 'REJECT', type: 'reject', x: 880,  y: 500 },
      { id: 'R1',  label: 'R1',  type: 'state', x: 200,  y: 160 },
      { id: 'R2',  label: 'R2',  type: 'state', x: 380,  y: 160 },
      { id: 'R3',  label: 'R3',  type: 'state', x: 200,  y: 300 },
      { id: 'R4',  label: 'R4',  type: 'state', x: 380,  y: 300 },
      { id: 'R5',  label: 'R5',  type: 'state', x: 200,  y: 440 },
      { id: 'R6',  label: 'R6',  type: 'state', x: 460,  y: 440 },
      { id: 'R7',  label: 'R7',  type: 'state', x: 620,  y: 440 },
      { id: 'R8',  label: 'R8',  type: 'state', x: 460,  y: 580 },
      { id: 'R9',  label: 'R9',  type: 'state', x: 780,  y: 440 },
      { id: 'R10', label: 'R10', type: 'state', x: 780,  y: 580 },
      { id: 'R11', label: 'R11', type: 'state', x: 780,  y: 720 },
      { id: 'R12', label: 'R12', type: 'state', x: 940,  y: 440 },
      { id: 'R13', label: 'R13', type: 'state', x: 940,  y: 720 },
    ],
    links: [
      { src: 'START', tgt: 'R1',  label: '' },
      { src: 'R1',  tgt: 'rj1',  label: 'Δ' },
      { src: 'R1',  tgt: 'R2',   label: '1' },
      { src: 'R1',  tgt: 'R3',   label: '0' },
      { src: 'R2',  tgt: 'rj2',  label: 'Δ' },
      { src: 'R2',  tgt: 'R2',   label: '1' },
      { src: 'R2',  tgt: 'R4',   label: '0' },
      { src: 'R3',  tgt: 'rj3',  label: 'Δ' },
      { src: 'R3',  tgt: 'R6',   label: '1' },
      { src: 'R3',  tgt: 'R5',   label: '0' },
      { src: 'R4',  tgt: 'R5',   label: '0' },
      { src: 'R4',  tgt: 'rj4',  label: 'Δ' },
      { src: 'R4',  tgt: 'R6',   label: '1' },
      { src: 'R5',  tgt: 'rj5',  label: 'Δ' },
      { src: 'R5',  tgt: 'R6',   label: '0,1' },
      { src: 'R6',  tgt: 'rj6',  label: 'Δ' },
      { src: 'R6',  tgt: 'R7',   label: '1' },
      { src: 'R6',  tgt: 'R8',   label: '0' },
      { src: 'R7',  tgt: 'rj7',  label: 'Δ' },
      { src: 'R7',  tgt: 'R9',   label: '1' },
      { src: 'R7',  tgt: 'R10',  label: '0' },
      { src: 'R8',  tgt: 'rj8',  label: 'Δ' },
      { src: 'R8',  tgt: 'R11',  label: '0' },
      { src: 'R8',  tgt: 'R7',   label: '1' },
      { src: 'R9',  tgt: 'rj9',  label: 'Δ' },
      { src: 'R9',  tgt: 'R12',  label: '1' },
      { src: 'R9',  tgt: 'R7',   label: '0', sweep: 1 },
      { src: 'R10', tgt: 'rj10', label: 'Δ' },
      { src: 'R10', tgt: 'R12',  label: '1' },
      { src: 'R11', tgt: 'R13',  label: '0', curve: 1.4, sweep: 1 },
      { src: 'R11', tgt: 'R12',  label: '1' },
      { src: 'R11', tgt: 'ACCEPT',label: 'Δ',  curve: 10001 },
      { src: 'R12', tgt: 'R13',  label: '0,1', curve: 10001 },
      { src: 'R12', tgt: 'ACCEPT',label: 'Δ',  curve: 10001 },
      { src: 'R13', tgt: 'R13',  label: '0,1' },
      { src: 'R13', tgt: 'ACCEPT',label: 'Δ',  curve: 10001 },
    ]
  },

  // ── Problem 1  (S states, a/b alphabet) ─────────────────────────────────
  1: {
    nodes: [
      { id: 'START',  label: 'START',  type: 'start',  x:  130, y:  80 },
      { id: 'ACCEPT', label: 'ACCEPT', type: 'accept', x: 2440, y: 340 },
      { id: 'rjS1',  label: 'REJECT', type: 'reject', x:   30, y: 160 },
      { id: 'rjS2',  label: 'REJECT', type: 'reject', x:  310, y:  40 },
      { id: 'rjS3',  label: 'REJECT', type: 'reject', x:  460, y:  40 },
      { id: 'rjS4',  label: 'REJECT', type: 'reject', x:  310, y: 300 },
      { id: 'rjS5',  label: 'REJECT', type: 'reject', x:  460, y: 400 },
      { id: 'rjS6',  label: 'REJECT', type: 'reject', x:  560, y: 400 },
      { id: 'rjS7',  label: 'REJECT', type: 'reject', x:  720, y: 400 },
      { id: 'rjS8',  label: 'REJECT', type: 'reject', x:  920, y:  80 },
      { id: 'rjS9',  label: 'REJECT', type: 'reject', x: 1100, y:  80 },
      { id: 'rjS11', label: 'REJECT', type: 'reject', x:  820, y: 260 },
      { id: 'rjS12', label: 'REJECT', type: 'reject', x: 1020, y: 440 },
      { id: 'rjS13', label: 'REJECT', type: 'reject', x: 1280, y: 440 },
      { id: 'rjS14', label: 'REJECT', type: 'reject', x: 1460, y: 600 },
      { id: 'rjS15', label: 'REJECT', type: 'reject', x: 1740, y:  80 },
      { id: 'rjS17', label: 'REJECT', type: 'reject', x: 1100, y: 660 },
      { id: 'rjS21', label: 'REJECT', type: 'reject', x: 1960, y:  80 },
      { id: 'rjS18', label: 'REJECT', type: 'reject', x: 1280, y: 660 },
      { id: 'rjS22', label: 'REJECT', type: 'reject', x: 2040, y: 200 },
      { id: 'rjS23', label: 'REJECT', type: 'reject', x: 2240, y: 200 },
      { id: 'S1',  label: 'S1',  type: 'state', x:  130, y: 160 },
      { id: 'S2',  label: 'S2',  type: 'state', x:  310, y: 160 },
      { id: 'S3',  label: 'S3',  type: 'state', x:  460, y: 160 },
      { id: 'S4',  label: 'S4',  type: 'state', x:  310, y: 240 },
      { id: 'S5',  label: 'S5',  type: 'state', x:  460, y: 240 },
      { id: 'S6',  label: 'S6',  type: 'state', x:  560, y: 240 },
      { id: 'S7',  label: 'S7',  type: 'state', x:  720, y: 240 },
      { id: 'S8',  label: 'S8',  type: 'state', x:  920, y: 160 },
      { id: 'S9',  label: 'S9',  type: 'state', x: 1100, y: 160 },
      { id: 'S11', label: 'S11', type: 'state', x:  920, y: 340 },
      { id: 'S12', label: 'S12', type: 'state', x: 1100, y: 340 },
      { id: 'S13', label: 'S13', type: 'state', x: 1280, y: 340 },
      { id: 'S14', label: 'S14', type: 'state', x: 1460, y: 500 },
      { id: 'S15', label: 'S15', type: 'state', x: 1640, y: 500 },
      { id: 'S17', label: 'S17', type: 'state', x: 1100, y: 520 },
      { id: 'S18', label: 'S18', type: 'state', x: 1280, y: 520 },
      { id: 'S20', label: 'S20', type: 'state', x: 1640, y: 340 },
      { id: 'S21', label: 'S21', type: 'state', x: 1840, y: 160 },
      { id: 'S22A',label: 'S22A',type: 'state', x: 1840, y: 340 },
      { id: 'S22B',label: 'S22B',type: 'state', x: 1840, y: 460 },
      { id: 'S23A',label: 'S23A',type: 'state', x: 2040, y: 340 },
      { id: 'S23B',label: 'S23B',type: 'state', x: 2040, y: 460 },
      { id: 'S30', label: 'S30', type: 'state', x: 2240, y: 340 },
    ],
    links: [
      { src: 'START', tgt: 'S1',   label: '' },
      { src: 'S1',  tgt: 'rjS1',  label: 'a,Δ' },
      { src: 'S1',  tgt: 'S2',    label: 'b' },
      { src: 'S2',  tgt: 'rjS2',  label: 'Δ' },
      { src: 'S2',  tgt: 'S3',    label: 'a' },
      { src: 'S2',  tgt: 'S4',    label: 'b' },
      { src: 'S3',  tgt: 'rjS3',  label: 'a,Δ' },
      { src: 'S3',  tgt: 'S5',    label: 'b', curve: 1.5, sweep: 1 },
      { src: 'S4',  tgt: 'rjS4',  label: 'a,Δ' },
      { src: 'S4',  tgt: 'S5',    label: 'b' },
      { src: 'S5',  tgt: 'S5',    label: 'a' },
      { src: 'S5',  tgt: 'S6',    label: 'b', curve: 10001 },
      { src: 'S5',  tgt: 'rjS5',  label: 'Δ' },
      { src: 'S6',  tgt: 'rjS6',  label: 'Δ' },
      { src: 'S6',  tgt: 'S6',    label: 'b' },
      { src: 'S6',  tgt: 'S7',    label: 'a' },
      { src: 'S7',  tgt: 'rjS7',  label: 'Δ' },
      { src: 'S7',  tgt: 'S8',    label: 'b' },
      { src: 'S8',  tgt: 'rjS8',  label: 'Δ' },
      { src: 'S8',  tgt: 'S9',    label: 'b' },
      { src: 'S8',  tgt: 'S11',   label: 'a' },
      { src: 'S9',  tgt: 'rjS9',  label: 'b,Δ' },
      { src: 'S9',  tgt: 'S8',    label: 'a', sweep: 1 },
      { src: 'S11', tgt: 'S12',   label: 'b' },
      { src: 'S11', tgt: 'rjS11', label: 'a,Δ' },
      { src: 'S12', tgt: 'rjS12', label: 'b,Δ' },
      { src: 'S12', tgt: 'S13',   label: 'a' },
      { src: 'S13', tgt: 'rjS13', label: 'Δ' },
      { src: 'S13', tgt: 'S14',   label: 'b' },
      { src: 'S13', tgt: 'S17',   label: 'a', curve: 1.4, sweep: 0 },
      { src: 'S14', tgt: 'rjS14', label: 'Δ' },
      { src: 'S14', tgt: 'S15',   label: 'a' },
      { src: 'S14', tgt: 'S20',   label: 'b' },
      { src: 'S15', tgt: 'S13',   label: 'b' },
      { src: 'S15', tgt: 'rjS14', label: 'a,Δ' },
      { src: 'S20', tgt: 'S21',   label: 'b' },
      { src: 'S20', tgt: 'rjS15', label: 'a,Δ' },
      { src: 'S21', tgt: 'S22A',  label: 'a' },
      { src: 'S21', tgt: 'S22B',  label: 'b', curve: 1.4, sweep: 1 },
      { src: 'S21', tgt: 'rjS21', label: 'Δ' },
      { src: 'S22A',tgt: 'rjS22', label: 'Δ' },
      { src: 'S22A',tgt: 'S22A',  label: 'a' },
      { src: 'S22A',tgt: 'S23A',  label: 'b', curve: 10001 },
      { src: 'S22B',tgt: 'S23B',  label: 'a' },
      { src: 'S22B',tgt: 'S22B',  label: 'b' },
      { src: 'S22B',tgt: 'rjS22', label: 'Δ', curve: 1.4, sweep: 0 },
      { src: 'S23A',tgt: 'rjS23', label: 'Δ' },
      { src: 'S23A',tgt: 'S30',   label: 'a', curve: 10001 },
      { src: 'S23A',tgt: 'S22A',  label: 'b' },
      { src: 'S23B',tgt: 'S30',   label: 'b' },
      { src: 'S23B',tgt: 'S22A',  label: 'a', curve: 1.5, sweep: 1 },
      { src: 'S23B',tgt: 'rjS23', label: 'Δ' },
      { src: 'S30', tgt: 'S30',   label: 'a,b' },
      { src: 'S30', tgt: 'ACCEPT',label: 'Δ' },
      { src: 'S17', tgt: 'S18',   label: 'b' },
      { src: 'S17', tgt: 'rjS17', label: 'a,Δ' },
      { src: 'S18', tgt: 'S13',   label: 'a' },
      { src: 'S18', tgt: 'rjS18', label: 'b,Δ' },
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

  const path = []
  let node = 'START'
  let pos  = 0
  const max = (input.length + 1) * 10 + 40

  for (let step = 0; step < max; step++) {
    const char = pos < input.length ? input[pos] : null
    path.push({ nodeId: node, linkIdx: -1, charIdx: pos, char: char ?? 'Δ' })

    if (node === 'ACCEPT' || node.startsWith('rj')) break

    const avail = transMap[node] || []
    let matched = null

    // priority 1 — match current character
    if (char !== null) {
      for (const t of avail) {
        if (t.label.split(',').map(s => s.trim()).includes(char)) { matched = t; break }
      }
    }
    // priority 2 — epsilon / Δ transition
    if (!matched) {
      for (const t of avail) {
        const parts = t.label.split(',').map(s => s.trim())
        if (parts.includes('Δ') || parts.includes('')) { matched = t; break }
      }
    }

    if (!matched) break
    path[path.length - 1].linkIdx = matched.idx

    // advance position only when a real character was consumed
    if (char !== null && matched.label.split(',').map(s => s.trim()).includes(char)) pos++

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
const labelFill = (i) => activeLinkIdx.value === i ? '#b45309' : '#dc2626'
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
      <div class="viz-head">Pushdown Automata Diagram</div>
      <div class="image-viewport">
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
.header-left { display:flex; align-items:center; gap:0.6rem; }
.badge {
  background: linear-gradient(135deg, #1e293b, #0f172a);
  color: #cbd5e1;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  padding: 3px 9px;
  border-radius: 6px;
  text-transform: uppercase;
}
.title { font-size:17px; font-weight:700; color:#0f172a; letter-spacing:-0.01em; }
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
.viz-head { padding:9px 16px; background:#f8fafc; border-bottom:1.5px solid #e8ecf0; font-size:10.5px; font-weight:700; color:#64748b; text-transform:uppercase; letter-spacing:0.08em; }
.image-viewport { padding:20px; display:flex; justify-content:center; background:#fafbfc; overflow-x:auto; }

.no-scrollbar-x { scrollbar-width:none; }
.no-scrollbar-x::-webkit-scrollbar { display:none; }

.pop-enter-active { animation:popIn 0.3s cubic-bezier(0.34,1.56,0.64,1); }
@keyframes popIn { from { transform:scale(0.9); opacity:0; } to { transform:scale(1); opacity:1; } }
</style>
