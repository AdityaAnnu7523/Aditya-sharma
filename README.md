<!doctype html>
<html lang="hi">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Private Vault — Secure Setup</title>
<style>
  :root{font-family:system-ui,Segoe UI,Roboto,Arial}
  body{margin:0;padding:0;background:#05060a;color:#fff}
  body::before{content:'';position:fixed;inset:0;background:url('https://i.imgur.com/4AiXzf8.jpg') center/cover no-repeat;opacity:0.25;z-index:-1}
  #vaultName{position:fixed;left:16px;top:14px;font-weight:700;color:#1f6feb;font-size:20px;text-shadow:1px 1px 6px #000;z-index:50}
  .centerBox{max-width:520px;margin:80px auto 40px;padding:18px;background:rgba(8,10,14,0.85);border-radius:10px;box-shadow:0 8px 30px rgba(0,0,0,0.6)}
  h2{margin:0 0 8px 0}
  label{display:block;margin-top:10px;font-size:14px;color:#c9d2ff}
  input[type=password], input[type=text]{width:100%;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.06);background:rgba(255,255,255,0.02);color:#fff;margin-top:6px}
  button{margin-top:12px;padding:10px 12px;border-radius:8px;border:0;background:#1f6feb;color:#fff;cursor:pointer}
  button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.06);color:#dfe9ff}
  .small{font-size:13px;color:rgba(255,255,255,0.8);margin-top:8px}
  .flex{display:flex;gap:8px;align-items:center}
  .folders{display:grid;grid-template-columns:repeat(auto-fit,minmax(120px,1fr));gap:12px;margin-top:16px}
  .folder{background:rgba(255,255,255,0.03);padding:12px;border-radius:8px;text-align:center;cursor:pointer}
  .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(160px,1fr));gap:12px;margin-top:12px}
  .card{background:rgba(255,255,255,0.03);padding:10px;border-radius:8px;text-align:center}
  .thumb{width:100%;height:110px;object-fit:cover;border-radius:6px}
  .viewer{position:fixed;inset:0;background:rgba(0,0,0,0.92);display:none;align-items:center;justify-content:center;flex-direction:column;z-index:200}
  .viewer img,.viewer video{max-width:96%;max-height:86%;border-radius:8px}
  .metaRow{display:flex;gap:8px;justify-content:center;margin-top:8px}
  .hint{font-size:13px;color:rgba(255,255,255,0.7);margin-top:8px}
</style>
</head>
<body>
<div id="vaultName">Aditya Hacker</div>

<div class="centerBox" id="setupBox" style="display:none;">
  <h2>Setup your Vault</h2>
  <div class="small">Ye pehli baar run ho raha hai. Please ek strong password create karo. Plain password file me save nahi hoga.</div>

  <label for="newPwd">Naya password</label>
  <input id="newPwd" type="password" autocomplete="new-password" />

  <label for="newPwd2">Confirm password</label>
  <input id="newPwd2" type="password" autocomplete="new-password" />

  <div class="hint">Important: Agar password bhool gaye to data recover nahi hoga. Backup rakhna na bhoolen.</div>
  <div style="display:flex;gap:8px;margin-top:12px">
    <button id="createBtn">Create Password & Initialize Vault</button>
    <button id="cancelSetup" class="ghost">Cancel</button>
  </div>
  <div id="setupMsg" class="small" style="color:#ff8b8b;margin-top:8px"></div>
</div>

<div class="centerBox" id="loginBox" style="display:none;">
  <h2>Unlock Vault</h2>
  <div class="small">Password enter karo taaki vault dikh sake. (Password kahin bhi plain text me stored nahi hai.)</div>
  <label for="pwdInput">Password</label>
  <input id="pwdInput" type="password" autocomplete="current-password" />
  <div style="display:flex;gap:8px">
    <button id="unlockBtn">Unlock</button>
    <button id="resetBtn" class="ghost" title="Reset removes all saved files and allows creating a new password">Reset Vault</button>
  </div>
  <div id="lockMsg" class="small" style="color:#ff8b8b;margin-top:8px"></div>
</div>

