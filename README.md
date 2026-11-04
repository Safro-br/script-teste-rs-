<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Visual Assist HUD</title>
<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Segoe UI', 'Inter', system-ui, -apple-system, sans-serif;
    user-select: none;
    -webkit-user-select: none;
  }

  body {
    background: radial-gradient(circle at 50% 30%, #1a1f2e 0%, #0a0d14 100%);
    min-height: 100vh;
    overflow: hidden;
    position: relative;
  }

  /* Fundo decorativo simulando jogo */
  body::before {
    content: '';
    position: absolute;
    inset: 0;
    background-image: 
      linear-gradient(rgba(0, 200, 255, 0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0, 200, 255, 0.03) 1px, transparent 1px);
    background-size: 60px 60px;
    pointer-events: none;
  }

  /* ============ BOTÃO FLUTUANTE ============ */
  #floatingBtn {
    position: fixed;
    top: 24px;
    left: 24px;
    width: 52px;
    height: 52px;
    border-radius: 50%;
    background: linear-gradient(135deg, #00d4ff 0%, #0066ff 100%);
    border: none;
    cursor: grab;
    z-index: 10000;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 
      0 0 0 1px rgba(255, 255, 255, 0.15),
      0 8px 24px rgba(0, 150, 255, 0.4),
      0 0 30px rgba(0, 212, 255, 0.2);
    transition: transform 0.2s ease, box-shadow 0.3s ease;
  }

  #floatingBtn:hover {
    transform: scale(1.08);
    box-shadow: 
      0 0 0 1px rgba(255, 255, 255, 0.25),
      0 10px 30px rgba(0, 150, 255, 0.6),
      0 0 40px rgba(0, 212, 255, 0.35);
  }

  #floatingBtn:active {
    cursor: grabbing;
    transform: scale(0.95);
  }

  #floatingBtn.dragging {
    cursor: grabbing;
    transition: none;
  }

  #floatingBtn svg {
    width: 26px;
    height: 26px;
    fill: white;
    transition: transform 0.3s ease;
  }

  #floatingBtn.collapsed svg {
    transform: rotate(180deg);
  }

  /* ============ PAINEL HUD ============ */
  #hudPanel {
    position: fixed;
    top: 90px;
    left: 24px;
    width: 300px;
    background: rgba(15, 20, 30, 0.82);
    backdrop-filter: blur(20px) saturate(180%);
    -webkit-backdrop-filter: blur(20px) saturate(180%);
    border-radius: 18px;
    border: 1px solid rgba(255, 255, 255, 0.08);
    box-shadow: 
      0 20px 60px rgba(0, 0, 0, 0.6),
      inset 0 1px 0 rgba(255, 255, 255, 0.06);
    z-index: 9999;
    overflow: hidden;
    transition: opacity 0.35s cubic-bezier(0.4, 0, 0.2, 1),
                transform 0.35s cubic-bezier(0.4, 0, 0.2, 1),
                visibility 0.35s;
    transform-origin: top left;
  }

  #hudPanel.hidden {
    opacity: 0;
    transform: scale(0.9) translateY(-10px);
    visibility: hidden;
    pointer-events: none;
  }

  /* Header */
  .hud-header {
    padding: 18px 20px 14px;
    display: flex;
    align-items: center;
    gap: 10px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.06);
    background: linear-gradient(180deg, rgba(0, 212, 255, 0.06), transparent);
  }

  .hud-header .dot {
    width: 9px;
    height: 9px;
    border-radius: 50%;
    background: #00ff88;
    box-shadow: 0 0 10px #00ff88, 0 0 20px rgba(0, 255, 136, 0.5);
    animation: pulse 2s ease-in-out infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; transform: scale(1); }
    50% { opacity: 0.6; transform: scale(0.85); }
  }

  .hud-header h1 {
    font-size: 13px;
    font-weight: 700;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: #ffffff;
    flex: 1;
  }

  .hud-header .version {
    font-size: 10px;
    color: #00d4ff;
    font-weight: 600;
    letter-spacing: 0.5px;
    padding: 3px 8px;
    background: rgba(0, 212, 255, 0.1);
    border-radius: 6px;
    border: 1px solid rgba(0, 212, 255, 0.2);
  }

  /* Sections */
  .hud-section {
    padding: 14px 20px;
  }

  .hud-section + .hud-section {
    border-top: 1px solid rgba(255, 255, 255, 0.05);
  }

  .section-label {
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 1.2px;
    text-transform: uppercase;
    color: #6b7a94;
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .section-label::after {
    content: '';
    flex: 1;
    height: 1px;
    background: linear-gradient(90deg, rgba(255,255,255,0.08), transparent);
  }

  /* Toggle rows */
  .toggle-row {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 9px 0;
    cursor: pointer;
    transition: opacity 0.2s;
  }

  .toggle-row:hover {
    opacity: 0.85;
  }

  .toggle-info {
    display: flex;
    align-items: center;
    gap: 11px;
  }

  .toggle-icon {
    width: 28px;
    height: 28px;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.06);
    transition: all 0.3s;
  }

  .toggle-icon svg {
    width: 15px;
    height: 15px;
    fill: #8892a6;
    transition: fill 0.3s;
  }

  .toggle-row.active .toggle-icon {
    background: rgba(0, 212, 255, 0.12);
    border-color: rgba(0, 212, 255, 0.3);
    box-shadow: 0 0 12px rgba(0, 212, 255, 0.15);
  }

  .toggle-row.active .toggle-icon svg {
    fill: #00d4ff;
  }

  .toggle-text {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .toggle-name {
    font-size: 12.5px;
    font-weight: 600;
    color: #dbe2f0;
    letter-spacing: 0.2px;
  }

  .toggle-desc {
    font-size: 9.5px;
    color: #5a6478;
    font-weight: 500;
    letter-spacing: 0.2px;
  }

  /* Switch */
  .switch {
    position: relative;
    width: 40px;
    height: 22px;
    background: rgba(255, 255, 255, 0.08);
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.1);
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    flex-shrink: 0;
  }

  .switch::after {
    content: '';
    position: absolute;
    top: 2px;
    left: 2px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #8892a6;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }

  .toggle-row.active .switch {
    background: linear-gradient(135deg, #00d4ff, #0066ff);
    border-color: rgba(0, 212, 255, 0.5);
    box-shadow: 0 0 15px rgba(0, 212, 255, 0.35);
  }

  .toggle-row.active .switch::after {
    left: 20px;
    background: #ffffff;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.4);
  }

  /* Slider de FOV */
  .fov-control {
    padding: 10px 0 4px;
  }

  .fov-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 8px;
  }

  .fov-label {
    font-size: 11px;
    color: #8892a6;
    font-weight: 500;
  }

  .fov-value {
    font-size: 11px;
    color: #00d4ff;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
  }

  .fov-slider {
    width: 100%;
    height: 4px;
    -webkit-appearance: none;
    appearance: none;
    background: rgba(255, 255, 255, 0.08);
    border-radius: 4px;
    outline: none;
    cursor: pointer;
  }

  .fov-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: linear-gradient(135deg, #00d4ff, #0066ff);
    cursor: pointer;
    box-shadow: 0 0 10px rgba(0, 212, 255, 0.6);
    border: 2px solid #0a0d14;
  }

  .fov-slider::-moz-range-thumb {
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: linear-gradient(135deg, #00d4ff, #0066ff);
    cursor: pointer;
    box-shadow: 0 0 10px rgba(0, 212, 255, 0.6);
    border: 2px solid #0a0d14;
  }

  /* Footer */
  .hud-footer {
    padding: 12px 20px;
    border-top: 1px solid rgba(255, 255, 255, 0.05);
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: rgba(0, 0, 0, 0.15);
  }

  .status {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 10px;
    font-weight: 600;
    color: #00ff88;
    letter-spacing: 0.5px;
  }

  .status::before {
    content: '';
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #00ff88;
    box-shadow: 0 0 6px #00ff88;
  }

  .fps {
    font-size: 10px;
    color: #6b7a94;
    font-weight: 600;
    font-variant-numeric: tabular-nums;
  }

  /* FOV Circle visual no centro */
  #fovCircle {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border: 1.5px solid rgba(0, 212, 255, 0.5);
    border-radius: 50%;
    pointer-events: none;
    z-index: 100;
    box-shadow: 
      0 0 20px rgba(0, 212, 255, 0.2),
      inset 0 0 20px rgba(0, 212, 255, 0.05);
    transition: opacity 0.3s, width 0.3s, height 0.3s;
    display: none;
  }

  #fovCircle.active {
    display: block;
  }

  #fovCircle::before,
  #fovCircle::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    background: rgba(0, 212, 255, 0.4);
    transform: translate(-50%, -50%);
  }

  #fovCircle::before {
    width: 12px;
    height: 1px;
  }

  #fovCircle::after {
    width: 1px;
    height: 12px;
  }

  /* Custom scrollbar */
  ::-webkit-scrollbar { width: 6px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgba(0, 212, 255, 0.3); border-radius: 3px; }
