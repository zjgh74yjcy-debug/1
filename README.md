<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<title>字卡 · Flashcards</title>
<style>
  :root {
    --bg: #FAF7F5;
    --card: #FFFFFF;
    --text: #2D2A2E;
    --text-sub: #9B9598;
    --accent: #FF8FA3;
    --accent-dark: #E87A8E;
    --accent-soft: #FFE4E9;
    --border: #F0EBE9;
    --shadow-sm: 0 2px 8px rgba(45, 42, 46, 0.04);
    --shadow-md: 0 8px 24px rgba(45, 42, 46, 0.06);
    --radius: 28px;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  html, body { height: 100%; overflow: hidden; }

  body {
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro SC", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
    background: var(--bg);
    background-image: radial-gradient(ellipse at top, #FFF5F7 0%, #FAF7F5 45%, #F5F3F0 100%);
    color: var(--text);
    -webkit-font-smoothing: antialiased;
    user-select: none;
  }

  .app {
    height: 100%; width: 100%; max-width: 460px; margin: 0 auto;
    display: flex; flex-direction: column; position: relative;
    padding: max(env(safe-area-inset-top), 16px) 20px max(env(safe-area-inset-bottom), 20px);
  }

  /* 头部 */
  .header { display: flex; align-items: center; justify-content: space-between; padding: 8px 0 4px; flex-shrink: 0; }
  .brand { display: flex; align-items: center; gap: 8px; }
  .brand-icon {
    width: 32px; height: 32px; border-radius: 10px;
    background: linear-gradient(135deg, #FFB3C1, #FF8FA3);
    display: flex; align-items: center; justify-content: center; font-size: 16px;
    box-shadow: 0 4px 12px rgba(255, 143, 163, 0.3);
  }
  .brand-title { font-size: 17px; font-weight: 600; letter-spacing: -0.01em; }
  .icon-btn {
    width: 40px; height: 40px; border-radius: 14px; background: var(--card);
    border: 1px solid var(--border); display: flex; align-items: center; justify-content: center;
    cursor: pointer; transition: all 0.2s; color: var(--text-sub); box-shadow: var(--shadow-sm);
  }
  .icon-btn:active { transform: scale(0.92); background: var(--accent-soft); }
  .icon-btn svg { width: 18px; height: 18px; }

  /* 进度条 */
  .progress-section { padding: 16px 0 8px; flex-shrink: 0; }
  .progress-info { display: flex; justify-content: space-between; align-items: center; font-size: 13px; color: var(--text-sub); margin-bottom: 10px; font-weight: 500; }
  .progress-info .current { color: var(--accent); font-weight: 700; }
  .progress-track { height: 5px; background: var(--border); border-radius: 99px; overflow: hidden; }
  .progress-fill { height: 100%; background: linear-gradient(90deg, #FFB3C1, #FF8FA3); border-radius: 99px; transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1); width: 0%; }

  /* 卡片 */
  .card-scene { flex: 1; min-height: 0; perspective: 1400px; margin: 12px 0; cursor: pointer; }
  .card { position: relative; width: 100%; height: 100%; transform-style: preserve-3d; transition: transform 0.65s cubic-bezier(0.4, 0.0, 0.2, 1); }
  .card.flipped { transform: rotateY(180deg); }
  .card-face {
    position: absolute; inset: 0; backface-visibility: hidden; -webkit-backface-visibility: hidden;
    background: var(--card); border-radius: var(--radius); box-shadow: var(--shadow-md);
    border: 1px solid var(--border); display: flex; flex-direction: column;
    align-items: center; justify-content: center; padding: 36px 32px; text-align: center; overflow: hidden;
  }
  .card-face::before { content: ''; position: absolute; top: 0; left: 0; right: 0; height: 3px; background: linear-gradient(90deg, transparent, var(--accent-soft), transparent); }
  .card-back { transform: rotateY(180deg); background: linear-gradient(160deg, #FFF9FB 0%, #FFFFFF 60%); }
  .card-back::before { background: linear-gradient(90deg, transparent, #FFB3C1, transparent); }
  .card-label { font-size: 11px; font-weight: 600; letter-spacing: 0.08em; text-transform: uppercase; color: var(--text-sub); margin-bottom: 20px; opacity: 0.7; }
  .card-text { font-size: 30px; font-weight: 600; line-height: 1.4; color: var(--text); word-break: break-word; letter-spacing: -0.01em; }
  .card-back .card-text { color: var(--accent-dark); }

  /* 控制栏 */
  .controls { display: flex; align-items: center; justify-content: center; gap: 12px; padding: 8px 0 4px; flex-shrink: 0; }
  .ctrl-btn {
    flex: 1; height: 56px; border-radius: 20px; background: var(--card);
    border: 1px solid var(--border); display: flex; align-items: center; justify-content: center;
    gap: 8px; cursor: pointer; transition: all 0.2s; color: var(--text);
    font-size: 15px; font-weight: 600; font-family: inherit; box-shadow: var(--shadow-sm);
  }
  .ctrl-btn:active { transform: scale(0.96); }
  .ctrl-btn:disabled { opacity: 0.35; cursor: not-allowed; transform: none; }
  .ctrl-btn svg { width: 18px; height: 18px; }
  .ctrl-btn.primary { background: linear-gradient(135deg, #FFB3C1, #FF8FA3); border-color: transparent; color: white; box-shadow: 0 6px 20px rgba(255, 143, 163, 0.35); }

  /* 模态框 */
  .modal-overlay {
    position: fixed; inset: 0; background: rgba(45, 42, 46, 0.4);
    backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
    display: flex; align-items: flex-end; justify-content: center;
    z-index: 100; opacity: 0; pointer-events: none; transition: opacity 0.3s;
  }
  .modal-overlay.show { opacity: 1; pointer-events: auto; }
  .modal {
    width: 100%; max-width: 460px; background: var(--bg); border-radius: 28px 28px 0 0;
    padding: 20px 20px max(env(safe-area-inset-bottom), 24px);
    transform: translateY(100%); transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    max-height: 80vh; display: flex; flex-direction: column;
  }
  .modal-overlay.show .modal { transform: translateY(0); }
  .modal-handle { width: 40px; height: 4px; background: var(--border); border-radius: 99px; margin: 0 auto 16px; flex-shrink: 0; }
  .modal-title { font-size: 18px; font-weight: 700; margin-bottom: 16px; text-align: center; letter-spacing: -0.01em; }
  .modal-body { overflow-y: auto; flex: 1; margin: 0 -8px; padding: 0 8px; }
  .modal-body::-webkit-scrollbar { width: 0; }

  /* 表单 */
  .field { margin-bottom: 14px; }
  .field-label { font-size: 13px; font-weight: 600; color: var(--text-sub); margin-bottom: 6px; display: block; }
  .field textarea {
    width: 100%; padding: 14px 16px; border-radius: 16px; border: 1px solid var(--border);
    background: var(--card); font-size: 15px; font-family: inherit; color: var(--text);
    outline: none; transition: all 0.2s; resize: none;
  }
  .field textarea:focus { border-color: var(--accent); box-shadow: 0 0 0 3px var(--accent-soft); }

  .btn-row { display: flex; gap: 10px; margin-top: 8px; }
  .btn {
    flex: 1; height: 50px; border-radius: 16px; border: none; font-size: 15px;
    font-weight: 600; font-family: inherit; cursor: pointer; transition: all 0.2s;
    display: flex; align-items: center; justify-content: center; gap: 6px;
  }
  .btn:active { transform: scale(0.97); }
  .btn.primary { background: linear-gradient(135deg, #FFB3C1, #FF8FA3); color: white; box-shadow: 0 6px 20px rgba(255, 143, 163, 0.35); }
  .btn.secondary { background: var(--card); color: var(--text); border: 1px solid var(--border); }

  /* 列表 */
  .card-list { display: flex; flex-direction: column; gap: 8px; }
  .list-item { display: flex; align-items: center; gap: 12px; padding: 14px 16px; background: var(--card); border: 1px solid var(--border); border-radius: 16px; transition: all 0.2s; }
  .list-item:active { background: var(--accent-soft); }
  .list-index { width: 28px; height: 28px; border-radius: 10px; background: var(--accent-soft); color: var(--accent-dark); display: flex; align-items: center; justify-content: center; font-size: 12px; font-weight: 700; flex-shrink: 0; }
  .list-content { flex: 1; min-width: 0; cursor: pointer; }
  .list-front { font-size: 14px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .list-back { font-size: 12px; color: var(--text-sub); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; margin-top: 2px; }
  .list-delete { width: 32px; height: 32px; border-radius: 10px; border: none; background: transparent; color: #ccc; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; transition: all 0.2s; }
  .list-delete:active { background: #FFEBEE; color: #E53935; }

  .empty-state { text-align: center; padding: 40px 20px; color: var(--text-sub); }
  .empty-state-icon { font-size: 40px; margin-bottom: 12px; opacity: 0.5; }
  .empty-state-text { font-size: 14px; }
</style>
</head>
<body>

  <div class="app">
    <header class="header">
      <div class="brand">
        <div class="brand-icon">🥛</div>
        <div class="brand-title">字卡</div>
      </div>
      <button class="icon-btn" id="menuBtn" aria-label="菜单">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M4 6h16M4 12h16M4 18h16"/></svg>
      </button>
    </header>

    <div class="progress-section">
      <div class="progress-info">
        <span><span class="current" id="curNum">0</span> / <span id="totNum">0</span></span>
        <span id="pctText">0%</span>
      </div>
      <div class="progress-track"><div class="progress-fill" id="progressFill"></div></div>
    </div>

    <div class="card-scene" id="cardScene">
      <div class="card" id="card">
        <div class="card-face card-front">
          <div class="card-label">正面</div>
          <div class="card-text" id="frontText">还没有字卡</div>
        </div>
        <div class="card-face card-back">
          <div class="card-label">背面</div>
          <div class="card-text" id="backText">点击菜单添加</div>
        </div>
      </div>
    </div>

    <div class="controls">
      <button class="ctrl-btn" id="prevBtn" aria-label="上一张">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M15 18l-6-6 6-6"/></svg>
      </button>
      <button class="ctrl-btn primary" id="flipBtn" aria-label="翻面">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12a9 9 0 11-3-6.7M21 3v6h-6"/></svg>
      </button>
      <button class="ctrl-btn" id="nextBtn" aria-label="下一张">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 18l6-6-6-6"/></svg>
      </button>
    </div>
  </div>

  <!-- 菜单 -->
  <div class="modal-overlay" id="menuModal">
    <div class="modal">
      <div class="modal-handle"></div>
      <h2 class="modal-title">管理字卡</h2>
      <div class="modal-body" id="menuBody"></div>
    </div>
  </div>

  <!-- 编辑 -->
  <div class="modal-overlay" id="editModal">
    <div class="modal">
      <div class="modal-handle"></div>
      <h2 class="modal-title" id="editTitle">添加字卡</h2>
      <div class="modal-body">
        <div class="field">
          <label class="field-label">正面</label>
          <textarea id="editFront" rows="2" placeholder="输入正面内容..."></textarea>
        </div>
        <div class="field">
          <label class="field-label">背面</label>
          <textarea id="editBack" rows="2" placeholder="输入背面内容..."></textarea>
        </div>
        <div class="btn-row">
          <button class="btn secondary" id="editCancel">取消</button>
          <button class="btn primary" id="editSave">保存</button>
        </div>
      </div>
    </div>
  </div>

<script>
  const STORAGE_KEY = 'flashcards_v1';
  let cards = [];
  let currentIndex = 0;
  let editingIndex = -1;
  let isFlipped = false;
  let didSwipe = false;

  const $ = id => document.getElementById(id);
  const cardEl = $('card'), frontTextEl = $('frontText'), backTextEl = $('backText');
  const curNumEl = $('curNum'), totNumEl = $('totNum'), pctTextEl = $('pctText'), progressFillEl = $('progressFill');
  const prevBtn = $('prevBtn'), nextBtn = $('nextBtn'), flipBtn = $('flipBtn'), menuBtn = $('menuBtn');
  const menuModal = $('menuModal'), menuBody = $('menuBody');
  const editModal = $('editModal'), editTitle = $('editTitle'), editFront = $('editFront'), editBack = $('editBack');
  const editCancel = $('editCancel'), editSave = $('editSave');

  const DEFAULT_CARDS = [
    { front: 'apple', back: '苹果' },
    { front: 'banana', back: '香蕉' },
    { front: 'cat', back: '猫' },
    { front: 'dog', back: '狗' },
    { front: 'hello', back: '你好' },
  ];

  function save() { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(cards)); } catch(e){} }
  function load() {
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) { const p = JSON.parse(raw); if (Array.isArray(p) && p.length) { cards = p; return; } }
    } catch(e){}
    cards = DEFAULT_CARDS.slice();
  }

  function renderCard() {
    if (!cards.length) {
      frontTextEl.textContent = '还没有字卡';
      backTextEl.textContent = '点击菜单添加';
      curNumEl.textContent = '0'; totNumEl.textContent = '0'; pctTextEl.textContent = '0%';
      progressFillEl.style.width = '0%';
      prevBtn.disabled = nextBtn.disabled = true;
      cardEl.classList.remove('flipped'); isFlipped = false;
      return;
    }
    const c = cards[currentIndex];
    frontTextEl.textContent = c.front;
    backTextEl.textContent = c.back;
    cardEl.classList.remove('flipped'); isFlipped = false;

    curNumEl.textContent = currentIndex + 1;
    totNumEl.textContent = cards.length;
    const pct = Math.round(((currentIndex + 1) / cards.length) * 100);
    pctTextEl.textContent = pct + '%';
    progressFillEl.style.width = pct + '%';

    prevBtn.disabled = currentIndex === 0;
    nextBtn.disabled = currentIndex === cards.length - 1;
  }

  function flip() { if (!cards.length) return; isFlipped = !isFlipped; cardEl.classList.toggle('flipped', isFlipped); }
  function prev() { if (currentIndex > 0) { currentIndex--; renderCard(); } }
  function next() { if (currentIndex < cards.length - 1) { currentIndex++; renderCard(); } }

  function openEdit(i = -1) {
    editingIndex = i;
    if (i >= 0) { editTitle.textContent = '编辑字卡'; editFront.value = cards[i].front; editBack.value = cards[i].back; }
    else { editTitle.textContent = '添加字卡'; editFront.value = ''; editBack.value = ''; }
    editModal.classList.add('show');
    setTimeout(() => editFront.focus(), 300);
  }

  function closeEdit() { editModal.classList.remove('show'); editFront.value = ''; editBack.value = ''; editingIndex = -1; }

  function saveEdit() {
    const front = editFront.value.trim(), back = editBack.value.trim();
    if (!front || !back) return;
    if (editingIndex >= 0) cards[editingIndex] = { front, back };
    else { cards.push({ front, back }); currentIndex = cards.length - 1; }
    save(); closeEdit(); renderCard();
    if (menuModal.classList.contains('show')) renderMenu();
  }

  function deleteCard(i) {
    if (cards.length <= 1) return;
    cards.splice(i, 1);
    if (currentIndex >= cards.length) currentIndex = cards.length - 1;
    save(); renderCard(); renderMenu();
  }

  function esc(s) { return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;').replace(/'/g,'&#39;'); }

  function renderMenu() {
    let html = '<div class="btn-row" style="margin-bottom:16px;"><button class="btn primary" id="addCardBtn"><svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M12 5v14M5 12h14"/></svg>添加字卡</button></div>';

    if (cards.length) {
      html += '<div class="card-list">';
      cards.forEach((c, i) => {
        html += `<div class="list-item">
          <div class="list-index">${i + 1}</div>
          <div class="list-content" data-edit="${i}">
            <div class="list-front">${esc(c.front)}</div>
            <div class="list-back">${esc(c.back)}</div>
          </div>
          <button class="list-delete" data-delete="${i}" aria-label="删除">
            <svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M3 6h18M8 6V4a2 2 0 012-2h4a2 2 0 012 2v2M19 6l-1 14a2 2 0 01-2 2H8a2 2 0 01-2-2L5 6"/></svg>
          </button>
        </div>`;
      });
      html += '</div>';
    } else {
      html += '<div class="empty-state"><div class="empty-state-icon">📝</div><div class="empty-state-text">还没有字卡，点击上方添加</div></div>';
    }

    menuBody.innerHTML = html;

    const addBtn = menuBody.querySelector('#addCardBtn');
    if (addBtn) addBtn.addEventListener('click', () => { menuModal.classList.remove('show'); setTimeout(() => openEdit(-1), 200); });

    menuBody.querySelectorAll('[data-edit]').forEach(el => el.addEventListener('click', () => {
      const i = parseInt(el.dataset.edit, 10);
      menuModal.classList.remove('show');
      setTimeout(() => openEdit(i), 200);
    }));

    menuBody.querySelectorAll('[data-delete]').forEach(el => el.addEventListener('click', e => {
      e.stopPropagation();
      deleteCard(parseInt(el.dataset.delete, 10));
    }));
  }

  // 事件
  cardEl.addEventListener('click', () => { if (didSwipe) return; flip(); });
  flipBtn.addEventListener('click', e => { e.stopPropagation(); flip(); });
  prevBtn.addEventListener('click', prev);
  nextBtn.addEventListener('click', next);
  menuBtn.addEventListener('click', () => { renderMenu(); menuModal.classList.add('show'); });

  menuModal.addEventListener('click', e => { if (e.target === menuModal) menuModal.classList.remove('show'); });
  editCancel.addEventListener('click', closeEdit);
  editSave.addEventListener('click', saveEdit);
  editModal.addEventListener('click', e => { if (e.target === editModal) closeEdit(); });

  document.addEventListener('keydown', e => {
    if (editModal.classList.contains('show') || menuModal.classList.contains('show')) {
      if (e.key === 'Escape') { closeEdit(); menuModal.classList.remove('show'); }
      return;
    }
    if (e.key === 'ArrowLeft') prev();
    if (e.key === 'ArrowRight') next();
    if (e.key === ' ') { e.preventDefault(); flip(); }
  });

  // 触摸滑动
  let touchStartX = 0, touchStartY = 0;
  cardEl.addEventListener('touchstart', e => {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
  }, { passive: true });

  cardEl.addEventListener('touchend', e => {
    const dx = e.changedTouches[0].clientX - touchStartX;
    const dy = e.changedTouches[0].clientY - touchStartY;
    if (Math.abs(dx) > 60 && Math.abs(dx) > Math.abs(dy)) {
      didSwipe = true;
      if (dx > 0) prev(); else next();
      setTimeout(() => { didSwipe = false; }, 300);
    }
  }, { passive: true });

  load();
  renderCard();
</script>
</body>
</html>
