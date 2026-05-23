<script setup>
const sections = [
  {
    id: 'introduction',
    title: 'Introduction',
    content: `AutomaticQuatro is an interactive, browser-based educational tool built for S-CSPC327 Theory of Automata and Formal Languages at De La Salle University – Dasmariñas. It lets you test strings against two pre-defined regular expressions and instantly see how each string is processed by three different formal models — a Deterministic Finite Automaton (DFA), a Context-Free Grammar (CFG), and a Pushdown Automaton (PDA). Each model is animated step-by-step so you can follow exactly which state is active, which character is being read, and whether the string is ultimately accepted or rejected.`
  },
  {
    id: 'getting-started',
    title: 'Getting Started',
    subsections: [
      {
        title: 'System Requirements',
        body: 'AutomaticQuatro runs entirely in the browser — no installation needed. Use a modern desktop browser (Chrome 100+, Firefox 100+, or Edge 100+) for the best experience. The complex automata diagrams are optimized for larger screens; a minimum width of 900 px is recommended.'
      },
      {
        title: 'Basic Workflow',
        items: [
          { label: '1 — Select a Problem', text: 'Pick Problem 1 or Problem 2 from the left sidebar. Each problem is tied to a specific regular expression shown in the Regex block below the selector.' },
          { label: '2 — Choose an Automaton tab', text: 'Click DFA, CFG, or PDA in the top tab bar of the right panel to select the model you want to visualize.' },
          { label: '3 — Enter a test string', text: 'Type a string in the input field. The character preview below the field shows each character individually. You can also click a suggestion chip to auto-fill and run a known example string.' },
          { label: '4 — Run the simulation', text: 'Press Enter or click Run. The selected automaton animates the trace step by step — highlighting the active state and the character being consumed at each transition.' },
          { label: '5 — Review the result', text: 'A green Accepted or red Rejected badge appears when the trace finishes. Previously run strings are saved in the Recent history list for quick re-simulation.' },
        ]
      }
    ]
  },
  {
    id: 'features',
    title: 'Core Features',
    subsections: [
      {
        title: 'Smart Input Panel',
        body: 'The left sidebar input area is designed for fast experimentation.',
        items: [
          { label: 'Single-input field', text: 'Enter any string and press Enter or click Run. No need to manage multiple rows.' },
          { label: 'Character preview', text: 'As you type, each character is shown as an individual chip so you can spot typos before running.' },
          { label: 'Suggestion chips', text: 'Clickable example strings are shown for each problem. Clicking a chip auto-fills the input and immediately starts the simulation.' },
          { label: 'Recent history', text: 'The last 10 strings you simulated are listed below the input. Click the play button on any history entry to re-run it instantly.' },
        ]
      },
      {
        title: 'Deterministic Finite Automaton (DFA)',
        body: 'The DFA tab renders the full state-transition graph for the selected problem and animates a trace when a string is submitted.',
        items: [
          { label: 'State colors', text: 'Start state — blue circle; non-final states — amber; accepting state — green; trap/dead states — red.' },
          { label: 'Transition arrows', text: 'Labeled directed edges show which input symbol causes each state change. Self-loops and curved back-edges are drawn to avoid overlap.' },
          { label: 'Step-by-step animation', text: 'The active state is highlighted with a glow, the tape shows each character being consumed, and the current state label updates live.' },
          { label: 'Zoom and pan', text: 'Scroll the mouse wheel to zoom in or out. Click and drag to pan. Use the + / − / ⊙ buttons in the top-right corner of the diagram for keyboard-friendly zoom control.' },
        ]
      },
      {
        title: 'Context-Free Grammar (CFG)',
        body: 'The CFG tab displays the complete set of production rules derived from the regular expression.',
        items: [
          { label: 'Production rules', text: 'Each rule shows its left-hand non-terminal, the → arrow, and all right-hand alternatives separated by |.' },
          { label: 'Color coding', text: 'Non-terminal symbols appear in amber; terminal symbols appear in green for immediate visual distinction.' },
          { label: 'String validation', text: 'Submit a string to see a pass/fail result checked against the grammar.' },
        ]
      },
      {
        title: 'Pushdown Automaton (PDA)',
        body: 'The PDA tab shows a full NFA-style pushdown automaton diagram with animated tracing.',
        items: [
          { label: 'Node shapes', text: 'READ states are drawn as yellow diamonds; START and ACCEPT use rounded blue/green rectangles; REJECT states use red rounded rectangles.' },
          { label: 'Epsilon (Δ) transitions', text: 'Arrows labeled Δ fire without consuming any input character, enabling the NFA to explore multiple paths through the diagram.' },
          { label: 'NFA simulation', text: 'The simulator uses a depth-first search over all non-rejecting paths to find an accepting trace, so valid strings are never incorrectly rejected due to a greedy choice.' },
          { label: 'Zoom and pan', text: 'The PDA diagram is large — use the scroll wheel to zoom and drag to pan, or use the + / − / ⊙ buttons above the diagram.' },
        ]
      }
    ]
  },
  {
    id: 'faq',
    title: 'Troubleshooting',
    items: [
      { label: 'String is rejected but I think it should be accepted', text: 'Double-check the string character-by-character using the character preview chips. Then compare against the regex shown in the sidebar. Use the DFA trace to see exactly which state causes rejection.' },
      { label: 'Simulation does not start', text: 'Make sure you pressed Enter or clicked the Run button after typing the string. Clicking a suggestion chip runs automatically.' },
      { label: 'Diagram is off-screen or hard to read', text: 'Use the + / − zoom buttons or scroll the mouse wheel over the diagram. Click and drag to reposition. Press ⊙ to reset the view.' },
      { label: 'CFG shows no rules', text: 'The CFG panel requires a problem to be selected. If the regex field is empty, go back to the Simulations view and choose a problem first.' },
      { label: 'Animation is very slow', text: 'For long strings, each step adds a small delay. Wait for the animation to complete or refresh the page and try a shorter string.' },
      { label: 'Logo or images are not loading', text: 'Try a hard refresh (Ctrl + Shift + R on Windows, Cmd + Shift + R on Mac) to clear the browser cache.' },
    ]
  }
]
</script>

