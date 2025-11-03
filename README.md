<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Agro Farms Limited</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    :root{--accent:#1473e6;--success:#16a34a;--danger:#dc2626;--muted:#6b7280;font-family:Inter,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial;}
    *{box-sizing:border-box}
    body{margin:0;background:#f3f4f6;color:#111;font-family:var(--font,Inter),sans-serif;line-height:1.4}
    .container{max-width:980px;margin:28px auto;padding:20px;background:#fff;border-radius:10px;box-shadow:0 6px 24px rgba(16,24,40,.08)}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px}
    header h1{margin:0;font-size:20px;color:var(--accent)}
    nav button{margin-left:8px;padding:8px 12px;border-radius:8px;border:1px solid #e6e9ef;background:#fff;cursor:pointer}
    nav .primary{background:var(--accent);color:#fff;border-color:var(--accent)}
    .top-note{font-size:13px;color:var(--muted);margin-top:8px}
    .grid{display:grid;gap:16px}
    .grid.cols-2{grid-template-columns:1fr 1fr}
    .card{background:#f9fafb;padding:14px;border-radius:8px;border:1px solid #eef2ff}
    .muted{color:var(--muted)}
    input, select{width:100%;padding:10px;border:1px solid #e6e9ef;border-radius:8px}
    .btn{display:inline-block;padding:10px 14px;border-radius:8px;border:0;cursor:pointer}
    .btn.primary{background:var(--accent);color:#fff}
    .btn.green{background:var(--success);color:#fff}
    .btn.danger{background:var(--danger);color:#fff}
    .small{font-size:13px}
    .list{list-style:none;padding:0;margin:0}
    .list li{padding:8px;border-radius:8px;background:#fff;margin-bottom:8px;border:1px solid #eee;display:flex;justify-content:space-between;align-items:center}
    .center{display:flex;align-items:center;gap:10px}
    footer{margin-top:18px;font-size:12px;color:var(--muted)}
    .hidden{display:none}
    .notice{padding:8px;background:#fef3c7;border-radius:8px;border:1px solid #fde68a;color:#92400e}
    .ok{padding:8px;background:#ecfeee;border-radius:8px;border:1px solid #bbf7d0;color:#065f46}
    .admin-panel{max-height:420px;overflow:auto;padding:6px}
    .inline{display:inline-block}
    .small-muted{font-size:12px;color:var(--muted)}
    .invite-input{display:flex;gap:8px}
    .invite-input input{flex:1}
    .actions{display:flex;gap:8px;flex-wrap:wrap}
    @media (max-width:680px){ .grid.cols-2{grid-template-columns:1fr} header{flex-direction:column;align-items:flex-start} nav{margin-top:8px}}
  </style>
</head>
<body>
  <div class="container">
    <header>
      <div>
        <h1>Agro Farms Limited</h1>
        <div class="top-note">Invest in real farm animals • Deposits to <strong>0206059987</strong></div>
      </div>

      <nav id="main-nav">
        <button onclick="go('home')" class="small">Home</button>
        <button onclick="go('about')" class="small">About</button>
        <button onclick="go('signup')" id="nav-signup" class="small primary">Sign Up</button>
        <button onclick="go('login')" id="nav-login" class="small">Login</button>
        <button onclick="go('dashboard')" id="nav-dash" class="small hidden">Dashboard</button>
        <button onclick="go('admin')" id="nav-admin" class="small">Admin</button>
      </nav>
    </header>

    <main id="app" style="margin-top:16px">
      <!-- Views will be inserted here -->
    </main>

    <footer>
      Prototype — stores data locally in the browser. Payments are manual: customers must send funds to 0206059987 and record the deposit here. Do not use this file to process real money without a secure backend and required licensing.
    </footer>
  </div>

<script>
/*
  Agro Farms Limited - Single-file HTML/JS prototype
  - localStorage keys: agro_users_v1, agro_session_v1
  - Admin password (demo): admin
  - Features: signup/login (phone+password), referral (?ref=PHONE), deposit recording (to 0206059987), admin confirm deposits,
              buy animals (deduct balance), claim daily +5 GHS (once per 24h), withdrawal requests (min 35), invite link.
*/

const ANIMALS = [
  { id:'cock', label:'Cock', price:30 },
  { id:'goat', label:'Goat', price:100 },
  { id:'horse', label:'Horse', price:500 },
  { id:'pig', label:'Pig', price:200 },
  { id:'duck', label:'Duck', price:60 },
  { id:'fish', label:'Fish', price:30 }
];

const LS_USERS = 'agro_users_v1';
const LS_SESS  = 'agro_session_v1';

// utilities
function readUsers(){ try{ return JSON.parse(localStorage.getItem(LS_USERS) || '{}') }catch(e){ return {} } }
function saveUsers(u){ localStorage.setItem(LS_USERS, JSON.stringify(u)) }
function saveSession(phone){ localStorage.setItem(LS_SESS, phone) }
function clearSession(){ localStorage.removeItem(LS_SESS) }
function getSession(){ return localStorage.getItem(LS_SESS) }

function uid(prefix='id'){ return prefix + '_' + Date.now() + '_' + Math.floor(Math.random()*999) }

// query param helper
function getQueryParam(name){
  const params = new URLSearchParams(window.location.search);
  return params.get(name);
}

// current view rendering
const app = document.getElementById('app');
let currentUser = null;
function refreshSession(){
  const phone = getSession();
  if(phone){
    const u = readUsers()[phone];
    if(u) currentUser = u;
    else { clearSession(); currentUser = null; }
  } else currentUser = null;
  updateNav();
}
function updateNav(){
  document.getElementById('nav-signup').classList.toggle('hidden', !!currentUser);
  document.getElementById('nav-login').classList.toggle('hidden', !!currentUser);
  document.getElementById('nav-dash').classList.toggle('hidden', !currentUser);
}
refreshSession();

// routing
function go(view){ render(view); window.history.pushState({view}, '', window.location.pathname + window.location.search) }

window.addEventListener('popstate', (e)=>{ const v = (e.state && e.state.view) || 'home'; render(v) })

// Render views
function render(view='home'){
  refreshSession();
  app.innerHTML = '';
  if(view === 'home'){ renderHome(); return; }
  if(view === 'about'){ renderAbout(); return; }
  if(view === 'signup'){ renderSignup(); return; }
  if(view === 'login'){ renderLogin(); return; }
  if(view === 'dashboard'){ if(!currentUser){ renderLogin('Please login to access dashboard.'); } else renderDashboard(); return; }
  if(view === 'deposit'){ if(!currentUser) renderLogin('Please login to record deposit.'); else renderDeposit(); return; }
  if(view === 'withdraw'){ if(!currentUser) renderLogin('Please login to request withdrawal.'); else renderWithdraw(); return; }
  if(view === 'shop'){ if(!currentUser) renderLogin('Please login to invest.'); else renderShop(); return; }
  if(view === 'invite'){ if(!currentUser) renderLogin('Please login to get invite link.'); else renderInvite(); return; }
  if(view === 'admin'){ renderAdmin(); return; }
  renderHome();
}

/* ------------------ Views ------------------ */

function renderHome(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="grid cols-2">
      <div>
        <div class="card">
          <h2 style="margin:0">Welcome to Agro Farms Limited</h2>
          <p class="muted small">Invest in real farm animals and earn daily income. Payments are sent manually to <strong>0206059987</strong>. This site records deposits and allows an admin to confirm them.</p>
          <div style="margin-top:12px" class="actions">
            <button class="btn primary" onclick="go('signup')">Get Started</button>
            <button class="btn" onclick="go('about')">Learn more</button>
          </div>
        </div>

        <div style="margin-top:12px" class="card">
          <h3 style="margin:0">Animals & Prices</h3>
          <ul class="list" style="margin-top:8px">
            ${ANIMALS.map(a => `<li><div>${a.label} <span class="small-muted">(${a.id})</span></div><div>${a.price} GHS</div></li>`).join('')}
          </ul>
          <div class="small-muted">Tip: Deposit funds and buy animals to invest. Admin confirms deposits before balance increases.</div>
        </div>
      </div>

      <div>
        <div class="card">
          <h3 style="margin:0">Quick actions</h3>
          <div style="margin-top:8px" class="actions">
            <button class="btn" onclick="go('signup')">Sign up</button>
            <button class="btn" onclick="go('login')">Login</button>
            <button class="btn" onclick="go('admin')">Admin</button>
          </div>
        </div>

        <div style="margin-top:12px" class="card">
          <h3 style="margin:0">How deposits work</h3>
          <ol style="margin-top:8px" class="small-muted">
            <li>Send money to <strong>0206059987</strong> (MoMo / Telecel / Tigo / Bank).</li>
            <li>Record the deposit on the Deposit page and choose a method.</li>
            <li>Admin confirms deposit manually, then your balance is credited.</li>
          </ol>
        </div>

        <div style="margin-top:12px" class="card">
          <h3 style="margin:0">Contact</h3>
          <p class="small-muted" style="margin-top:8px">Phone / WhatsApp: <strong>0206059987</strong></p>
          <p class="small-muted">Location: Ghana (add your town/region on About page)</p>
        </div>
      </div>
    </div>
  `;
  app.appendChild(el);
}

function renderAbout(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">About Agro Farms Limited</h2>
      <p class="muted small" style="margin-top:8px">We rear chickens, goats, horses, pigs, ducks and fish. We invite investors to support our operations — funds are used to buy and care for animals. This platform records investments and returns and requires manual verification for deposits and withdrawals.</p>

      <h3 style="margin-top:12px">Contact</h3>
      <p class="small-muted">Phone / WhatsApp: <strong>0206059987</strong></p>
      <p class="small-muted">Purpose: Local sales, breeding, and investment income.</p>
    </div>
  `;
  app.appendChild(el);
}

function renderSignup(){
  const suggestedRef = getQueryParam('ref') || '';
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Create an account</h2>
      <p class="small-muted" style="margin-top:8px">Phone number and password required.</p>

      <div style="margin-top:12px">
        <label class="small-muted">Full name (optional)</label>
        <input id="su-name" placeholder="Full name" />
      </div>
      <div style="margin-top:8px">
        <label class="small-muted">Phone</label>
        <input id="su-phone" placeholder="e.g. 0206xxxxxx" />
      </div>
      <div style="margin-top:8px">
        <label class="small-muted">Password</label>
        <input id="su-pass" type="password" placeholder="Choose a password" />
      </div>
      <div style="margin-top:8px">
        <label class="small-muted">Referral (optional)</label>
        <input id="su-ref" placeholder="Ref phone or leave blank" value="${suggestedRef}" />
        <div class="small-muted" style="margin-top:6px">If you signed up using someone's invite link, they receive 2 GHS automatically.</div>
      </div>

      <div style="margin-top:12px" class="actions">
        <button class="btn primary" id="su-btn">Create account</button>
        <button class="btn" onclick="go('login')">Already have account?</button>
      </div>

      <div id="su-msg" style="margin-top:12px"></div>
    </div>
  `;
  app.appendChild(el);

  document.getElementById('su-btn').onclick = ()=>{
    const name = document.getElementById('su-name').value.trim();
    const phone = document.getElementById('su-phone').value.trim();
    const pass  = document.getElementById('su-pass').value;
    const ref   = document.getElementById('su-ref').value.trim() || null;
    const msgEl = document.getElementById('su-msg');

    if(!phone || !pass){ msgEl.innerHTML = `<div class="notice">Phone and password required.</div>`; return; }
    const users = readUsers();
    if(users[phone]){ msgEl.innerHTML = `<div class="notice">Phone already registered. Try login.</div>`; return; }

    users[phone] = {
      name: name || phone,
      phone,
      password: pass,
      balance: 0,
      deposits: [],
      withdrawals: [],
      investments: [],
      createdAt: Date.now(),
      dailyClaimTs: 0,
      refBy: ref || null,
      referralEarnings: 0
    };

    // referral bonus: 2 GHS
    if(ref && users[ref]){
      users[ref].balance = (users[ref].balance || 0) + 2;
      users[ref].referralEarnings = (users[ref].referralEarnings || 0) + 2;
    }

    saveUsers(users);
    saveSession(phone);
    refreshSession();
    msgEl.innerHTML = `<div class="ok">Account created. Logged in.</div>`;
    setTimeout(()=>go('dashboard'),900);
  };
}

function renderLogin(prefMsg=''){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Login</h2>
      <div class="small-muted" style="margin-top:8px">${prefMsg}</div>

      <div style="margin-top:12px">
        <label class="small-muted">Phone</label>
        <input id="li-phone" placeholder="e.g. 0206xxxxxx" />
      </div>
      <div style="margin-top:8px">
        <label class="small-muted">Password</label>
        <input id="li-pass" type="password" placeholder="Password" />
      </div>

      <div style="margin-top:12px" class="actions">
        <button class="btn primary" id="li-btn">Login</button>
        <button class="btn" onclick="go('signup')">Create account</button>
      </div>

      <div id="li-msg" style="margin-top:12px"></div>
    </div>
  `;
  app.appendChild(el);

  document.getElementById('li-btn').onclick = ()=>{
    const phone = document.getElementById('li-phone').value.trim();
    const pass  = document.getElementById('li-pass').value;
    const users = readUsers();
    const u = users[phone];
    const msgEl = document.getElementById('li-msg');
    if(!u || u.password !== pass){ msgEl.innerHTML = `<div class="notice">Invalid phone or password.</div>`; return; }
    saveSession(phone);
    refreshSession();
    msgEl.innerHTML = `<div class="ok">Logged in.</div>`;
    setTimeout(()=>go('dashboard'),700);
  };
}

function renderDashboard(){
  refreshSession();
  const u = currentUser;
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="grid cols-2">
      <div class="card">
        <h3 style="margin:0">Dashboard</h3>
        <div style="margin-top:8px" class="small-muted">Welcome, <strong>${u.name}</strong> — ${u.phone}</div>

        <div style="margin-top:12px">
          <div class="card" style="padding:10px">
            <div class="small-muted">Balance</div>
            <div style="font-size:20px;font-weight:700">${u.balance} GHS</div>
            <div style="margin-top:8px" class="actions">
              <button class="btn" onclick="go('deposit')">Deposit</button>
              <button class="btn" onclick="go('withdraw')">Withdraw</button>
              <button class="btn" onclick="claimDaily()">Claim Daily +5 GHS</button>
              <button class="btn" onclick="go('shop')">Invest / Buy</button>
            </div>
            <div class="small-muted" style="margin-top:8px">Daily claim: +5 GHS once per 24 hours.</div>
          </div>
        </div>

        <div style="margin-top:12px">
          <h4 style="margin:0">Investments</h4>
          <ul class="list" style="margin-top:8px" id="inv-list">
            ${(u.investments || []).map(inv => `<li><div>${inv.animal}</div><div>${inv.price} GHS</div></li>`).join('') || '<li class="small-muted">No investments yet</li>'}
          </ul>
        </div>
      </div>

      <div>
        <div class="card">
          <h4 style="margin:0">Deposits</h4>
          <div class="small-muted" style="margin-top:8px">Send money to <strong>0206059987</strong>. Record deposit here and admin will confirm it.</div>
          <ul class="list" id="dep-list" style="margin-top:8px">
            ${(u.deposits || []).map(d => `<li><div>${d.amount} GHS — ${d.method}</div><div>${d.status}</div></li>`).join('') || '<li class="small-muted">No deposits recorded</li>'}
          </ul>
        </div>

        <div class="card" style="margin-top:12px">
          <h4 style="margin:0">Withdrawals</h4>
          <div class="small-muted" style="margin-top:8px">Minimum withdrawal: 35 GHS. Approved withdrawals will be paid to your registered phone number.</div>
          <ul class="list" id="wd-list" style="margin-top:8px">
            ${(u.withdrawals || []).map(w => `<li><div>${w.amount} GHS</div><div>${w.status}</div></li>`).join('') || '<li class="small-muted">No withdrawals</li>'}
          </ul>
        </div>

        <div style="margin-top:12px" class="card">
          <h4 style="margin:0">Referral & Invite</h4>
          <div class="small-muted" style="margin-top:8px">Your referral earnings: <strong>${u.referralEarnings || 0} GHS</strong></div>
          <div style="margin-top:8px" class="invite-input">
            <input id="invite-link" readonly value="${window.location.origin + window.location.pathname + '?ref=' + encodeURIComponent(u.phone)}" />
            <button class="btn green" onclick="copyInvite()">Copy</button>
          </div>
        </div>

        <div style="margin-top:12px" class="card">
          <div class="actions">
            <button class="btn" onclick="logout()">Logout</button>
            <button class="btn" onclick="go('shop')">Buy animals</button>
          </div>
        </div>
      </div>
    </div>
  `;
  app.appendChild(el);
}

function renderDeposit(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Record a deposit</h2>
      <div class="small-muted" style="margin-top:8px">Make payment to <strong>0206059987</strong>. After you pay, record the amount and method here. Admin will confirm to credit your balance.</div>

      <div style="margin-top:12px">
        <label class="small-muted">Amount (GHS)</label>
        <input id="dep-amt" placeholder="e.g. 100" />
      </div>
      <div style="margin-top:8px">
        <label class="small-muted">Method</label>
        <select id="dep-method">
          <option>MoMo</option>
          <option>Telecel Cash</option>
          <option>Tigo Cash</option>
          <option>Bank Transfer</option>
        </select>
      </div>

      <div style="margin-top:12px" class="actions">
        <button class="btn primary" id="dep-btn">Record deposit</button>
        <button class="btn" onclick="go('dashboard')">Back to dashboard</button>
      </div>

      <div id="dep-msg" style="margin-top:12px"></div>
    </div>
  `;
  app.appendChild(el);

  document.getElementById('dep-btn').onclick = ()=>{
    const amt = Number(document.getElementById('dep-amt').value);
    const method = document.getElementById('dep-method').value;
    const msg = document.getElementById('dep-msg');
    if(!amt || amt <= 0){ msg.innerHTML = `<div class="notice">Enter a valid amount.</div>`; return; }
    const users = readUsers();
    const phone = getSession();
    if(!phone){ msg.innerHTML = `<div class="notice">Login required.</div>`; return; }
    const u = users[phone];
    const dep = { id: uid('dep'), amount: amt, method, number: '0206059987', status: 'pending', ts: Date.now() };
    u.deposits = u.deposits || [];
    u.deposits.push(dep);
    users[phone] = u;
    saveUsers(users);
    msg.innerHTML = `<div class="ok">Deposit recorded as pending. Admin will confirm after verifying payment to 0206059987.</div>`;
  };
}

function renderWithdraw(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Request withdrawal</h2>
      <div class="small-muted" style="margin-top:8px">Minimum withdrawal: 35 GHS. Withdrawals are processed manually and will be paid to your registered phone number.</div>

      <div style="margin-top:12px">
        <label class="small-muted">Amount (GHS)</label>
        <input id="wd-amt" placeholder="e.g. 100" />
      </div>

      <div style="margin-top:12px" class="actions">
        <button class="btn primary" id="wd-btn">Request withdrawal</button>
        <button class="btn" onclick="go('dashboard')">Back</button>
      </div>

      <div id="wd-msg" style="margin-top:12px"></div>
    </div>
  `;
  app.appendChild(el);

  document.getElementById('wd-btn').onclick = ()=>{
    const amt = Number(document.getElementById('wd-amt').value);
    const msg = document.getElementById('wd-msg');
    if(!amt || amt <= 0){ msg.innerHTML = `<div class="notice">Enter valid amount.</div>`; return; }
    if(amt < 35){ msg.innerHTML = `<div class="notice">Minimum withdrawal is 35 GHS.</div>`; return; }
    const users = readUsers();
    const phone = getSession();
    const u = users[phone];
    if(!u){ msg.innerHTML = `<div class="notice">Login required.</div>`; return; }
    if(u.balance < amt){ msg.innerHTML = `<div class="notice">Insufficient balance.</div>`; return; }
    const req = { id: uid('wd'), amount: amt, status: 'pending', ts: Date.now(), to: u.phone };
    u.withdrawals = u.withdrawals || [];
    u.withdrawals.push(req);
    users[phone] = u;
    saveUsers(users);
    msg.innerHTML = `<div class="ok">Withdrawal requested. Admin will process and send to your registered phone number when approved.</div>`;
  };
}

function renderShop(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div>
      <h2 style="margin:0">Buy animals (Invest)</h2>
      <div class="small-muted" style="margin-top:8px">Buying an animal deducts its price from your balance and records an investment.</div>
      <div style="margin-top:12px" class="grid" id="animals-grid"></div>
    </div>
  `;
  app.appendChild(el);

  const grid = document.getElementById('animals-grid');
  ANIMALS.forEach(a=>{
    const c = document.createElement('div');
    c.className = 'card';
    c.innerHTML = `<div style="font-weight:700">${a.label}</div><div class="small-muted">Price: ${a.price} GHS</div><div style="margin-top:8px" class="actions"><button class="btn primary" onclick="buyAnimal('${a.id}')">Buy</button></div>`;
    grid.appendChild(c);
  });
}

function renderInvite(){
  refreshSession();
  const u = currentUser;
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Invite & Earn</h2>
      <div class="small-muted" style="margin-top:8px">Share your invite link. When someone signs up using your link, you get 2 GHS credited automatically.</div>
      <div style="margin-top:12px" class="invite-input">
        <input id="invite-link-2" readonly value="${window.location.origin + window.location.pathname + '?ref=' + encodeURIComponent(u.phone)}" />
        <button class="btn green" onclick="copyInvite()">Copy</button>
      </div>
    </div>
  `;
  app.appendChild(el);
}

function renderAdmin(){
  const el = document.createElement('div');
  el.innerHTML = `
    <div class="card">
      <h2 style="margin:0">Admin Panel</h2>
      <div class="small-muted" style="margin-top:8px">Enter admin password to manage deposits and withdrawals. (Demo password: <strong>admin</strong>)</div>

      <div style="margin-top:12px">
        <input id="admin-pass" placeholder="Admin password" />
        <div style="margin-top:8px">
          <button class="btn primary" id="admin-login">Login</button>
          <button class="btn" onclick="renderAdmin()">Refresh</button>
        </div>
      </div>

      <div id="admin-area" class="admin-panel" style="margin-top:12px;display:none"></div>
    </div>
  `;
  app.appendChild(el);

  document.getElementById('admin-login').onclick = ()=>{
    const p = document.getElementById('admin-pass').value;
    if(p !== 'admin'){ alert('Wrong admin password'); return; }
    document.getElementById('admin-area').style.display = 'block';
    loadAdminArea();
  };
}

function loadAdminArea(){
  const area = document.getElementById('admin-area');
  const users = readUsers();
  if(!Object.keys(users).length){ area.innerHTML = '<div class="small-muted">No users registered.</div>'; return; }
  area.innerHTML = '';
  Object.keys(users).forEach(phone=>{
    const u = users[phone];
    const card = document.createElement('div');
    card.className = 'card';
    card.style.marginBottom = '10px';
    card.innerHTML = `
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div>
          <div style="font-weight:700">${u.name} — ${u.phone}</div>
          <div class="small-muted">Balance: ${u.balance} GHS • Created: ${new Date(u.createdAt).toLocaleString()}</div>
        </div>
      </div>
      <div style="margin-top:8px">
        <h4 style="margin:0">Deposits</h4>
        <div id="deps-${phone}" style="margin-top:6px"></div>
      </div>
      <div style="margin-top:8px">
        <h4 style="margin:0">Withdrawals</h4>
        <div id="wds-${phone}" style="margin-top:6px"></div>
      </div>
    `;
    area.appendChild(card);

    const depsDiv = card.querySelector(`#deps-${phone}`);
    if(u.deposits && u.deposits.length){
      u.deposits.forEach(d=>{
        const dEl = document.createElement('div');
        dEl.style.display='flex'; dEl.style.justifyContent='space-between'; dEl.style.marginTop='6px'; dEl.style.alignItems='center';
        dEl.innerHTML = `<div>${d.amount} GHS — ${d.method} — ${d.status}</div>`;
        if(d.status === 'pending'){
          const btn = document.createElement('button');
          btn.className = 'btn green';
          btn.textContent = 'Confirm';
          btn.onclick = ()=>{ adminConfirmDeposit(phone, d.id); loadAdminArea(); };
          dEl.appendChild(btn);
        }
        depsDiv.appendChild(dEl);
      });
    } else depsDiv.innerHTML = '<div class="small-muted">No deposits</div>';

    const wdsDiv = card.querySelector(`#wds-${phone}`);
    if(u.withdrawals && u.withdrawals.length){
      u.withdrawals.forEach(w=>{
        const wEl = document.createElement('div');
        wEl.style.display='flex'; wEl.style.justifyContent='space-between'; wEl.style.marginTop='6px'; wEl.style.alignItems='center';
        wEl.innerHTML = `<div>${w.amount} GHS — ${w.status}</div>`;
        if(w.status === 'pending'){
          const btn = document.createElement('button');
          btn.className = 'btn';
          btn.textContent = 'Approve';
          btn.onclick = ()=>{ adminApproveWithdrawal(phone, w.id); loadAdminArea(); };
          wEl.appendChild(btn);
        }
        wdsDiv.appendChild(wEl);
      });
    } else wdsDiv.innerHTML = '<div class="small-muted">No withdrawals</div>';
  });
}

/* ------------------ Actions ------------------ */

function logout(){ clearSession(); refreshSession(); go('home'); }

function copyInvite(){
  const el = document.getElementById('invite-link') || document.getElementById('invite-link-2');
  if(!el) return;
  el.select ? el.select() : null;
  navigator.clipboard.writeText(el.value || el.getAttribute('value')).then(()=> alert('Invite link copied') ).catch(()=> alert('Copy failed'));
}

function claimDaily(){
  refreshSession();
  const phone = getSession();
  if(!phone){ alert('Please login'); go('login'); return; }
  const users = readUsers();
  const u = users[phone];
  const now = Date.now();
  const last = u.dailyClaimTs || 0;
  if(now - last < 24*60*60*1000){ alert('Daily income already claimed. Try again later.'); return; }
  u.balance = (u.balance || 0) + 5;
  u.dailyClaimTs = now;
  users[phone] = u;
  saveUsers(users);
  alert('Daily income of 5 GHS credited to your balance.');
  go('dashboard');
}

function buyAnimal(animalId){
  refreshSession();
  const phone = getSession();
  if(!phone){ alert('Login required'); go('login'); return; }
  const users = readUsers();
  const u = users[phone];
  const a = ANIMALS.find(x => x.id === animalId);
  if(!a){ alert('Invalid animal'); return; }
  if((u.balance || 0) < a.price){ alert('Insufficient balance. Please deposit.'); return; }
  u.balance -= a.price;
  u.investments = u.investments || [];
  u.investments.push({ id: uid('inv'), animal: a.label, price: a.price, ts: Date.now() });
  users[phone] = u;
  saveUsers(users);
  alert(`${a.label} purchased as investment.`);
  go('dashboard');
}

/* ------------------ Admin functions ------------------ */

function adminConfirmDeposit(phone, depId){
  const users = readUsers();
  const u = users[phone];
  if(!u) return;
  const d = (u.deposits || []).find(x=>x.id===depId);
  if(!d) return;
  if(d.status === 'confirmed') return;
  d.status = 'confirmed';
  u.balance = (u.balance || 0) + Number(d.amount);
  users[phone] = u;
  saveUsers(users);
  alert('Deposit confirmed and balance updated.');
}

function adminApproveWithdrawal(phone, wdId){
  const users = readUsers();
  const u = users[phone];
  if(!u) return;
  const w = (u.withdrawals || []).find(x=>x.id===wdId);
  if(!w) return;
  if(w.status !== 'pending') return;
  if((u.balance || 0) < w.amount){ w.status = 'rejected'; saveUsers(users); alert('Insufficient balance: withdrawal rejected.'); return; }
  u.balance -= w.amount;
  w.status = 'approved';
  users[phone] = u;
  saveUsers(users);
  alert(`Withdrawal approved. Send amount to ${u.phone} (user's registered phone).`);
}

/* ------------------ Init: check referral param and auto-fill sign-up if present ------------------ */

(function init(){
  // If ?ref=... present and user not logged in, show sign-up page with prefilled ref
  const ref = getQueryParam('ref');
  if(!getSession() && ref){
    // pre-load signup view with ref
    // We'll navigate to signup; signup page reads param to fill the ref field
    render('signup');
  } else {
    render('home');
  }
})();

</script>
</body>
</html>

Would you like any of these next:

    I can split this into separate index.html, styles.css, and app.js files for cleaner structure.

    Add simple images or family-friendly icons for each animal.

    Add a downloadable ZIP containing the single file and instructions.

    Help convert this into a real website with a secure backend (Node.js + SQLite) and payment integrations later.

Which of those would you like me to do next?