<!-- Vault main UI -->
<div style="max-width:1100px;margin:24px auto;padding:16px;display:none" id="vaultUI">
  <div style="display:flex;justify-content:space-between;align-items:center">
    <div>
      <h2 style="margin:0">Private Photo & Video Vault</h2>
      <div class="small">Folders: Aditya1…Aditya10</div>
    </div>
    <div style="display:flex;gap:8px;align-items:center">
      <button id="exportBtn" class="ghost">Export</button>
      <label class="ghost" style="padding:8px;border-radius:8px;cursor:pointer"><input id="importFile" type="file" accept="application/json" style="display:none"> Import</label>
    </div>
  </div>

  <div class="folders" id="foldersGrid"></div>

  <div id="folderView" style="display:none;margin-top:12px">
    <div style="display:flex;gap:8px;align-items:center">
      <button id="backBtn" class="ghost">← Back</button>
      <h3 id="folderTitle" style="margin:0"></h3>
    </div>

    <div style="display:flex;gap:8px;align-items:center;margin-top:8px">
      <label class="ghost" style="padding:8px;border-radius:8px;cursor:pointer"><input id="fileAdd" type="file" multiple accept="image/*,video/*" style="display:none"> Add Files</label>
      <button id="toggleAutosave" class="ghost">Auto-Save: OFF</button>
      <button id="clearFolder" class="ghost" title="Remove saved files in this folder">Clear Saved</button>
    </div>

    <div id="grid" class="grid"></div>
  </div>

</div>

<!-- Viewer -->
<div class="viewer" id="viewer">
  <div id="viewerContent"></div>
  <div class="metaRow">
    <button id="prevBtn" class="ghost">Prev</button>
    <button id="nextBtn" class="ghost">Next</button>
    <button id="closeViewer" class="ghost">Close</button>
  </div>
</div>

<script>
/*
  Secure-first vault:
  - On first run: ask user to create password.
  - Use WebCrypto PBKDF2 (SHA-256) with random salt and store {salt, derived} in localStorage.
  - No plaintext password stored anywhere in file.
  - On login: derive and compare.
  - Reset option wipes DB + stored auth so new password can be set.
*/

// CONFIG
const DB_NAME = 'private-vault-secure-db';
const STORE = 'files';
const AUTH_KEY = 'vault_auth_v1'; // localStorage key for auth {salt,hash}
const FOLDERS = Array.from({length:10},(_,i)=>'Aditya'+(i+1));
let db;

// helpers: encode/decode
function toHex(buffer){
  return Array.from(new Uint8Array(buffer)).map(b=>b.toString(16).padStart(2,'0')).join('');
}
function fromHex(hex){
  const len = hex.length/2;
  const u = new Uint8Array(len);
  for(let i=0;i<len;i++) u[i]=parseInt(hex.substr(i*2,2),16);
  return u.buffer;
}
function randBytes(len){
  const b = new Uint8Array(len); crypto.getRandomValues(b); return b.buffer;
}

// derive PBKDF2 hash
async function deriveKey(password, saltHex, iterations=150000){
  const enc = new TextEncoder();
  const pwKey = await crypto.subtle.importKey('raw', enc.encode(password), {name:'PBKDF2'}, false, ['deriveBits']);
  const salt = fromHex(saltHex);
  const bits = await crypto.subtle.deriveBits(
    {name:'PBKDF2', salt, iterations, hash:'SHA-256'},
    pwKey,
    256
  );
  return toHex(bits); // hex string of derived 32 bytes
}

// Auth helpers
function authExists(){ return !!localStorage.getItem(AUTH_KEY); }
function storeAuth(saltHex, derivedHex){
  localStorage.setItem(AUTH_KEY, JSON.stringify({salt:saltHex,hash:derivedHex}));
}
function readAuth(){
  const j = localStorage.getItem(AUTH_KEY);
  if(!j) return null;
  try{ return JSON.parse(j); }catch(e){ return null; }
}
function clearAuth(){ localStorage.removeItem(AUTH_KEY); }

