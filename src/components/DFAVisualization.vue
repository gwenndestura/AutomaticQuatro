<script setup>
import { ref, watch, onMounted, computed, onUnmounted } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
    problemId: { type: Number, required: true },
    testString: { type: String, default: '' },
    simKey: { type: Number, default: 0 }
})

const svgRef = ref(null)
const stepIndex = ref(0)
const isRunning = ref(false)
const done = ref(false)
const simResult = ref(null)
const autoTimer = ref(null)

// ── Zoom / pan ────────────────────────────────────────────────
const zoomScale  = ref(1)
const panX       = ref(0)
const panY       = ref(0)
const isDragging = ref(false)
const dragStart  = ref({ x: 0, y: 0, px: 0, py: 0 })
const ZOOM_MIN   = 0.25
const ZOOM_MAX   = 5

const zoomIn    = () => { zoomScale.value = Math.min(ZOOM_MAX, +(zoomScale.value * 1.2).toFixed(3)) }
const zoomOut   = () => { zoomScale.value = Math.max(ZOOM_MIN, +(zoomScale.value / 1.2).toFixed(3)) }
const resetZoom = () => { zoomScale.value = 1; panX.value = 0; panY.value = 0 }

const onWheel = (e) => {
  const f = e.deltaY < 0 ? 1.1 : 1 / 1.1
  zoomScale.value = Math.min(ZOOM_MAX, Math.max(ZOOM_MIN, +(zoomScale.value * f).toFixed(3)))
}
const onDragStart = (e) => {
  isDragging.value = true
  dragStart.value  = { x: e.clientX, y: e.clientY, px: panX.value, py: panY.value }
}
const onDrag = (e) => {
  if (!isDragging.value) return
  panX.value = dragStart.value.px + (e.clientX - dragStart.value.x)
  panY.value = dragStart.value.py + (e.clientY - dragStart.value.y)
}
const onDragEnd = () => { isDragging.value = false }

