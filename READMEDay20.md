# Day20
AI-Powered Face Puzzle Game with Claude
You are an expert front-end developer. Build me a complete, fully working face puzzle game as a single self-contained HTML file (no external dependencies except what can load from cdnjs.cloudflare.com, cdn.jsdelivr.net, or unpkg.com).

FEATURES REQUIRED — deliver ALL of these in one complete response:

1. CAMERA ACCESS
   - On load, request webcam permission using getUserMedia()
   - Show a live video preview (front-facing camera preferred)
   - Display a 'Take Photo' button to snapshot the user's face onto a canvas

2. PUZZLE GENERATION
   - After snapshot, let the user choose difficulty: 3×3, 4×4, or 5×5 grid
   - Slice the captured face image into equal puzzle pieces
   - Randomly scramble the pieces (guarantee it is solvable)
   - Render each piece as a draggable tile at its scrambled position

3. DRAG & TOUCH GESTURE CONTROLS
   - Support both mouse drag (desktop) and touch drag (mobile/tablet)
   - When a piece is dropped onto another piece's cell, swap their positions
   - Snap pieces to the nearest grid cell on release
   - Highlight a piece with a coloured border while it is being dragged
   - Show a green border on pieces that land in their correct position

4. TIMER & MOVE COUNTER
   - Start the timer the moment the puzzle begins
   - Display elapsed time live (format: mm:ss.t)
   - Count and display total moves made
   - Show how many pieces are correctly placed out of the total

5. WIN DETECTION & RESULTS SCREEN
   - Detect automatically when all pieces are in the correct position
   - Stop the timer immediately on win
   - Show a results overlay with: final time, total moves, and difficulty
   - Save the top 5 best times to localStorage with date, time, moves, and difficulty
   - Display a leaderboard of saved best times

6. UI & POLISH
   - Clean, modern design
   - Works on desktop and mobile
   - 'Retake Photo' button
   - 'Play Again' button
   - 'New Photo' button
   - Responsive layout

TECHNICAL REQUIREMENTS:
- Single HTML file
- All CSS and JS inline
- No frameworks
- Must work in Chrome, Firefox, and Safari
- Camera must work over HTTPS or localhost
- Handle camera permission denied gracefully
- Do NOT leave placeholder comments

Output the complete HTML file in one code block. Do not truncate or summarise any section.

