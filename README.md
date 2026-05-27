<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Night Vision Mode</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Rajdhani:wght@300;400;500;600;700&display=swap" rel="stylesheet">
<style>
/* ── Reset ─────────────────────────────────────────── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* ── Variables ─────────────────────────────────────── */
:root {
  --bg:          #06090f;
  --bg-card:     #0b1018;
  --bg-panel:    #0d1520;
  --border:      rgba(255,255,255,0.06);
  --border-hi:   rgba(255,255,255,0.12);
  --text-1:      #c8cdd8;
  --text-2:      #5a6478;
  --text-dim:    #2a3040;
  --nv-green:    #39ff7a;
  --nv-mid:      #1aff5a;
  --nv-glow:     rgba(57,255,122,0.15);
  --nv-glow-hi:  rgba(57,255,122,0.35);
  --red:         #ff4444;
  --font-mono:   'Share Tech Mono', monospace;
  --font-body:   'Rajdhani', sans-serif;
  --radius:      12px;
  --radius-lg:   20px;
}

html, body {
  height: 100%;
  background: var(--bg);
  color: var(--text-1);
  font-family: var(--font-body);
  font-size: 16px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  overflow-x: hidden;
}

/* ── App wrapper — filter applied here ─────────────── */
#app {
  min-height: 100vh;
  transition: filter 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
}

/* ── Scanline overlay (always present, more visible in NV) ── */
#app::before {
  content: '';
  position: fixed; inset: 0; z-index: 9000;
  pointer-events: none;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0,0,0,0.04) 2px,
    rgba(0,0,0,0.04) 4px
  );
  opacity: 0;
  transition: opacity 0.6s ease;
}
#app.nv-active::before { opacity: 1; }

/* ── Vignette overlay ──────────────────────────────── */
#app::after {
  content: '';
  position: fixed; inset: 0; z-index: 8999;
  pointer-events: none;
  background: radial-gradient(ellipse at center, transparent 40%, rgba(0,0,0,0.65) 100%);
  opacity: 0;
  transition: opacity 0.6s ease;
}
#app.nv-active::after { opacity: 1; }

/* ── Grid noise texture ────────────────────────────── */
body::before {
  content: '';
  position: fixed; inset: 0; z-index: -1;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.025'/%3E%3C/svg%3E");
  pointer-events: none;
}

/* ── Layout ────────────────────────────────────────── */
.page-wrap {
  max-width: 1100px;
  margin: 0 auto;
  padding: 40px 24px 80px;
}

/* ── Top bar ───────────────────────────────────────── */
.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 48px;
  gap: 16px;
  flex-wrap: wrap;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-icon {
  width: 42px; height: 42px;
  border-radius: 10px;
  background: linear-gradient(135deg, #0f1a12, #1a3320);
  border: 1px solid rgba(57,255,122,0.2);
  display: flex; align-items: center; justify-content: center;
  box-shadow: 0 0 16px rgba(57,255,122,0.12);
}

.brand-title {
  font-family: var(--font-mono);
  font-size: 1rem;
  letter-spacing: .12em;
  color: var(--text-1);
  text-transform: uppercase;
}
.brand-sub {
  font-size: 11px;
  letter-spacing: .1em;
  color: var(--text-2);
  text-transform: uppercase;
  margin-top: 2px;
}

/* ── Status indicator ──────────────────────────────── */
.status-pill {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 18px;
  border-radius: 100px;
  border: 1px solid var(--border-hi);
  background: var(--bg-card);
  font-family: var(--font-mono);
  font-size: 12px;
  letter-spacing: .12em;
  text-transform: uppercase;
  transition: all 0.4s ease;
}

.status-dot {
  width: 8px; height: 8px;
  border-radius: 50%;
  background: var(--text-2);
  transition: all 0.4s ease;
  flex-shrink: 0;
}

.status-text {
  color: var(--text-2);
  transition: color 0.4s ease;
}

.nv-active .status-pill {
  border-color: rgba(57,255,122,0.4);
  background: rgba(57,255,122,0.06);
  box-shadow: 0 0 20px rgba(57,255,122,0.15);
}
.nv-active .status-dot {
  background: var(--nv-green);
  box-shadow: 0 0 8px var(--nv-green);
  animation: blink 2s ease infinite;
}
.nv-active .status-text { color: var(--nv-green); }

@keyframes blink {
  0%,100% { opacity: 1; }
  50% { opacity: 0.4; }
}

/* ── Main grid ─────────────────────────────────────── */
.main-grid {
  display: grid;
  grid-template-columns: 1fr 340px;
  gap: 20px;
  align-items: start;
}

@media (max-width: 820px) {
  .main-grid { grid-template-columns: 1fr; }
}

/* ── Panel / Card ──────────────────────────────────── */
.panel {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 28px;
  position: relative;
  overflow: hidden;
}

.panel::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.06), transparent);
}