const DFA_CONFIGS = {
  1:  {
    start: 'q0',
    accept: ['q17'],

    nodes: [
      { id: 'q0',   label: '-', type: 'start',  fx: 0,    fy: 0 },
      { id: 'q1',   label: '',  type: 'state',  fx: 200,  fy: 0 },
      { id: 'q2',   label: '',  type: 'state',  fx: 380,  fy: -110 },
      { id: 'q3',   label: '',  type: 'state',  fx: 380,  fy: 110 },
      { id: 'q4',   label: '',  type: 'state',  fx: 560,  fy: 0 },
      { id: 'q5',   label: '',  type: 'state',  fx: 720,  fy: -120 },
      { id: 'q6',   label: '',  type: 'state',  fx: 720,  fy: 140 },
      { id: 'q7',   label: '',  type: 'state',  fx: 900,  fy: 0 },
      { id: 'q8',   label: '',  type: 'state',  fx: 1080, fy: 0 },
      { id: 'q9',   label: '',  type: 'state',  fx: 1260, fy: -120 },
      { id: 'q10',  label: '',  type: 'state',  fx: 1260, fy: 120 },
      { id: 'q11',  label: '',  type: 'state',  fx: 1440, fy: 0 },
      { id: 'q12',  label: '',  type: 'state',  fx: 1620, fy: 0 },
      { id: 'q13',  label: '',  type: 'state',  fx: 1800, fy: -120 },
      { id: 'q14',  label: '',  type: 'state',  fx: 1800, fy: 120 },
      { id: 'q15',  label: '',  type: 'state',  fx: 1980, fy: -120 },
      { id: 'q16',  label: '',  type: 'state',  fx: 1980, fy: 120 },
      { id: 'q17',  label: '+', type: 'accept', fx: 2160, fy: 0 },
      // Separate trap nodes — one per source that goes to trap
      { id: 'qT_q0',  label: 'T', type: 'trap', fx: 0,    fy: -140 },
      { id: 'qT_q2',  label: 'T', type: 'trap', fx: 380,  fy: -260 },
      { id: 'qT_q3',  label: 'T', type: 'trap', fx: 380,  fy: 260 },
      { id: 'qT_q9',  label: 'T', type: 'trap', fx: 1260, fy: -280 },
      { id: 'qT_q10', label: 'T', type: 'trap', fx: 1340, fy: 280 },
    ],

    links: [
      { source: 'q0',  target: 'qT_q0',  label: 'a' },
      { source: 'q0',  target: 'q1',     label: 'b' },

      { source: 'q1',  target: 'q2',     label: 'a' },
      { source: 'q1',  target: 'q3',     label: 'b' },

      { source: 'q2',  target: 'qT_q2',  label: 'a' },
      { source: 'q2',  target: 'q4',     label: 'b' },

      { source: 'q3',  target: 'qT_q3',  label: 'a' },
      { source: 'q3',  target: 'q4',     label: 'b' },

      { source: 'q4',  target: 'q5',     label: 'a' },
      { source: 'q4',  target: 'q6',     label: 'b' },

      { source: 'q5',  target: 'q5',     label: 'a' },
      { source: 'q5',  target: 'q7',     label: 'b' },

      { source: 'q6',  target: 'q5',     label: 'a' },
      { source: 'q6',  target: 'q6',     label: 'b' },

      { source: 'q7',  target: 'q8',     label: 'a' },
      { source: 'q7',  target: 'q4',     label: 'b' },

      { source: 'q8',  target: 'q9',     label: 'a' },
      { source: 'q8',  target: 'q11',    label: 'b' },

      { source: 'q9',  target: 'qT_q9',  label: 'a' },
      { source: 'q9',  target: 'q10',    label: 'b' },

      { source: 'q10', target: 'q8',     label: 'a' },
      { source: 'q10', target: 'qT_q10', label: 'b' },

      { source: 'q11', target: 'q8',     label: 'a' },
      { source: 'q11', target: 'q12',    label: 'b' },

      { source: 'q12', target: 'q13',    label: 'a' },
      { source: 'q12', target: 'q14',    label: 'b' },

      { source: 'q13', target: 'q13',    label: 'a' },
      { source: 'q13', target: 'q15',    label: 'b' },

      { source: 'q14', target: 'q16',    label: 'a' },
      { source: 'q14', target: 'q14',    label: 'b' },

      { source: 'q15', target: 'q17',    label: 'a' },
      { source: 'q15', target: 'q14',    label: 'b' },

      { source: 'q16', target: 'q13',    label: 'a' },
      { source: 'q16', target: 'q17',    label: 'b' },

      { source: 'q17', target: 'q17',    label: 'a, b' },

      // Trap self-loops
      { source: 'qT_q0',  target: 'qT_q0',  label: 'a, b' },
      { source: 'qT_q2',  target: 'qT_q2',  label: 'a, b' },
      { source: 'qT_q3',  target: 'qT_q3',  label: 'a, b' },
      { source: 'qT_q9',  target: 'qT_q9',  label: 'a, b' },
      { source: 'qT_q10', target: 'qT_q10', label: 'a, b' }
    ],

    transitions: {
      q0:      { a: 'qT_q0',  b: 'q1' },
      qT_q0:   { a: 'qT_q0',  b: 'qT_q0' },
      q1:      { a: 'q2',     b: 'q3' },
      q2:      { a: 'qT_q2',  b: 'q4' },
      qT_q2:   { a: 'qT_q2',  b: 'qT_q2' },
      q3:      { a: 'qT_q3',  b: 'q4' },
      qT_q3:   { a: 'qT_q3',  b: 'qT_q3' },
      q4:      { a: 'q5',     b: 'q6' },
      q5:      { a: 'q5',     b: 'q7' },
      q6:      { a: 'q5',     b: 'q6' },
      q7:      { a: 'q8',     b: 'q4' },
      q8:      { a: 'q9',     b: 'q11' },
      q9:      { a: 'qT_q9',  b: 'q10' },
      qT_q9:   { a: 'qT_q9',  b: 'qT_q9' },
      q10:     { a: 'q8',     b: 'qT_q10' },
      qT_q10:  { a: 'qT_q10', b: 'qT_q10' },
      q11:     { a: 'q8',     b: 'q12' },
      q12:     { a: 'q13',    b: 'q14' },
      q13:     { a: 'q13',    b: 'q15' },
      q14:     { a: 'q16',    b: 'q14' },
      q15:     { a: 'q17',    b: 'q14' },
      q16:     { a: 'q13',    b: 'q17' },
      q17:     { 'a, b': 'q17' }
    }
  },

  2: {
    start: 'p0',
    accept: ['p10'],
    nodes: [
      { id: 'p0',  label: '-', type: 'start',  fx: 0,   fy: 0 },
      { id: 'p1',  label: '',  type: 'state',  fx: 160, fy: -80 },
      { id: 'p2',  label: '',  type: 'state',  fx: 160, fy: 80 },
      { id: 'p3',  label: '',  type: 'state',  fx: 320, fy: -80 },
      { id: 'p4',  label: '',  type: 'state',  fx: 320, fy: 80 },
      { id: 'p5',  label: '',  type: 'state',  fx: 480, fy: 0 },
      { id: 'p6',  label: '',  type: 'state',  fx: 640, fy: -80 },
      { id: 'p7',  label: '',  type: 'state',  fx: 640, fy: 80 },
      { id: 'p8',  label: '',  type: 'state',  fx: 800, fy: -80 },
      { id: 'p9',  label: '',  type: 'state',  fx: 830, fy: 130 },
      { id: 'p10', label: '+', type: 'accept', fx: 970, fy: 0 }
    ],
    links: [
      { source: 'p0',  target: 'p1',  label: '0' },
      { source: 'p0',  target: 'p2',  label: '1' },
      { source: 'p1',  target: 'p4',  label: '0' },
      { source: 'p1',  target: 'p5',  label: '1', curve: 1.6, sweep: 0 },
      { source: 'p2',  target: 'p3',  label: '0' },
      { source: 'p2',  target: 'p2',  label: '1' },
      { source: 'p3',  target: 'p5',  label: '1', curve: 1.6, sweep: 1 },
      { source: 'p3',  target: 'p1',  label: '0', curve: 1.4, sweep: 1 },
      { source: 'p4',  target: 'p5',  label: '0' },
      { source: 'p4',  target: 'p2',  label: '1', curve: 1.4, sweep: 0 },
      { source: 'p5',  target: 'p6',  label: '0' },
      { source: 'p5',  target: 'p7',  label: '1' },
      { source: 'p6',  target: 'p10', label: '0', curve: 1.6, sweep: 0 },
      { source: 'p6',  target: 'p7',  label: '1', curve: 10001 },
      { source: 'p7',  target: 'p8',  label: '0' },
      { source: 'p7',  target: 'p9',  label: '1' },
      { source: 'p8',  target: 'p10', label: '1', curve: 1.6, sweep: 1 },
      { source: 'p8',  target: 'p6',  label: '0', curve: 1.4, sweep: 1 },
      { source: 'p9',  target: 'p10', label: '1' },
      { source: 'p9',  target: 'p6',  label: '0', curve: 1.4, sweep: 0 },
      { source: 'p10', target: 'p10', label: '0, 1' }
    ],
    transitions: {
      p0:  { '0': 'p1',  '1': 'p2' },
      p1:  { '0': 'p4',  '1': 'p5' },
      p2:  { '0': 'p3',  '1': 'p2' },
      p3:  { '0': 'p1',  '1': 'p5' },
      p4:  { '0': 'p5',  '1': 'p2' },
      p5:  { '0': 'p6',  '1': 'p7' },
      p6:  { '0': 'p10', '1': 'p7' },
      p7:  { '0': 'p8',  '1': 'p9' },
      p8:  { '0': 'p6',  '1': 'p10' },
      p9:  { '0': 'p6',  '1': 'p10' },
      p10: { '0, 1': 'p10' }
    }
  }
}