<template>
  <div class="manual-root">
    <div class="manual-inner">

      <!-- Page header -->
      <header class="manual-header">
        <div class="manual-header-tag">Documentation</div>
        <h1 class="manual-title">User Manual</h1>
        <p class="manual-intro">
          A reference guide for using the Automata Simulator — a tool built for
          <strong class="manual-strong">S-CSPC327 Theory of Automata and Formal Languages</strong>
          at De La Salle University – Dasmariñas.
        </p>
        <a
          href="https://docs.google.com/document/d/1sWP7bERrjxlXzy6-0Ccm8Wy5c0E_gGDu3ShWwXFDZGM/edit?usp=sharing"
          target="_blank"
          rel="noopener noreferrer"
          class="manual-link"
        >
          View full technical documentation
          <svg width="12" height="12" viewBox="0 0 14 14" fill="none" aria-hidden="true"
            style="margin-left:4px;vertical-align:middle">
            <path d="M3 11L11 3M11 3H6M11 3v5" stroke="currentColor" stroke-width="1.8"
              stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </a>
      </header>

      <!-- Sections -->
      <div class="manual-body">

        <!-- Navigation TOC -->
        <nav class="manual-toc" aria-label="Table of contents">
          <p class="toc-heading">Contents</p>
          <ol class="toc-list">
            <li v-for="s in sections" :key="s.id">
              <a :href="`#${s.id}`" class="toc-link">{{ s.title }}</a>
            </li>
          </ol>
        </nav>

        <!-- Content -->
        <div class="manual-content">
          <section
            v-for="s in sections"
            :key="s.id"
            :id="s.id"
            class="manual-section"
          >
            <h2 class="manual-section-title">{{ s.title }}</h2>

            <p v-if="s.content" class="manual-p">{{ s.content }}</p>

            <template v-if="s.items">
              <ul class="manual-list">
                <li v-for="item in s.items" :key="item.label" class="manual-li">
                  <strong class="manual-strong">{{ item.label }}:</strong>
                  {{ item.text }}
                </li>
              </ul>
            </template>

            <template v-if="s.subsections">
              <div
                v-for="sub in s.subsections"
                :key="sub.title"
                class="manual-subsection"
              >
                <h3 class="manual-h3">{{ sub.title }}</h3>
                <p v-if="sub.body" class="manual-p">{{ sub.body }}</p>
                <ul v-if="sub.items" class="manual-list">
                  <li v-for="item in sub.items" :key="item.label" class="manual-li">
                    <strong class="manual-strong">{{ item.label }}:</strong>
                    {{ item.text }}
                  </li>
                </ul>
              </div>
            </template>
          </section>

          <!-- Tech stack footer -->
          <div class="manual-stack">
            <p class="stack-label">Technology Stack</p>
            <div class="stack-tags">
              <span class="manual-tag">Vue.js 3</span>
              <span class="manual-tag">D3.js</span>
              <span class="manual-tag">Vite</span>
              <span class="manual-tag">Git / GitHub</span>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<style scoped>