// IndexedDB helpers
function openDB(){
  return new Promise((res,rej)=>{
    const r = indexedDB.open(DB_NAME,1);
    r.onupgradeneeded = e=>{
      const d = e.target.result;
      if(!d.objectStoreNames.contains(STORE)){
        d.createObjectStore(STORE,{keyPath:'id',autoIncrement:true});
      }
    }
    r.onsuccess = e=>{ db = e.target.result; res(db); }
    r.onerror = e=>rej(e.target.error);
  });
}
function addItem(folder,name,type,dataUrl){
  return new Promise((res,rej)=>{
    const tx = db.transaction(STORE,'readwrite'); const s = tx.objectStore(STORE);
    const req = s.add({folder,name,type,dataUrl,created:Date.now()});
    req.onsuccess = ()=>res(req.result); req.onerror = e=>rej(e.target.error);
  });
}
function getAllItems(){
  return new Promise((res,rej)=>{
    const tx = db.transaction(STORE,'readonly'); const s = tx.objectStore(STORE);
    const req = s.getAll();
    req.onsuccess = ()=>res(req.result); req.onerror = e=>rej(e.target.error);
  });
}
function deleteItem(id){
  return new Promise((res,rej)=>{
    const tx = db.transaction(STORE,'readwrite'); const s = tx.objectStore(STORE);
    const req = s.delete(id);
    req.onsuccess = ()=>res(); req.onerror = e=>rej(e.target.error);
  });
}
function clearAllItems(){
  return new Promise((res,rej)=>{
    const tx = db.transaction(STORE,'readwrite'); const s = tx.objectStore(STORE);
    const req = s.clear();
    req.onsuccess = ()=>res(); req.onerror = e=>rej(e.target.error);
  });
}

// UI elements
const setupBox = document.getElementById('setupBox');
const loginBox = document.getElementById('loginBox');
const vaultUI = document.getElementById('vaultUI');
const unlockBtn = document.getElementById('unlockBtn');
const createBtn = document.getElementById('createBtn');
const resetBtn = document.getElementById('resetBtn');
const lockMsg = document.getElementById('lockMsg');
const setupMsg = document.getElementById('setupMsg');

async function init(){
  // if auth exists -> show login; else show setup
  if(authExists()){
    setupBox.style.display = 'none';
    loginBox.style.display = 'block';
    vaultUI.style.display = 'none';
  } else {
    setupBox.style.display = 'block';
    loginBox.style.display = 'none';
    vaultUI.style.display = 'none';
  }
  // pre-open DB so IndexedDB prompt doesn't block later
  try{ await openDB(); }catch(e){ console.warn('DB init failed', e); }
}
init();

// Setup flow
createBtn.addEventListener('click', async ()=>{
  setupMsg.textContent = '';
  const p1 = document.getElementById('newPwd').value || '';
  const p2 = document.getElementById('newPwd2').value || '';
  if(p1.length < 6){ setupMsg.textContent = 'Password kam se kam 6 characters hona chahiye.'; return; }
  if(p1 !== p2){ setupMsg.textContent = 'Passwords match nahi kar rahe.'; return; }
  // create salt, derive, store
  try{
    const saltBuf = randBytes(16);
    const saltHex = toHex(saltBuf);
    const derived = await deriveKey(p1, saltHex);
    storeAuth(saltHex, derived);
    // clear password fields
    document.getElementById('newPwd').value=''; document.getElementById('newPwd2').value='';
    // show login
    setupBox.style.display = 'none';
    loginBox.style.display = 'block';
    lockMsg.textContent = 'Password created. Please login.';
  }catch(err){
    console.error(err); setupMsg.textContent = 'Setup failed: '+err.message;
  }
});
document.getElementById('cancelSetup').addEventListener('click', ()=>{ setupBox.style.display='none'; loginBox.style.display='block'; });

// Reset (dangerous): remove auth + DB
resetBtn.addEventListener('click', async ()=>{
  if(!confirm('Reset karne se saari saved files delete ho jayengi aur aap naya password set kar paoge. Continue?')) return;
  try{
    await clearAllItems();
  }catch(e){ console.warn('clear items error', e); }
  clearAuth();
  location.reload();
});

// Unlock flow
unlockBtn.addEventListener('click', async ()=>{
  lockMsg.textContent = '';
  const pw = document.getElementById('pwdInput').value || '';
  if(!pw){ lockMsg.textContent = 'Password dalen.'; return; }
  const auth = readAuth();
  if(!auth){ lockMsg.textContent = 'Auth missing — reload page.'; return; }
  try{
    const derived = await deriveKey(pw, auth.salt);
    if(derived === auth.hash){
      // success
      document.getElementById('pwdInput').value = '';
      loginBox.style.display = 'none';
      await openDB();
      await loadSavedIntoMemory();
      await renderGallery();
      vaultUI.style.display = 'block';
      showFolders();
    } else {
      lockMsg.textContent = 'Wrong password!';
    }
  }catch(err){
    console.error(err);
    lockMsg.textContent = 'Unlock error: '+err.message;
  }
});

