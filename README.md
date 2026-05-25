<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Dual Axis Solar Tracker — Logic Gate Design</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet"/>
<style>
  :root {
    --sun: #FFB627;
    --sun-glow: #FF8C00;
    --sky: #0A1628;
    --sky-mid: #0D2137;
    --panel: #1A3A5C;
    --accent: #00E5FF;
    --accent2: #FF4F5E;
    --accent3: #39FF14;
    --text: #E8F4FD;
    --text-muted: #7AACCC;
    --border: rgba(0,229,255,0.2);
    --card: rgba(13,33,55,0.85);
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--sky);
    color: var(--text);
    font-family: 'Syne', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* ── ANIMATED SKY BACKGROUND ── */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background:
      radial-gradient(ellipse 80% 40% at 50% -10%, rgba(255,182,39,0.18) 0%, transparent 60%),
      radial-gradient(ellipse 60% 60% at 80% 100%, rgba(0,229,255,0.07) 0%, transparent 60%),
      linear-gradient(180deg, #050D1A 0%, #0A1628 40%, #0D2137 100%);
    z-index: -2;
  }

  /* Sun */
  .sun-orb {
    position: fixed;
    top: -80px; left: 50%;
    transform: translateX(-50%);
    width: 200px; height: 200px;
    border-radius: 50%;
    background: radial-gradient(circle, #FFF5C0 0%, #FFB627 40%, #FF8C00 70%, transparent 100%);
    filter: blur(2px);
    opacity: 0.7;
    animation: sun-pulse 6s ease-in-out infinite;
    z-index: -1;
  }
  @keyframes sun-pulse {
    0%,100% { transform: translateX(-50%) scale(1); opacity: 0.7; }
    50% { transform: translateX(-50%) scale(1.08); opacity: 0.9; }
  }

  /* ── GRID LINES ── */
  body::after {
    content: '';
    position: fixed; inset: 0;
    background-image:
      linear-gradient(rgba(0,229,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    z-index: -1;
  }

  /* ── HEADER ── */
  header {
    text-align: center;
    padding: 100px 24px 60px;
    position: relative;
  }

  .badge {
    display: inline-block;
    background: rgba(0,229,255,0.1);
    border: 1px solid var(--accent);
    color: var(--accent);
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    letter-spacing: 3px;
    padding: 5px 14px;
    border-radius: 20px;
    text-transform: uppercase;
    margin-bottom: 24px;
  }

  h1 {
    font-size: clamp(2.4rem, 6vw, 4.5rem);
    font-weight: 800;
    line-height: 1.05;
    letter-spacing: -1px;
    background: linear-gradient(135deg, #FFFFFF 0%, #FFB627 50%, #FF8C00 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin-bottom: 20px;
  }

  .subtitle {
    color: var(--text-muted);
    font-size: 1.05rem;
    max-width: 580px;
    margin: 0 auto 36px;
    line-height: 1.7;
    font-weight: 400;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    justify-content: center;
    margin-bottom: 40px;
  }
  .tag {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    padding: 4px 12px;
    border-radius: 4px;
    border: 1px solid;
  }
  .tag.g { color: var(--accent3); border-color: rgba(57,255,20,0.3); background: rgba(57,255,20,0.06); }
  .tag.b { color: var(--accent); border-color: rgba(0,229,255,0.3); background: rgba(0,229,255,0.06); }
  .tag.r { color: var(--accent2); border-color: rgba(255,79,94,0.3); background: rgba(255,79,94,0.06); }
  .tag.y { color: var(--sun); border-color: rgba(255,182,39,0.3); background: rgba(255,182,39,0.06); }

  /* ── OVERVIEW DIAGRAM ── */
  .diagram-section {
    max-width: 900px;
    margin: 0 auto 60px;
    padding: 0 24px;
  }

  .diagram-box {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 40px;
    backdrop-filter: blur(12px);
  }

  .section-label {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: var(--accent);
    letter-spacing: 3px;
    text-transform: uppercase;
    margin-bottom: 24px;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .section-label::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--border);
  }

  /* ── SYSTEM DIAGRAM SVG ── */
  .sys-diagram {
    width: 100%;
    max-width: 780px;
    margin: 0 auto;
    display: block;
  }

  /* ── MAIN CONTENT ── */
  main {
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 24px 100px;
  }

  /* ── SECTION HEADERS ── */
  .sec-title {
    font-size: 1.6rem;
    font-weight: 800;
    margin-bottom: 24px;
    color: #fff;
    display: flex;
    align-items: center;
    gap: 14px;
  }
  .sec-title span { font-size: 1.3rem; }

  /* ── GRID LAYOUTS ── */
  .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 60px; }
  .grid-3 { display: grid; grid-template-columns: repeat(3,1fr); gap: 20px; margin-bottom: 60px; }
  .grid-4 { display: grid; grid-template-columns: repeat(4,1fr); gap: 16px; margin-bottom: 60px; }

  @media (max-width: 768px) {
    .grid-2, .grid-3, .grid-4 { grid-template-columns: 1fr; }
  }

  /* ── CARDS ── */
  .card {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 24px;
    backdrop-filter: blur(10px);
    transition: border-color 0.2s, transform 0.2s;
  }
  .card:hover {
    border-color: rgba(0,229,255,0.5);
    transform: translateY(-2px);
  }

  .card-icon {
    font-size: 2rem;
    margin-bottom: 12px;
    display: block;
  }

  .card h3 {
    font-size: 1rem;
    font-weight: 700;
    margin-bottom: 8px;
    color: var(--accent);
  }

  .card p, .card li {
    font-size: 0.875rem;
    color: var(--text-muted);
    line-height: 1.7;
  }

  .card ul { padding-left: 16px; }

  /* ── SENSOR MAP ── */
  .sensor-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 4px;
    width: 200px;
    margin: 0 auto 20px;
  }
  .sensor-cell {
    aspect-ratio: 1;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-family: 'Space Mono', monospace;
    font-size: 13px;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.2s;
    border: 2px solid rgba(255,255,255,0.1);
    position: relative;
  }
  .sensor-cell.active {
    background: rgba(255,182,39,0.25);
    border-color: var(--sun);
    box-shadow: 0 0 20px rgba(255,182,39,0.4);
  }
  .sensor-cell.inactive {
    background: rgba(30,50,80,0.6);
  }
  .sensor-cell .label { font-size: 9px; color: var(--text-muted); margin-top: 2px; }
  .sensor-cell .dot {
    width: 10px; height: 10px; border-radius: 50%;
    background: var(--sun);
    margin-bottom: 4px;
    box-shadow: 0 0 8px var(--sun);
  }
  .sensor-cell.inactive .dot { background: #2A4A6A; box-shadow: none; }

  /* ── TRUTH TABLE ── */
  .truth-table-wrap { overflow-x: auto; margin-bottom: 60px; }
  table {
    width: 100%;
    border-collapse: collapse;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
  }
  th {
    background: rgba(0,229,255,0.1);
    color: var(--accent);
    padding: 10px 14px;
    text-align: center;
    border: 1px solid var(--border);
    font-size: 11px;
    letter-spacing: 1px;
    white-space: nowrap;
  }
  td {
    padding: 8px 14px;
    border: 1px solid rgba(0,229,255,0.08);
    text-align: center;
    color: var(--text-muted);
  }
  tr:hover td { background: rgba(0,229,255,0.04); }
  .one { color: var(--accent3) !important; font-weight: 700; }
  .zero { color: #2A4A6A !important; }
  .act { color: var(--sun) !important; font-weight: 700; background: rgba(255,182,39,0.06); }

  /* ── COMPONENT TABLE ── */
  .comp-table { width: 100%; border-collapse: collapse; margin-bottom: 60px; }
  .comp-table th {
    background: rgba(255,182,39,0.1);
    color: var(--sun);
    text-align: left;
    padding: 12px 16px;
    border: 1px solid rgba(255,182,39,0.2);
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    letter-spacing: 1px;
  }
  .comp-table td {
    padding: 10px 16px;
    border: 1px solid rgba(255,182,39,0.08);
    text-align: left;
    font-size: 13px;
    color: var(--text-muted);
  }
  .comp-table tr:hover td { background: rgba(255,182,39,0.03); }
  .comp-table .ref { color: var(--accent); font-family: 'Space Mono', monospace; font-size: 12px; }
  .comp-table .ic { color: var(--sun); font-family: 'Space Mono', monospace; }

  /* ── CODE BLOCK ── */
  .code-block {
    background: rgba(0,0,0,0.4);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 24px;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    color: #A8D8EA;
    line-height: 1.9;
    overflow-x: auto;
    margin-bottom: 24px;
    position: relative;
  }
  .code-block .kw { color: var(--accent2); }
  .code-block .cm { color: #4A7A9B; }
  .code-block .val { color: var(--accent3); }
  .code-block .sig { color: var(--sun); }

  /* ── L293D DIAGRAM ── */
  .driver-pins {
    display: grid;
    grid-template-columns: 1fr auto 1fr;
    align-items: center;
    gap: 0;
    max-width: 460px;
    margin: 0 auto 20px;
    font-family: 'Space Mono', monospace;
    font-size: 11px;
  }
  .driver-pins .left { text-align: right; color: var(--text-muted); }
  .driver-pins .right { text-align: left; color: var(--text-muted); }
  .driver-chip {
    background: rgba(0,229,255,0.08);
    border: 2px solid var(--accent);
    border-radius: 8px;
    padding: 16px 20px;
    text-align: center;
    color: var(--accent);
    font-weight: 700;
    font-size: 13px;
    margin: 0 10px;
    min-width: 80px;
  }
  .pin-row {
    display: flex;
    justify-content: flex-end;
    align-items: center;
    gap: 6px;
    padding: 3px 0;
    color: var(--text-muted);
  }
  .pin-row.r { justify-content: flex-start; }
  .pin-dot {
    width: 8px; height: 8px; border-radius: 50%;
    background: var(--accent);
    flex-shrink: 0;
  }
  .pin-dot.m { background: var(--sun); }
  .pin-dot.g { background: var(--accent3); }
  .pin-dot.r { background: var(--accent2); }

  /* ── LOGIC EQUATIONS ── */
  .eq-box {
    background: rgba(0,0,0,0.35);
    border-left: 3px solid var(--sun);
    padding: 16px 20px;
    margin-bottom: 12px;
    border-radius: 0 8px 8px 0;
    font-family: 'Space Mono', monospace;
    font-size: 13px;
    line-height: 1.8;
  }
  .eq-box .axis { color: var(--sun); font-size: 11px; letter-spacing: 2px; text-transform: uppercase; margin-bottom: 6px; }
  .eq-box .eq { color: var(--text); }
  .eq-box .eq .var { color: var(--accent); }
  .eq-box .eq .bar { text-decoration: overline; }

  /* ── REPO STRUCTURE ── */
  .tree {
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    line-height: 2;
    color: var(--text-muted);
    background: rgba(0,0,0,0.35);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 24px 28px;
    margin-bottom: 60px;
  }
  .tree .folder { color: var(--sun); }
  .tree .file { color: var(--accent); }
  .tree .desc { color: #3A6A8A; }

  /* ── FUTURE IMPROVEMENTS ── */
  .todo-list { list-style: none; margin-bottom: 60px; }
  .todo-list li {
    display: flex;
    align-items: flex-start;
    gap: 12px;
    padding: 12px 16px;
    border-bottom: 1px solid var(--border);
    font-size: 0.9rem;
    color: var(--text-muted);
    transition: background 0.2s;
  }
  .todo-list li:hover { background: rgba(0,229,255,0.04); }
  .todo-list .checkbox {
    width: 18px; height: 18px;
    border: 2px solid var(--border);
    border-radius: 4px;
    flex-shrink: 0;
    margin-top: 2px;
  }
  .todo-list .title { color: var(--text); font-weight: 600; display: block; font-size: 0.88rem; }

  /* ── INTERACTIVE SIMULATOR ── */
  .sim-area {
    background: var(--card);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 36px;
    margin-bottom: 60px;
  }
  .sim-sensors {
    display: flex;
    justify-content: center;
    gap: 16px;
    margin-bottom: 28px;
    flex-wrap: wrap;
  }
  .sim-btn {
    background: rgba(30,50,80,0.8);
    border: 2px solid rgba(255,255,255,0.1);
    color: var(--text-muted);
    border-radius: 10px;
    padding: 14px 20px;
    cursor: pointer;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    text-align: center;
    transition: all 0.15s;
    min-width: 80px;
  }
  .sim-btn:hover { border-color: rgba(255,182,39,0.5); }
  .sim-btn.on {
    background: rgba(255,182,39,0.2);
    border-color: var(--sun);
    color: var(--sun);
    box-shadow: 0 0 16px rgba(255,182,39,0.25);
  }
  .sim-btn .ldr { font-size: 1.4rem; display: block; margin-bottom: 4px; }
  .sim-btn .s-name { font-size: 10px; letter-spacing: 1px; }

  .motor-display {
    display: flex;
    justify-content: center;
    gap: 24px;
    flex-wrap: wrap;
    margin-bottom: 16px;
  }
  .motor-card {
    background: rgba(0,0,0,0.35);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 20px 28px;
    text-align: center;
    min-width: 160px;
  }
  .motor-label {
    font-family: 'Space Mono', monospace;
    font-size: 10px;
    color: var(--text-muted);
    letter-spacing: 2px;
    text-transform: uppercase;
    margin-bottom: 12px;
  }
  .motor-arrow {
    font-size: 2rem;
    transition: all 0.3s;
    filter: grayscale(1);
    opacity: 0.3;
  }
  .motor-arrow.active { filter: none; opacity: 1; }
  .motor-state {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    margin-top: 8px;
    padding: 3px 10px;
    border-radius: 4px;
    background: rgba(0,0,0,0.3);
    color: var(--text-muted);
    transition: all 0.3s;
  }
  .motor-state.running { color: var(--accent3); background: rgba(57,255,20,0.1); }

  .sim-result {
    text-align: center;
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    color: var(--text-muted);
    padding: 12px;
    background: rgba(0,0,0,0.2);
    border-radius: 8px;
    margin-top: 12px;
    min-height: 38px;
    transition: all 0.3s;
  }

  /* ── FOOTER ── */
  footer {
    text-align: center;
    padding: 40px 24px;
    border-top: 1px solid var(--border);
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: #2A4A6A;
    letter-spacing: 2px;
  }

  /* ── SECTION DIVIDER ── */
  .divider { height: 1px; background: var(--border); margin: 60px 0; }

  h2 { font-size: 1.5rem; font-weight: 800; margin-bottom: 20px; }
</style>
</head>
<body>

<div class="sun-orb"></div>

<!-- HEADER -->
<header>
  <div class="badge">☀ Hardware Project</div>
  <h1>Dual Axis<br/>Solar Tracker</h1>
  <p class="subtitle">A microcontroller-free, logic-gate-only solar tracking system using LDR sensors, combinational AND/OR/NOT gates, and an L293D H-bridge to drive two DC motors on azimuth and elevation axes.</p>
  <div class="tags">
    <span class="tag g">Logic Gates</span>
    <span class="tag b">74HC21 · 4073 · 4075</span>
    <span class="tag r">L293D Motor Driver</span>
    <span class="tag y">Proteus Simulation</span>
    <span class="tag b">Dual Axis</span>
    <span class="tag g">No Microcontroller</span>
  </div>
</header>

<!-- SYSTEM OVERVIEW DIAGRAM -->
<div class="diagram-section">
  <div class="diagram-box">
    <div class="section-label">System Architecture</div>
    <svg class="sys-diagram" viewBox="0 0 780 260" fill="none" xmlns="http://www.w3.org/2000/svg">
      <!-- Sun -->
      <circle cx="390" cy="30" r="22" fill="#FFB627" opacity="0.9"/>
      <circle cx="390" cy="30" r="30" fill="#FFB627" opacity="0.15"/>
      <circle cx="390" cy="30" r="38" fill="#FFB627" opacity="0.07"/>
      <!-- Sun rays -->
      <g stroke="#FFB627" stroke-width="2" opacity="0.6">
        <line x1="390" y1="0" x2="390" y2="-8" transform="translate(0,8)"/>
        <line x1="362" y1="8" x2="356" y2="2"/>
        <line x1="418" y1="8" x2="424" y2="2"/>
        <line x1="352" y1="30" x2="344" y2="30"/>
        <line x1="428" y1="30" x2="436" y2="30"/>
      </g>

      <!-- Block: Sensors -->
      <rect x="30" y="80" width="140" height="100" rx="10" fill="rgba(13,33,55,0.9)" stroke="rgba(0,229,255,0.35)" stroke-width="1.5"/>
      <text x="100" y="104" text-anchor="middle" fill="#00E5FF" font-family="Space Mono" font-size="9" letter-spacing="2">SENSORS</text>
      <!-- 4 LDR boxes -->
      <rect x="46" y="112" width="40" height="26" rx="5" fill="rgba(255,182,39,0.15)" stroke="#FFB627" stroke-width="1"/>
      <text x="66" y="123" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="8">S1</text>
      <text x="66" y="133" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">Top-L</text>
      <rect x="94" y="112" width="40" height="26" rx="5" fill="rgba(255,182,39,0.15)" stroke="#FFB627" stroke-width="1"/>
      <text x="114" y="123" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="8">S2</text>
      <text x="114" y="133" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">Top-R</text>
      <rect x="46" y="144" width="40" height="26" rx="5" fill="rgba(255,182,39,0.1)" stroke="rgba(255,182,39,0.4)" stroke-width="1"/>
      <text x="66" y="155" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="8">S3</text>
      <text x="66" y="165" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">Bot-L</text>
      <rect x="94" y="144" width="40" height="26" rx="5" fill="rgba(255,182,39,0.1)" stroke="rgba(255,182,39,0.4)" stroke-width="1"/>
      <text x="114" y="155" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="8">S4</text>
      <text x="114" y="165" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">Bot-R</text>

      <!-- Arrow: Sensors → NOT -->
      <line x1="170" y1="130" x2="195" y2="130" stroke="#00E5FF" stroke-width="1.5" stroke-dasharray="4,2"/>
      <polygon points="195,126 203,130 195,134" fill="#00E5FF"/>

      <!-- Block: NOT Gates -->
      <rect x="204" y="100" width="80" height="60" rx="8" fill="rgba(13,33,55,0.9)" stroke="rgba(0,229,255,0.3)" stroke-width="1.5"/>
      <text x="244" y="122" text-anchor="middle" fill="#00E5FF" font-family="Space Mono" font-size="8" letter-spacing="1">NOT</text>
      <text x="244" y="134" text-anchor="middle" fill="#00E5FF" font-family="Space Mono" font-size="8" letter-spacing="1">GATES</text>
      <text x="244" y="148" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Space Mono" font-size="7">74HC04 ×4</text>

      <!-- Arrow: NOT → AND -->
      <line x1="284" y1="130" x2="309" y2="130" stroke="#00E5FF" stroke-width="1.5" stroke-dasharray="4,2"/>
      <polygon points="309,126 317,130 309,134" fill="#00E5FF"/>

      <!-- Block: AND Gate Network -->
      <rect x="318" y="80" width="120" height="100" rx="8" fill="rgba(13,33,55,0.9)" stroke="rgba(255,182,39,0.4)" stroke-width="1.5"/>
      <text x="378" y="104" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="9" letter-spacing="1">AND LAYER</text>
      <text x="378" y="120" text-anchor="middle" fill="rgba(255,182,39,0.7)" font-family="Space Mono" font-size="7">74HC21</text>
      <text x="378" y="132" text-anchor="middle" fill="rgba(255,182,39,0.7)" font-family="Space Mono" font-size="7">4073 (×2)</text>
      <text x="378" y="156" text-anchor="middle" fill="rgba(255,182,39,0.5)" font-family="Space Mono" font-size="7">Minterm</text>
      <text x="378" y="168" text-anchor="middle" fill="rgba(255,182,39,0.5)" font-family="Space Mono" font-size="7">generation</text>

      <!-- Arrow: AND → OR -->
      <line x1="438" y1="130" x2="463" y2="130" stroke="#39FF14" stroke-width="1.5" stroke-dasharray="4,2"/>
      <polygon points="463,126 471,130 463,134" fill="#39FF14"/>

      <!-- Block: OR Gate -->
      <rect x="472" y="100" width="80" height="60" rx="8" fill="rgba(13,33,55,0.9)" stroke="rgba(57,255,20,0.35)" stroke-width="1.5"/>
      <text x="512" y="122" text-anchor="middle" fill="#39FF14" font-family="Space Mono" font-size="8" letter-spacing="1">OR</text>
      <text x="512" y="134" text-anchor="middle" fill="#39FF14" font-family="Space Mono" font-size="8" letter-spacing="1">GATES</text>
      <text x="512" y="148" text-anchor="middle" fill="rgba(57,255,20,0.5)" font-family="Space Mono" font-size="7">4072 / 4075</text>

      <!-- Arrow: OR → L293D -->
      <line x1="552" y1="130" x2="577" y2="130" stroke="#FF4F5E" stroke-width="1.5" stroke-dasharray="4,2"/>
      <polygon points="577,126 585,130 577,134" fill="#FF4F5E"/>

      <!-- Block: L293D -->
      <rect x="586" y="80" width="90" height="100" rx="8" fill="rgba(13,33,55,0.9)" stroke="rgba(255,79,94,0.45)" stroke-width="1.5"/>
      <text x="631" y="104" text-anchor="middle" fill="#FF4F5E" font-family="Space Mono" font-size="9" letter-spacing="1">L293D</text>
      <text x="631" y="119" text-anchor="middle" fill="rgba(255,79,94,0.6)" font-family="Space Mono" font-size="7">H-Bridge</text>
      <text x="631" y="131" text-anchor="middle" fill="rgba(255,79,94,0.6)" font-family="Space Mono" font-size="7">Motor Driver</text>
      <text x="631" y="154" text-anchor="middle" fill="rgba(255,79,94,0.4)" font-family="Space Mono" font-size="7">9V supply</text>
      <text x="631" y="166" text-anchor="middle" fill="rgba(255,79,94,0.4)" font-family="Space Mono" font-size="7">600mA/ch</text>

      <!-- Arrows: L293D → Motors -->
      <line x1="676" y1="110" x2="718" y2="90" stroke="#FFB627" stroke-width="1.5"/>
      <polygon points="716,85 724,89 718,96" fill="#FFB627"/>
      <line x1="676" y1="155" x2="718" y2="175" stroke="#FFB627" stroke-width="1.5"/>
      <polygon points="716,170 724,174 718,181" fill="#FFB627"/>

      <!-- Motor A -->
      <circle cx="736" cy="86" r="22" fill="rgba(13,33,55,0.9)" stroke="#FFB627" stroke-width="1.5"/>
      <text x="736" y="84" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">MOTOR</text>
      <text x="736" y="94" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">A (H)</text>

      <!-- Motor B -->
      <circle cx="736" cy="178" r="22" fill="rgba(13,33,55,0.9)" stroke="#FFB627" stroke-width="1.5"/>
      <text x="736" y="176" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">MOTOR</text>
      <text x="736" y="186" text-anchor="middle" fill="#FFB627" font-family="Space Mono" font-size="7">B (V)</text>

      <!-- Label: Sun arrows -->
      <line x1="100" y1="60" x2="100" y2="78" stroke="#FFB627" stroke-width="1.5" stroke-dasharray="3,2"/>
      <polygon points="96,78 100,86 104,78" fill="#FFB627"/>
      <line x1="244" y1="60" x2="244" y2="98" stroke="#FFB627" stroke-width="1.5" stroke-dasharray="3,2" opacity="0.3"/>
      <line x1="378" y1="60" x2="378" y2="78" stroke="#FFB627" stroke-width="1.5" stroke-dasharray="3,2" opacity="0.2"/>

      <!-- Labels at bottom -->
      <text x="100" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">LDR Sensors</text>
      <text x="244" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">Inversion</text>
      <text x="378" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">AND Network</text>
      <text x="512" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">OR Layer</text>
      <text x="631" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">H-Bridge</text>
      <text x="736" y="230" text-anchor="middle" fill="rgba(0,229,255,0.5)" font-family="Syne" font-size="10">Motors</text>
    </svg>
  </div>
</div>

<!-- MAIN CONTENT -->
<main>

  <!-- INTERACTIVE SIMULATOR -->
  <div class="sim-area">
    <div class="section-label">Interactive Logic Simulator</div>
    <p style="color:var(--text-muted);font-size:0.85rem;text-align:center;margin-bottom:24px;">Toggle LDR sensors ON/OFF to see which motor activates and in which direction.</p>

    <div class="sim-sensors" id="sensorBtns">
      <button class="sim-btn on" data-s="s1" onclick="toggleSensor('s1')">
        <span class="ldr">☀️</span>
        <span class="s-name">S1 — TOP LEFT</span>
        <span style="font-size:10px;color:var(--sun);margin-top:4px;display:block;">BRIGHT</span>
      </button>
      <button class="sim-btn" data-s="s2" onclick="toggleSensor('s2')">
        <span class="ldr">🌑</span>
        <span class="s-name">S2 — TOP RIGHT</span>
        <span style="font-size:10px;color:#2A4A6A;margin-top:4px;display:block;">DARK</span>
      </button>
      <button class="sim-btn on" data-s="s3" onclick="toggleSensor('s3')">
        <span class="ldr">☀️</span>
        <span class="s-name">S3 — BOT LEFT</span>
        <span style="font-size:10px;color:var(--sun);margin-top:4px;display:block;">BRIGHT</span>
      </button>
      <button class="sim-btn" data-s="s4" onclick="toggleSensor('s4')">
        <span class="ldr">🌑</span>
        <span class="s-name">S4 — BOT RIGHT</span>
        <span style="font-size:10px;color:#2A4A6A;margin-top:4px;display:block;">DARK</span>
      </button>
    </div>

    <div class="motor-display">
      <div class="motor-card">
        <div class="motor-label">Motor A — Horizontal</div>
        <div class="motor-arrow" id="arrowH">◀ ▶</div>
        <div class="motor-state" id="stateH">STOPPED</div>
      </div>
      <div class="motor-card">
        <div class="motor-label">Motor B — Vertical</div>
        <div class="motor-arrow" id="arrowV">▲ ▼</div>
        <div class="motor-state" id="stateV">STOPPED</div>
      </div>
    </div>

    <div class="sim-result" id="simResult">Inputs: S1=1 S2=0 S3=1 S4=0 → Left side brighter → Motor A: ROTATE LEFT</div>
  </div>

  <!-- COMPONENTS -->
  <h2 class="sec-title"><span>🔩</span> Bill of Materials</h2>
  <table class="comp-table">
    <thead>
      <tr><th>REF</th><th>COMPONENT</th><th>QTY</th><th>FUNCTION</th><th>NOTES</th></tr>
    </thead>
    <tbody>
      <tr><td class="ref">S1–S4</td><td class="ic">LDR + 10kΩ Divider</td><td>4</td><td>Light sensing</td><td>Output HIGH when bright</td></tr>
      <tr><td class="ref">NOT0 ×4</td><td class="ic">74HC04</td><td>1</td><td>Signal inversion</td><td>Hex inverter; 4 used</td></tr>
      <tr><td class="ref">U16, U8</td><td class="ic">74HC21</td><td>2</td><td>4-input AND</td><td>Dual 4-input AND per chip</td></tr>
      <tr><td class="ref">U5, U6</td><td class="ic">4073</td><td>2</td><td>3-input AND</td><td>Triple 3-input AND per chip</td></tr>
      <tr><td class="ref">U10,U12,U13</td><td class="ic">Generic AND</td><td>3</td><td>2/3-input AND</td><td>From same packages above</td></tr>
      <tr><td class="ref">U7</td><td class="ic">4072</td><td>1</td><td>4-input OR</td><td>Dual 4-input OR per chip</td></tr>
      <tr><td class="ref">U11, U14</td><td class="ic">4075</td><td>1</td><td>3-input OR</td><td>Triple 3-input OR per chip</td></tr>
      <tr><td class="ref">U9</td><td class="ic">OR Gate</td><td>1</td><td>OR combination</td><td>Any 2-input OR</td></tr>
      <tr><td class="ref">U15</td><td class="ic" style="color:var(--accent2)">L293D</td><td>1</td><td>H-Bridge driver</td><td>Dual-channel, 600mA/ch</td></tr>
      <tr><td class="ref">V1</td><td class="ic">9V Battery</td><td>1</td><td>Motor supply</td><td>Also feeds 5V regulator</td></tr>
      <tr><td class="ref">M1, M2</td><td class="ic">DC Motor</td><td>2</td><td>Axis movement</td><td>3–6V, low-torque</td></tr>
    </tbody>
  </table>

  <!-- LOGIC EQUATIONS -->
  <h2 class="sec-title"><span>🔢</span> Logic Equations</h2>
  <p style="color:var(--text-muted);font-size:0.875rem;margin-bottom:20px;line-height:1.8;">
    Let <code style="color:var(--accent)">A=S1</code>, <code style="color:var(--accent)">B=S2</code>, <code style="color:var(--accent)">C=S3</code>, <code style="color:var(--accent)">D=S4</code>. Overline = NOT. Each term represents a combination where one side is definitively brighter than the other.
  </p>
  <div class="eq-box">
    <div class="axis">── Horizontal Axis (Motor A) ──</div>
    <div class="eq">Motor_A_LEFT  &nbsp;= (A · <span style="text-decoration:overline">B</span> · C · <span style="text-decoration:overline">D</span>) + (A · <span style="text-decoration:overline">B</span> · <span style="text-decoration:overline">C</span> · <span style="text-decoration:overline">D</span>) + (<span style="text-decoration:overline">A</span> · <span style="text-decoration:overline">B</span> · C · <span style="text-decoration:overline">D</span>)</div>
    <div class="eq" style="margin-top:6px;">Motor_A_RIGHT &nbsp;= (<span style="text-decoration:overline">A</span> · B · <span style="text-decoration:overline">C</span> · D) + (<span style="text-decoration:overline">A</span> · B · <span style="text-decoration:overline">C</span> · <span style="text-decoration:overline">D</span>) + (<span style="text-decoration:overline">A</span> · <span style="text-decoration:overline">B</span> · <span style="text-decoration:overline">C</span> · D)</div>
  </div>
  <div class="eq-box" style="margin-bottom:60px;">
    <div class="axis">── Vertical Axis (Motor B) ──</div>
    <div class="eq">Motor_B_UP   &nbsp;&nbsp;&nbsp;= (A · B · <span style="text-decoration:overline">C</span> · <span style="text-decoration:overline">D</span>) + (A · <span style="text-decoration:overline">B</span> · <span style="text-decoration:overline">C</span> · <span style="text-decoration:overline">D</span>) + (<span style="text-decoration:overline">A</span> · B · <span style="text-decoration:overline">C</span> · <span style="text-decoration:overline">D</span>)</div>
    <div class="eq" style="margin-top:6px;">Motor_B_DOWN &nbsp;= (<span style="text-decoration:overline">A</span> · <span style="text-decoration:overline">B</span> · C · D) + (<span style="text-decoration:overline">A</span> · <span style="text-decoration:overline">B</span> · C · <span style="text-decoration:overline">D</span>) + (<span style="text-decoration:overline">A</span> · <span style="text-decoration:overline">B</span> · <span style="text-decoration:overline">C</span> · D)</div>
  </div>

  <!-- TRUTH TABLES -->
  <h2 class="sec-title"><span>📊</span> Truth Tables</h2>
  <div class="truth-table-wrap">
    <table>
      <thead>
        <tr>
          <th>S1(A)</th><th>S2(B)</th><th>S3(C)</th><th>S4(D)</th>
          <th>MOTOR A LEFT</th><th>MOTOR A RIGHT</th>
          <th>MOTOR B UP</th><th>MOTOR B DOWN</th>
          <th>CONDITION</th>
        </tr>
      </thead>
      <tbody id="truthTableBody"></tbody>
    </table>
  </div>

  <!-- L293D WIRING -->
  <h2 class="sec-title"><span>⚡</span> L293D Wiring</h2>
  <div class="grid-2" style="margin-bottom:60px;">
    <div class="card">
      <h3>Pin Mapping</h3>
      <div class="code-block" style="font-size:11px;">
<span class="cm">-- Motor A (Horizontal) --</span>
<span class="sig">IN1</span>  → Motor_A_LEFT  (from OR gate)
<span class="sig">IN2</span>  → Motor_A_RIGHT (from OR gate)
<span class="sig">EN1</span>  → VCC (always enabled)
<span class="sig">OUT1</span> → Motor A terminal (+)
<span class="sig">OUT2</span> → Motor A terminal (-)

<span class="cm">-- Motor B (Vertical) --</span>
<span class="sig">IN3</span>  → Motor_B_UP   (from OR gate)
<span class="sig">IN4</span>  → Motor_B_DOWN (from OR gate)
<span class="sig">EN2</span>  → VCC (always enabled)
<span class="sig">OUT3</span> → Motor B terminal (+)
<span class="sig">OUT4</span> → Motor B terminal (-)

<span class="cm">-- Power --</span>
<span class="val">VS</span>   → 9V (motor supply)
<span class="val">VCC</span>  → 5V (logic supply)
<span class="val">GND</span>  → Common ground (×4 pins)</div>
    </div>
    <div class="card">
      <h3>H-Bridge Truth Table</h3>
      <table style="width:100%;margin-top:8px;">
        <thead>
          <tr><th>EN</th><th>IN1/IN3</th><th>IN2/IN4</th><th>Motor</th></tr>
        </thead>
        <tbody>
          <tr><td class="one">1</td><td class="one">1</td><td class="zero">0</td><td style="color:var(--accent3);font-size:11px;">Forward (CW)</td></tr>
          <tr><td class="one">1</td><td class="zero">0</td><td class="one">1</td><td style="color:var(--accent2);font-size:11px;">Reverse (CCW)</td></tr>
          <tr><td class="one">1</td><td class="one">1</td><td class="one">1</td><td style="color:var(--text-muted);font-size:11px;">Brake</td></tr>
          <tr><td class="one">1</td><td class="zero">0</td><td class="zero">0</td><td style="color:var(--text-muted);font-size:11px;">Coast/Stop</td></tr>
          <tr><td class="zero">0</td><td>X</td><td>X</td><td style="color:var(--text-muted);font-size:11px;">Disabled</td></tr>
        </tbody>
      </table>
    </div>
  </div>

  <!-- TEST CASES -->
  <h2 class="sec-title"><span>🧪</span> Simulation Test Cases</h2>
  <div class="code-block" style="margin-bottom:60px;">
<span class="cm">// Proteus ISIS — Switch Positions for Testing</span>

<span class="kw">Test 1</span>  │ S1=<span class="val">1</span> S2=<span class="zero">0</span> S3=<span class="val">1</span> S4=<span class="zero">0</span> │ Left bright  → Motor A <span class="sig">ROTATES LEFT</span>
<span class="kw">Test 2</span>  │ S1=<span class="zero">0</span> S2=<span class="val">1</span> S3=<span class="zero">0</span> S4=<span class="val">1</span> │ Right bright → Motor A <span class="sig">ROTATES RIGHT</span>
<span class="kw">Test 3</span>  │ S1=<span class="val">1</span> S2=<span class="val">1</span> S3=<span class="zero">0</span> S4=<span class="zero">0</span> │ Top bright   → Motor B <span class="sig">TILTS UP</span>
<span class="kw">Test 4</span>  │ S1=<span class="zero">0</span> S2=<span class="zero">0</span> S3=<span class="val">1</span> S4=<span class="val">1</span> │ Bottom bright→ Motor B <span class="sig">TILTS DOWN</span>
<span class="kw">Test 5</span>  │ S1=<span class="val">1</span> S2=<span class="val">1</span> S3=<span class="val">1</span> S4=<span class="val">1</span> │ All bright   → Both motors <span class="sig">STOP</span>
<span class="kw">Test 6</span>  │ S1=<span class="zero">0</span> S2=<span class="zero">0</span> S3=<span class="zero">0</span> S4=<span class="zero">0</span> │ No light     → Both motors <span class="sig">STOP</span>
<span class="kw">Test 7</span>  │ S1=<span class="val">1</span> S2=<span class="zero">0</span> S3=<span class="zero">0</span> S4=<span class="zero">0</span> │ TL only      → Motor A LEFT + Motor B UP
<span class="kw">Test 8</span>  │ S1=<span class="zero">0</span> S2=<span class="zero">0</span> S3=<span class="zero">0</span> S4=<span class="val">1</span> │ BR only      → Motor A RIGHT + Motor B DOWN</div>

  <!-- REPO STRUCTURE -->
  <h2 class="sec-title"><span>📁</span> Repository Structure</h2>
  <div class="tree">
<span class="folder">dual-axis-solar-tracker/</span>
├── <span class="file">README.md</span>                         <span class="desc">← Project overview, logic tables, IC ref</span>
├── <span class="file">index.html</span>                        <span class="desc">← This documentation page</span>
│
├── <span class="folder">docs/</span>
│   ├── <span class="file">circuit-analysis.md</span>             <span class="desc">← Gate-by-gate schematic walkthrough</span>
│   ├── <span class="file">truth-tables.md</span>                 <span class="desc">← Full 16-row truth tables</span>
│   ├── <span class="file">component-datasheets.md</span>         <span class="desc">← Links to IC datasheets</span>
│   └── <span class="folder">images/</span>
│       └── <span class="file">schematic.png</span>               <span class="desc">← Proteus schematic screenshot</span>
│
├── <span class="folder">simulation/</span>
│   └── <span class="file">dual_axis_tracker.pdsprj</span>        <span class="desc">← Proteus ISIS simulation file</span>
│
├── <span class="folder">hardware/</span>
│   ├── <span class="file">BOM.csv</span>                         <span class="desc">← Bill of materials with part numbers</span>
│   └── <span class="file">pcb-layout-notes.md</span>             <span class="desc">← Notes for PCB design (KiCad)</span>
│
└── <span class="file">LICENSE</span>                           <span class="desc">← MIT License</span></div>

  <!-- FUTURE IMPROVEMENTS -->
  <h2 class="sec-title"><span>🚀</span> Future Improvements</h2>
  <ul class="todo-list">
    <li><div class="checkbox"></div><div><span class="title">Hysteresis / Deadband Comparator</span>Add a window comparator so the motors don't hunt or oscillate when sensors are nearly equal — prevents mechanical wear at equilibrium.</div></li>
    <li><div class="checkbox"></div><div><span class="title">PWM Speed Control</span>Replace binary ON/OFF motor drive with PWM for gradual, smoother movement — especially useful during slow sun movement.</div></li>
    <li><div class="checkbox"></div><div><span class="title">Mechanical Limit Switches</span>Add hardware end-stops on both axes to prevent over-rotation and mechanical damage.</div></li>
    <li><div class="checkbox"></div><div><span class="title">Night Mode / Park Position</span>Add a global light threshold (LDR + comparator) that parks the panel facing east at sunset and wakes it at sunrise.</div></li>
    <li><div class="checkbox"></div><div><span class="title">PCB Design (KiCad)</span>Convert the breadboard prototype to a compact custom PCB — reduces wiring errors and improves reliability.</div></li>
    <li><div class="checkbox"></div><div><span class="title">Arduino Comparison Branch</span>Implement the same tracking logic in an Arduino sketch for efficiency benchmarking vs. the pure logic-gate approach.</div></li>
    <li><div class="checkbox"></div><div><span class="title">Efficiency Logging</span>Add a current + voltage sensor on the solar panel output to measure and log actual power gain from tracking vs. fixed mount.</div></li>
  </ul>

</main>

<footer>
  ☀ DUAL AXIS SOLAR TRACKER — LOGIC GATE IMPLEMENTATION &nbsp;·&nbsp; MIT LICENSE &nbsp;·&nbsp; BUILT WITH PROTEUS ISIS
</footer>

<script>
// ── SENSOR STATE ──
const state = { s1: true, s2: false, s3: true, s4: false };

function toggleSensor(id) {
  state[id] = !state[id];
  const btn = document.querySelector(`[data-s="${id}"]`);
  const on = state[id];
  btn.classList.toggle('on', on);
  btn.querySelector('.ldr').textContent = on ? '☀️' : '🌑';
  btn.querySelector('span:last-child').textContent = on ? 'BRIGHT' : 'DARK';
  btn.querySelector('span:last-child').style.color = on ? 'var(--sun)' : '#2A4A6A';
  updateSim();
}

function updateSim() {
  const A = state.s1 ? 1 : 0;
  const B = state.s2 ? 1 : 0;
  const C = state.s3 ? 1 : 0;
  const D = state.s4 ? 1 : 0;
  const nA = 1-A, nB = 1-B, nC = 1-C, nD = 1-D;

  const mAL = (A&&!B&&C&&!D) || (A&&!B&&!C&&!D) || (!A&&!B&&C&&!D);
  const mAR = (!A&&B&&!C&&D) || (!A&&B&&!C&&!D) || (!A&&!B&&!C&&D);
  const mBU = (A&&B&&!C&&!D) || (A&&!B&&!C&&!D) || (!A&&B&&!C&&!D);
  const mBD = (!A&&!B&&C&&D) || (!A&&!B&&C&&!D) || (!A&&!B&&!C&&D);

  // Horizontal motor
  const aH = document.getElementById('arrowH');
  const sH = document.getElementById('stateH');
  if (mAL) { aH.textContent = '◀'; aH.className = 'motor-arrow active'; aH.style.color = 'var(--sun)'; sH.textContent = 'ROTATE LEFT'; sH.className = 'motor-state running'; }
  else if (mAR) { aH.textContent = '▶'; aH.className = 'motor-arrow active'; aH.style.color = 'var(--sun)'; sH.textContent = 'ROTATE RIGHT'; sH.className = 'motor-state running'; }
  else { aH.textContent = '◀ ▶'; aH.className = 'motor-arrow'; aH.style.color = ''; sH.textContent = 'STOPPED'; sH.className = 'motor-state'; }

  // Vertical motor
  const aV = document.getElementById('arrowV');
  const sV = document.getElementById('stateV');
  if (mBU) { aV.textContent = '▲'; aV.className = 'motor-arrow active'; aV.style.color = 'var(--accent)'; sV.textContent = 'TILT UP'; sV.className = 'motor-state running'; }
  else if (mBD) { aV.textContent = '▼'; aV.className = 'motor-arrow active'; aV.style.color = 'var(--accent2)'; sV.textContent = 'TILT DOWN'; sV.className = 'motor-state running'; }
  else { aV.textContent = '▲ ▼'; aV.className = 'motor-arrow'; aV.style.color = ''; sV.textContent = 'STOPPED'; sV.className = 'motor-state'; }

  const parts = [];
  if (mAL) parts.push('Motor A → LEFT');
  if (mAR) parts.push('Motor A → RIGHT');
  if (mBU) parts.push('Motor B → UP');
  if (mBD) parts.push('Motor B → DOWN');
  if (!mAL && !mAR && !mBU && !mBD) parts.push('Both motors STOPPED (balanced or no light)');

  const cond = mAL||mAR ? (mAL ? 'Left side brighter' : 'Right side brighter') :
               mBU||mBD ? (mBU ? 'Top side brighter' : 'Bottom side brighter') : 'Balanced';

  document.getElementById('simResult').textContent =
    `Inputs: S1=${A} S2=${B} S3=${C} S4=${D} → ${cond} → ${parts.join(' | ')}`;
}

// ── TRUTH TABLE GENERATION ──
function buildTruthTable() {
  const tbody = document.getElementById('truthTableBody');
  const conditions = [
    [1,1,1,1,'All bright'], [0,0,0,0,'No light'],
    [1,0,1,0,'Left only'], [0,1,0,1,'Right only'],
    [1,1,0,0,'Top only'], [0,0,1,1,'Bottom only'],
    [1,0,0,0,'S1 only'], [0,1,0,0,'S2 only'],
    [0,0,1,0,'S3 only'], [0,0,0,1,'S4 only'],
    [1,0,1,1,'S1+S3+S4'], [1,1,1,0,'S1+S2+S3'],
    [0,1,1,0,'S2+S3'], [1,0,0,1,'S1+S4'],
    [1,1,0,1,'S1+S2+S4'], [0,1,1,1,'S2+S3+S4'],
  ];

  conditions.forEach(([A,B,C,D,label]) => {
    const mAL = (A&&!B&&C&&!D)||(A&&!B&&!C&&!D)||(!A&&!B&&C&&!D);
    const mAR = (!A&&B&&!C&&D)||(!A&&B&&!C&&!D)||(!A&&!B&&!C&&D);
    const mBU = (A&&B&&!C&&!D)||(A&&!B&&!C&&!D)||(!A&&B&&!C&&!D);
    const mBD = (!A&&!B&&C&&D)||(!A&&!B&&C&&!D)||(!A&&!B&&!C&&D);

    const f = (v, active) => `<td class="${active ? 'act' : v ? 'one' : 'zero'}">${active ? (v === 'L' ? '← L' : v === 'R' ? 'R →' : v === 'U' ? '↑ UP' : '↓ DN') : (v ? '1' : '0')}</td>`;

    tbody.innerHTML += `<tr>
      ${f(A,false)}${f(B,false)}${f(C,false)}${f(D,false)}
      ${mAL ? `<td class="act">← LEFT</td>` : `<td class="zero">0</td>`}
      ${mAR ? `<td class="act">RIGHT →</td>` : `<td class="zero">0</td>`}
      ${mBU ? `<td class="act">↑ UP</td>` : `<td class="zero">0</td>`}
      ${mBD ? `<td class="act">↓ DOWN</td>` : `<td class="zero">0</td>`}
      <td style="color:var(--text-muted);font-size:11px;">${label}</td>
    </tr>`;
  });
}

buildTruthTable();
updateSim();
</script>
</body>
</html>