</style>
</head>
<body>

<!-- FOV Circle -->
<div id="fovCircle"></div>

<!-- Botão Flutuante -->
<button id="floatingBtn" title="Visual Assist HUD">
  <svg viewBox="0 0 24 24">
    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm0-14c-3.31 0-6 2.69-6 6s2.69 6 6 6 6-2.69 6-6-2.69-6-6-6zm0 10c-2.21 0-4-1.79-4-4s1.79-4 4-4 4 1.79 4 4-1.79 4-4 4z"/>
  </svg>
</button>

<!-- Painel HUD -->
<div id="hudPanel">
  <div class="hud-header">
    <div class="dot"></div>
    <h1>Visual Assist</h1>
    <span class="version">v2.4</span>
  </div>

  <!-- Seção de Checks -->
  <div class="hud-section">
    <div class="section-label">Detecção</div>

    <div class="toggle-row" data-toggle="wallCheck">
      <div class="toggle-info">
        <div class="toggle-icon">
          <svg viewBox="0 0 24 24"><path d="M4 4h16v16H4V4zm2 2v12h12V6H6z"/></svg>
        </div>
        <div class="toggle-text">
          <span class="toggle-name">Wall Check</span>
          <span class="toggle-desc">Indicador de obstruções</span>
        </div>
      </div>
      <div class="switch"></div>
    </div>

    <div class="toggle-row" data-toggle="teamCheck">
      <div class="toggle-info">
        <div class="toggle-icon">
          <svg viewBox="0 0 24 24"><path d="M16 11c1.66 0 2.99-1.34 2.99-3S17.66 5 16 5c-1.66 0-3 1.34-3 3s1.34 3 3 3zm-8 0c1.66 0 2.99-1.34 2.99-3S9.66 5 8 5C6.34 5 5 6.34 5 8s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V19h14v-2.5c0-2.33-4.67-3.5-7-3.5zm8 0c-.29 0-.62.02-.97.05 1.16.84 1.97 1.97 1.97 3.45V19h6v-2.5c0-2.33-4.67-3.5-7-3.5z"/></svg>
        </div>
        <div class="toggle-text">
          <span class="toggle-name">Team Check</span>
          <span class="toggle-desc">Aliados e inimigos</span>
        </div>
      </div>
      <div class="switch"></div>
    </div>

    <div class="toggle-row" data-toggle="friendCheck">
      <div class="toggle-info">
        <div class="toggle-icon">
          <svg viewBox="0 0 24 24"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>
        </div>
        <div class="toggle-text">
          <span class="toggle-name">Friend Check</span>
          <span class="toggle-desc">Identificação de amigos</span>
        </div>
      </div>
      <div class="switch"></div>
    </div>
  </div>

  <!-- Seção de Visuais -->
  <div class="hud-section">
    <div class="section-label">Visuais</div>

    <div class="toggle-row" data-toggle="targetIndicator">
      <div class="toggle-info">
        <div class="toggle-icon">
          <svg viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 15c-2.76 0-5-2.24-5-5s2.24-5 5-5 5 2.24 5 5-2.24 5-5 5zm0-8c-1.66 0-3 1.34-3 3s1.34 3 3 3 3-1.34 3-3-1.34-3-3-3z"/></svg>
        </div>
        <div class="toggle-text">
          <span class="toggle-name">Target Indicator</span>
          <span class="toggle-desc">Indicador de alvo</span>
        </div>
      </div>
      <div class="switch"></div>
    </div>

    <div class="toggle-row" data-toggle="fovCircle">
      <div class="toggle-info">
        <div class="toggle-icon">
          <svg viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8z"/></svg>
        </div>
        <div class="toggle-text">
          <span class="toggle-name">FOV Circle</span>
          <span class="toggle-desc">Círculo de referência</span>
        </div>
      </div>
      <div class="switch"></div>
    </div>

    <div class="fov-control" id="fovControl" style="display:none;">
      <div class="fov-header">
        <span class="fov-label">Raio do FOV</span>
        <span class="fov-value" id="fovValue">180px</span>
      </div>
      <input type="range" class="fov-slider" id="fovSlider" min="80" max="400" value="180">
    </div>
  </div>

  <!-- Footer -->
  <div class="hud-footer">
    <div class="status">ATIVO</div>
    <div class="fps" id="fpsCounter">60 FPS</div>
  </div>
