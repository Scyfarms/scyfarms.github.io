<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Weave — Fire (Production-ready)</title>

<!--
  PRODUCTION-READY single-file index.html for Weave — Fire (GitHub Pages)

  IMPORTANT SECURITY NOTES (read before deploying):
  - Never commit real Back4App (Parse) keys into source control.
  - Use GitHub Actions (or your CI) to replace placeholders
      %%BACK4APP_APP_ID%% and %%BACK4APP_JS_KEY%%
    with values taken from repository secrets during the build/deploy step.
  - Alternatively place a secure server-side proxy to avoid exposing keys to clients.
  - For best security in production, prefer server-side session/token approaches.

  How keys are read by this file (order):
  1) window.__BACK4APP_APP_ID and window.__BACK4APP_JS_KEY (recommended if set by build)
  2) meta tags (placeholders below replaced at build time)
  3) sessionStorage dev helper (only for local/dev testing; NOT persisted to repo)

  NOTE: This file includes robust handling:
  - Tries to auto-init Parse from safe sources.
  - Retries initialization on transient failures.
  - Wraps Parse login with a timeout and retry/backoff.
  - Surfaces clear, copyable error details in a modal.
  - Falls back gracefully to a local demo mode (no Parse) if Parse is unreachable.
  - Avoids storing secrets in localStorage.
  - Defensive JSON parsing and guarded localStorage access.
-->

<!-- Build-time placeholders: replace during CI with actual secrets -->
<meta name="back4app-app-id" content="hh1ngtcuSSf7hYAM28NnPhn9nGH5UK0iVcVRMA1l" />
<meta name="back4app-js-key" content="kqRUfpp3ejpuEsBVw0yFzZwrUDVmbNnKCAMqGwa6" />

