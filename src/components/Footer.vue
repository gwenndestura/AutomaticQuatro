<script setup>
// Glob all images in assets so missing files don't break the build
const assets = import.meta.glob('../assets/*', { eager: true, import: 'default' })
const img = (filename) => assets[`../assets/${filename}`] ?? null

const developers = [
  {
    name: 'Janmikael A. Samson',
    role: 'Developer',
    img: 'jm.jpg',
    linkedin: 'https://www.linkedin.com/in/janmikael-samson-8691b23b9/',
    initials: 'JM'
  },
  {
    name: 'Princess Gwenn A. Destura',
    role: 'Developer',
    img: 'pg.png',
    linkedin: 'https://www.linkedin.com/in/princess-gwenn-destura',
    initials: 'GD'
  },
  {
    name: 'Edwin Carl R. Demonteverde Jr.',
    role: 'Developer',
    img: 'cj.png',
    linkedin: 'https://www.linkedin.com/in/edwin-carl-demonteverde-4602923b7/',
    initials: 'EC'
  },
  {
    name: 'John Carlo V. Salvidar',
    role: 'Developer',
    img: 'jc.jpg',
    linkedin: 'https://www.linkedin.com/in/john-carlo-salvidar-053114282/',
    initials: 'JS'
  }
]
</script>

<template>
  <footer class="footer">

    <div class="footer-inner">

      <!-- Top row -->
      <div class="footer-top">
        <div class="footer-brand">
          <svg width="22" height="22" viewBox="0 0 28 28" fill="none" style="flex-shrink:0">
            <circle cx="14" cy="14" r="13" stroke="#4caf50" stroke-width="2"/>
            <circle cx="14" cy="14" r="4" fill="#4caf50"/>
            <line x1="14" y1="1" x2="14" y2="7"  stroke="#4caf50" stroke-width="1.5" stroke-linecap="round"/>
            <line x1="14" y1="21" x2="14" y2="27" stroke="#4caf50" stroke-width="1.5" stroke-linecap="round"/>
            <line x1="1"  y1="14" x2="7"  y2="14" stroke="#4caf50" stroke-width="1.5" stroke-linecap="round"/>
            <line x1="21" y1="14" x2="27" y2="14" stroke="#4caf50" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <div>
            <div class="brand-name">Automata Simulator</div>
            <div class="brand-sub">Automata Theory &amp; Formal Languages · DLSUD</div>
          </div>
        </div>

        <div class="footer-note">
          Built as part of the Automata Theory &amp; Formal Languages course at<br/>
          <strong>De La Salle University – Dasmariñas</strong>
        </div>
      </div>

      <!-- Divider -->
      <div class="footer-divider"></div>

      <!-- Developers -->
      <div class="footer-devs-label">Developed by</div>
      <div class="footer-devs">
        <a
          v-for="dev in developers"
          :key="dev.name"
          :href="dev.linkedin"
          target="_blank"
          rel="noopener noreferrer"
          class="dev-card"
          :title="`${dev.name} on LinkedIn`"
        >
          <div class="dev-avatar">
            <img
              v-if="img(dev.img)"
              :src="img(dev.img)"
              :alt="dev.name"
              class="dev-photo"
            />
            <div v-else class="dev-initials">{{ dev.initials }}</div>
            <div class="linkedin-overlay">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="white">
                <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
              </svg>
            </div>
          </div>
          <div class="dev-name">{{ dev.name }}</div>
        </a>
      </div>

      <!-- Bottom -->
      <div class="footer-bottom">
        <span>© {{ new Date().getFullYear() }} Automata Theory Group · DLSUD</span>
      </div>

    </div>
  </footer>
</template>

<style scoped>
.footer {
  background: #0a1a0d;
  color: #7aaa82;
  font-family: 'Inter', 'Segoe UI', sans-serif;
  margin-top: 0;
}

.footer-inner {
  max-width: 1100px;
  margin: 0 auto;
  padding: 40px 24px 28px;
}

/* Top row */
.footer-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 24px;
  flex-wrap: wrap;
}
.footer-brand {
  display: flex;
  align-items: center;
  gap: 12px;
}
.brand-name {
  font-size: 15px;
  font-weight: 700;
  color: #e6f4ea;
}
.brand-sub {
  font-size: 11px;
  color: #3d6645;
  margin-top: 2px;
  letter-spacing: 0.03em;
}
.footer-note {
  font-size: 12px;
  color: #3d6645;
  text-align: right;
  line-height: 1.6;
}
.footer-note strong {
  color: #7aaa82;
}

/* Divider */
.footer-divider {
  height: 1px;
  background: #132a17;
  margin: 28px 0 24px;
}

/* Developers */
.footer-devs-label {
  font-size: 10px;
  font-weight: 700;
  color: #2e5435;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin-bottom: 16px;
}

.footer-devs {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
}

.dev-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  text-decoration: none;
  transition: transform 0.2s ease;
}
.dev-card:hover {
  transform: translateY(-4px);
}
.dev-card:hover .dev-avatar {
  box-shadow: 0 0 0 3px #4caf50;
}
.dev-card:hover .linkedin-overlay {
  opacity: 1;
}

.dev-avatar {
  position: relative;
  width: 72px;
  height: 72px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid #132a17;
  transition: box-shadow 0.2s ease;
  background: #132a17;
  flex-shrink: 0;
}
.dev-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}
.dev-initials {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 700;
  color: #4caf50;
  background: #0f2a1a;
}

.linkedin-overlay {
  position: absolute;
  inset: 0;
  background: rgba(10, 102, 194, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.2s ease;
  border-radius: 50%;
}

.dev-name {
  font-size: 11px;
  color: #7aaa82;
  text-align: center;
  max-width: 90px;
  line-height: 1.4;
  font-weight: 500;
}
.dev-card:hover .dev-name {
  color: #e6f4ea;
}

/* Bottom */
.footer-bottom {
  margin-top: 28px;
  padding-top: 16px;
  border-top: 1px solid #132a17;
  font-size: 11px;
  color: #2e5435;
  text-align: center;
}

@media (max-width: 640px) {
  .footer-top { flex-direction: column; }
  .footer-note { text-align: left; }
  .footer-devs { gap: 16px; }
  .dev-avatar { width: 60px; height: 60px; }
  .dev-name { font-size: 10px; max-width: 70px; }
}
</style>