const REGEX_MAP = {
  1: '(bab+bbb)a*b*(a*+b*)(ba)*(aba)(bab+aba)*bb(a+b)*(bab+aba)(a+b)*',
  2: '(1+0)*1*0*(101+01+000)(1+0)*(101+00)*(111+00+101)(1+0)*'
}

const dfa = computed(() => DFA_CONFIGS[props.problemId])
const problemRegex = computed(() => REGEX_MAP[props.problemId])

const steps = computed(() =>
  simResult.value ? simResult.value.steps : [{ state: dfa.value.start, charIndex: -1, char: null }]
)

const currentStep = computed(() => steps.value[stepIndex.value] || steps.value[steps.value.length - 1])
const currentState = computed(() => currentStep.value?.state ?? null)
const currentCharIdx = computed(() => simResult.value ? (steps.value[stepIndex.value]?.charIndex ?? -1) : -1)

const resultAccepted = computed(() => done.value && !!simResult.value?.accepted)

const tape = computed(() => {
  if (!props.testString) return []
  return props.testString.split('').map((ch, i) => {
    const idx = currentCharIdx.value
    if (!simResult.value) return { ch, status: 'pending' }
    if (i < idx) return { ch, status: 'done' }
    if (i === idx) return { ch, status: 'active' }
    return { ch, status: 'pending' }
  })
})