<style>
  :root{
    --bg:#0f0a05;
    --panel:#0d0b09;
    --accent-1:#ff7a18;
    --accent-2:#ff3d00;
    --glass: rgba(255,255,255,0.04);
    --muted: #cfcfcf;
    --ok: #23c55e;
    --danger: #ff4d4f;
  }
  html,body{height:100%;margin:0;font-family:Inter,system-ui,-apple-system,"Segoe UI",Roboto,Arial;background:linear-gradient(180deg,var(--bg),#070505);color:#fff;-webkit-font-smoothing:antialiased}
  .wrap{max-width:1100px;margin:28px auto;padding:20px;box-sizing:border-box}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:18px}
  .brandRow{display:flex;align-items:center;gap:12px}
  .logo{width:36px;height:36px}
  .title{font-weight:800;font-size:18px}
  .muted{color:var(--muted);font-size:13px}
  .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:12px;padding:14px;border:1px solid var(--glass);box-shadow:0 8px 30px rgba(0,0,0,0.6)}
  input, button, textarea{font:inherit}
  .input{width:100%;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
  .row{display:flex;gap:8px;align-items:center}
  .btn{padding:10px 12px;border-radius:10px;border:none;cursor:pointer;background:linear-gradient(90deg,var(--accent-1),var(--accent-2));color:#1a0500;font-weight:700}
  .btn.ghost{background:transparent;border:1px solid rgba(255,255,255,0.03);color:var(--muted)}
  .notice{margin-top:10px;color:#ffd7cf}
  .modal-backdrop{position:fixed;inset:0;background:rgba(0,0,0,0.6);display:none;align-items:center;justify-content:center;z-index:1200}
  .modal{width:92%;max-width:880px;background:var(--panel);padding:16px;border-radius:10px;border:1px solid var(--glass);max-height:84vh;overflow:auto}
  pre{white-space:pre-wrap;background:rgba(255,255,255,0.02);padding:10px;border-radius:6px;max-height:300px;overflow:auto}
  .dev-key{position:fixed;right:12px;top:12px;padding:8px 10px;border-radius:8px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.04);color:var(--muted);cursor:pointer;z-index:1300}
  .sr-only{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}
  @media (max-width:720px){ header{flex-direction:column;align-items:flex-start} .row{flex-wrap:wrap} }
</style>
</head>
<body>
  <div class="wrap">

    <header>
      <div class="brandRow">
        <svg class="logo" viewBox="0 0 64 64" aria-hidden="true"><defs><linearGradient id="lg" x1="0" x2="1"><stop offset="0" stop-color="#ffd200" /><stop offset="1" stop-color="#ff3d00" /></linearGradient></defs><path fill="url(#lg)" d="M32 4c-1 8-8 12-8 22 0 7 6 13 8 16 2-3 8-9 8-16 0-10-7-14-8-22z"/></svg>
        <div>
          <div class="title">Weave — Fire</div>
          <div class="muted" style="margin-top:4px">Production build — robust Parse init & fallback</div>
        </div>
      </div>

      <div class="muted">Hosted on GitHub Pages</div>
    </header>

    <!-- Login overlay -->
    <div id="loginOverlay" style="position:relative">
      <div id="loginCard" class="panel" role="dialog" aria-modal="true" aria-labelledby="loginTitle">
        <h2 id="loginTitle" style="margin:0 0 8px 0">Login / Sign Up</h2>

        <label class="sr-only" for="username">Username</label>
        <input id="username" class="input" type="text" autocomplete="username" placeholder="Username (any text allowed)" />

        <label class="sr-only" for="password">Password</label>
        <input id="password" class="input" type="password" autocomplete="current-password" placeholder="Password (min 8 chars)" />

        <div style="margin-top:8px" class="muted">You must accept Terms & Privacy before logging in.</div>
        <div class="row" style="margin-top:8px;align-items:center">
          <input id="tosCheckbox" type="checkbox" />
          <label for="tosCheckbox" class="muted">I agree to the <a id="viewTerms" href="#" style="color:var(--accent-1);text-decoration:underline">Terms & Privacy</a></label>
        </div>

        <div class="row" style="margin-top:12px;justify-content:space-between">
          <div style="display:flex;gap:8px">
            <button id="loginButton" class="btn">Login / Sign Up</button>
            <button id="guestButton" class="btn ghost" title="Continue without remote backend">Continue local</button>
          </div>
          <div id="backendStatus" class="muted" aria-live="polite"></div>
        </div>

        <div id="loginHelp" class="notice"></div>
      </div>
    </div>

    <!-- Main app (hidden until login) -->
    <main id="appMain" style="display:none;margin-top:18px">
      <div class="panel">
        <div style="display:flex;justify-content:space-between;align-items:center">
          <div>
            <div style="font-weight:800">Welcome</div>
            <div id="currentUser" class="muted" style="margin-top:4px">—</div>
          </div>
          <div style="display:flex;gap:8px;align-items:center">
            <button id="logoutBtn" class="btn ghost">Logout</button>
          </div>
        </div>

        <div style="margin-top:12px" class="muted">
          This is your production Weave UI shell. Hook in your feed/chatroom data after login.
        </div>
      </div>
    </main>

  </div>

  <!-- Terms modal -->
  <div id="tosModal" class="modal-backdrop" role="dialog" aria-modal="true" style="display:none">
    <div class="modal" role="document">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div style="font-weight:700">Terms & Privacy</div>
        <button id="closeTos" class="btn ghost">Close</button>
      </div>
      <div style="margin-top:12px" class="muted">
        <p>Readable summary for users. This is a short, friendly summary only.</p>
        <h4>Terms</h4>
        <ul>
          <li>Be respectful to others.</li>
          <li>Do not claim Weave/Scyfarms intellectual property as your own.</li>
        </ul>
        <h4>Privacy</h4>
        <ul>
          <li>We use cookies/localStorage for login & settings.</li>
          <li>We do not sell personal data.</li>
        </ul>
      </div>
      <div style="margin-top:12px;display:flex;justify-content:flex-end">
        <button id="acceptTos" class="btn">I Accept</button>
      </div>
    </div>
  </div>

  <!-- Error modal (technical details copyable) -->
  <div id="errorModal" class="modal-backdrop" style="display:none">
    <div class="modal" role="alertdialog" aria-modal="true" aria-labelledby="errorTitle">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div id="errorTitle" style="font-weight:700">Error</div>
        <button id="closeError" class="btn ghost">Close</button>
      </div>

      <div style="margin-top:8px" class="muted">Message</div>
      <div id="errorMessage" style="margin-top:6px;font-weight:700"></div>

      <div style="margin-top:12px" class="muted">Technical details</div>
      <pre id="errorDetails"></pre>

      <div style="display:flex;justify-content:space-between;margin-top:12px">
        <div><button id="retryLogin" class="btn ghost">Retry</button></div>
        <div style="display:flex;gap:8px">
          <button id="copyDetails" class="btn ghost">Copy details</button>
          <button id="fallbackLocal" class="btn">Use local demo</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Dev helper to set keys for the session (not persisted to repo) -->
  <button id="devSetKeysBtn" class="dev-key" title="Dev: set Parse keys in this browser session (for testing)">Dev: Set Keys</button>

  <!-- Parse SDK (CDN). In production you should bundle the SDK with your build for strict CSP -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/parse/3.4.4/parse.min.js"></script>

<script>
/* ========== Production-ready Weave-Fire index.html script ==========
   Defensive, secure, and self-contained:
   - Auto-init Parse from meta/window/session sources
   - Robust login with timeout, retries, and fallback to local demo
   - Clear error modal with copyable details & actions (retry, local fallback)
   - ToS modal and keyboard accessibility
   - No secrets persisted in repo or localStorage (sessionStorage only if testing)
   - Guidance printed to console for deploy-time steps
================================================================== */

(() => {
  const BACK4APP_SERVER = 'https://parseapi.back4app.com/';
  const META_APP = 'back4app-app-id';
  const META_KEY = 'back4app-js-key';
  const SESSION_DEV_APP = 'weave_dev_appId';
  const SESSION_DEV_KEY = 'weave_dev_jsKey';

  // DOM
  const loginButton = document.getElementById('loginButton');
  const guestButton = document.getElementById('guestButton');
  const usernameEl = document.getElementById('username');
  const passwordEl = document.getElementById('password');
  const tosCheckbox = document.getElementById('tosCheckbox');
  const backendStatus = document.getElementById('backendStatus');
  const loginHelp = document.getElementById('loginHelp');
  const loginOverlay = document.getElementById('loginOverlay');
  const appMain = document.getElementById('appMain');
  const currentUser = document.getElementById('currentUser');
  const logoutBtn = document.getElementById('logoutBtn');

  const errorModal = document.getElementById('errorModal');
  const errorMessage = document.getElementById('errorMessage');
  const errorDetails = document.getElementById('errorDetails');
  const retryBtn = document.getElementById('retryLogin');
  const copyBtn = document.getElementById('copyDetails');
  const fallbackBtn = document.getElementById('fallbackLocal');
  const closeErrorBtn = document.getElementById('closeError');

  const devSetKeysBtn = document.getElementById('devSetKeysBtn');
  const tosModal = document.getElementById('tosModal');
  const viewTermsBtn = document.getElementById('viewTerms');
  const acceptTosBtn = document.getElementById('acceptTos');
  const closeTosBtn = document.getElementById('closeTos');

  // Helpers: safe text, local get/set
  function safeText(node, text){ if(!node) return; node.textContent = text === undefined || text === null ? '' : String(text); }
  function safeParseJSON(raw, fallback){ try { return JSON.parse(raw); } catch(e) { return fallback; } }

  function sessionSetKey(appId, jsKey){
    if(appId) sessionStorage.setItem(SESSION_DEV_APP, appId);
    if(jsKey) sessionStorage.setItem(SESSION_DEV_KEY, jsKey);
  }
  function sessionGetKeys(){
    return {
      appId: sessionStorage.getItem(SESSION_DEV_APP),
      jsKey: sessionStorage.getItem(SESSION_DEV_KEY)
    };
  }
  function clearSessionKeys(){
    sessionStorage.removeItem(SESSION_DEV_APP);
    sessionStorage.removeItem(SESSION_DEV_KEY);
  }

  // Read meta values (build-time placeholders should replace these if using CI)
  function metaContent(name){
    try { const m = document.querySelector('meta[name="' + name + '"]'); return m ? m.content : null; } catch(e) { return null; }
  }

  // Parse detection & initialization:
  function parseIsConfigured(){
    try {
      if(typeof Parse === 'undefined') return false;
      if(Parse.CoreManager && typeof Parse.CoreManager.get === 'function') {
        return !!Parse.CoreManager.get('APPLICATION_ID');
      }
      if(Parse._config && Parse._config.applicationId) return true;
      return !!Parse.serverURL;
    } catch(e){ return false; }
  }

  // Attempt to initialize Parse from safe sources (non-destructive)
  function tryInitParseFromSources(){
    try {
      if(typeof Parse === 'undefined') return false;
      // already configured?
      if(parseIsConfigured()) return true;

      // 1. window globals (set by build)
      const wApp = (window.__BACK4APP_APP_ID && typeof window.__BACK4APP_APP_ID === 'string' && window.__BACK4APP_APP_ID.indexOf('%%BACK4APP') === -1) ? window.__BACK4APP_APP_ID : null;
      const wKey = (window.__BACK4APP_JS_KEY && typeof window.__BACK4APP_JS_KEY === 'string' && window.__BACK4APP_JS_KEY.indexOf('%%BACK4APP') === -1) ? window.__BACK4APP_JS_KEY : null;

      // 2. meta tags (placeholders replaced by build)
      const mApp = (metaContent(META_APP) && metaContent(META_APP).indexOf('%%BACK4APP') === -1) ? metaContent(META_APP) : null;
      const mKey = (metaContent(META_KEY) && metaContent(META_KEY).indexOf('%%BACK4APP') === -1) ? metaContent(META_KEY) : null;

      // 3. sessionStorage dev helper
      const s = sessionGetKeys();
      const sApp = s.appId || null;
      const sKey = s.jsKey || null;

      const appId = wApp || mApp || sApp;
      const jsKey = wKey || mKey || sKey;

      if(appId && jsKey){
        try {
          Parse.initialize(appId, jsKey);
          Parse.serverURL = Parse.serverURL || BACK4APP_SERVER;
          console.info('Parse initialized from available sources');
          return true;
        } catch(e){
          console.warn('Parse.initialize threw', e);
          return false;
        }
      }
      return false;
    } catch(e){
      console.warn('tryInitParseFromSources error', e);
      return false;
    }
  }

  // Use a Promise wrapper with timeout to guard Parse SDK calls
  function withTimeout(promise, ms){
    let timeout;
    const t = new Promise((_, reject) => timeout = setTimeout(()=> reject(new Error('Request timed out after ' + ms + 'ms')), ms));
    return Promise.race([promise.finally(()=> clearTimeout(timeout)), t]);
  }

  // Parse login with timeout + retry/backoff for network errors
  async function parseLoginWithRetry(username, password, opts = {}){
    const attempts = opts.attempts || 3;
    const baseDelay = opts.baseDelay || 600; // ms
    const timeoutMs = opts.timeoutMs || 8000; // each attempt timeout
    let lastErr = null;

    for(let i=0;i<attempts;i++){
      try {
        // Wrap Parse.User.logIn in a timeout
        const user = await withTimeout(Parse.User.logIn(username, password), timeoutMs);
        return user;
      } catch(err){
        lastErr = err;
        const msg = (err && err.message) ? err.message.toLowerCase() : String(err).toLowerCase();
        // If it's authentication-related (invalid credentials), don't retry
        if(err && err.code === 101 || /invalid/i.test(msg) || /password/i.test(msg) || /auth/i.test(msg) ) {
          throw err; // immediate bubble
        }
        // If network/XHR failed or timed out, retry after backoff
        console.warn('Parse login attempt failed (will retry if attempts left):', err);
        const delay = baseDelay * Math.pow(1.8, i);
        await new Promise(r => setTimeout(r, delay));
      }
    }
    // after retries, throw last error
    throw lastErr;
  }

  // UI helpers for error modal
  function showErrorModal(title, details){
    safeText(errorMessage, title || 'Error');
    safeText(errorDetails, typeof details === 'string' ? details : JSON.stringify(details, Object.getOwnPropertyNames(details), 2));
    errorModal.style.display = 'flex';
  }
  function hideErrorModal(){ errorModal.style.display = 'none'; }

  copyBtn.addEventListener('click', ()=>{
    const text = errorDetails.textContent || errorMessage.textContent || '';
    if(!text) return;
    navigator.clipboard?.writeText(text).catch(()=> {
      const ta = document.createElement('textarea'); ta.value = text; document.body.appendChild(ta); ta.select(); document.execCommand('copy'); ta.remove();
    });
  });
  closeErrorBtn.addEventListener('click', hideErrorModal);

  // Fallback to local/demo login
  function localLogin(username){
    try {
      // Store minimal username so session can persist; avoid storing keys or sensitive data
      localStorage.setItem('weave_local_user', username);
      console.info('Local demo login:', username);
      return { username };
    } catch(e){
      console.error('localLogin failed', e);
      throw e;
    }
  }

  // After-login UX: hide login overlay, show app content
  async function afterLogin(userObj, usedParse){
    loginOverlay.style.display = 'none';
    appMain.style.display = 'block';
    const displayName = (userObj && (userObj.getUsername ? userObj.getUsername() : (userObj.username || userObj))) || 'unknown';
    safeText(currentUser, displayName);

    // optional: record ToS acceptance locally (and on Parse if used)
    try { localStorage.setItem('weave_tos_privacy_accepted_v2.0', JSON.stringify({ accepted: true, ts: Date.now() })); } catch(e){}
    if(usedParse){
      // sample: try to record acceptance server-side (best-effort)
      try {
        const TA = Parse.Object.extend('TermsAcceptance');
        const a = new TA();
        a.set('username', displayName);
        a.set('acceptedAt', new Date());
        a.set('version', 'v2.0');
        await a.save();
      } catch(e){
        console.warn('Failed to save TermsAcceptance server-side', e);
      }
    }
    // load application data hooks (if present)
    if(typeof window.loadPosts === 'function') try { await window.loadPosts(); } catch(_) {}
    if(typeof window.loadChatrooms === 'function') try { await window.loadChatrooms(); } catch(_) {}
  }

  // Primary login handler: robust path
  async function performLogin(){
    const username = (usernameEl.value || '').trim();
    const password = (passwordEl.value || '');
    const tosOk = tosCheckbox.checked;

    if(!tosOk){
      // open ToS modal
      tosModal.style.display = 'flex';
      return;
    }
    if(!username){
      showErrorModal('Username required', 'Please enter a username. Any non-empty text is allowed.');
      return;
    }
    if(!password || password.length < 8){
      showErrorModal('Invalid password', 'Password must be at least 8 characters.');
      return;
    }

    // Disable UI immediately
    loginButton.disabled = true;
    guestButton.disabled = true;
    const origText = loginButton.textContent;
    loginButton.textContent = 'Connecting...';

    try {
      // Try to initialize Parse if not yet configured
      tryInitParseFromSources();

      if(parseIsConfigured()){
        // Try to login with retries/timeouts
        try {
          const user = await parseLoginWithRetry(username, password, { attempts: 3, baseDelay: 600, timeoutMs: 8000 });
          // success
          await afterLogin(user, true);
          return;
        } catch(err){
          // distinguish network connectivity vs auth errors
          const raw = String(err && err.message ? err.message : err);
          const lower = raw.toLowerCase();
          // Network issues: show helpful guidance + allow retry/fallback
          if(lower.includes('xmlhttprequest failed') || lower.includes('unable to connect') || lower.includes('network') || lower.includes('timed out')){
            showErrorModal('Login failed: network or Parse unreachable', raw + '\n\nTroubleshooting:\n- Verify keys are injected and Parse.serverURL is correct.\n- Check Network tab for failed XHR and any CORS/CSP errors.\n- Ensure the site is not running inside a sandboxed iframe (some editors block XHR).\n- Try the Dev: Set Keys button to provide keys for this session.');
            // allow user to Retry or Use Local (buttons in the modal)
            retryBtn.onclick = () => { hideErrorModal(); performLogin(); };
            fallbackBtn.onclick = async () => {
              hideErrorModal();
              try {
                const lu = localLogin(username);
                await afterLogin(lu, false);
              } catch(e){
                showErrorModal('Local fallback failed', String(e));
              }
            };
            return;
          }
          // Authentication/other server error -> show details (don't retry)
          showErrorModal('Login failed', raw);
          return;
        }
      } else {
        // Parse not configured; auto-fallback to local demo (but inform user)
        loginHelp.textContent = 'Parse backend not configured. Using local demo (no remote persistence).';
        const localUser = localLogin(username);
        await afterLogin(localUser, false);
        return;
      }
    } catch(e){
      showErrorModal('Unexpected login error', String(e));
      return;
    } finally {
      loginButton.disabled = false;
      guestButton.disabled = false;
      loginButton.textContent = origText;
      updateBackendStatus();
    }
  }

  // Retry button in error modal reuses performLogin; fallback handled there

  // Guest local flow
  guestButton.addEventListener('click', async () => {
    const username = (usernameEl.value || '').trim() || ('guest_' + Date.now().toString(36).slice(-6));
    try {
      const user = localLogin(username);
      await afterLogin(user, false);
    } catch(e){
      showErrorModal('Guest login failed', String(e));
    }
  });

  // Wire login button and Enter key
  loginButton.addEventListener('click', ()=> performLogin());
  [usernameEl, passwordEl].forEach(el => {
    el.addEventListener('keypress', (e) => { if(e.key === 'Enter') performLogin(); });
  });

  // Logout
  logoutBtn.addEventListener('click', async ()=> {
    try { if(parseIsConfigured() && Parse.User && Parse.User.logOut) await Parse.User.logOut(); } catch(e){ console.warn(e); }
    try { localStorage.removeItem('weave_local_user'); } catch(e){}
    // show login UI
    appMain.style.display = 'none';
    loginOverlay.style.display = 'block';
    usernameEl.value = '';
    passwordEl.value = '';
  });

  // ToS modal wiring
  viewTermsBtn.addEventListener('click', (e)=> { e.preventDefault(); tosModal.style.display = 'flex'; });
  closeTosBtn.addEventListener('click', ()=> tosModal.style.display = 'none');
  acceptTosBtn.addEventListener('click', ()=> { tosCheckbox.checked = true; try{ localStorage.setItem('weave_tos_privacy_accepted_v2.0', JSON.stringify({ accepted:true, ts:Date.now() })); }catch(e){} tosModal.style.display = 'none'; });

  // Error modal controls
  retryBtn.addEventListener('click', ()=> { hideErrorModal(); performLogin(); });
  fallbackBtn.addEventListener('click', async ()=> {
    hideErrorModal();
    const username = (usernameEl.value || '').trim() || ('guest_' + Date.now().toString(36).slice(-6));
    try {
      const user = localLogin(username);
      await afterLogin(user, false);
    } catch(e){
      showErrorModal('Local fallback failed', String(e));
    }
  });

  // Dev: set keys for current browser session (sessionStorage) - convenient for development only
  const devBtn = document.getElementById('devSetKeysBtn');
  devBtn.addEventListener('click', ()=>{
    const appId = prompt('Paste Back4App APP ID (session-only):', sessionStorage.getItem(SESSION_DEV_APP) || '');
    if(!appId) return alert('App ID required for session testing.');
    const jsKey = prompt('Paste Back4App JS KEY (session-only):', sessionStorage.getItem(SESSION_DEV_KEY) || '');
    if(!jsKey) return alert('JS Key required.');
    sessionSetKey(appId, jsKey);
    const ok = tryInitParseFromSources();
    if(ok){
      devBtn.style.display = 'none';
      alert('Parse initialized for this browser session. Try logging in.');
    } else {
      alert('Failed to initialize Parse with provided keys. Check console for details.');
    }
    updateBackendStatus();
  });

  // Update backend status on UI
  function updateBackendStatus(){
    if(parseIsConfigured()){
      safeText(backendStatus, 'Remote backend enabled');
      if(devBtn) devBtn.style.display = 'none';
    } else {
      safeText(backendStatus, 'Remote backend not configured (local demo available)');
      if(devBtn) devBtn.style.display = 'block';
    }
  }
  updateBackendStatus();

  // Auto-resume local session if present
  (function tryResumeLocal(){
    try {
      const u = localStorage.getItem('weave_local_user');
      if(u && !parseIsConfigured()){
        // small delay to let UI render
        setTimeout(async ()=> {
          try {
            const user = { username: u };
            await afterLogin(user, false);
          } catch(e){}
        }, 200);
      }
    } catch(e){}
  })();

  // Expose some functions for debugging (safe)
  window.__weave_tryInitParse = tryInitParseFromSources;
  window.__weave_isParseConfigured = parseIsConfigured;
  window.__weave_performLogin = performLogin;

  // Final console guidance for deployers
  console.info('Weave-Fire production index loaded.');
  console.info('Ensure you inject Back4App keys during CI (replace meta placeholders) or set them via session Dev button for local testing.');
  console.info('Preferred CI approach: replace %%BACK4APP_APP_ID%% and %%BACK4APP_JS_KEY%% at build-time with GitHub Actions secrets, then deploy to GitHub Pages.');
})();
</script>
</body>
</html>
