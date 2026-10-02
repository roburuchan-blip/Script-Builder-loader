<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<!-- ▼ スマホ対応の必須タグ -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>Script Loader Builder</title>
<style>
  /* ===== テーマ変数 ===== */
  :root {
    --bg: #1e1e2e;
    --panel: #2a2a3d;
    --panel-2: #32324a;
    --text: #eeeeee;
    --text-dim: #9a9ab0;
    --accent: #7c5cff;
    --accent-hover: #6a4ce0;
    --output-text: #9cff9c;
    --copy: #27ae60;
    --copy-hover: #219150;
    --border: rgba(255,255,255,.08);
    --radius: 10px;
    --gap: 16px;
  }
  body.light {
    --bg: #f4f5f7;
    --panel: #ffffff;
    --panel-2: #eef0f4;
    --text: #1e1e2e;
    --text-dim: #6b6b80;
    --accent: #6a4ce0;
    --accent-hover: #5a3dd0;
    --output-text: #1a7f37;
    --copy: #27ae60;
    --copy-hover: #219150;
    --border: rgba(0,0,0,.08);
  }

  * {
    box-sizing: border-box;              /* ← 幅計算を楽に */
    transition: background .25s ease, color .25s ease, border-color .25s ease;
  }

  /* ▼ 画面いっぱいをキレイに使う */
  html, body {
    margin: 0;
    padding: 0;
    min-height: 100%;
    height: 100%;
  }

  body {
    font-family: system-ui, -apple-system, "Segoe UI", sans-serif;
    background: var(--bg);
    color: var(--text);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: var(--gap);
    padding: 80px 20px 40px;             /* 上はハンバーガー分の余白 */
    overflow-x: hidden;
    -webkit-tap-highlight-color: transparent;
  }

  h1 {
    font-size: clamp(16px, 4vw, 22px);   /* ← 画面幅で自動調整 */
    font-weight: 700;
    margin: 0;
    text-align: center;
  }

  /* ===== 左上ハンバーガー ===== */
  .hamburger {
    position: fixed;
    top: max(16px, env(safe-area-inset-top));   /* ノッチ対応 */
    left: max(16px, env(safe-area-inset-left));
    width: 44px;
    height: 44px;
    border-radius: var(--radius);
    border: none;
    background: var(--panel);
    color: var(--text);
    font-size: 22px;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(0,0,0,.2);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 20;
  }
  .hamburger:hover { background: var(--panel-2); }

  /* ===== サイドバー ===== */
  .sidebar {
    position: fixed;
    top: 0;
    left: 0;
    width: min(320px, 85vw);              /* ← スマホでは85vwまで */
    height: 100dvh;                       /* モバイルブラウザ対応 */
    background: var(--panel);
    box-shadow: 4px 0 20px rgba(0,0,0,.3);
    transform: translateX(-100%);
    transition: transform .3s ease;
    display: flex;
    flex-direction: column;
    z-index: 30;
  }
  .sidebar.open { transform: translateX(0); }

  .sidebar-header {
    padding: 20px 20px 12px;
    font-size: 16px;
    font-weight: 700;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid var(--border);
  }
  .clear-btn {
    font-size: 12px;
    padding: 6px 10px;
    border-radius: 6px;
    border: none;
    background: transparent;
    color: var(--text-dim);
    cursor: pointer;
  }
  .clear-btn:hover { color: #e74c3c; }

  .history-list {
    flex: 1;
    overflow-y: auto;
    padding: 8px 12px;
    -webkit-overflow-scrolling: touch;   /* iOSスムーズスクロール */
  }
  .history-list::-webkit-scrollbar { width: 6px; }
  .history-list::-webkit-scrollbar-thumb {
    background: var(--border);
    border-radius: 3px;
  }

  .history-item {
    display: block;
    width: 100%;
    text-align: left;
    padding: 12px;
    margin-bottom: 6px;
    border-radius: 8px;
    border: none;
    background: var(--panel-2);
    color: var(--text);
    font-size: 13px;
    cursor: pointer;
    word-break: break-all;
    transition: background .15s, transform .1s;
  }
  .history-item:hover { background: var(--accent); color: #fff; }
  .history-item:active { transform: scale(.98); }

  .empty-msg {
    color: var(--text-dim);
    font-size: 13px;
    text-align: center;
    padding: 30px 10px;
  }

  .sidebar-footer {
    border-top: 1px solid var(--border);
    padding: 14px 16px calc(14px + env(safe-area-inset-bottom));
  }
  .theme-toggle {
    width: 100%;
    padding: 12px;
    border-radius: 8px;
    border: none;
    background: var(--panel-2);
    color: var(--text);
    font-size: 14px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
  }
  .theme-toggle:hover { background: var(--accent); color: #fff; }

  /* ===== オーバーレイ ===== */
  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,.4);
    opacity: 0;
    pointer-events: none;
    transition: opacity .3s ease;
    z-index: 25;
  }
  .overlay.show { opacity: 1; pointer-events: auto; }

  /* ===== メイン：幅を自動調整 ===== */
  .container {
    width: 100%;
    max-width: 520px;                    /* PCでは最大520px */
    display: flex;
    flex-direction: column;
    gap: 14px;
    align-items: stretch;
  }

  .row {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;                     /* 幅が狭いと折り返し */
  }

  input {
    flex: 1;
    min-width: 0;                        /* ← flex内での縮小を許可 */
    padding: 12px 14px;
    border: none;
    border-radius: var(--radius);
    background: var(--panel);
    color: var(--text);
    font-size: 15px;                     /* iOSで拡大されない16px近辺 */
    outline: none;
  }
  input:focus { box-shadow: 0 0 0 2px var(--accent); }

  button.primary {
    padding: 12px 20px;
    border: none;
    border-radius: var(--radius);
    background: var(--accent);
    color: #fff;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: background .2s, transform .1s;
    min-height: 46px;                    /* タップしやすい高さ */
  }
  button.primary:hover { background: var(--accent-hover); }
  button.primary:active { transform: scale(.97); }

  textarea {
    width: 100%;
    min-height: 100px;
    padding: 12px;
    border-radius: var(--radius);
    border: none;
    background: var(--panel);
    color: var(--output-text);
    font-family: ui-monospace, Menlo, monospace;
    font-size: 13px;
    resize: vertical;
    outline: none;
    word-break: break-all;
  }

  .copy { background: var(--copy) !important; }
  .copy:hover { background: var(--copy-hover) !important; }

  /* ===== スマホ用調整 ===== */
  @media (max-width: 480px) {
    body {
      padding: 74px 14px 30px;
      gap: 12px;
    }
    .row {
      flex-direction: column;            /* ← 縦並び */
    }
    button.primary {
      width: 100%;
    }
    h1 { font-size: 17px; }
  }

  /* ===== 小さいスマホ ===== */
  @media (max-width: 360px) {
    input, textarea, button.primary { font-size: 14px; }
    body { padding: 70px 10px 24px; }
  }

  /* ===== PC大画面 ===== */
  @media (min-width: 1024px) {
    .container { max-width: 560px; }
  }
</style>
</head>
<body>

  <button class="hamburger" id="hamburger" title="履歴">☰</button>

  <div class="overlay" id="overlay"></div>

  <aside class="sidebar" id="sidebar">
    <div class="sidebar-header">
      <span>📋 履歴</span>
      <button class="clear-btn" id="clearBtn">全削除</button>
    </div>

    <div class="history-list" id="historyList"></div>

    <div class="sidebar-footer">
      <button class="theme-toggle" id="themeToggle">🌙 ダークモード</button>
    </div>
  </aside>

  <div class="container">
    <h1>Script Loader Builder</h1>

    <div class="row">
      <input id="urlInput" type="text" placeholder="URLを入力（例: example.com/script.lua）" inputmode="url" autocomplete="off" spellcheck="false">
      <button class="primary" id="insertBtn">入れる</button>
    </div>

    <textarea id="output" readonly placeholder="ここに生成結果が表示されます"></textarea>

    <button class="primary copy" id="copyBtn">コピー</button>
  </div>

<script>
  /* ========== 要素取得 ========== */
  const hamburger   = document.getElementById('hamburger');
  const sidebar     = document.getElementById('sidebar');
  const overlay     = document.getElementById('overlay');
  const historyList = document.getElementById('historyList');
  const clearBtn    = document.getElementById('clearBtn');
  const urlInput    = document.getElementById('urlInput');
  const insertBtn   = document.getElementById('insertBtn');
  const output      = document.getElementById('output');
  const copyBtn     = document.getElementById('copyBtn');
  const themeToggle = document.getElementById('themeToggle');

  /* ========== テーマ ========== */
  const savedTheme = localStorage.getItem('theme');
  function applyTheme(isLight) {
    document.body.classList.toggle('light', isLight);
    themeToggle.textContent = isLight ? '☀️ ライトモード' : '🌙 ダークモード';
  }
  applyTheme(savedTheme === 'light');

  themeToggle.addEventListener('click', () => {
    const isLight = !document.body.classList.contains('light');
    applyTheme(isLight);
    localStorage.setItem('theme', isLight ? 'light' : 'dark');
  });

  /* ========== 履歴管理 ========== */
  const HISTORY_KEY = 'urlHistory';
  let history = JSON.parse(localStorage.getItem(HISTORY_KEY) || '[]');

  function saveHistory() {
    localStorage.setItem(HISTORY_KEY, JSON.stringify(history));
  }

  function renderHistory() {
    historyList.innerHTML = '';

    if (history.length === 0) {
      historyList.innerHTML = '<div class="empty-msg">まだ履歴がありません</div>';
      return;
    }

    history.forEach((url) => {
      const btn = document.createElement('button');
      btn.className = 'history-item';
      btn.textContent = url;
      btn.title = url;
      btn.addEventListener('click', () => {
        urlInput.value = url;
        output.value = `loadstring(game:HttpGet("${url}"))()`;
        closeSidebar();
        urlInput.focus();
      });
      historyList.appendChild(btn);
    });
  }

  function addHistory(url) {
    history = history.filter(u => u !== url);
    history.unshift(url);
    if (history.length > 50) history.pop();
    saveHistory();
    renderHistory();
  }

  clearBtn.addEventListener('click', () => {
    if (history.length === 0) return;
    if (confirm('履歴を全て削除しますか？')) {
      history = [];
      saveHistory();
      renderHistory();
    }
  });

  /* ========== サイドバー開閉 ========== */
  function openSidebar() {
    sidebar.classList.add('open');
    overlay.classList.add('show');
  }
  function closeSidebar() {
    sidebar.classList.remove('open');
    overlay.classList.remove('show');
  }

  hamburger.addEventListener('click', () => {
    sidebar.classList.contains('open') ? closeSidebar() : openSidebar();
  });
  overlay.addEventListener('click', closeSidebar);

  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') closeSidebar();
  });

  /* ========== ローダー生成 ========== */
  insertBtn.addEventListener('click', () => {
    const url = urlInput.value.trim();
    if (!url) {
      output.value = '⚠ URLを入力してください';
      return;
    }
    output.value = `loadstring(game:HttpGet("${url}"))()`;
    addHistory(url);
  });

  urlInput.addEventListener('keydown', (e) => {
    if (e.key === 'Enter') insertBtn.click();
  });

  copyBtn.addEventListener('click', async () => {
    if (!output.value || output.value.startsWith('⚠')) return;
    try {
      await navigator.clipboard.writeText(output.value);
      copyBtn.textContent = 'コピー完了✅';
      setTimeout(() => copyBtn.textContent = 'コピー', 1200);
    } catch {
      output.select();
      document.execCommand('copy');
    }
  });

  renderHistory();
</script>
</body>
</html>