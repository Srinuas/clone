<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <title>PixelEdit — Interactive Canvas Engine</title>
  <link rel="icon" type="image/jpeg" href="favicon.jpg">
  <!-- Cache-buster added to force new styles -->
  <link rel="stylesheet" href="style.css?v=2.0">
</head>
<body>
  <div id="appBackground"></div>

  <!-- Draggable Universal Navigation Widget -->
  <div id="dragNavWidget" class="drag-nav">
      <div id="dragNavHandle" class="drag-handle-bar">☰ Move Page</div>
      <div class="nav-content">
          <button onclick="window.scrollBy({top: -400, behavior: 'smooth'})" title="Scroll Up">⬆</button>
          <button onclick="window.scrollBy({left: -400, behavior: 'smooth'})" title="Scroll Left">⬅</button>
          <button onclick="window.scrollBy({left: 400, behavior: 'smooth'})" title="Scroll Right">➡</button>
          <button onclick="window.scrollBy({top: 400, behavior: 'smooth'})" title="Scroll Down">⬇</button>
      </div>
  </div>

  <div id="toastNotification" class="toast hidden" style="opacity: 0;">Changes Applied Successfully!</div>

  <div class="app-shell">
    <header class="topbar">
      <div class="brand">
        <div class="brand-mark">✦</div>
        <div><h1>PixelEdit</h1><p>Stateful Memory & Manual Region Engine</p></div>
      </div>
    </header>

    <main class="workspace">
      <!-- LEFT SIDEBAR -->
      <section class="panel controls">
        <div id="uploadSection">
          <div class="section-title"><h2>01. Upload Document</h2></div>
          <label class="dropzone" id="dropzone">
            <input id="fileInput" type="file" accept=".pdf,image/*">
            <div class="upload-icon">↑</div>
            <strong>Drop PDF or image here</strong>
          </label>
        </div>

        <div id="editSection" class="hidden slide-in">
          <div class="mode-toggle">
            <button class="mode-btn active" id="btnAutoMode">Find & Replace</button>
            <button class="mode-btn" id="btnManualMode">Manual Selection</button>
          </div>

          <div id="autoUI">
            <div class="section-title edits-head"><h2>Auto Replace</h2><button class="text-btn" id="addBtn">+ Add</button></div>
            <div id="edits"></div>
            <button class="primary-btn apply-btn" onclick="submitEdit('auto', false)">Replace & Save</button>
          </div>

          <div id="manualUI" class="hidden">
            <div class="section-title edits-head"><h2>Region Editor</h2></div>
            <div class="tip">Draw a box on the document preview to extract and replace text or images.</div>
            
            <div id="manualForm" class="hidden manual-form">
              <div class="info-box" id="scanResults">Scanned: 0 Lines</div>
              
              <div class="manual-tabs">
                  <button id="tabText" class="manual-tab active" onclick="setManualTab('text')">Text</button>
                  <button id="tabImage" class="manual-tab" onclick="setManualTab('image')">Image</button>
              </div>

              <div id="manualTextInput">
                  <input type="text" id="manualSearchInput" placeholder="Target Text inside box (Optional)" style="margin-bottom: 10px;">
                  <div style="display: flex; gap: 10px;">
                      <input type="text" id="manualReplaceInput" placeholder="Replace With..." style="flex: 1; margin-top: 0;">
                      <input type="color" id="manualTextColor" style="height: 38px; width: 45px; border-radius: 6px; background: #111829;" title="Override Color">
                  </div>
                  
                  <div style="display: flex; gap: 5px; margin-top: 10px; margin-bottom: 10px;">
                      <button class="color-swatch" style="background: #000000;" onclick="setColor('#000000')"></button>
                      <button class="color-swatch" style="background: #ef4444;" onclick="setColor('#ef4444')"></button>
                      <button class="color-swatch" style="background: #3b82f6;" onclick="setColor('#3b82f6')"></button>
                      <button class="color-swatch" style="background: #10b981;" onclick="setColor('#10b981')"></button>
                      <button class="color-swatch" id="originalColorSwatch" style="background: #ffffff;" title="Reset Color" onclick="setColor(this.dataset.color || '#000000')">↻</button>
                  </div>
                  
                  <input type="hidden" id="manualOffsetX" value="0">
                  <input type="hidden" id="manualOffsetY" value="0">
                  <input type="hidden" id="manualOffsetSize" value="0">
                  <input type="hidden" id="manualOffsetBold" value="0">
              </div>

              <div id="manualImageInput" class="hidden">
                  <label class="image-upload-btn" for="manualReplaceImage">
                      <span id="imageSelectName">Choose Image...</span>
                      <input type="file" id="manualReplaceImage" accept="image/*" class="hidden">
                  </label>
              </div>

              <button id="previewEditBtn" class="primary-btn apply-btn" onclick="submitEdit('manual', true)">1. Preview Replace</button>

              <div id="nudgeBlock" class="hidden" style="margin-top: 15px; background: #131a2c; padding: 15px; border-radius: 8px;">
                  <div style="display: flex; justify-content: center; gap: 20px; margin-bottom: 15px;">
                      <button class="nudge-btn bold-btn" onclick="nudgeBold(-1)">B-</button>
                      <div class="d-pad" style="display: grid; grid-template-areas: '. up .' 'left . right' '. down .'; gap: 4px;">
                          <button class="nudge-btn" style="grid-area: up;" onclick="nudgePos(0, -0.5)">▲</button>
                          <button class="nudge-btn" style="grid-area: left;" onclick="nudgePos(-0.5, 0)">◀</button>
                          <button class="nudge-btn" style="grid-area: right;" onclick="nudgePos(0.5, 0)">▶</button>
                          <button class="nudge-btn" style="grid-area: down;" onclick="nudgePos(0, 0.5)">▼</button>
                      </div>
                      <button class="nudge-btn bold-btn" onclick="nudgeBold(1)">B+</button>
                  </div>
                  <div id="boldLevelDisplay" style="color: #a78bfa; font-size: 11px; font-weight: bold; margin-bottom: 10px; text-align: center;">Weight: Normal</div>
                  <div style="display: flex; justify-content: center; gap: 10px;">
                      <button class="tool-btn" onclick="nudgeSize(0.5)" style="flex: 1;">🔍 Zoom In (+)</button>
                      <button class="tool-btn" onclick="nudgeSize(-0.5)" style="flex: 1;">🔍 Zoom Out (-)</button>
                  </div>
                  <button class="success-btn apply-btn" style="margin-top: 10px;" onclick="submitEdit('manual', false)">2. Confirm Edit</button>
                  <button class="secondary-btn apply-btn" style="margin-top: 5px;" onclick="cancelPreview()">Cancel</button>
              </div>
            </div>
          </div>

          <hr class="divider">
          <div class="download-grid">
            <button class="success-btn" id="downloadBtn" onclick="downloadFile()">Download Final ↓</button>
            <button class="secondary-btn hidden" id="newEditBtn" onclick="clearFile()">Upload New ⟲</button>
          </div>
        </div>
      </section>

      <!-- RIGHT PANEL: CANVAS VIEWPORT -->
      <section class="panel preview">
        <div class="preview-top">
          <h2>Interactive Canvas</h2>
          <div class="history-controls">
            <div class="zoom-controls">
              <button class="tool-btn" onclick="setZoom(0.2)">🔍 +</button>
              <button class="tool-btn" onclick="setZoom(-0.2)">🔍 -</button>
              <button class="tool-btn" onclick="resetZoom()">🔄</button>
            </div>
            <button class="tool-btn" id="undoBtn" disabled>↶ Undo</button>
            <button class="tool-btn" id="redoBtn" disabled>↷ Redo</button>
          </div>
        </div>
        
        <div class="viewer" id="viewer">
          <div class="empty-state" id="emptyState">Upload a file to begin stateful editing.</div>
          <div class="canvas-viewport hidden fade-in" id="canvasViewport">
              <div class="canvas-container" id="canvasContainer">
                <img id="previewImg" draggable="false">
                <canvas id="drawCanvas"></canvas>
              </div>
          </div>
        </div>
      </section>
    </main>

  </div>

  <div id="adminBtn" class="admin-trigger" onclick="openAdmin()">🛡️</div>

  <!-- Included Modal snippets (Pin, Admin, Confirm, Alert, Preview) -->
  <div id="pinModal" class="admin-modal hidden">
    <div class="pin-content scale-in">
      <h3>Admin Access</h3>
      <input type="password" id="pinInput" placeholder="••••" maxlength="4" autocomplete="off">
      <div class="pin-actions">
        <button class="secondary-btn" onclick="document.getElementById('pinModal').classList.add('hidden')">Cancel</button>
        <button class="primary-btn" id="submitPinBtn" onclick="submitPin()">Unlock 🔓</button>
      </div>
    </div>
  </div>

  <!-- Cache-buster added to force new JS -->
  <script src="script.js?v=2.0"></script>