</div>

<script>
  // ============ ELEMENTOS ============
  const floatingBtn = document.getElementById('floatingBtn');
  const hudPanel = document.getElementById('hudPanel');
  const fovCircle = document.getElementById('fovCircle');
  const fovControl = document.getElementById('fovControl');
  const fovSlider = document.getElementById('fovSlider');
  const fovValue = document.getElementById('fovValue');
  const fpsCounter = document.getElementById('fpsCounter');

  // ============ ESTADO ============
  const state = {
    panelOpen: true,
    wallCheck: false,
    teamCheck: false,
    friendCheck: false,
    targetIndicator: false,
    fovCircle: false,
    fovRadius: 180
  };

  // ============ DRAG DO BOTÃO FLUTUANTE ============
  let isDragging = false;
  let hasMoved = false;
  let offsetX = 0, offsetY = 0;

  floatingBtn.addEventListener('mousedown', (e) => {
    isDragging = true;
    hasMoved = false;
    floatingBtn.classList.add('dragging');
    
    const rect = floatingBtn.getBoundingClientRect();
    offsetX = e.clientX - rect.left;
    offsetY = e.clientY - rect.top;
    
    // Remove restrições de posicionamento inicial
    floatingBtn.style.left = rect.left + 'px';
    floatingBtn.style.top = rect.top + 'px';
    floatingBtn.style.right = 'auto';
    floatingBtn.style.bottom = 'auto';
    
    e.preventDefault();
  });

  document.addEventListener('mousemove', (e) => {
    if (!isDragging) return;
    
    hasMoved = true;
    
    let x = e.clientX - offsetX;
    let y = e.clientY - offsetY;
    
    // Limitar à tela
    const maxX = window.innerWidth - floatingBtn.offsetWidth;
    const maxY = window.innerHeight - floatingBtn.offsetHeight;
    x = Math.max(0, Math.min(x, maxX));
    y = Math.max(0, Math.min(y, maxY));
    
    floatingBtn.style.left = x + 'px';
    floatingBtn.style.top = y + 'px';
    
    // Reposicionar painel próximo ao botão
    updatePanelPosition();
  });

  document.addEventListener('mouseup', () => {
    if (isDragging) {
      isDragging = false;
      floatingBtn.classList.remove('dragging');
    }
  });

  // Reposicionar painel baseado na posição do botão
  function updatePanelPosition() {
    if (!state.panelOpen) return;
    
    const btnRect = floatingBtn.getBoundingClientRect();
    const panelWidth = 300;
    const panelHeight = hudPanel.offsetHeight || 500;
    const margin = 12;
    
    let panelLeft, panelTop;
    
    // Tentar abrir à direita do botão
    if (btnRect.right + margin + panelWidth <= window.innerWidth) {
      panelLeft = btnRect.right + margin;
    } else {
      // Abrir à esquerda
      panelLeft = btnRect.left - panelWidth - margin;
      if (panelLeft < 0) panelLeft = margin;
    }
    
    // Vertical: alinhar ao topo do botão
    panelTop = btnRect.top;
    if (panelTop + panelHeight > window.innerHeight) {
      panelTop = window.innerHeight - panelHeight - margin;
    }
    if (panelTop < 0) panelTop = margin;
    
    hudPanel.style.left = panelLeft + 'px';
    hudPanel.style.top = panelTop + 'px';
    hudPanel.style.right = 'auto';
    hudPanel.style.bottom = 'auto';
  }

  // ============ CLICK NO BOTÃO (toggle painel) ============
  floatingBtn.addEventListener('click', (e) => {
    if (hasMoved) {
      // Se arrastou, não executa o toggle
      hasMoved = false;
      return;
    }
    
    state.panelOpen = !state.panelOpen;
    
    if (state.panelOpen) {
      hudPanel.classList.remove('hidden');
      floatingBtn.classList.remove('collapsed');
      updatePanelPosition();
    } else {
      hudPanel.classList.add('hidden');
      floatingBtn.classList.add('collapsed');
    }
  });

  // ============ TOGGLES ============
  document.querySelectorAll('.toggle-row').forEach(row => {
    row.addEventListener('click', () => {
      const key = row.dataset.toggle;
      state[key] = !state[key];
      
      if (state[key]) {
        row.classList.add('active');
      } else {
        row.classList.remove('active');
      }
      
      // Lógica específica
      if (key === 'fovCircle') {
        fovCircle.classList.toggle('active', state.fovCircle);
        fovControl.style.display = state.fovCircle ? 'block' : 'none';
        if (state.fovCircle) {
          updateFovCircle();
        }
      }
    });
  });

  // ============ FOV SLIDER ============
  fovSlider.addEventListener('input', (e) => {
    state.fovRadius = parseInt(e.target.value);
    fovValue.textContent = state.fovRadius + 'px';
    updateFovCircle();
  });

  function updateFovCircle() {
    const size = state.fovRadius * 2;
    fovCircle.style.width = size + 'px';
    fovCircle.style.height = size + 'px';
  }

  // ============ FPS COUNTER ============
  let lastFrameTime = performance.now();
  let frameCount = 0;
  let fps = 60;

  function measureFPS() {
    frameCount++;
    const now = performance.now();
    const delta = now - lastFrameTime;
    
    if (delta >= 500) {
      fps = Math.round((frameCount * 1000) / delta);
      fpsCounter.textContent = fps + ' FPS';
      frameCount = 0;
      lastFrameTime = now;
    }
    
    requestAnimationFrame(measureFPS);
  }
  requestAnimationFrame(measureFPS);

  // ============ RESPONSIVO ============
  window.addEventListener('resize', () => {
    // Garantir que botão fique visível
    const rect = floatingBtn.getBoundingClientRect();
    if (rect.right > window.innerWidth || rect.bottom > window.innerHeight) {
      floatingBtn.style.left = '24px';
      floatingBtn.style.top = '24px';
    }
    updatePanelPosition();
  });

  // ============ INICIALIZAÇÃO ============
  updatePanelPosition();
  
  // Ativar alguns toggles para demonstração
  document.querySelector('[data-toggle="teamCheck"]').click();
  document.querySelector('[data-toggle="fovCircle"]').click();
</script>

</body>
</html>