const runSimulation = (input) => {
  const d = dfa.value
  const trapIds = new Set(d.nodes.filter(n => n.type === 'trap').map(n => n.id))
  const stepsList = [{ state: d.start, charIndex: -1, char: null }]
  let current = d.start
  for (let i = 0; i < input.length; i++) {
    const ch = input[i]
    let next = d.transitions[current]?.[ch]

    // Handle split transitions string like "0, 1"
    if (!next) {
        for (const [key, val] of Object.entries(d.transitions[current])) {
            if (key.includes(ch)) next = val;
        }
    }

    if (next === undefined) {
      stepsList.push({ state: null, charIndex: i, char: ch, dead: true })
      return { steps: stepsList, accepted: false }
    }
    current = next
    stepsList.push({ state: current, charIndex: i, char: ch })
    if (trapIds.has(current)) {
      return { steps: stepsList, accepted: false }
    }
  }
  return { steps: stepsList, accepted: d.accept.includes(current) }
}

const initSim = () => {
  const result = runSimulation(props.testString)
  simResult.value = result
  return result
}

const highlightElements = (fromId, toId) => {
    d3.select(svgRef.value).selectAll('circle').attr('stroke', '#fff').attr('stroke-width', 1.5).style('filter', null);
    d3.select(svgRef.value).selectAll('path.edge').attr('stroke', 'black').attr('stroke-width', 2);
    
    const isAccepted = resultAccepted.value;
    const isDone = done.value;
    const color = isDone ? (isAccepted ? '#22c55e' : '#ef4444') : '#f59e0b';
    
    if (toId) {
        d3.select(svgRef.value).select(`#node-${toId}`)
          .attr('stroke', color)
          .attr('stroke-width', 3.5)
          .style('filter', `drop-shadow(0 0 8px ${color})`);
    } else if (fromId && !toId) {
        // Dead state highlight previous node red
        d3.select(svgRef.value).select(`#node-${fromId}`)
          .attr('stroke', '#ef4444')
          .attr('stroke-width', 3.5)
          .style('filter', `drop-shadow(0 0 8px #ef4444)`);
    }
    
    if (fromId && toId) {
        // Highlight all lines belonging to the correct char index (there might be multiple chars)
        const char = currentStep.value?.char;
        if (char) {
            const specificLink = d3.select(svgRef.value).select(`#link-${fromId}-${toId}-${char}`);
            if (!specificLink.empty()) {
                specificLink.attr('stroke', color).attr('stroke-width', 3);
            } else {
                d3.select(svgRef.value).select(`#link-${fromId}-${toId}`).attr('stroke', color).attr('stroke-width', 3);
            }
        } else {
            d3.select(svgRef.value).select(`path[id^="link-${fromId}-${toId}"]`).attr('stroke', color).attr('stroke-width', 3);
        }
    }
}

const advance = (result, idx) => {
  const from = result.steps[idx].state
  const to = result.steps[idx + 1]?.state
  stepIndex.value = idx + 1
  
  highlightElements(from, to)

  if (idx + 1 >= result.steps.length - 1) done.value = true
}

const runAuto = () => {
  if (props.testString === null || props.testString === undefined) return;
  doReset();
  const result = initSim()
  isRunning.value = true
  let idx = 0
  stepIndex.value = 0
  done.value = false
  
  highlightElements(null, result.steps[0].state)
  
  const max = result.steps.length - 1
  autoTimer.value = setInterval(() => {
    if (idx >= max) {
      clearInterval(autoTimer.value)
      autoTimer.value = null
      isRunning.value = false
      done.value = true
      // Trigger final highlight update
      highlightElements(null, result.steps[max].state)
      return
    }
    advance(result, idx)
    idx++
  }, 800)
}

const doReset = () => {
  clearInterval(autoTimer.value)
  autoTimer.value = null
  isRunning.value = false
  stepIndex.value = 0
  done.value = false
  simResult.value = null
  
  if (svgRef.value) {
      d3.select(svgRef.value).selectAll('circle').attr('stroke', '#fff').attr('stroke-width', 1.5).style('filter', null);
      d3.select(svgRef.value).selectAll('path.edge').attr('stroke', 'black').attr('stroke-width', 2);
  }
}