// Load saved items into memory mapping
async function loadSavedIntoMemory(){
  items = {};
  for(const f of FOLDERS) items[f]=[];
  try{
    const all = await getAllItems();
    for(const it of all){
      if(!items[it.folder]) items[it.folder]=[];
      items[it.folder].push({id:it.id,name:it.name,type:it.type,dataUrl:it.dataUrl,saved:true});
    }
  }catch(e){ console.warn('load saved error', e); }
}

// Vault UI functions (same as before)
function showFolders(){
  const grid = document.getElementById('foldersGrid'); grid.innerHTML='';
  for(const f of FOLDERS){
    const div = document.createElement('div'); div.className='folder';
    div.innerHTML = `📁<br><strong>${f}</strong>`;
    div.onclick = ()=> openFolder(f);
    grid.appendChild(div);
  }
}
let currentFolder=null, galleryList=[], viewerIndex=-1;

function openFolder(folder){
  currentFolder = folder;
  document.getElementById('foldersGrid').style.display='none';
  document.getElementById('folderView').style.display='block';
  document.getElementById('folderTitle').textContent = folder;
  renderFolder();
}

function renderFolder(){
  const grid = document.getElementById('grid'); grid.innerHTML='';
  if(!items[currentFolder]) items[currentFolder]=[];
  items[currentFolder].forEach((it,idx)=>{
    const card = document.createElement('div'); card.className='card';
    if(it.type && it.type.startsWith('image')){
      const img = document.createElement('img'); img.className='thumb'; img.src = it.dataUrl; img.alt = it.name||'';
      img.onclick = ()=> openViewerByIdOrIndex(it);
      card.appendChild(img);
    } else {
      const v = document.createElement('video'); v.className='thumb'; v.src = it.dataUrl; v.preload='metadata'; v.controls = false;
      v.onclick = ()=> openViewerByIdOrIndex(it);
      card.appendChild(v);
    }
    const meta = document.createElement('div'); meta.className='metaRow';
    const nm = document.createElement('div'); nm.className='small'; nm.textContent = it.name || '(no name)';
    const btnSave = document.createElement('button'); btnSave.textContent = it.saved ? 'Saved' : 'Save';
    btnSave.style.background = it.saved ? '#0bbf7c' : '';
    btnSave.onclick = async (e)=>{ e.stopPropagation(); if(it.saved){ alert('Already saved.'); return; } try{ const id = await addItem(currentFolder, it.name || ('file-'+Date.now()), it.type, it.dataUrl); it.saved = true; it.id = id; btnSave.textContent='Saved'; btnSave.style.background='#0bbf7c'; await renderGallery(); }catch(err){ console.error(err); alert('Save failed: '+err.message); } };
    const btnRemove = document.createElement('button'); btnRemove.textContent = it.saved ? 'Delete' : 'Remove';
    btnRemove.onclick = async (e)=>{ e.stopPropagation(); if(it.saved){ if(confirm('Permanently delete this saved file?')){ await deleteItem(it.id); items[currentFolder] = items[currentFolder].filter(x=>x!==it); renderFolder(); await renderGallery(); } } else { items[currentFolder].splice(idx,1); renderFolder(); } };
    meta.appendChild(nm); meta.appendChild(btnSave); meta.appendChild(btnRemove); card.appendChild(meta); grid.appendChild(card);
  });
}

document.getElementById('fileAdd').addEventListener('change', async e=>{
  const files = Array.from(e.target.files || []);
  if(!currentFolder){ alert('Pehle folder kholen.'); e.target.value=''; return; }
  for(const f of files){
    const dataUrl = await new Promise(res=>{
      const r = new FileReader(); r.onload = ()=>res(r.result); r.readAsDataURL(f);
    });
    const type = f.type && f.type.startsWith('video') ? 'video' : 'image';
    const obj = {name:f.name, type, dataUrl, saved:false};
    items[currentFolder].push(obj);
    if(AUTOSAVE){
      try{ const id = await addItem(currentFolder, obj.name, obj.type, obj.dataUrl); obj.saved=true; obj.id=id; }catch(err){ console.error('autosave failed', err); }
    }
  }
  renderFolder(); e.target.value='';
});