solution : [face-puzzle.html](https://github.com/user-attachments/files/29158836/face-puzzle.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<title>Face Puzzle</title>
<style>
  :root{
    --accent:#6d5dfc;
    --accent-dark:#5142d4;
    --accent-light:#a99bff;
    --success:#22c55e;
    --danger:#ef4444;
    --warning:#f59e0b;
    --bg-card:#ffffff;
    --text-dark:#1f2330;
    --text-muted:#6b7280;
    --radius-lg:20px;
    --radius-md:12px;
  }
  *{box-sizing:border-box;}
  html,body{
    margin:0;padding:0;
    font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif;
    background:linear-gradient(135deg,#6d5dfc 0%,#8e6ff0 45%,#c46ee0 100%);
    min-height:100vh;
    color:var(--text-dark);
    -webkit-tap-highlight-color:transparent;
    touch-action:manipulation;
  }
  #app{
    min-height:100vh;
    display:flex;
    flex-direction:column;
    align-items:center;
    padding:18px 12px 40px;
  }
  header.app-header{
    text-align:center;
    color:#fff;
    margin-bottom:18px;
    user-select:none;
  }
  header.app-header h1{
    margin:0;
    font-size:clamp(1.6rem,5vw,2.3rem);
    font-weight:800;
    letter-spacing:-0.5px;
    text-shadow:0 2px 12px rgba(0,0,0,.2);
  }
  header.app-header p{
    margin:4px 0 0;
    font-size:.95rem;
    opacity:.9;
  }

  .stage{display:none;}
  .stage.active{display:flex;}
  .stage{
    flex-direction:column;
    align-items:center;
    width:100%;
    max-width:640px;
  }

  .card{
    background:var(--bg-card);
    border-radius:var(--radius-lg);
    box-shadow:0 20px 50px rgba(20,10,60,.25);
    padding:24px;
    width:100%;
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:16px;
    animation:fadeIn .35s ease;
  }
  @keyframes fadeIn{
    from{opacity:0;transform:translateY(8px);}
    to{opacity:1;transform:translateY(0);}
  }

  .video-frame{
    position:relative;
    width:100%;
    max-width:420px;
    aspect-ratio:1/1;
    border-radius:16px;
    overflow:hidden;
    background:#0f1117;
    box-shadow:inset 0 0 0 4px rgba(109,93,252,.15);
  }
  #video{
    width:100%;
    height:100%;
    object-fit:cover;
    transform:scaleX(-1);
    display:block;
  }
  .video-loading{
    position:absolute;inset:0;
    display:flex;
    flex-direction:column;
    gap:10px;
    align-items:center;
    justify-content:center;
    color:#cfd2e6;
    font-size:.9rem;
    background:rgba(15,17,23,.55);
  }
  .spinner{
    width:34px;height:34px;
    border:4px solid rgba(255,255,255,.25);
    border-top-color:#fff;
    border-radius:50%;
    animation:spin .9s linear infinite;
  }
  @keyframes spin{to{transform:rotate(360deg);}}

  .camera-error{
    display:none;
    width:100%;
    max-width:420px;
    background:#fef2f2;
    border:1px solid #fecaca;
    color:#991b1b;
    border-radius:var(--radius-md);
    padding:14px 16px;
    font-size:.92rem;
    line-height:1.4;
    text-align:left;
  }
  .camera-error button{
    margin-top:10px;
  }

  .btn{
    border:none;
    border-radius:999px;
    padding:12px 22px;
    font-size:.95rem;
    font-weight:700;
    cursor:pointer;
    transition:transform .12s ease, box-shadow .12s ease, background .15s ease, opacity .15s ease;
    display:inline-flex;
    align-items:center;
    gap:8px;
    user-select:none;
  }
  .btn:active{transform:scale(.96);}
  .btn:disabled{opacity:.5;cursor:not-allowed;}
  .btn-primary{
    background:linear-gradient(135deg,var(--accent),var(--accent-dark));
    color:#fff;
    box-shadow:0 8px 20px rgba(109,93,252,.4);
  }
  .btn-primary:hover:not(:disabled){box-shadow:0 10px 24px rgba(109,93,252,.55);}
  .btn-secondary{
    background:#f1f0ff;
    color:var(--accent-dark);
  }
  .btn-secondary:hover{background:#e6e3ff;}
  .btn-ghost{
    background:rgba(255,255,255,.15);
    color:#fff;
    border:1px solid rgba(255,255,255,.4);
    padding:8px 14px;
    font-size:.82rem;
  }
  .btn-ghost:hover{background:rgba(255,255,255,.28);}
  .btn-danger{
    background:#fff1f1;
    color:var(--danger);
  }

  .btn-row{
    display:flex;
    flex-wrap:wrap;
    gap:10px;
    justify-content:center;
    width:100%;
  }

  /* ---- Preview stage ---- */
  #preview-img-wrap{
    width:100%;
    max-width:380px;
    aspect-ratio:1/1;
    border-radius:16px;
    overflow:hidden;
    box-shadow:0 6px 24px rgba(0,0,0,.18);
  }
  #preview-img{
    width:100%;height:100%;
    object-fit:cover;
    display:block;
  }
  .diff-label{
    font-weight:700;
    color:var(--text-muted);
    font-size:.85rem;
    text-transform:uppercase;
    letter-spacing:.06em;
    margin-top:4px;
  }
  .diff-row{
    display:flex;
    gap:10px;
  }
  .diff-btn{
    border:2px solid #e3e1fb;
    background:#fff;
    color:var(--accent-dark);
    border-radius:14px;
    padding:10px 18px;
    font-weight:700;
    font-size:.95rem;
    cursor:pointer;
    transition:all .15s ease;
  }
  .diff-btn:hover{border-color:var(--accent-light);}
  .diff-btn.active{
    background:var(--accent);
    border-color:var(--accent);
    color:#fff;
    box-shadow:0 6px 16px rgba(109,93,252,.4);
  }

  /* ---- Puzzle stage ---- */
  .puzzle-topbar{
    width:100%;
    display:flex;
    justify-content:flex-end;
    margin-bottom:10px;
  }
  .stats-bar{
    width:100%;
    display:flex;
    justify-content:space-between;
    gap:8px;
    background:rgba(255,255,255,.92);
    border-radius:16px;
    padding:14px 16px;
    margin-bottom:14px;
    box-shadow:0 8px 24px rgba(20,10,60,.2);
    flex-wrap:wrap;
  }
  .stat{
    display:flex;
    flex-direction:column;
    align-items:center;
    min-width:64px;
    flex:1;
  }
  .stat .label{
    font-size:.68rem;
    text-transform:uppercase;
    letter-spacing:.05em;
    color:var(--text-muted);
    font-weight:700;
  }
  .stat .value{
    font-size:1.15rem;
    font-weight:800;
    color:var(--text-dark);
    font-variant-numeric:tabular-nums;
  }

  #puzzle-board{
    position:relative;
    margin:0 auto;
    border-radius:14px;
    overflow:hidden;
    background:#0f1117;
    box-shadow:0 18px 40px rgba(20,10,60,.3);
  }
  .tile{
    position:absolute;
    box-sizing:border-box;
    border:1px solid rgba(255,255,255,.28);
    cursor:grab;
    touch-action:none;
    user-select:none;
    -webkit-user-select:none;
    background-repeat:no-repeat;
    transition:left .25s cubic-bezier(.22,.8,.32,1), top .25s cubic-bezier(.22,.8,.32,1), box-shadow .2s ease, transform .15s ease, border-color .15s ease;
    will-change:left, top;
  }
  .tile.dragging{
    transition:box-shadow .15s ease, transform .15s ease;
    cursor:grabbing;
    z-index:1000;
    border:3px solid var(--warning);
    box-shadow:0 14px 30px rgba(0,0,0,.4);
    transform:scale(1.05);
  }
  .tile.correct{
    border:3px solid var(--success);
    box-shadow:0 0 0 2px rgba(34,197,94,.25);
  }

  /* ---- Results overlay ---- */
  .overlay{
    position:fixed;
    inset:0;
    background:rgba(15,17,30,.72);
    display:flex;
    align-items:center;
    justify-content:center;
    padding:18px;
    opacity:0;
    pointer-events:none;
    transition:opacity .25s ease;
    z-index:3000;
  }
  .overlay.visible{
    opacity:1;
    pointer-events:auto;
  }
  .result-card{
    background:#fff;
    border-radius:22px;
    padding:28px 26px;
    width:100%;
    max-width:420px;
    text-align:center;
    box-shadow:0 30px 80px rgba(0,0,0,.4);
    max-height:92vh;
    overflow-y:auto;
    animation:popIn .3s cubic-bezier(.2,.9,.3,1.2);
  }
  @keyframes popIn{
    from{opacity:0; transform:scale(.85);}
    to{opacity:1; transform:scale(1);}
  }
  .result-card h2{
    margin:0 0 4px;
    font-size:1.6rem;
    color:var(--accent-dark);
  }
  .new-best-banner{
    display:none;
    background:linear-gradient(135deg,#fde68a,#f59e0b);
    color:#78350f;
    font-weight:800;
    border-radius:10px;
    padding:8px 12px;
    font-size:.88rem;
    margin:10px 0;
  }
  .result-stats{
    display:flex;
    justify-content:space-around;
    gap:8px;
    margin:14px 0 6px;
  }
  .result-stats .stat .value{font-size:1.3rem;}
  .leaderboard-wrap{
    margin-top:16px;
    text-align:left;
  }
  .leaderboard-wrap h3{
    margin:0 0 8px;
    font-size:1rem;
    color:var(--text-dark);
  }
  .lb-tabs{
    display:flex;
    gap:6px;
    margin-bottom:10px;
  }
  .lb-tab{
    flex:1;
    border:none;
    background:#f1f0ff;
    color:var(--accent-dark);
    font-weight:700;
    padding:7px 0;
    border-radius:8px;
    font-size:.85rem;
    cursor:pointer;
  }
  .lb-tab.active{
    background:var(--accent);
    color:#fff;
  }
  table.leaderboard{
    width:100%;
    border-collapse:collapse;
    font-size:.86rem;
  }
  table.leaderboard th{
    text-align:left;
    color:var(--text-muted);
    font-size:.7rem;
    text-transform:uppercase;
    letter-spacing:.04em;
    padding:4px 6px;
    border-bottom:1px solid #eee;
  }
  table.leaderboard td{
    padding:7px 6px;
    border-bottom:1px solid #f3f3f7;
    font-variant-numeric:tabular-nums;
  }
  .empty-row{
    text-align:center;
    color:var(--text-muted);
    padding:14px 0;
  }

  footer.note{
    color:rgba(255,255,255,.75);
    font-size:.78rem;
    margin-top:24px;
    text-align:center;
    max-width:420px;
  }

  @media (max-width:480px){
    .card{padding:18px;}
    .result-card{padding:22px 18px;}
  }
</style>
</head>
<body>
<div id="app">
  <header class="app-header">
    <h1>🧩 Face Puzzle</h1>
    <p>Snap a selfie, scramble it, and race to put your face back together.</p>
  </header>

  <!-- CAMERA STAGE -->
  <section class="stage active" id="stage-camera">
    <div class="card">
      <div class="video-frame">
        <video id="video" autoplay muted playsinline></video>
        <div class="video-loading" id="video-loading">
          <div class="spinner"></div>
          <span>Requesting camera access…</span>
        </div>
      </div>
      <div class="camera-error" id="camera-error">
        <strong>Camera unavailable.</strong>
        <div id="camera-error-msg"></div>
        <button class="btn btn-secondary" id="retry-camera-btn">🔄 Try Again</button>
      </div>
      <div class="btn-row">
        <button class="btn btn-primary" id="take-photo-btn" disabled>📸 Take Photo</button>
      </div>
    </div>
    <footer class="note">Your camera feed stays on your device — nothing is uploaded anywhere. Requires camera permission over HTTPS (or localhost).</footer>
  </section>

  <!-- PREVIEW / DIFFICULTY STAGE -->
  <section class="stage" id="stage-preview">
    <div class="card">
      <div id="preview-img-wrap">
        <img id="preview-img" alt="Captured photo preview">
      </div>
      <div class="diff-label">Choose difficulty</div>
      <div class="diff-row">
        <button class="diff-btn" data-size="3">3×3</button>
        <button class="diff-btn active" data-size="4">4×4</button>
        <button class="diff-btn" data-size="5">5×5</button>
      </div>
      <div class="btn-row">
        <button class="btn btn-secondary" id="retake-photo-btn">📷 Retake Photo</button>
        <button class="btn btn-primary" id="start-puzzle-btn">▶ Start Puzzle</button>
      </div>
    </div>
  </section>

  <!-- PUZZLE STAGE -->
  <section class="stage" id="stage-puzzle">
    <div class="puzzle-topbar">
      <button class="btn btn-ghost" id="mid-retake-btn">↺ Retake Photo</button>
    </div>
    <div class="stats-bar">
      <div class="stat">
        <span class="label">Time</span>
        <span class="value" id="stat-time">00:00.0</span>
      </div>
      <div class="stat">
        <span class="label">Moves</span>
        <span class="value" id="stat-moves">0</span>
      </div>
      <div class="stat">
        <span class="label">Correct</span>
        <span class="value" id="stat-correct">0 / 0</span>
      </div>
      <div class="stat">
        <span class="label">Grid</span>
        <span class="value" id="stat-difficulty">4×4</span>
      </div>
    </div>
    <div id="puzzle-board"></div>
  </section>

  <!-- RESULTS OVERLAY -->
  <div class="overlay" id="results-overlay">
    <div class="result-card">
      <h2>🎉 Puzzle Solved!</h2>
      <p style="color:var(--text-muted);margin:0;">Great job putting your face back together.</p>
      <div class="new-best-banner" id="new-best-banner">🏆 New Best Time for this difficulty!</div>
      <div class="result-stats">
        <div class="stat">
          <span class="label">Time</span>
          <span class="value" id="final-time">00:00.0</span>
        </div>
        <div class="stat">
          <span class="label">Moves</span>
          <span class="value" id="final-moves">0</span>
        </div>
        <div class="stat">
          <span class="label">Grid</span>
          <span class="value" id="final-difficulty">4×4</span>
        </div>
      </div>
      <div class="btn-row">
        <button class="btn btn-secondary" id="new-photo-btn">📷 New Photo</button>
        <button class="btn btn-primary" id="play-again-btn">🔁 Play Again</button>
      </div>
      <div class="leaderboard-wrap">
        <h3>🏆 Best Times</h3>
        <div class="lb-tabs">
          <button class="lb-tab" data-size="3">3×3</button>
          <button class="lb-tab" data-size="4">4×4</button>
          <button class="lb-tab" data-size="5">5×5</button>
        </div>
        <table class="leaderboard">
          <thead>
            <tr><th>#</th><th>Time</th><th>Moves</th><th>Date</th></tr>
          </thead>
          <tbody id="leaderboard-body"></tbody>
        </table>
      </div>
    </div>
  </div>

  <canvas id="capture-canvas" style="display:none;"></canvas>
</div>

<script>
(function(){
  'use strict';

  /* ---------------- State ---------------- */
  var stream = null;
  var capturedImage = null;
  var selectedGridSize = 4;
  var gridSize = 4;

  var slotContents = [];   // slotContents[slot] = pieceId
  var pieceSlot = [];      // pieceSlot[pieceId] = slot
  var pieceEls = [];       // pieceEls[pieceId] = DOM element

  var boardSizePx = 480;
  var pieceSizePx = 120;

  var moveCount = 0;
  var timerInterval = null;
  var startTime = 0;
  var elapsedMs = 0;
  var gameActive = false;
  var dragData = null;
  var currentLeaderboardTab = 4;

  /* ---------------- DOM refs ---------------- */
  var video = document.getElementById('video');
  var videoLoading = document.getElementById('video-loading');
  var cameraError = document.getElementById('camera-error');
  var cameraErrorMsg = document.getElementById('camera-error-msg');
  var takePhotoBtn = document.getElementById('take-photo-btn');
  var retryCameraBtn = document.getElementById('retry-camera-btn');
  var captureCanvas = document.getElementById('capture-canvas');
  var previewImg = document.getElementById('preview-img');
  var retakePhotoBtn = document.getElementById('retake-photo-btn');
  var midRetakeBtn = document.getElementById('mid-retake-btn');
  var startPuzzleBtn = document.getElementById('start-puzzle-btn');
  var boardEl = document.getElementById('puzzle-board');
  var timeEl = document.getElementById('stat-time');
  var movesEl = document.getElementById('stat-moves');
  var correctCountEl = document.getElementById('stat-correct');
  var difficultyLabelEl = document.getElementById('stat-difficulty');
  var resultsOverlay = document.getElementById('results-overlay');
  var finalTimeEl = document.getElementById('final-time');
  var finalMovesEl = document.getElementById('final-moves');
  var finalDifficultyEl = document.getElementById('final-difficulty');
  var newBestBanner = document.getElementById('new-best-banner');
  var newPhotoBtn = document.getElementById('new-photo-btn');
  var playAgainBtn = document.getElementById('play-again-btn');
  var leaderboardBody = document.getElementById('leaderboard-body');

  /* ---------------- Utilities ---------------- */
  function pad(n){ return n < 10 ? '0' + n : '' + n; }

  function formatTime(ms){
    var totalSeconds = Math.max(0, ms) / 1000;
    var mm = Math.floor(totalSeconds / 60);
    var ss = Math.floor(totalSeconds % 60);
    var t = Math.floor((ms % 1000) / 100);
    return pad(mm) + ':' + pad(ss) + '.' + t;
  }

  function debounce(fn, wait){
    var timer;
    return function(){
      var args = arguments, ctx = this;
      clearTimeout(timer);
      timer = setTimeout(function(){ fn.apply(ctx, args); }, wait);
    };
  }

  function showScreen(name){
    ['camera', 'preview', 'puzzle'].forEach(function(s){
      document.getElementById('stage-' + s).classList.toggle('active', s === name);
    });
  }

  function shuffleArray(arr){
    for (var i = arr.length - 1; i > 0; i--){
      var j = Math.floor(Math.random() * (i + 1));
      var tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
    }
    return arr;
  }

  function countCorrect(arr){
    var c = 0;
    for (var i = 0; i < arr.length; i++){ if (arr[i] === i) c++; }
    return c;
  }

  function generateScramble(n){
    var arr = [];
    for (var i = 0; i < n; i++) arr.push(i);
    var attempts = 0;
    var maxAllowedCorrect = Math.max(1, Math.floor(n * 0.3));
    do {
      shuffleArray(arr);
      attempts++;
    } while ((countCorrect(arr) > maxAllowedCorrect || countCorrect(arr) === n) && attempts < 200);
    return arr;
  }

  /* ---------------- Camera handling ---------------- */
  function showCameraError(msg){
    cameraErrorMsg.textContent = msg;
    cameraError.style.display = 'block';
    videoLoading.style.display = 'none';
    takePhotoBtn.disabled = true;
  }

  function hideCameraError(){
    cameraError.style.display = 'none';
  }

  function initCamera(){
    hideCameraError();
    takePhotoBtn.disabled = true;
    videoLoading.style.display = 'flex';
    videoLoading.querySelector('span').textContent = 'Requesting camera access…';

    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia){
      showCameraError('Camera access is not supported in this browser. Please try a recent version of Chrome, Firefox, or Safari on a device with a camera.');
      return;
    }

    if (stream){
      stream.getTracks().forEach(function(t){ t.stop(); });
      stream = null;
    }

    navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user', width: { ideal: 1280 }, height: { ideal: 1280 } },
      audio: false
    }).then(function(s){
      stream = s;
      video.srcObject = stream;
      var playPromise = video.play();
      if (playPromise && playPromise.catch) playPromise.catch(function(){});
    }).catch(function(err){
      var msg = 'Unable to access the camera.';
      if (err && err.name === 'NotAllowedError'){
        msg = 'Camera access was denied. Please allow camera permissions for this site in your browser settings, then click "Try Again".';
      } else if (err && err.name === 'NotFoundError'){
        msg = 'No camera was found on this device.';
      } else if (err && err.name === 'NotReadableError'){
        msg = 'The camera appears to be in use by another application.';
      } else if (location.protocol !== 'https:' && location.hostname !== 'localhost'){
        msg = 'Camera access requires a secure connection (HTTPS) or localhost. Please load this page over HTTPS.';
      }
      showCameraError(msg);
    });
  }

  video.addEventListener('loadedmetadata', function(){
    videoLoading.style.display = 'none';
    takePhotoBtn.disabled = false;
  });

  retryCameraBtn.addEventListener('click', initCamera);

  function takePhoto(){
    if (!video.videoWidth || !video.videoHeight) return;
    var vw = video.videoWidth, vh = video.videoHeight;
    var size = Math.min(vw, vh);
    var sx = (vw - size) / 2, sy = (vh - size) / 2;
    var OUT = 720;
    captureCanvas.width = OUT;
    captureCanvas.height = OUT;
    var ctx = captureCanvas.getContext('2d');
    ctx.save();
    ctx.translate(OUT, 0);
    ctx.scale(-1, 1);
    ctx.drawImage(video, sx, sy, size, size, 0, 0, OUT, OUT);
    ctx.restore();
    capturedImage = captureCanvas.toDataURL('image/jpeg', 0.92);
    previewImg.src = capturedImage;

    if (stream){
      stream.getTracks().forEach(function(t){ t.stop(); });
      stream = null;
    }
    showScreen('preview');
  }

  takePhotoBtn.addEventListener('click', takePhoto);

  function retakePhoto(){
    if (gameActive){
      gameActive = false;
      stopTimer();
    }
    capturedImage = null;
    showScreen('camera');
    initCamera();
  }
  retakePhotoBtn.addEventListener('click', retakePhoto);
  midRetakeBtn.addEventListener('click', retakePhoto);

  /* ---------------- Difficulty selection ---------------- */
  var diffBtns = Array.prototype.slice.call(document.querySelectorAll('.diff-btn'));
  diffBtns.forEach(function(btn){
    btn.addEventListener('click', function(){
      diffBtns.forEach(function(b){ b.classList.remove('active'); });
      btn.classList.add('active');
      selectedGridSize = parseInt(btn.getAttribute('data-size'), 10);
    });
  });

  startPuzzleBtn.addEventListener('click', function(){
    startPuzzle();
  });

  /* ---------------- Board geometry ---------------- */
  function computeBoardSize(){
    var available = Math.min(window.innerWidth * 0.92, window.innerHeight * 0.56, 560);
    pieceSizePx = Math.floor(available / gridSize);
    if (pieceSizePx < 30) pieceSizePx = 30;
    boardSizePx = pieceSizePx * gridSize;
  }

  function layoutTile(id){
    var slot = pieceSlot[id];
    var row = Math.floor(slot / gridSize), col = slot % gridSize;
    var el = pieceEls[id];
    el.style.left = (col * pieceSizePx) + 'px';
    el.style.top = (row * pieceSizePx) + 'px';
  }

  function buildBoard(){
    boardEl.innerHTML = '';
    pieceEls = [];
    var total = gridSize * gridSize;
    boardEl.style.width = boardSizePx + 'px';
    boardEl.style.height = boardSizePx + 'px';

    for (var id = 0; id < total; id++){
      var correctRow = Math.floor(id / gridSize), correctCol = id % gridSize;
      var tile = document.createElement('div');
      tile.className = 'tile';
      tile.setAttribute('data-id', id);
      tile.style.width = pieceSizePx + 'px';
      tile.style.height = pieceSizePx + 'px';
      tile.style.backgroundImage = 'url(' + capturedImage + ')';
      tile.style.backgroundSize = boardSizePx + 'px ' + boardSizePx + 'px';
      tile.style.backgroundPosition = (-correctCol * pieceSizePx) + 'px ' + (-correctRow * pieceSizePx) + 'px';
      (function(pieceId, el){
        el.addEventListener('pointerdown', function(e){ onPointerDown(e, pieceId); });
      })(id, tile);
      boardEl.appendChild(tile);
      pieceEls[id] = tile;
    }

    for (var i = 0; i < total; i++) layoutTile(i);
    updateAllCorrectness();
  }

  function relayoutBoard(){
    if (!pieceEls.length) return;
    computeBoardSize();
    boardEl.style.width = boardSizePx + 'px';
    boardEl.style.height = boardSizePx + 'px';
    for (var id = 0; id < pieceEls.length; id++){
      var el = pieceEls[id];
      var correctRow = Math.floor(id / gridSize), correctCol = id % gridSize;
      el.style.width = pieceSizePx + 'px';
      el.style.height = pieceSizePx + 'px';
      el.style.backgroundSize = boardSizePx + 'px ' + boardSizePx + 'px';
      el.style.backgroundPosition = (-correctCol * pieceSizePx) + 'px ' + (-correctRow * pieceSizePx) + 'px';
      layoutTile(id);
    }
  }
  window.addEventListener('resize', debounce(relayoutBoard, 150));
  window.addEventListener('orientationchange', debounce(relayoutBoard, 150));

  /* ---------------- Correctness / win ---------------- */
  function updateCorrectness(id){
    var isCorrect = pieceSlot[id] === id;
    pieceEls[id].classList.toggle('correct', isCorrect);
  }

  function updateAllCorrectness(){
    var count = 0;
    for (var id = 0; id < pieceEls.length; id++){
      updateCorrectness(id);
      if (pieceSlot[id] === id) count++;
    }
    correctCountEl.textContent = count + ' / ' + pieceEls.length;
    return count;
  }

  /* ---------------- Drag & touch (Pointer Events) ---------------- */
  function onPointerDown(e, id){
    if (!gameActive) return;
    e.preventDefault();
    var tile = pieceEls[id];
    try { tile.setPointerCapture(e.pointerId); } catch (err){}
    var startLeft = parseFloat(tile.style.left) || 0;
    var startTop = parseFloat(tile.style.top) || 0;
    dragData = {
      id: id,
      pointerId: e.pointerId,
      startClientX: e.clientX,
      startClientY: e.clientY,
      startLeft: startLeft,
      startTop: startTop
    };
    tile.classList.add('dragging');
    tile.style.zIndex = 1000;
    tile.addEventListener('pointermove', onPointerMove);
    tile.addEventListener('pointerup', onPointerUp);
    tile.addEventListener('pointercancel', onPointerUp);
  }

  function onPointerMove(e){
    if (!dragData || e.pointerId !== dragData.pointerId) return;
    var dx = e.clientX - dragData.startClientX;
    var dy = e.clientY - dragData.startClientY;
    var tile = pieceEls[dragData.id];
    tile.style.left = (dragData.startLeft + dx) + 'px';
    tile.style.top = (dragData.startTop + dy) + 'px';
  }

  function onPointerUp(e){
    if (!dragData || e.pointerId !== dragData.pointerId) return;
    var id = dragData.id;
    var tile = pieceEls[id];

    tile.removeEventListener('pointermove', onPointerMove);
    tile.removeEventListener('pointerup', onPointerUp);
    tile.removeEventListener('pointercancel', onPointerUp);
    tile.classList.remove('dragging');
    tile.style.zIndex = '';

    var left = parseFloat(tile.style.left) || 0;
    var top = parseFloat(tile.style.top) || 0;
    var centerX = left + pieceSizePx / 2;
    var centerY = top + pieceSizePx / 2;
    var col = Math.floor(centerX / pieceSizePx);
    var row = Math.floor(centerY / pieceSizePx);
    col = Math.max(0, Math.min(gridSize - 1, col));
    row = Math.max(0, Math.min(gridSize - 1, row));
    var targetSlot = row * gridSize + col;
    var currentSlot = pieceSlot[id];

    if (targetSlot !== currentSlot){
      var otherId = slotContents[targetSlot];
      slotContents[currentSlot] = otherId;
      slotContents[targetSlot] = id;
      pieceSlot[id] = targetSlot;
      pieceSlot[otherId] = currentSlot;
      layoutTile(id);
      layoutTile(otherId);

      moveCount++;
      movesEl.textContent = moveCount;

      var correct = updateAllCorrectness();
      if (correct === pieceEls.length){
        onWin();
      }
    } else {
      layoutTile(id);
    }

    dragData = null;
  }

  /* ---------------- Timer ---------------- */
  function startTimer(){
    startTime = Date.now();
    elapsedMs = 0;
    timeEl.textContent = formatTime(0);
    clearInterval(timerInterval);
    timerInterval = setInterval(function(){
      elapsedMs = Date.now() - startTime;
      timeEl.textContent = formatTime(elapsedMs);
    }, 100);
  }

  function stopTimer(){
    clearInterval(timerInterval);
    timerInterval = null;
  }

  /* ---------------- Game flow ---------------- */
  function startPuzzle(){
    gridSize = selectedGridSize;
    computeBoardSize();
    var total = gridSize * gridSize;

    slotContents = generateScramble(total);
    pieceSlot = new Array(total);
    for (var slot = 0; slot < total; slot++){
      pieceSlot[slotContents[slot]] = slot;
    }

    moveCount = 0;
    movesEl.textContent = '0';
    difficultyLabelEl.textContent = gridSize + '×' + gridSize;
    currentLeaderboardTab = gridSize;

    buildBoard();
    gameActive = true;
    resultsOverlay.classList.remove('visible');
    showScreen('puzzle');
    startTimer();
  }

  function onWin(){
    gameActive = false;
    stopTimer();

    var scoresBefore = loadScores();
    var prevBest = (scoresBefore[gridSize] && scoresBefore[gridSize][0]) ? scoresBefore[gridSize][0].timeMs : null;

    var record = {
      date: new Date().toLocaleString(),
      timeMs: elapsedMs,
      moves: moveCount,
      difficulty: gridSize
    };
    saveScore(record);

    var isNewBest = (prevBest === null || elapsedMs < prevBest);
    newBestBanner.style.display = isNewBest ? 'block' : 'none';

    finalTimeEl.textContent = formatTime(elapsedMs);
    finalMovesEl.textContent = moveCount;
    finalDifficultyEl.textContent = gridSize + '×' + gridSize;

    setLeaderboardTab(gridSize);
    resultsOverlay.classList.add('visible');
  }

  function playAgain(){
    resultsOverlay.classList.remove('visible');
    startPuzzle();
  }

  function newPhoto(){
    resultsOverlay.classList.remove('visible');
    gameActive = false;
    stopTimer();
    capturedImage = null;
    showScreen('camera');
    initCamera();
  }

  playAgainBtn.addEventListener('click', playAgain);
  newPhotoBtn.addEventListener('click', newPhoto);

  /* ---------------- Leaderboard / localStorage ---------------- */
  var STORAGE_KEY = 'facePuzzleScores';

  function loadScores(){
    try {
      var raw = localStorage.getItem(STORAGE_KEY);
      var parsed = raw ? JSON.parse(raw) : null;
      return parsed || { 3: [], 4: [], 5: [] };
    } catch (e){
      return { 3: [], 4: [], 5: [] };
    }
  }

  function saveScore(record){
    var scores = loadScores();
    var key = record.difficulty;
    if (!scores[key]) scores[key] = [];
    scores[key].push(record);
    scores[key].sort(function(a, b){ return a.timeMs - b.timeMs; });
    scores[key] = scores[key].slice(0, 5);
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(scores));
    } catch (e){ /* storage unavailable - ignore */ }
  }

  function renderLeaderboard(difficulty){
    var scores = loadScores();
    var list = scores[difficulty] || [];
    leaderboardBody.innerHTML = '';
    if (list.length === 0){
      leaderboardBody.innerHTML = '<tr><td colspan="4" class="empty-row">No times yet — be the first!</td></tr>';
      return;
    }
    list.forEach(function(rec, i){
      var tr = document.createElement('tr');
      tr.innerHTML = '<td>' + (i + 1) + '</td><td>' + formatTime(rec.timeMs) + '</td><td>' + rec.moves + '</td><td>' + rec.date + '</td>';
      leaderboardBody.appendChild(tr);
    });
  }

  var lbTabs = Array.prototype.slice.call(document.querySelectorAll('.lb-tab'));
  function setLeaderboardTab(size){
    currentLeaderboardTab = size;
    lbTabs.forEach(function(tab){
      tab.classList.toggle('active', parseInt(tab.getAttribute('data-size'), 10) === size);
    });
    renderLeaderboard(size);
  }
  lbTabs.forEach(function(tab){
    tab.addEventListener('click', function(){
      setLeaderboardTab(parseInt(tab.getAttribute('data-size'), 10));
    });
  });

  /* ---------------- Init ---------------- */
  initCamera();
})();
</script>
</body>
</html>
https://claude.ai/share/eb040171-7e52-4c02-9208-f9632f405bd5<img width="980" height="852" alt="Image 20-06-26 at 2 36 PM" src="https://github.com/user-attachments/assets/10d45694-a418-401f-bc5b-616147bd2b06" />
<img width="1426" height="1366" alt="Image 20-06-26 at 2 37 PM" src="https://github.com/user-attachments/assets/8283360d-cece-4a3f-9b15-dd1af0cdc289" />
<img width="624" height="36" alt="image" src="https://github.com/user-attachments/assets/0b5d5f72-948b-47da-8237-5b799aa59334" />
