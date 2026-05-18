<script setup>
const sections = [
  {
    id: 'introduction',
    title: 'Introduction',
    content: `The Automata Simulator is an educational tool designed to bridge the gap between theoretical computation and visual representation. It allows users to input regular expressions and observe how they translate into mathematical models — including Deterministic Finite Automata (DFA), Context-Free Grammars (CFG), and Pushdown Automata (PDA).`
  },
  {
    id: 'getting-started',
    title: 'Getting Started',
    subsections: [
      {
        title: 'System Requirements',
        body: 'The simulator runs on any modern browser (Chrome, Firefox, or Edge). It is optimized for desktop use to provide the best experience for complex automata visualizations.'
      },
      {
        title: 'Basic Workflow',
        items: [
          { label: 'Select a Problem', text: 'Choose between Problem 1 or Problem 2 from the left sidebar. Each problem defines a distinct regular expression.' },
          { label: 'Review the Regex', text: 'The selected regular expression is displayed below the problem tabs for reference.' },
          { label: 'Enter a Test String', text: 'Type a string into one of the five input fields. A colored indicator shows whether the string is valid or invalid for the regex.' },
          { label: 'Run the Simulation', text: 'Click Run to animate the automata. The visualization panel on the right will show the step-by-step state transitions.' },
        ]
      }
    ]
  },
  {
    id: 'features',
    title: 'Core Features',
    subsections: [
      {
        title: 'Deterministic Finite Automata (DFA)',
        body: 'The DFA module converts the selected regular expression into a state-transition diagram.',
        items: [
          { label: 'States', text: 'Represented by circles. The start state (blue), non-final states (yellow), accepting states (green), and trap/dead states (red) are visually distinguished.' },
          { label: 'Transitions', text: 'Labeled arrows indicate movement between states based on input symbols.' },
          { label: 'Interactive Playback', text: 'Click Run on any test string to animate the DFA step by step. The current state and the character being read are highlighted in real time.' },
        ]
      },
      {
        title: 'Context-Free Grammar (CFG)',
        body: 'This module displays the formal production rules corresponding to the selected regular expression.',
        items: [
          { label: 'Production Rules', text: 'Each rule is listed with its left-hand side non-terminal, the derivation arrow, and the right-hand alternatives.' },
          { label: 'Color Coding', text: 'Non-terminals are shown in amber; terminals are shown in green for quick visual scanning.' },
        ]
      },
      {
        title: 'Pushdown Automata (PDA)',
        body: 'The PDA module models the same language using a stack-based automaton.',
        items: [
          { label: 'State Shapes', text: 'States are drawn as oblongs. Accepting states are marked with a double border.' },
          { label: 'Transition Labels', text: 'Each arrow is labeled in the format: input, pop → push. The symbol Δ denotes the blank stack symbol.' },
        ]
      }
    ]
  },
  {
    id: 'faq',
    title: 'Troubleshooting',
    items: [
      { label: 'String rejected unexpectedly', text: 'Verify the string against the regex shown in the sidebar. Trace the DFA diagram to identify which state causes rejection.' },
      { label: 'Visualization not appearing', text: 'Ensure you have clicked Run. If the DFA tab is active, the animation starts automatically on Run.' },
      { label: 'Slow rendering', text: 'For complex strings, the DFA animation may take a moment. Ensure your browser is up to date.' },
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
          href="https://docs.google.com/document/d/1ny4ccAVK-HR_crp0zBQ0JsBoIXSl3KXw1x4W2Kmc4O8/edit?usp=sharing"
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