const renderDFA = () => {
    if (!svgRef.value) return;
    
    const data = dfa.value;
    d3.select(svgRef.value).selectAll("*").remove();

    const svg = d3.select(svgRef.value).style("overflow", "hidden"); 

    const defs = svg.append("defs");

    // Arrow for regular edges (path ends at node center, refX offsets back to circle edge)
    defs.append("marker")
        .attr("id", "arrow")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 22)
        .attr("refY", 0)
        .attr("markerWidth", 6)
        .attr("markerHeight", 6)
        .attr("orient", "auto")
        .append("path")
        .attr("d", "M0,-5L10,0L0,5")
        .attr("fill", "#000");

    // Arrow for self-loops (path ends at circle edge, needs small refX)
    defs.append("marker")
        .attr("id", "arrow-self")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 9)
        .attr("refY", 0)
        .attr("markerWidth", 6)
        .attr("markerHeight", 6)
        .attr("orient", "auto")
        .append("path")
        .attr("d", "M0,-5L10,0L0,5")
        .attr("fill", "#000");

    const simulation = d3.forceSimulation(data.nodes)
        .force("link", d3.forceLink(data.links).id(d => d.id));

    const linkGroup = svg.append("g")
    const link = linkGroup.selectAll("path")
        .data(data.links)
        .join("path")
        .attr("class", "edge")
        .attr("id", d => `link-${d.source.id ?? d.source}-${d.target.id ?? d.target}-${d.label}`)
        .attr("fill", "none")
        .attr("stroke", "black")
        .attr("stroke-width", 2)
        .attr("marker-end", d => {
            const src = d.source.id ?? d.source;
            const tgt = d.target.id ?? d.target;
            return src === tgt ? "url(#arrow-self)" : "url(#arrow)";
        });

    const linkLabel = svg.append("g").selectAll("text")
        .data(data.links).join("text").text(d => d.label)
        .attr("font-size", "16px")
        .attr("fill", "#1d4ed8")
        .attr("font-weight", "bold")
        .attr("text-anchor", "middle")
        // Create the white background aura effect using stroke
        .style("paint-order", "stroke")
        .style("stroke", "#ffffff")
        .style("stroke-width", "5px")
        .style("stroke-linejoin", "round");

    const nodeGroup = svg.append("g")
    const node = nodeGroup.selectAll("circle")
        .data(data.nodes)
        .join("circle")
        .attr("id", d => `node-${d.id}`)
        .attr("stroke", "#fff")
        .attr("stroke-width", 1.5)
        .attr("r", 20)
        .attr("fill", d => d.type === 'accept' ? '#4caf50' : d.type === 'start' ? '#2563eb' : d.type === 'trap' ? '#ef4444' : '#f59e0b');

    const label = svg.append("g").selectAll("text")
        .data(data.nodes).join("text").text(d => d.label)
        .attr("dy", d => (d.label === '-' || d.label === '+') ? 8 : 5)
        .attr("text-anchor", "middle")
        .attr("font-size", d => (d.label === '-' || d.label === '+') ? "28px" : "14px")
        .attr("pointer-events", "none")
        .attr("fill", "white").attr("font-weight", "bold");

    simulation.tick(300);

