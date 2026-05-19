<template>
  <div class="manual-root">
    <div class="manual-bar">
      <span class="manual-label">User Manual</span>
      <a
        href="https://docs.google.com/document/d/1sWP7bERrjxlXzy6-0Ccm8Wy5c0E_gGDu3ShWwXFDZGM/edit?usp=sharing"
        target="_blank"
        rel="noopener noreferrer"
        class="manual-open"
      >
        Open in Google Docs
        <svg width="12" height="12" viewBox="0 0 14 14" fill="none" aria-hidden="true">
          <path d="M3 11L11 3M11 3H6M11 3v5" stroke="currentColor" stroke-width="1.8"
            stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </a>
    </div>
    <iframe
      src="https://docs.google.com/document/d/1sWP7bERrjxlXzy6-0Ccm8Wy5c0E_gGDu3ShWwXFDZGM/preview"
      class="manual-frame"
      allow="autoplay"
      title="User Manual"
    />
  </div>
</template>

<style scoped>
.manual-root {
  display: flex;
  flex-direction: column;
  height: calc(100vh - 60px);
  background: #f6f8fa;
}

.manual-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 24px;
  background: #ffffff;
  border-bottom: 1px solid #e1e4e8;
  flex-shrink: 0;
}

.manual-label {
  font-size: 13px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.01em;
}

.manual-open {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  font-size: 12px;
  font-weight: 600;
  color: #0969da;
  text-decoration: none;
  transition: color 0.15s;
}
.manual-open:hover { color: #0550ae; text-decoration: underline; }

.manual-frame {
  flex: 1;
  width: 100%;
  border: none;
}
</style>