</body>
</html>



const $ = (s) => document.querySelector(s);
const $$ = (s) => document.querySelectorAll(s);

let session = { id: null, nativeWidth: 0, nativeHeight: 0 };
let currentMode = 'auto'; 
let manualType = 'text'; 
let manualBoxNative = null; 
let suggestedFontSize = 12;
let manualTextProps = { angle: 0, is_bold: false, color: [0,0,0] };
let manualReplaceImageBase64 = null;
let currentZoom = 1; 

// FIXED: Removed 'assets/' path
let currentBg = 'bg_img_1.jpg';

// Draggable Widget Logic
const dragNav = $("#dragNavWidget");
const dragHandle = $("#dragNavHandle");
let isNavDragging = false;
let navStartX, navStartY, initX, initY;

dragHandle.addEventListener('mousedown', (e) => {
    isNavDragging = true;
    dragNav.classList.add("operating");
    navStartX = e.clientX; navStartY = e.clientY;
    const rect = dragNav.getBoundingClientRect();
    initX = rect.left; initY = rect.top;
});
document.addEventListener('mousemove', (e) => {
    if (!isNavDragging) return;
    const dx = e.clientX - navStartX;
    const dy = e.clientY - navStartY;
    dragNav.style.right = 'auto';
    dragNav.style.bottom = 'auto';
    dragNav.style.left = `${initX + dx}px`;
    dragNav.style.top = `${initY + dy}px`;
});
document.addEventListener('mouseup', () => { 
    isNavDragging = false; 
    dragNav.classList.remove("operating"); 
});
dragNav.addEventListener('mouseenter', () => dragNav.classList.add("operating"));
dragNav.addEventListener('mouseleave', () => dragNav.classList.remove("operating"));