link.attr("d", d => {
    const dx = d.target.x - d.source.x;
    const dy = d.target.y - d.source.y;
    
    // 1. Handle Self-Loops
    if (d.source === d.target) {
        const loopR = 16;
        const edgeDx = 9;
        const edgeDy = 17;
        const cx = d.source.x;
        const cy = d.source.y;
        return `M${cx - edgeDx},${cy - edgeDy} A ${loopR} ${loopR} 0 1 1 ${cx + edgeDx},${cy - edgeDy}`;
    }

    // 2. Calculate Distance
    let dr = Math.sqrt(dx * dx + dy * dy);
    
    // 3. Determine Sweep (Direction of the curve)
    // If d.sweep is provided (0 or 1), use it. 
    // Otherwise, use your ID-based direction logic.
    let finalSweep;
    if (d.sweep !== undefined) {
        finalSweep = d.sweep;
    } else {
        const sourceNum = parseInt((d.source.id ?? d.source).replace(/\D/g, '')) || 0;
        const targetNum = parseInt((d.target.id ?? d.target).replace(/\D/g, '')) || 0;
        finalSweep = sourceNum < targetNum ? 1 : 0;
    }

    // 4. Determine Radius (Intensity of the curve)
    // If d.curve is a huge number (like 50000), it makes the line straight.
    // Otherwise, we apply the multiplier to the distance.
    if (d.curve && d.curve > 1000) {
        // High curve value = Straight line
        return `M${d.source.x},${d.source.y} L${d.target.x},${d.target.y}`;
    } else {
        // Standard curved arc
        dr = dr * (d.curve || 1.3);
        return `M${d.source.x},${d.source.y} A ${dr} ${dr} 0 0 ${finalSweep} ${d.target.x},${d.target.y}`;
    }
});

    linkLabel
        .attr("x", d => {
            if (d.source === d.target) return d.source.x;
            return link.nodes()[data.links.indexOf(d)].getPointAtLength(0.5 * link.nodes()[data.links.indexOf(d)].getTotalLength()).x;
        })
        .attr("y", d => {
            if (d.source === d.target) return d.source.y - 52;
            return link.nodes()[data.links.indexOf(d)].getPointAtLength(0.5 * link.nodes()[data.links.indexOf(d)].getTotalLength()).y;
        });

    node.attr("cx", d => d.x).attr("cy", d => d.y);
    label.attr("x", d => d.x).attr("y", d => d.y);

    if (data.nodes.length > 0) {
        const minX = Math.min(...data.nodes.map(n => n.x));
        const maxX = Math.max(...data.nodes.map(n => n.x));
        const minY = Math.min(...data.nodes.map(n => n.y));
        const maxY = Math.max(...data.nodes.map(n => n.y));
        const padding = 80;
        const width = maxX - minX + padding * 2;
        const height = maxY - minY + padding * 2;
        svg.attr("viewBox", `${minX - padding} ${minY - padding} ${width} ${height}`)
           .style("width", "100%")
           .style("max-width", "100%")
           .style("max-height", "500px");
    }
    simulation.stop();
};

watch(() => props.problemId, () => {
    doReset();
    renderDFA();
});

watch(() => props.testString, (newStr) => {
    if (newStr !== null && newStr !== undefined) {
        runAuto();
    } else {
        doReset();
    }
});

watch(() => props.simKey, () => {
    if (props.testString !== null && props.testString !== undefined) {
        runAuto();
    }
});

onMounted(() => {
    renderDFA();
});

onUnmounted(() => {
    clearInterval(autoTimer.value);
});
</script>

<template>
  <div class="dfa-wrap">
    
    <!-- Header -->
    <div class="dfa-header">
      <div class="header-left">
        <div class="header-tag">
          <span class="tag-type">DFA</span>
          <span class="tag-prob">Problem {{ problemId }}</span>
        </div>
      </div>
      <div class="header-right">
        <span class="dot start-dot"></span><span class="leg">Start</span>
        <span class="dot state-dot"></span><span class="leg">State</span>
        <span class="dot accept-dot"></span><span class="leg">Accept</span>
        <span class="dot trap-dot"></span><span class="leg">Trap</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap" v-if="problemRegex">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ problemRegex }}</code>
    </div>

    <!-- Simulation panel -->
    <div class="sim-panel" :class="done ? (resultAccepted ? 'sim-ok' : 'sim-fail') : 'sim-running'">

      <!-- Left: state + reading -->
      <div class="sim-left">
        <div class="sim-state-wrap" v-if="simResult && currentState">
          <span class="sim-micro-label">State</span>
          <span :class="['sim-state', done ? (resultAccepted ? 'ok' : 'fail') : 'running']">
            {{ currentState }}
          </span>
        </div>
        <div v-if="simResult && currentState" class="sim-divider-v"></div>
        <div class="sim-reading-wrap">
          <span class="sim-micro-label">Reading</span>
          <span v-if="currentStep?.char != null" class="sim-char">{{ currentStep.char }}</span>
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
          <div v-if="done" :class="['sim-result', resultAccepted ? 'result-ok' : 'result-fail']">
            <span class="result-icon">{{ resultAccepted ? '✓' : '✕' }}</span>
            <span class="result-text">{{ resultAccepted ? 'Accepted' : 'Rejected' }}</span>
          </div>
          <div v-else class="sim-result result-running">
            <span class="result-icon spin">◌</span>
            <span class="result-text">Tracing…</span>
          </div>
        </transition>
      </div>

    </div>

    <!-- SVG Visualization -->
    <div class="viz-container">
      <div class="zoom-controls">
        <button class="zoom-btn" title="Zoom in"  @click="zoomIn">+</button>
        <button class="zoom-btn" title="Zoom out" @click="zoomOut">−</button>
        <button class="zoom-btn" title="Reset view" @click="resetZoom">⊙</button>
      </div>
      <div
        class="viz-scroll-area"
        @wheel.prevent="onWheel"
        @mousedown.prevent="onDragStart"
        @mousemove="onDrag"
        @mouseup="onDragEnd"
        @mouseleave="onDragEnd"
      >
        <div :style="{ transform: `translate(${panX}px,${panY}px) scale(${zoomScale})`, transformOrigin: '0 0', display: 'inline-block', width: '100%' }">
          <svg ref="svgRef"></svg>
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
.dfa-wrap {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    max-width: 920px;
    margin: 16px auto;
    padding: 1.2rem;
    background: #ffffff;
    border-radius: 14px;
    box-shadow: 0 4px 24px rgba(0,0,0,0.07), 0 1px 4px rgba(0,0,0,0.04);
    font-family: 'Inter', 'Segoe UI', sans-serif;
    border: 1px solid #e8ecf0;
}