async function renderGallery(){ galleryList = await getAllItems(); }

async function openViewerByIdOrIndex(it){
  galleryList = await getAllItems();
  if(it.saved && it.id){
    const idx = galleryList.findIndex(x=>x.id === it.id);
    if(idx !== -1){ openViewerByIndex(idx); return; }
  }
  // fallback: show unsaved item
  viewerIndex = -1;
  const vcont = document.getElementById('viewerContent'); vcont.innerHTML = '';
  if(it.type && it.type.startsWith('image')){ const img = document.createElement('img'); img.src = it.dataUrl; vcont.appendChild(img); }
  else { const vid = document.createElement('video'); vid.src = it.dataUrl; vid.controls = true; vid.autoplay = true; vcont.appendChild(vid); }
  document.getElementById('viewer').style.display = 'flex';
}

function openViewerByIndex(idx){
  viewerIndex = idx;
  const it = galleryList[viewerIndex];
  const vcont = document.getElementById('viewerContent'); vcont.innerHTML = '';
  if(it.type && it.type.startsWith('image')){ const img = document.createElement('img'); img.src = it.dataUrl; vcont.appendChild(img); }
  else { const vid = document.createElement('video'); vid.src = it.dataUrl; vid.controls = true; vid.autoplay = true; vcont.appendChild(vid); }
  document.getElementById('viewer').style.display = 'flex';
}
document.getElementById('prevBtn').addEventListener('click', ()=>{ if(viewerIndex>0){ viewerIndex--; openViewerByIndex(viewerIndex); }});
document.getElementById('nextBtn').addEventListener('click', ()=>{ if(viewerIndex < galleryList.length-1){ viewerIndex++; openViewerByIndex(viewerIndex); }});
document.getElementById('closeViewer').addEventListener('click', ()=>{ document.getElementById('viewer').style.display='none'; });

document.getElementById('backBtn').addEventListener('click', ()=>{ document.getElementById('folderView').style.display='none'; document.getElementById('foldersGrid').style.display='grid'; });
document.getElementById('toggleAutosave').addEventListener('click', (e)=>{ AUTOSAVE = !AUTOSAVE; e.target.textContent = 'Auto-Save: ' + (AUTOSAVE ? 'ON' : 'OFF'); });
document.getElementById('clearFolder').addEventListener('click', async ()=>{
  if(!currentFolder) return;
  if(!confirm('Remove all saved files in this folder?')) return;
  // remove saved items in DB for this folder
  const all = await getAllItems();
  const toDelete = all.filter(x=>x.folder === currentFolder).map(x=>x.id);
  for(const id of toDelete) await deleteItem(id);
  items[currentFolder] = items[currentFolder].filter(x=>!x.saved);
  renderFolder();
  await renderGallery();
});

// Export / Import
document.getElementById('exportBtn').addEventListener('click', async ()=>{
  const all = await getAllItems();
  const blob = new Blob([JSON.stringify(all)], {type:'application/json'});
  const a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'vault-backup-'+new Date().toISOString()+'.json'; document.body.appendChild(a); a.click(); a.remove();
});
document.getElementById('importFile').addEventListener('change', async (e)=>{
  const f = e.target.files[0]; if(!f) return;
  const txt = await f.text();
  try{
    const arr = JSON.parse(txt);
    if(!Array.isArray(arr)) throw new Error('Invalid backup');
    for(const it of arr){
      if(it.dataUrl && it.folder){
        await addItem(it.folder, it.name || 'file', it.type || 'image', it.dataUrl);
      }
    }
    await loadSavedIntoMemory();
    await renderGallery();
    alert('Import complete');
  }catch(err){ alert('Import failed: '+err.message); }
  e.target.value = '';
});

// initial DB open
(async ()=>{ try{ await openDB(); }catch(e){ console.warn('DB init pre-open failed', e); } })();
</script>
</body>
</html>