function setBackground(imgName) {
    if (currentBg === imgName && $("#appBackground").style.backgroundImage) return;
    currentBg = imgName;
    const bgDiv = $('#appBackground');
    bgDiv.style.opacity = 0;
    setTimeout(() => {
        bgDiv.style.backgroundImage = `url('${imgName}')`;
        bgDiv.style.opacity = 1;
    }, 500); 
}

// FIXED: Removed 'assets/' path
document.addEventListener("DOMContentLoaded", () => setBackground('bg_img_1.jpg'));

// Canvas Initialization
const drawCanvas = $("#drawCanvas");
const ctx = drawCanvas.getContext("2d");
let isDrawing = false, startX, startY, currentX, currentY;

function getCoords(clientX, clientY) {
    const rect = drawCanvas.getBoundingClientRect();
    const scaleX = drawCanvas.width / rect.width;
    const scaleY = drawCanvas.height / rect.height;
    return { x: (clientX - rect.left) * scaleX, y: (clientY - rect.top) * scaleY };
}

drawCanvas.addEventListener("mousedown", (e) => {
  if (currentMode !== 'manual') return;
  const pos = getCoords(e.clientX, e.clientY);
  startX = pos.x; startY = pos.y;
  isDrawing = true;
});

drawCanvas.addEventListener("mousemove", (e) => {
  if (!isDrawing) return;
  const pos = getCoords(e.clientX, e.clientY);
  currentX = pos.x; currentY = pos.y;
  ctx.clearRect(0, 0, drawCanvas.width, drawCanvas.height);
  ctx.strokeStyle = "#ff3366"; ctx.lineWidth = 2; ctx.fillStyle = "rgba(255, 51, 102, 0.2)";
  ctx.fillRect(startX, startY, currentX - startX, currentY - startY);
  ctx.strokeRect(startX, startY, currentX - startX, currentY - startY);
});