.panel-label {
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: .18em;
  text-transform: uppercase;
  color: var(--text-2);
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 8px;
}
.panel-label::after {
  content: '';
  flex: 1;
  height: 1px;
  background: var(--border);
}

/* ── Scene — the dark visual area ─────────────────── */
.scene {
  border-radius: var(--radius);
  background: #030507;
  border: 1px solid var(--border);
  min-height: 380px;
  position: relative;
  overflow: hidden;
  margin-bottom: 20px;
}

/* Stars */
.scene-stars {
  position: absolute; inset: 0;
  background-image:
    radial-gradient(1px 1px at 15% 20%, #ffffff18, transparent),
    radial-gradient(1px 1px at 70% 10%, #ffffff12, transparent),
    radial-gradient(1.5px 1.5px at 40% 60%, #ffffff15, transparent),
    radial-gradient(1px 1px at 80% 40%, #ffffff10, transparent),
    radial-gradient(1px 1px at 25% 75%, #ffffff14, transparent),
    radial-gradient(1px 1px at 55% 30%, #ffffff11, transparent),
    radial-gradient(1.5px 1.5px at 90% 70%, #ffffff16, transparent),
    radial-gradient(1px 1px at 10% 50%, #ffffff10, transparent),
    radial-gradient(1px 1px at 60% 85%, #ffffff13, transparent),
    radial-gradient(1px 1px at 35% 45%, #ffffff09, transparent);
}

/* Mountain silhouette */
.scene-mountains {
  position: absolute;
  bottom: 0; left: 0; right: 0;
}

/* Trees */
.scene-objects {
  position: absolute; inset: 0;
}

/* Moon */
.moon {
  position: absolute;
  top: 18%; right: 18%;
  width: 38px; height: 38px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #3a3f30, #1c2018);
  box-shadow: 0 0 0 1px rgba(255,255,255,0.04), 0 0 20px rgba(255,255,255,0.05);
}

/* Hidden figures */
.figure {
  position: absolute;
  transition: opacity 0.3s ease;
}

.figure-person {
  bottom: 28%; left: 22%;
  width: 12px; height: 28px;
  background: #0d1018;
  border-radius: 3px 3px 0 0;
  opacity: 0.55;
}
.figure-person::before {
  content: '';
  position: absolute;
  top: -10px; left: 50%;
  transform: translateX(-50%);
  width: 10px; height: 10px;
  border-radius: 50%;
  background: #0d1018;
}

.figure-animal {
  bottom: 25%; right: 28%;
  width: 22px; height: 12px;
  background: #0a0e14;
  border-radius: 50% 50% 0 0;
  opacity: 0.5;
}
.figure-animal::before {
  content: '';
  position: absolute;
  top: -6px; right: 3px;
  width: 7px; height: 9px;
  background: #0a0e14;
  border-radius: 3px 3px 0 0;
}

.figure-vehicle {
  bottom: 22%; left: 50%;
  transform: translateX(-50%);
  width: 40px; height: 16px;
  background: #080c12;
  border-radius: 3px;
  opacity: 0.45;
}
.figure-vehicle::before {
  content: '';
  position: absolute;
  top: -8px; left: 8px;
  width: 24px; height: 10px;
  background: #080c12;
  border-radius: 2px 2px 0 0;
}

/* Grid overlay (for night vision feel) */
.scene-grid {
  position: absolute; inset: 0;
  background-image:
    linear-gradient(rgba(57,255,122,0) 1px, transparent 1px),
    linear-gradient(90deg, rgba(57,255,122,0) 1px, transparent 1px);
  background-size: 40px 40px;
  opacity: 0;
  transition: opacity 0.6s ease;
  pointer-events: none;
}

.nv-active .scene-grid { opacity: 0.07; }

/* Crosshair */
.crosshair {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  width: 40px; height: 40px;
  opacity: 0;
  transition: opacity 0.4s ease;
  pointer-events: none;
}
.crosshair::before, .crosshair::after {
  content: '';
  position: absolute;
  background: rgba(57,255,122,0.6);
}
.crosshair::before { width: 1px; height: 100%; left: 50%; }
.crosshair::after  { width: 100%; height: 1px; top: 50%; }
.crosshair-ring {
  position: absolute; inset: 4px;
  border: 1px solid rgba(57,255,122,0.4);
  border-radius: 50%;
}

.nv-active .crosshair { opacity: 1; }

/* Corner brackets */
.corner { position: absolute; width: 14px; height: 14px; opacity: 0; transition: opacity 0.4s ease; }
.corner::before, .corner::after { content: ''; position: absolute; background: rgba(57,255,122,0.7); }
.corner-tl { top: 12px; left: 12px; }
.corner-tl::before { width: 2px; height: 100%; top: 0; left: 0; }
.corner-tl::after  { width: 100%; height: 2px; top: 0; left: 0; }
.corner-tr { top: 12px; right: 12px; }
.corner-tr::before { width: 2px; height: 100%; top: 0; right: 0; }
.corner-tr::after  { width: 100%; height: 2px; top: 0; right: 0; }
.corner-bl { bottom: 12px; left: 12px; }
.corner-bl::before { width: 2px; height: 100%; bottom: 0; left: 0; }
.corner-bl::after  { width: 100%; height: 2px; bottom: 0; left: 0; }
.corner-br { bottom: 12px; right: 12px; }
.corner-br::before { width: 2px; height: 100%; bottom: 0; right: 0; }
.corner-br::after  { width: 100%; height: 2px; bottom: 0; right: 0; }

.nv-active .corner { opacity: 1; }

/* HUD text */
.hud-text {
  position: absolute;
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: .1em;
  color: rgba(57,255,122,0.7);
  opacity: 0;
  transition: opacity 0.4s ease;
  pointer-events: none;
}
.hud-tl { top: 14px; left: 14px; }
.hud-tr { top: 14px; right: 14px; text-align: right; }
.hud-bl { bottom: 14px; left: 14px; }
.hud-br { bottom: 14px; right: 14px; text-align: right; }

.nv-active .hud-text { opacity: 1; }

/* Scene caption */
.scene-caption {
  font-size: 12px;
  letter-spacing: .06em;
  color: var(--text-2);
  text-align: center;
  padding: 12px;
  font-family: var(--font-mono);
}

/* ── Info cards ────────────────────────────────────── */
.info-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

@media (max-width: 500px) { .info-grid { grid-template-columns: 1fr; } }

.info-card {
  background: var(--bg-panel);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 18px;
  transition: border-color 0.4s ease;
}

.nv-active .info-card {
  border-color: rgba(57,255,122,0.12);
}

.info-card-label {
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: .14em;
  text-transform: uppercase;
  color: var(--text-2);
  margin-bottom: 8px;
}

.info-card-value {
  font-family: var(--font-mono);
  font-size: 1.6rem;
  font-weight: 700;
  color: var(--text-dim);
  transition: color 0.4s ease;
  line-height: 1;
}

.nv-active .info-card-value { color: var(--nv-green); text-shadow: 0 0 12px rgba(57,255,122,0.4); }

.info-card-sub {
  font-size: 11px;
  color: var(--text-2);
  margin-top: 4px;
  letter-spacing: .05em;
}

/* ── Control panel (sidebar) ────────────────────────── */
.ctrl-panel {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Toggle button */
.nv-toggle-btn {
  width: 100%;
  padding: 18px 24px;
  border-radius: var(--radius-lg);
  border: 1px solid var(--border-hi);
  background: var(--bg-card);
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 14px;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.nv-toggle-btn::before {
  content: '';
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at center, var(--nv-glow), transparent 70%);
  opacity: 0;
  transition: opacity 0.4s ease;
}

.nv-toggle-btn:hover { border-color: var(--border-hi); transform: translateY(-1px); }
.nv-toggle-btn:hover::before { opacity: 0.5; }

.nv-toggle-btn.active {
  border-color: rgba(57,255,122,0.5);
  background: rgba(57,255,122,0.06);
  box-shadow: 0 0 30px rgba(57,255,122,0.2), inset 0 0 20px rgba(57,255,122,0.05);
}
.nv-toggle-btn.active::before { opacity: 1; }

.toggle-icon-wrap {
  width: 44px; height: 44px;
  border-radius: 10px;
  background: #0f1520;
  border: 1px solid var(--border);
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
  transition: all 0.4s ease;
}

.nv-toggle-btn.active .toggle-icon-wrap {
  background: rgba(57,255,122,0.1);
  border-color: rgba(57,255,122,0.3);
  box-shadow: 0 0 12px rgba(57,255,122,0.2);
}

.nv-toggle-btn.active .toggle-icon { stroke: var(--nv-green); filter: drop-shadow(0 0 4px var(--nv-green)); }

.toggle-label {
  flex: 1;
  text-align: left;
}

.toggle-label-main {
  font-family: var(--font-mono);
  font-size: 13px;
  letter-spacing: .1em;
  text-transform: uppercase;
  color: var(--text-1);
  display: block;
}

.toggle-label-sub {
  font-size: 11px;
  color: var(--text-2);
  letter-spacing: .05em;
  margin-top: 2px;
  display: block;
}

.toggle-switch {
  width: 42px; height: 24px;
  border-radius: 100px;
  background: var(--bg-panel);
  border: 1px solid var(--border-hi);
  position: relative;
  transition: all 0.3s ease;
  flex-shrink: 0;
}

.toggle-switch::after {
  content: '';
  position: absolute;
  top: 3px; left: 3px;
  width: 16px; height: 16px;
  border-radius: 50%;
  background: var(--text-2);
  transition: all 0.3s ease;
}

.nv-toggle-btn.active .toggle-switch {
  background: rgba(57,255,122,0.2);
  border-color: var(--nv-green);
}
.nv-toggle-btn.active .toggle-switch::after {
  left: calc(100% - 19px);
  background: var(--nv-green);
  box-shadow: 0 0 8px var(--nv-green);
}

/* ── Sliders panel ─────────────────────────────────── */
.sliders-panel {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
}

.slider-block { margin-bottom: 24px; }
.slider-block:last-child { margin-bottom: 0; }

.slider-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.slider-name {
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: .12em;
  text-transform: uppercase;
  color: var(--text-2);
}

.slider-val {
  font-family: var(--font-mono);
  font-size: 13px;
  color: var(--text-1);
  min-width: 44px;
  text-align: right;
  transition: color 0.3s ease;
}

.nv-active .slider-val { color: var(--nv-green); }

.slider-track {
  position: relative;
  height: 4px;
  background: var(--bg-panel);
  border-radius: 2px;
  border: 1px solid var(--border);
}

.slider-fill {
  position: absolute;
  top: 0; left: 0;
  height: 100%;
  border-radius: 2px;
  background: var(--text-dim);
  transition: background 0.4s ease;
  pointer-events: none;
}

.nv-active .slider-fill { background: var(--nv-green); box-shadow: 0 0 8px rgba(57,255,122,0.5); }

input[type=range] {
  position: absolute;
  inset: -6px 0;
  width: 100%;
  height: calc(100% + 12px);
  opacity: 0;
  cursor: pointer;
  margin: 0;
}

/* ── Preset buttons ────────────────────────────────── */
.presets-panel {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 20px;
}

.presets-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.preset-btn {
  padding: 10px 12px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: var(--bg-panel);
  color: var(--text-2);
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: .1em;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: center;
}

.preset-btn:hover {
  border-color: var(--border-hi);
  color: var(--text-1);
  background: rgba(255,255,255,0.04);
}

.preset-btn.active-preset {
  border-color: rgba(57,255,122,0.4);
  background: rgba(57,255,122,0.06);
  color: var(--nv-green);
}

/* ── Filter readout ────────────────────────────────── */
.filter-readout {
  background: var(--bg-panel);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 16px;
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--text-2);
  line-height: 1.8;
  letter-spacing: .06em;
  word-break: break-all;
}

.filter-readout span {
  color: var(--text-1);
  transition: color 0.3s ease;
}

.nv-active .filter-readout span { color: var(--nv-green); }

/* ── Bottom info bar ───────────────────────────────── */
.bottom-bar {
  margin-top: 24px;
  padding: 20px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.bottom-bar-icon {
  flex-shrink: 0;
}

.bottom-bar-text {
  flex: 1;
  font-size: 13px;
  color: var(--text-2);
  letter-spacing: .03em;
  min-width: 200px;
}

.bottom-bar-text strong { color: var(--text-1); font-weight: 600; }

.kbd {
  display: inline-flex;
  align-items: center;
  padding: 2px 8px;
  border-radius: 4px;
  background: var(--bg-panel);
  border: 1px solid var(--border-hi);
  font-family: var(--font-mono);
  font-size: 11px;
  color: var(--text-1);
}

/* ── Animations ────────────────────────────────────── */
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.page-wrap > * {
  animation: fadeUp 0.5s ease both;
}
.page-wrap > *:nth-child(1) { animation-delay: 0.05s; }
.page-wrap > *:nth-child(2) { animation-delay: 0.12s; }
.page-wrap > *:nth-child(3) { animation-delay: 0.2s; }
.page-wrap > *:nth-child(4) { animation-delay: 0.28s; }

@media (max-width: 600px) {
  .page-wrap { padding: 24px 16px 60px; }
  .topbar { margin-bottom: 28px; }
  .panel { padding: 20px; }
}
</style>
</head>
<body>
<div id="app">
<div class="page-wrap">

  <!-- ── Top bar ────────────────────────────────────── -->
  <div class="topbar">
    <div class="brand">
      <div class="brand-icon">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="#39ff7a" stroke-width="1.5">
          <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/>
          <circle cx="12" cy="12" r="3"/>
          <path d="M12 9v-2M12 17v2M9 12H7M17 12h2" stroke-width="1"/>
        </svg>
      </div>
      <div>
        <div class="brand-title">NightVision OS</div>
        <div class="brand-sub">Tactical Visual Enhancement</div>
      </div>
    </div>
    <div class="status-pill">
      <div class="status-dot"></div>
      <span class="status-text" id="status-text">SYSTEM STANDBY</span>
    </div>
  </div>

  <!-- ── Main grid ──────────────────────────────────── -->
  <div class="main-grid">

    <!-- Left: scene + info cards -->
    <div>
      <!-- Scene -->
      <div class="panel" style="padding:0;overflow:hidden;margin-bottom:16px;">
        <div class="scene">
          <div class="scene-stars"></div>

          <!-- Moon -->
          <div class="moon"></div>

          <!-- Mountains SVG -->
          <div class="scene-mountains">
            <svg viewBox="0 0 800 200" xmlns="http://www.w3.org/2000/svg" style="display:block;width:100%">
              <polygon points="0,200 120,60 240,120 380,20 520,90 640,50 760,110 800,80 800,200" fill="#08090e"/>
              <polygon points="0,200 80,110 160,150 300,80 440,130 560,95 680,140 800,120 800,200" fill="#0a0b10" opacity=".8"/>
          