/* Header */
.dfa-header {
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
.header-right {
    display: flex;
    align-items: center;
    gap: 0.4rem;
    flex-wrap: wrap;
}
.dot {
    width: 9px;
    height: 9px;
    border-radius: 50%;
    display: inline-block;
    flex-shrink: 0;
}
.start-dot  { background: #2563eb; }
.state-dot  { background: #f59e0b; margin-left: 0.5rem; }
.accept-dot { background: #10b981; margin-left: 0.5rem; }
.trap-dot   { background: #ef4444; margin-left: 0.5rem; }
.leg {
    font-size: 11.5px;
    color: #6b7280;
    font-weight: 500;
}

/* Regex */
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
.regex-label {
    font-size: 10px;
    font-weight: 700;
    color: #94a3b8;
    letter-spacing: 0.07em;
    text-transform: uppercase;
    padding-top: 2px;
    white-space: nowrap;
}
.regex-code {
    font-family: 'Courier New', monospace;
    font-size: 12.5px;
    color: #1e40af;
    word-break: break-all;
    line-height: 1.65;
}

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
.sim-char.muted { color: #cbd5e1; background: #f8fafc; border-color: #e2e8f0; }

.sim-divider-v {
    width: 1px;
    align-self: stretch;
    background: #e8ecf0;
    margin: 10px 0;
}

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
}

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
.result-icon { font-size: 20px; font-weight: 900; line-height: 1; }
.result-text { font-size: 10px; font-weight: 700; letter-spacing: 0.07em; text-transform: uppercase; }
.result-ok   .result-icon { color: #16a34a; }
.result-ok   .result-text { color: #16a34a; }
.result-fail .result-icon { color: #dc2626; }
.result-fail .result-text { color: #dc2626; }
.result-running .result-icon { color: #94a3b8; }
.result-running .result-text { color: #94a3b8; }

@keyframes spin { to { transform: rotate(360deg); } }
.spin { display: inline-block; animation: spin 1.2s linear infinite; }

/* Viz container */
.viz-container {
    border: 1.5px solid #e8ecf0;
    border-radius: 10px;
    background: #fafbfc;
    overflow: hidden;
    position: relative;
}
.zoom-controls {
    position: absolute;
    top: 8px;
    right: 8px;
    z-index: 10;
    display: flex;
    gap: 4px;
}
.zoom-btn {
    width: 28px;
    height: 28px;
    border: 1.5px solid #e1e4e8;
    border-radius: 6px;
    background: #fff;
    color: #57606a;
    font-size: 14px;
    font-weight: 700;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    transition: all 0.13s;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08);
    line-height: 1;
}
.zoom-btn:hover { background: #f0fdf4; border-color: #22c55e; color: #16a34a; }
.viz-scroll-area {
    overflow: hidden;
    cursor: grab;
    user-select: none;
}
.viz-scroll-area:active { cursor: grabbing; }

.pop-enter-active { animation: popIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
@keyframes popIn {
  from { transform: scale(0.9); opacity: 0; }
  to   { transform: scale(1);    opacity: 1; }
}

/* Hide scrollbar */
.no-scrollbar-x {
  scrollbar-width: none;
}
.no-scrollbar-x::-webkit-scrollbar {
  display: none;
}
</style>