.manual-root {
  background: #f6f8fa;
  min-height: calc(100vh - 60px);
  padding: 40px 24px;
}

.manual-inner {
  max-width: 960px;
  margin: 0 auto;
  background: #ffffff;
  border: 1px solid #e1e4e8;
  border-radius: 12px;
  overflow: hidden;
}

/* Header */
.manual-header {
  padding: 36px 40px 28px;
  border-bottom: 1px solid #e1e4e8;
  background: #f6f8fa;
}
.manual-header-tag {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #8c959f;
  margin-bottom: 8px;
}
.manual-title {
  font-size: 26px;
  font-weight: 800;
  color: #0f172a;
  letter-spacing: -0.02em;
  margin-bottom: 12px;
}
.manual-intro {
  font-size: 14px;
  color: #57606a;
  line-height: 1.65;
  max-width: 600px;
  margin-bottom: 16px;
}
.manual-strong { font-weight: 600; color: #0f172a; }
.manual-link {
  display: inline-flex;
  align-items: center;
  font-size: 13px;
  font-weight: 600;
  color: #0969da;
  text-decoration: none;
  transition: color 0.15s;
}
.manual-link:hover { color: #0550ae; text-decoration: underline; }

/* Body layout */
.manual-body {
  display: flex;
  align-items: flex-start;
}

/* TOC */
.manual-toc {
  width: 200px;
  flex-shrink: 0;
  padding: 28px 20px;
  border-right: 1px solid #e1e4e8;
  position: sticky;
  top: 60px;
  align-self: flex-start;
}
.toc-heading {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8c959f;
  margin-bottom: 10px;
}
.toc-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 4px;
  counter-reset: toc;
}
.toc-list li { counter-increment: toc; }
.toc-link {
  display: flex;
  align-items: baseline;
  gap: 6px;
  font-size: 12px;
  font-weight: 500;
  color: #57606a;
  text-decoration: none;
  padding: 4px 6px;
  border-radius: 5px;
  transition: all 0.12s;
}
.toc-link::before {
  content: counter(toc);
  font-size: 10px;
  color: #8c959f;
  font-weight: 600;
  flex-shrink: 0;
}
.toc-link:hover { background: #f6f8fa; color: #0f172a; }

/* Content */
.manual-content {
  flex: 1;
  min-width: 0;
  padding: 28px 36px 40px;
}

.manual-section {
  margin-bottom: 40px;
  scroll-margin-top: 80px;
}
.manual-section:last-of-type { margin-bottom: 0; }

.manual-section-title {
  font-size: 16px;
  font-weight: 700;
  color: #0f172a;
  padding-bottom: 10px;
  border-bottom: 1px solid #e1e4e8;
  margin-bottom: 16px;
}

.manual-subsection {
  margin-bottom: 24px;
}
.manual-h3 {
  font-size: 13px;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 8px;
  margin-top: 20px;
}

.manual-p {
  font-size: 13.5px;
  color: #57606a;
  line-height: 1.7;
  margin-bottom: 12px;
}

.manual-list {
  margin: 8px 0 0;
  padding-left: 0;
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.manual-li {
  font-size: 13.5px;
  color: #57606a;
  line-height: 1.65;
  padding-left: 16px;
  position: relative;
}
.manual-li::before {
  content: '';
  position: absolute;
  left: 4px;
  top: 9px;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #d0d7de;
}

/* Stack info */
.manual-stack {
  margin-top: 36px;
  padding-top: 24px;
  border-top: 1px solid #e1e4e8;
}
.stack-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #8c959f;
  margin-bottom: 10px;
}
.stack-tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}
.manual-tag {
  font-size: 11px;
  font-weight: 600;
  padding: 3px 10px;
  border-radius: 20px;
  background: #f6f8fa;
  border: 1px solid #e1e4e8;
  color: #57606a;
  font-family: 'SFMono-Regular', 'Consolas', monospace;
}

/* Responsive */
@media (max-width: 720px) {
  .manual-root { padding: 20px 12px; }
  .manual-header { padding: 24px 20px 20px; }
  .manual-body { flex-direction: column; }
  .manual-toc {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #e1e4e8;
    position: static;
    padding: 16px 20px;
  }
  .toc-list { flex-direction: row; flex-wrap: wrap; gap: 4px; }
  .manual-content { padding: 20px; }
}
</style>