drawCanvas.addEventListener("mouseup", async () => {
  if (!isDrawing) return;
  isDrawing = false;
  
  const scaleXNative = session.nativeWidth / drawCanvas.width;
  const scaleYNative = session.nativeHeight / drawCanvas.height;
  
  manualBoxNative = [
      Math.min(startX, currentX) * scaleXNative, Math.min(startY, currentY) * scaleYNative,
      Math.max(startX, currentX) * scaleXNative, Math.max(startY, currentY) * scaleYNative
  ];

  $("#manualForm").classList.remove("hidden");
  $("#nudgeBlock").classList.remove("hidden");
  
  const res = await fetch("/api/extract", {
    method: "POST", headers: {"Content-Type": "application/json"},
    body: JSON.stringify({ session_id: session.id, box: manualBoxNative })
  });
  
  const data = await res.json();
  suggestedFontSize = data.suggested_font_size || 12;
  manualTextProps.angle = data.angle || 0;
  manualTextProps.is_bold = data.is_bold || false;
  manualTextProps.color = data.color || [0,0,0];
  
  const toHex = (c) => { let h = Math.max(0, Math.min(255, Math.round(c * 255))).toString(16); return h.length === 1 ? "0"+h : h; };
  $("#manualTextColor").value = `#${toHex(manualTextProps.color[0])}${toHex(manualTextProps.color[1])}${toHex(manualTextProps.color[2])}`;
});

window.submitEdit = async function(mode, isPreview = false) {
  let edits = [];
  if (mode === 'auto') {
      edits = [...$$(".edit-row")].map(row => ({
          search: row.querySelector(".search-input").value.trim(), replace: row.querySelector(".replace-input").value
      })).filter(x => x.search);
  } else if (mode === 'manual') {
      if (!manualBoxNative) return alert("Draw a box first.");
      edits = [{ 
          type: manualType, box: manualBoxNative, 
          search: $("#manualSearchInput").value, replace: $("#manualReplaceInput").value, 
          replace_image: manualReplaceImageBase64,
          offset_x: parseFloat($("#manualOffsetX").value) || 0, offset_y: parseFloat($("#manualOffsetY").value) || 0,
          offset_size: parseFloat($("#manualOffsetSize").value) || 0, offset_bold: parseInt($("#manualOffsetBold").value) || 0,
          font_size: suggestedFontSize, angle: manualTextProps.angle, is_bold: manualTextProps.is_bold
      }];
  }

  const res = await fetch("/api/edit", {
      method: "POST", headers: {"Content-Type": "application/json"},
      body: JSON.stringify({ session_id: session.id, mode: mode, edits: edits, is_preview: isPreview })
  });

  const data = await res.json();
  if (data.status === "preview") {
      $("#previewImg").src = "data:image/png;base64," + data.image;
      ctx.clearRect(0,0, drawCanvas.width, drawCanvas.height);
  } else {
      loadPreview();
      ctx.clearRect(0,0, drawCanvas.width, drawCanvas.height);
      $("#manualForm").classList.add("hidden");
      $("#toastNotification").classList.remove("hidden");
      $("#toastNotification").style.opacity = 1;
      setTimeout(() => $("#toastNotification").style.opacity = 0, 3000);
  }
}


# 1. Stop and remove the old container
docker rm -f app 2>/dev/null || true

# 2. Remove the old image to guarantee a fresh copy
docker rmi srinuas/pdfimg:1 2>/dev/null || true

# 3. Build the new lightweight image (forcing it to ignore Docker's build cache)
docker build --no-cache -t srinuas/pdfimg:1 .

# 4. Run the container securely
docker run -itd --name app -p 333:3000 --env-file .env srinuas/pdfimg:1
