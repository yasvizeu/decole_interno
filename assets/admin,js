// ── Firebase config ───────────────────────────
firebase.initializeApp({
  apiKey:            'AIzaSyDnpVi7jwj53E7uzPds8xBz85rN3kPminw',
  authDomain:        'decole-d9f79.firebaseapp.com',
  projectId:         'decole-d9f79',
  storageBucket:     'decole-d9f79.firebasestorage.app',
  messagingSenderId: '878423782149',
  appId:             '1:878423782149:web:93e76ff1d44d3dbf6dfd6c'
});
const auth = firebase.auth();
const db   = firebase.firestore();

// ── State ──────────────────────────────────────
let cursosData = [];
let editingId  = null;
let isSaving   = false;

// ── Helpers ────────────────────────────────────
const $ = id => document.getElementById(id);
function esc(s){ return String(s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function fmt(v){ const n=parseFloat(v); return isNaN(n)||!v?null:'R$ '+n.toFixed(2).replace('.',','); }
let toastT;
function toast(msg, type=''){
  const t=$('toast'); t.textContent=msg; t.className='show'+(type?' '+type:'');
  clearTimeout(toastT); toastT=setTimeout(()=>t.className='',3500);
}

// ── Auth ───────────────────────────────────────
window.doLogin = async function(){
  const btn=$('loginBtn'), err=$('loginError');
  const email=$('loginEmail').value.trim();
  const pass =$('loginPass').value;
  if(!email||!pass){ err.textContent='Preencha e-mail e senha.'; err.style.display='block'; return; }
  btn.textContent='Entrando...'; btn.disabled=true; err.style.display='none';
  try {
    await auth.signInWithEmailAndPassword(email, pass);
  } catch(e){
    const msgs = {
      'auth/invalid-credential':    'E-mail ou senha incorretos.',
      'auth/user-not-found':        'Usuário não encontrado.',
      'auth/wrong-password':        'Senha incorreta.',
      'auth/too-many-requests':     'Muitas tentativas. Tente mais tarde.',
      'auth/invalid-email':         'E-mail inválido.'
    };
    err.textContent = msgs[e.code] || 'Erro ao entrar. Tente novamente.';
    err.style.display = 'block';
    btn.textContent='Entrar'; btn.disabled=false;
  }
};

window.doLogout = async function(){
  await auth.signOut();
};

auth.onAuthStateChanged(user => {
  if(user){
    $('loginScreen').style.display = 'none';
    $('app').style.display         = 'block';
    // User info
    const name = user.email.split('@')[0];
    $('userEmail').textContent  = user.email;
    $('userAvatar').textContent = name[0].toUpperCase();
    setStatus('ok');
    loadCursos();
  } else {
    $('loginScreen').style.display = 'flex';
    $('app').style.display         = 'none';
    $('loginBtn').textContent      = 'Entrar';
    $('loginBtn').disabled         = false;
  }
});

// ── Status ─────────────────────────────────────
function setStatus(s){
  const dot=$('statusDot'), txt=$('statusTxt');
  dot.className='status-dot';
  if(s==='ok') { dot.classList.add('ok'); txt.textContent='Firebase conectado'; }
  else if(s==='err'){ dot.classList.add('err'); txt.textContent='Erro na conexão'; }
  else { txt.textContent='Conectando...'; }
}

// ── Panels ─────────────────────────────────────
const PANELS = {
  dashboard:{ title:'Dashboard', sub:'Visão geral dos cursos' },
  cursos:   { title:'Cursos',    sub:'Gerencie os cursos da Decole' }
};
window.showPanel = function(id){
  document.querySelectorAll('.panel').forEach(p=>p.classList.remove('active'));
  document.querySelectorAll('.nav-item').forEach(n=>n.classList.remove('active'));
  $('panel-'+id).classList.add('active');
  $('nav-'+id).classList.add('active');
  $('pageTitle').textContent = PANELS[id].title;
  $('pageSub').textContent   = PANELS[id].sub;
  closeSidebar();
};

// ── Sidebar ────────────────────────────────────
window.toggleSidebar = function(){
  $('sidebar').classList.toggle('open');
  $('overlayBg').style.display = $('sidebar').classList.contains('open') ? 'block':'none';
};
function closeSidebar(){
  $('sidebar').classList.remove('open');
  $('overlayBg').style.display='none';
}
window.closeSidebar = closeSidebar;

// ── Load cursos (realtime) ─────────────────────
function loadCursos(){
  const q = db.collection('cursos').orderBy('nome');
  q.onSnapshot(snap => {
    cursosData = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    renderDash();
    renderCursos();
    $('navBadgeCursos').textContent = cursosData.length;
  }, err => {
    console.error(err);
    setStatus('err');
    toast('Erro ao carregar cursos.','err');
  });
}

// ── Dashboard ──────────────────────────────────
function renderDash(){
  const mods   = cursosData.reduce((s,c)=>s+(c.modulos||[]).length,0);
  const prices = cursosData.map(c=>parseFloat(c.mensalidade)||0).filter(v=>v>0);
  $('stCursos').textContent  = cursosData.length;
  $('stModulos').textContent = mods;
  $('stMensal').textContent  = prices.length ? 'R$ '+Math.min(...prices).toFixed(2).replace('.',',') : '—';
  $('stUpdate').textContent  = new Date().toLocaleString('pt-BR',{dateStyle:'short',timeStyle:'short'});

  const tb = $('dashTable');
  if(!cursosData.length){
    tb.innerHTML='<tr><td colspan="4" style="padding:30px;text-align:center;color:var(--muted)">Nenhum curso cadastrado.</td></tr>'; return;
  }
  tb.innerHTML = cursosData.slice(0,6).map(c=>`
    <tr>
      <td class="td-name">${esc(c.nome)}</td>
      <td class="td-muted">${(c.modulos||[]).length} módulos</td>
      <td>${fmt(c.mensalidade)||'—'}</td>
      <td>${fmt(c.valorTotal)||'—'}</td>
    </tr>`).join('');
}

// ── Cursos table ───────────────────────────────
function renderCursos(){
  const tb=$('cursosTable');
  if(!cursosData.length){
    tb.innerHTML=`<tr><td colspan="8"><div class="empty-state"><div class="empty-icon">📚</div><p>Nenhum curso ainda. Clique em <strong>＋ Novo curso</strong>.</p></div></td></tr>`;
    return;
  }
  tb.innerHTML = cursosData.map(c=>`
    <tr>
      <td class="td-name">${esc(c.nome)}</td>
      <td class="td-muted">${esc(c.paraQuem)}</td>
      <td class="td-muted">${esc(c.duracao)}</td>
      <td>${fmt(c.mensalidade)||'—'}</td>
      <td>${fmt(c.valorTotal)||'—'}</td>
      <td class="td-muted" style="font-size:0.78rem;line-height:1.7">
        ${c.materialTotal ? 'Total: '+fmt(c.materialTotal)+'<br>':''}
        ${c.materialVista ? 'À vista: '+fmt(c.materialVista)+'<br>':''}
        ${c.materialParcelado ? esc(c.materialParcelado):''}
        ${!c.materialTotal&&!c.materialVista&&!c.materialParcelado?'—':''}
      </td>
      <td><span class="badge badge-blue">${(c.modulos||[]).length} módulos</span></td>
      <td>
        <div class="actions-cell">
          <button class="btn btn-ghost btn-sm btn-icon" data-edit="${esc(c.id)}" title="Editar">✏️</button>
          <button class="btn btn-danger btn-sm btn-icon" data-del="${esc(c.id)}" title="Excluir">🗑️</button>
        </div>
      </td>
    </tr>`).join('');

  tb.querySelectorAll('[data-edit]').forEach(b=>b.addEventListener('click',()=>openModal(b.dataset.edit)));
  tb.querySelectorAll('[data-del]').forEach(b=>b.addEventListener('click',()=>delCurso(b.dataset.del)));
}

// ── Modal ──────────────────────────────────────
window.openModal = function(id){
  editingId = id || null;
  $('modalTitle').textContent = id ? 'Editar curso' : 'Novo curso';
  const c = id ? cursosData.find(x=>x.id===id) : null;

  $('f-nome').value      = c?.nome            || '';
  $('f-desc').value      = c?.descricao       || '';
  $('f-paraQuem').value  = c?.paraQuem        || '';
  $('f-duracao').value   = c?.duracao         || '';
  $('f-mensalidade').value = c?.mensalidade   || '';
  $('f-valorTotal').value  = c?.valorTotal    || '';
  $('f-matTotal').value  = c?.materialTotal   || '';
  $('f-matVista').value  = c?.materialVista   || '';
  $('f-matParc').value   = c?.materialParcelado|| '';

  $('modsContainer').innerHTML='';
  (c?.modulos||[]).forEach(m=>addMod(m.titulo,m.descricao,m.id));
  if(!(c?.modulos?.length)) addMod();

  const btn=$('btnSave');
  btn.textContent='💾 Salvar curso'; btn.disabled=false;
  isSaving=false;
  $('modalOverlay').classList.add('open');
  setTimeout(()=>$('f-nome').focus(),80);
};

window.closeModal = function(){
  if(isSaving) return;
  $('modalOverlay').classList.remove('open');
  editingId=null;
};

window.addMod = function(titulo='', desc='', modId=''){
  const wrap=$('modsContainer');
  const div=document.createElement('div');
  div.className='mod-item';
  div.dataset.id=modId||'mod_'+Date.now()+'_'+Math.random().toString(36).slice(2,6);
  div.innerHTML=`
    <div class="mod-fields">
      <input type="text" placeholder="Nome do módulo (ex: Introdução à Informática)" class="mod-t"/>
      <input type="text" placeholder="Descrição breve do módulo..." class="mod-d"/>
    </div>
    <button class="mod-del" type="button" title="Remover">✕</button>`;
  div.querySelector('.mod-t').value=titulo;
  div.querySelector('.mod-d').value=desc;
  div.querySelector('.mod-del').addEventListener('click',()=>div.remove());
  wrap.appendChild(div);
};

// ── Save ───────────────────────────────────────
window.saveCurso = async function(){
  if(isSaving) return;
  const nome=$('f-nome').value.trim();
  if(!nome){ toast('Informe o nome do curso!','err'); return; }

  isSaving=true;
  const btn=$('btnSave');
  btn.textContent='⏳ Salvando...'; btn.disabled=true;

  try{
    const id   = editingId || 'curso_'+Date.now();
    const slug = nome.toLowerCase()
      .normalize('NFD').replace(/[\u0300-\u036f]/g,'')
      .replace(/[^a-z0-9]+/g,'-').replace(/(^-|-$)/g,'');

    const modulos = Array.from($('modsContainer').querySelectorAll('.mod-item'))
      .map(el=>({
        id:        el.dataset.id,
        titulo:    el.querySelector('.mod-t').value.trim(),
        descricao: el.querySelector('.mod-d').value.trim()
      })).filter(m=>m.titulo);

    const data = {
      nome, slug,
      descricao:         $('f-desc').value.trim(),
      paraQuem:          $('f-paraQuem').value.trim(),
      duracao:           $('f-duracao').value.trim(),
      mensalidade:       $('f-mensalidade').value||'',
      valorTotal:        $('f-valorTotal').value||'',
      materialTotal:     $('f-matTotal').value||'',
      materialVista:     $('f-matVista').value||'',
      materialParcelado: $('f-matParc').value.trim(),
      modulos,
      updatedAt: new Date().toISOString()
    };

    await db.collection('cursos').doc(id).set(data);
    toast('Curso salvo com sucesso! ✓','ok');
    closeModal();

  } catch(e){
    console.error(e);
    toast('Erro ao salvar: '+e.message,'err');
    btn.textContent='💾 Salvar curso'; btn.disabled=false;
  } finally {
    isSaving=false;
  }
};

// ── Delete ─────────────────────────────────────
async function delCurso(id){
  if(!confirm('Excluir este curso permanentemente?')) return;
  try{
    await db.collection('cursos').doc(id).delete();
    toast('Curso excluído.','ok');
  } catch(e){
    toast('Erro ao excluir: '+e.message,'err');
  }
}

// Close modal clicking outside
$('modalOverlay').addEventListener('click', e=>{ if(e.target===$('modalOverlay')) closeModal(); });