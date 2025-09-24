<!--
  Minh Khai Primary School - Facilities Manager (server-ready)
  Single-file HTML/CSS/JS prototype WITH optional backend integration.

  How to use:
  - Local mode (default): runs fully in browser using localStorage.
    Open the file with browser or Live Server.

  - Server mode: set API_BASE to your backend URL (e.g. "http://localhost:3000/api").
    The front-end will call REST endpoints (rooms, assets, inventory, requests).
    Start backend (provided server.js) and then open this file.

  Notes:
  - This file keeps the same UI but the data layer now supports both localStorage and REST API.
  - You can deploy this file to GitHub Pages / Netlify and point API_BASE to a deployed backend.
-->

<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Minh Khai - Facilities Manager</title>
  <style>
    :root{--bg:#f6f8fa;--card:#ffffff;--accent:#0b6efd;--muted:#6b7280;--danger:#dc2626;--success:#16a34a;--maxw:1100px;--radius:10px;font-family:Inter,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial}
    *{box-sizing:border-box}
    body{margin:0;background:var(--bg);color:#0f172a;display:flex;flex-direction:column;align-items:center;padding:20px}
    .app{width:100%;max-width:var(--maxw)}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    header h1{font-size:18px;margin:0}
    .top-actions{display:flex;gap:8px}
    button{background:var(--accent);color:#fff;border:none;padding:8px 12px;border-radius:8px;cursor:pointer}
    button.ghost{background:transparent;color:var(--accent);border:1px solid var(--accent)}
    .layout{display:grid;grid-template-columns:220px 1fr;gap:18px}
    nav{background:var(--card);padding:12px;border-radius:var(--radius);box-shadow:0 2px 6px rgba(8,15,40,0.04)}
    nav ul{list-style:none;padding:0;margin:0}
    nav li{padding:8px;border-radius:8px;color:var(--muted);cursor:pointer}
    nav li.active{background:rgba(11,110,253,0.08);color:var(--accent);font-weight:600}
    main>.card{background:var(--card);padding:16px;border-radius:var(--radius);box-shadow:0 2px 6px rgba(8,15,40,0.04)}
    .grid{display:grid;grid-template-columns:repeat(3,1fr);gap:12px;margin-bottom:12px}
    .small{padding:12px;border-radius:8px;background:#fff;text-align:left}
    .small h3{margin:0;font-size:14px}
    .small p{margin:6px 0 0;color:var(--muted)}
    table{width:100%;border-collapse:collapse}
    th,td{padding:10px;border-bottom:1px solid #eee;text-align:left}
    th{background:#fafafa}
    .controls{display:flex;gap:8px}
    form{display:flex;flex-direction:column;gap:8px}
    label{font-size:13px;color:var(--muted)}
    input,select,textarea{padding:8px;border-radius:6px;border:1px solid #e6e9ef}
    .flex{display:flex;gap:8px;align-items:center}
    .right{justify-content:flex-end}
    .modal-backdrop{position:fixed;inset:0;background:rgba(4,6,12,0.45);display:none;align-items:center;justify-content:center}
    .modal{background:var(--card);padding:18px;border-radius:12px;width:100%;max-width:720px}
    footer{margin-top:18px;text-align:center;color:var(--muted)}
    @media (max-width:900px){.layout{grid-template-columns:1fr}.grid{grid-template-columns:1fr}nav{order:2}header{flex-direction:column;align-items:flex-start;gap:8px}}
  </style>
</head>
<body>
  <div class="app">
    <header>
      <h1>Quản lý cơ sở vật chất - Tiểu học Minh Khai</h1>
      <div class="top-actions">
        <button id="btn-export">Xuất dữ liệu</button>
        <button id="btn-import" class="ghost">Nhập dữ liệu</button>
        <input id="file-input" type="file" accept="application/json" style="display:none" />
      </div>
    </header>

    <div class="layout">
      <nav>
        <ul>
          <li data-view="dashboard" class="active">Bảng điều khiển</li>
          <li data-view="rooms">Phòng học</li>
          <li data-view="assets">Tài sản</li>
          <li data-view="inventory">Kho vật tư</li>
          <li data-view="maintenance">Yêu cầu bảo trì</li>
          <li data-view="reports">Báo cáo</li>
          <li data-view="settings">Cài đặt</li>
        </ul>
      </nav>

      <main>
        <section id="view-dashboard" class="card">
          <div class="grid">
            <div class="small">
              <h3 id="count-rooms">Rooms: 0</h3>
              <p>Tổng số phòng</p>
            </div>
            <div class="small">
              <h3 id="count-assets">Assets: 0</h3>
              <p>Tổng tài sản</p>
            </div>
            <div class="small">
              <h3 id="count-requests">Requests: 0</h3>
              <p>Yêu cầu bảo trì đang mở</p>
            </div>
          </div>

          <div style="display:flex;gap:12px;flex-direction:column">
            <div class="card">
              <h3 style="margin-top:0">Gần đây (last 10 changes)</h3>
              <div id="activity-log" style="max-height:220px;overflow:auto;padding-top:8px;color:var(--muted)"></div>
            </div>
          </div>
        </section>

        <section id="view-rooms" class="card" style="display:none">
          <div class="flex right"><button id="add-room">Thêm phòng</button></div>
          <table id="rooms-table">
            <thead><tr><th>Mã</th><th>Tên</th><th>Sức chứa</th><th>Ghi chú</th><th>Hành động</th></tr></thead>
            <tbody></tbody>
          </table>
        </section>

        <section id="view-assets" class="card" style="display:none">
          <div class="flex right"><button id="add-asset">Thêm tài sản</button></div>
          <table id="assets-table">
            <thead><tr><th>ID</th><th>Tên</th><th>Phòng</th><th>Trạng thái</th><th>Ngày mua</th><th>Hành động</th></tr></thead>
            <tbody></tbody>
          </table>
        </section>

        <section id="view-inventory" class="card" style="display:none">
          <div class="flex right"><button id="add-item">Nhập vật tư</button></div>
          <table id="inventory-table">
            <thead><tr><th>Mã</th><th>Tên</th><th>Số lượng</th><th>Đơn vị</th><th>Hành động</th></tr></thead>
            <tbody></tbody>
          </table>
        </section>

        <section id="view-maintenance" class="card" style="display:none">
          <div class="flex" style="margin-bottom:8px">
            <select id="filter-status"><option value="all">Tất cả</option><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select>
            <div style="flex:1"></div>
            <button id="new-request">Tạo yêu cầu</button>
          </div>
          <table id="requests-table">
            <thead><tr><th>Mã</th><th>Tiêu đề</th><th>Phòng</th><th>Người báo</th><th>Trạng thái</th><th>Hành động</th></tr></thead>
            <tbody></tbody>
          </table>
        </section>

        <section id="view-reports" class="card" style="display:none">
          <h3>Thống kê nhanh</h3>
          <div id="report-content" style="margin-top:12px"></div>
        </section>

        <section id="view-settings" class="card" style="display:none">
          <h3>Cài đặt & Dữ liệu mẫu</h3>
          <p class="muted">Bạn có thể reset dữ liệu mẫu hoặc tải dữ liệu hiện tại về.</p>
          <div class="flex" style="margin-top:12px">
            <button id="seed-data" class="ghost">Tạo dữ liệu mẫu</button>
            <button id="reset-data" class="ghost">Xóa toàn bộ dữ liệu</button>
          </div>
        </section>

      </main>
    </div>

    <footer>
      <small>Prototype - Quản lý cơ sở vật chất trường Tiểu học Minh Khai • Hỗ trợ localStorage & REST API</small>
    </footer>
  </div>

  <div id="modal" class="modal-backdrop">
    <div class="modal" role="dialog" aria-modal="true"></div>
  </div>

  <script>
    ////////////////////////////////////////////////////////////////////////////
    // CONFIG
    // If you want the frontend to call your backend, set API_BASE to its base URL.
    // Example: const API_BASE = 'http://localhost:3000/api';
    // If API_BASE is empty (''), the app runs entirely in localStorage mode.
    ////////////////////////////////////////////////////////////////////////////
    const API_BASE = ''; // <-- SET THIS to your backend URL to enable server mode
    const DB_KEY = 'minhkai_facilities_v1';

    // ---------- Data layer (supports localStorage OR REST API) ----------
    const dataLayer = (function(){
      const isServer = !!API_BASE;

      // local helpers
      async function localLoad(){
        const raw = localStorage.getItem(DB_KEY);
        if(!raw) return null;
        try{ return JSON.parse(raw); } catch(e){ console.error('Invalid local DB', e); return null; }
      }
      async function localSave(db){ localStorage.setItem(DB_KEY, JSON.stringify(db)); }

      // server helpers
      async function srvFetch(path, opts){
        const url = API_BASE + path;
        const res = await fetch(url, opts);
        if(!res.ok) throw new Error(`Server error ${res.status}`);
        return await res.json();
      }

      // public API
      return {
        // full DB (only used for dashboard/report when server mode)
        async getAll(){
          if(isServer){ return await srvFetch('/db'); }
          return await localLoad();
        },

        // rooms
        async getRooms(){ if(isServer) return await srvFetch('/rooms'); const db = await localLoad() || {rooms:[]}; return db.rooms; },
        async addRoom(room){ if(isServer) return await srvFetch('/rooms', {method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(room)}); const db=await localLoad()||{rooms:[],assets:[],inventory:[],requests:[],log:[]}; if(db.rooms.find(x=>x.id===room.id)) throw new Error('Mã phòng đã tồn tại'); db.rooms.push(room); await localSave(db); return room; },
        async updateRoom(id,patch){ if(isServer) return await srvFetch(`/rooms/${id}`,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db=await localLoad(); const idx=db.rooms.findIndex(r=>r.id===id); if(idx<0) throw new Error('Not found'); db.rooms[idx] = {...db.rooms[idx],...patch}; await localSave(db); return db.rooms[idx]; },
        async deleteRoom(id){ if(isServer) return await srvFetch(`/rooms/${id}`,{method:'DELETE'}); const db=await localLoad(); db.rooms = db.rooms.filter(r=>r.id!==id); await localSave(db); return {ok:true}; },

        // assets
        async getAssets(){ if(isServer) return await srvFetch('/assets'); const db=await localLoad()||{assets:[]}; return db.assets; },
        async addAsset(a){ if(isServer) return await srvFetch('/assets',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(a)}); const db=await localLoad()||{assets:[]}; if(db.assets.find(x=>x.id===a.id)) throw new Error('ID đã tồn tại'); db.assets.push(a); await localSave(db); return a; },
        async updateAsset(id,patch){ if(isServer) return await srvFetch(`/assets/${id}`,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db=await localLoad(); const idx=db.assets.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.assets[idx] = {...db.assets[idx],...patch}; await localSave(db); return db.assets[idx]; },
        async deleteAsset(id){ if(isServer) return await srvFetch(`/assets/${id}`,{method:'DELETE'}); const db=await localLoad(); db.assets = db.assets.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

        // inventory
        async getInventory(){ if(isServer) return await srvFetch('/inventory'); const db=await localLoad()||{inventory:[]}; return db.inventory; },
        async addItem(it){ if(isServer) return await srvFetch('/inventory',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(it)}); const db=await localLoad()||{inventory:[]}; if(db.inventory.find(x=>x.id===it.id)) throw new Error('ID đã tồn tại'); db.inventory.push(it); await localSave(db); return it; },
        async updateItem(id,patch){ if(isServer) return await srvFetch(`/inventory/${id}`,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db=await localLoad(); const idx=db.inventory.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.inventory[idx] = {...db.inventory[idx],...patch}; await localSave(db); return db.inventory[idx]; },
        async deleteItem(id){ if(isServer) return await srvFetch(`/inventory/${id}`,{method:'DELETE'}); const db=await localLoad(); db.inventory = db.inventory.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

        // requests
        async getRequests(){ if(isServer) return await srvFetch('/requests'); const db=await localLoad()||{requests:[]}; return db.requests; },
        async addRequest(req){ if(isServer) return await srvFetch('/requests',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(req)}); const db=await localLoad()||{requests:[]}; if(db.requests.find(x=>x.id===req.id)) throw new Error('ID đã tồn tại'); req.created = new Date().toISOString(); db.requests.push(req); await localSave(db); return req; },
        async updateRequest(id,patch){ if(isServer) return await srvFetch(`/requests/${id}`,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db=await localLoad(); const idx=db.requests.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.requests[idx] = {...db.requests[idx],...patch}; await localSave(db); return db.requests[idx]; },
        async deleteRequest(id){ if(isServer) return await srvFetch(`/requests/${id}`,{method:'DELETE'}); const db=await localLoad(); db.requests = db.requests.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

        // export / import
        async exportData(){ if(isServer){ window.open(API_BASE + '/export', '_blank'); return; } const db = await localLoad() || {rooms:[],assets:[],inventory:[],requests:[],log:[]}; const blob = new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href=url; a.download='minhkai_data.json'; a.click(); URL.revokeObjectURL(url); },
        async importData(file){ if(isServer){ const fd = new FormData(); fd.append('file', file); const res = await fetch(API_BASE + '/import',{method:'POST',body:fd}); if(!res.ok) throw new Error('Import failed'); return await res.json(); } const text = await file.text(); const data = JSON.parse(text); if(!data.rooms) throw new Error('Invalid format'); await localSave(data); return {ok:true}; },

        // seed/reset local-only (server can use import)
        async seedLocal(){ const now = new Date().toISOString(); const db = {rooms:[{id:'R101',name:'Lớp 1A',capacity:30,notes:'Tầng 1'},{id:'R102',name:'Lớp 1B',capacity:28,notes:'Tầng 1'}],assets:[{id:'A001',name:'Máy chiếu',room:'R101',status:'normal',purchased:'2021-08-15'},{id:'A002',name:'Bảng tương tác',room:'R102',status:'normal',purchased:'2022-09-01'}],inventory:[{id:'I001',name:'Phấn viết bảng',qty:50,unit:'bịch'},{id:'I002',name:'Giấy A4',qty:10,unit:'thùng'}],requests:[{id:'REQ001',title:'Máy chiếu không lên',room:'R101',reporter:'GV Nguyễn',desc:'Máy không khởi động',status:'open',created:now}],log:[]}; await localSave(db); return db; },
        async resetLocal(){ localStorage.removeItem(DB_KEY); return {ok:true}; }
      };
    })();

    // ---------- UI helpers & renderers (async) ----------
    const views = ['dashboard','rooms','assets','inventory','maintenance','reports','settings'];
    document.querySelectorAll('nav li').forEach(li=>li.addEventListener('click',()=>{setView(li.dataset.view)}));

    function setView(view){ views.forEach(v=>document.getElementById('view-'+v).style.display = (v===view? 'block':'none')); document.querySelectorAll('nav li').forEach(li=>li.classList.toggle('active', li.dataset.view===view)); if(view==='reports') renderReports(); }

    async function renderActivity(){ const db = await dataLayer.getAll(); const el = document.getElementById('activity-log'); el.innerHTML=''; if(!db || !db.log) return; db.log.slice(0,10).forEach(l=>{ const d=new Date(l.time); const p=document.createElement('div'); p.style.padding='6px 0'; p.textContent = `${d.toLocaleString()}: ${l.text}`; el.appendChild(p); }); }

    async function renderRooms(){ const rows = await dataLayer.getRooms(); const tbody = document.querySelector('#rooms-table tbody'); tbody.innerHTML=''; (rows||[]).forEach(r=>{ const tr=document.createElement('tr'); tr.innerHTML = `<td>${r.id}</td><td>${r.name}</td><td>${r.capacity||''}</td><td>${r.notes||''}</td><td class="controls"><button data-id="${r.id}" class="edit-room">Sửa</button><button data-id="${r.id}" class="del-room">Xóa</button></td>`; tbody.appendChild(tr); }); document.querySelectorAll('.edit-room').forEach(b=>b.addEventListener('click',e=>openRoomModal(e.target.dataset.id))); document.querySelectorAll('.del-room').forEach(b=>b.addEventListener('click',e=>{ if(confirm('Xóa phòng này?')) deleteRoom(e.target.dataset.id); })); }

    async function renderAssets(){ const rows = await dataLayer.getAssets(); const tbody = document.querySelector('#assets-table tbody'); tbody.innerHTML=''; (rows||[]).forEach(a=>{ const tr=document.createElement('tr'); tr.innerHTML = `<td>${a.id}</td><td>${a.name}</td><td>${a.room||''}</td><td>${a.status||''}</td><td>${a.purchased||''}</td><td class="controls"><button data-id="${a.id}" class="edit-asset">Sửa</button><button data-id="${a.id}" class="del-asset">Xóa</button></td>`; tbody.appendChild(tr); }); document.querySelectorAll('.edit-asset').forEach(b=>b.addEventListener('click',e=>openAssetModal(e.target.dataset.id))); document.querySelectorAll('.del-asset').forEach(b=>b.addEventListener('click',e=>{ if(confirm('Xóa tài sản?')) deleteAsset(e.target.dataset.id); })); }

    async function renderInventory(){ const rows = await dataLayer.getInventory(); const tbody = document.querySelector('#inventory-table tbody'); tbody.innerHTML=''; (rows||[]).forEach(i=>{ const tr=document.createElement('tr'); tr.innerHTML=`<td>${i.id}</td><td>${i.name}</td><td>${i.qty}</td><td>${i.unit||''}</td><td class="controls"><button data-id="${i.id}" class="edit-item">Sửa</button><button data-id="${i.id}" class="del-item">Xóa</button></td>`; tbody.appendChild(tr); }); document.querySelectorAll('.edit-item').forEach(b=>b.addEventListener('click',e=>openItemModal(e.target.dataset.id))); document.querySelectorAll('.del-item').forEach(b=>b.addEventListener('click',e=>{ if(confirm('Xóa vật tư?')) deleteItem(e.target.dataset.id); })); }

    async function renderRequests(filter){ const rows = await dataLayer.getRequests(); const tbody = document.querySelector('#requests-table tbody'); tbody.innerHTML=''; (rows||[]).forEach(r=>{ if(filter && filter!=='all' && r.status!==filter) return; const tr=document.createElement('tr'); tr.innerHTML=`<td>${r.id}</td><td>${r.title}</td><td>${r.room||''}</td><td>${r.reporter||''}</td><td>${r.status}</td><td class="controls"><button data-id="${r.id}" class="view-req">Xem</button><button data-id="${r.id}" class="edit-req">Sửa</button><button data-id="${r.id}" class="del-req">Xóa</button></td>`; tbody.appendChild(tr); }); document.querySelectorAll('.view-req').forEach(b=>b.addEventListener('click',e=>openRequestModal(e.target.dataset.id,false))); document.querySelectorAll('.edit-req').forEach(b=>b.addEventListener('click',e=>openRequestModal(e.target.dataset.id,true))); document.querySelectorAll('.del-req').forEach(b=>b.addEventListener('click',e=>{ if(confirm('Xóa yêu cầu?')) deleteRequest(e.target.dataset.id); })); }

    async function renderReports(){ const db = await dataLayer.getAll() || {rooms:[],assets:[],inventory:[],requests:[]}; const rc=(db.rooms||[]).length, ac=(db.assets||[]).length, ic=(db.inventory||[]).length; const open=(db.requests||[]).filter(r=>r.status==='open').length; const inprogress=(db.requests||[]).filter(r=>r.status==='in_progress').length; const done=(db.requests||[]).filter(r=>r.status==='done').length; const el=document.getElementById('report-content'); el.innerHTML=`<div class="grid"><div class="small"><h3>Tổng phòng</h3><p>${rc}</p></div><div class="small"><h3>Tổng tài sản</h3><p>${ac}</p></div><div class="small"><h3>Tổng vật tư</h3><p>${ic}</p></div></div><div style="margin-top:12px"><h4>Yêu cầu bảo trì</h4><p>Mới: ${open} · Đang xử lý: ${inprogress} · Hoàn tất: ${done}</p></div>`; }

    // ---------- Modal builder ----------
    const modalEl = document.getElementById('modal'); const modalBox = modalEl.querySelector('.modal'); function openModal(html){ modalBox.innerHTML = html; modalEl.style.display='flex'; modalEl.addEventListener('click',backdropClick); } function closeModal(){ modalEl.style.display='none'; modalBox.innerHTML=''; modalEl.removeEventListener('click',backdropClick); } function backdropClick(e){ if(e.target===modalEl) closeModal(); }

    // ---------- Modal forms ----------
    function openRoomModal(id){ (async ()=>{ const rows = await dataLayer.getRooms(); const room = (rows||[]).find(r=>r.id===id) || {id:'',name:'',capacity:'',notes:''}; openModal(`<h3>${id? 'Sửa phòng':'Thêm phòng'}</h3><form id="room-form"><label>Mã phòng<input required name="id" value="${room.id}" ${room.id? 'readonly':''}></label><label>Tên<input required name="name" value="${room.name}"></label><label>Sức chứa<input name="capacity" type="number" value="${room.capacity||''}"></label><label>Ghi chú<textarea name="notes">${room.notes||''}</textarea></label><div class="flex right"><button type="button" id="cancel">Hủy</button><button type="submit">Lưu</button></div></form>`); document.getElementById('cancel').addEventListener('click',closeModal); document.getElementById('room-form').addEventListener('submit',async e=>{ e.preventDefault(); const f=new FormData(e.target); const data={id:f.get('id').trim(),name:f.get('name').trim(),capacity:parseInt(f.get('capacity'))||null,notes:f.get('notes').trim()}; try{ if(id) await dataLayer.updateRoom(id,data); else await dataLayer.addRoom(data); closeModal(); await renderAll(); }catch(err){ alert(err.message); } }); })(); }

    function openAssetModal(id){ (async ()=>{ const rooms = await dataLayer.getRooms(); const assets = await dataLayer.getAssets(); const a = (assets||[]).find(x=>x.id===id) || {id:'',name:'',room:'',status:'normal',purchased:''}; const roomOptions = (rooms||[]).map(r=>`<option value="${r.id}" ${r.id===a.room? 'selected':''}>${r.name} (${r.id})</option>`).join(''); openModal(`<h3>${id? 'Sửa tài sản':'Thêm tài sản'}</h3><form id="asset-form"><label>ID<input required name="id" value="${a.id}" ${a.id? 'readonly':''}></label><label>Tên<input required name="name" value="${a.name}"></label><label>Phòng<select name="room"><option value="">-- Chọn phòng --</option>${roomOptions}</select></label><label>Trạng thái<select name="status"><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option></select></label><label>Ngày mua<input type="date" name="purchased" value="${a.purchased||''}"></label><div class="flex right"><button type="button" id="cancel">Hủy</button><button type="submit">Lưu</button></div></form>`); document.getElementById('cancel').addEventListener('click',closeModal); document.getElementById('asset-form').addEventListener('submit',async e=>{ e.preventDefault(); const f=new FormData(e.target); const data={id:f.get('id').trim(),name:f.get('name').trim(),room:f.get('room'),status:f.get('status'),purchased:f.get('purchased')}; try{ if(id) await dataLayer.updateAsset(id,data); else await dataLayer.addAsset(data); closeModal(); await renderAll(); }catch(err){ alert(err.message); } }); })(); }

    function openItemModal(id){ (async ()=>{ const inv = await dataLayer.getInventory(); const it = (inv||[]).find(x=>x.id===id)||{id:'',name:'',qty:0,unit:''}; openModal(`<h3>${id? 'Sửa vật tư':'Thêm vật tư'}</h3><form id="item-form"><label>ID<input required name="id" value="${it.id}" ${it.id? 'readonly':''}></label><label>Tên<input required name="name" value="${it.name}"></label><label>Số lượng<input required type="number" name="qty" value="${it.qty}"></label><label>Đơn vị<input name="unit" value="${it.unit}"></label><div class="flex right"><button type="button" id="cancel">Hủy</button><button type="submit">Lưu</button></div></form>`); document.getElementById('cancel').addEventListener('click',closeModal); document.getElementById('item-form').addEventListener('submit',async e=>{ e.preventDefault(); const f=new FormData(e.target); const data={id:f.get('id').trim(),name:f.get('name').trim(),qty:parseInt(f.get('qty'))||0,unit:f.get('unit').trim()}; try{ if(id) await dataLayer.updateItem(id,data); else await dataLayer.addItem(data); closeModal(); await renderAll(); }catch(err){ alert(err.message); } }); })(); }

    function openRequestModal(id,editable=true){ (async ()=>{ const rooms = await dataLayer.getRooms(); const reqs = await dataLayer.getRequests(); const r=(reqs||[]).find(x=>x.id===id) || {id:'',title:'',room:'',reporter:'',desc:'',status:'open'}; const roomOptions=(rooms||[]).map(rr=>`<option value="${rr.id}" ${rr.id===r.room? 'selected':''}>${rr.name} (${rr.id})</option>`).join(''); openModal(`<h3>${id? (editable? 'Sửa yêu cầu':'Xem yêu cầu') : 'Tạo yêu cầu'}</h3><form id="req-form">${!id? '<label>ID<input required name="id"></label>':''}<label>Tiêu đề<input required name="title" value="${r.title}"></label><label>Phòng<select name="room"><option value="">--Chọn--</option>${roomOptions}</select></label><label>Người báo<input name="reporter" value="${r.reporter}"></label><label>Mô tả<textarea name="desc">${r.desc||''}</textarea></label><label>Trạng thái<select name="status"><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select></label><div class="flex right"><button type="button" id="cancel">Hủy</button><button type="submit">Lưu</button></div></form>`); document.getElementById('cancel').addEventListener('click',closeModal); document.getElementById('req-form').addEventListener('submit',async e=>{ e.preventDefault(); const f=new FormData(e.target); const data={id: id? r.id : f.get('id').trim(), title:f.get('title').trim(), room:f.get('room'), reporter:f.get('reporter').trim(), desc:f.get('desc').trim(), status:f.get('status')}; try{ if(id) await dataLayer.updateRequest(id,data); else await dataLayer.addRequest({...data, created:new Date().toISOString()}); closeModal(); await renderAll(); }catch(err){ alert(err.message); } }); })(); }

    // ---------- CRUD wrappers that call dataLayer ----------
    async function deleteRoom(id){ try{ await dataLayer.deleteRoom(id); await renderAll(); }catch(err){ alert(err.message); } }
    async function deleteAsset(id){ try{ await dataLayer.deleteAsset(id); await renderAll(); }catch(err){ alert(err.message); } }
    async function deleteItem(id){ try{ await dataLayer.deleteItem(id); await renderAll(); }catch(err){ alert(err.message); } }
    async function deleteRequest(id){ try{ await dataLayer.deleteRequest(id); await renderAll(); }catch(err){ alert(err.message); } }

    // ---------- init & events ----------
    async function renderAll(){ await renderActivity(); await renderRooms(); await renderAssets(); await renderInventory(); await renderRequests(document.getElementById('filter-status')?.value || 'all'); const db = await dataLayer.getAll() || {rooms:[],assets:[],requests:[]}; document.getElementById('count-rooms').textContent = 'Rooms: '+ ((db.rooms||[]).length); document.getElementById('count-assets').textContent = 'Assets: '+ ((db.assets||[]).length); document.getElementById('count-requests').textContent = 'Requests: '+ ((db.requests||[]).filter(r=>r.status==='open').length); }

    document.getElementById('add-room').addEventListener('click',()=>openRoomModal(null));
    document.getElementById('add-asset').addEventListener('click',()=>openAssetModal(null));
    document.getElementById('add-item').addEventListener('click',()=>openItemModal(null));
    document.getElementById('new-request').addEventListener('click',()=>openRequestModal(null,true));
    document.getElementById('filter-status').addEventListener('change',e=>renderRequests(e.target.value));

    document.getElementById('seed-data').addEventListener('click',async ()=>{ try{ if(API_BASE){ // import seed to server
        const db = await dataLayer.seedLocal(); // create sample locally then import
        // perform import by creating a blob and using importData
        const blob = new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const file = new File([blob],'seed.json',{type:'application/json'}); await dataLayer.importData(file); alert('Đã tạo dữ liệu mẫu trên server');
      } else { await dataLayer.seedLocal(); alert('Đã tạo dữ liệu mẫu ở local'); } await renderAll(); }catch(err){ alert(err.message); } });

    document.getElementById('reset-data').addEventListener('click',async ()=>{ if(!confirm('Bạn có chắc muốn xóa toàn bộ dữ liệu?')) return; try{ if(API_BASE){ alert('Reset trên server không hỗ trợ từ đây — dùng API server-side hoặc import file rỗng.'); } else { await dataLayer.resetLocal(); alert('Đã xóa dữ liệu local'); await renderAll(); } }catch(err){ alert(err.message); } });

    document.getElementById('btn-export').addEventListener('click',async ()=>{ try{ await dataLayer.exportData(); }catch(err){ alert(err.message); } });
    document.getElementById('btn-import').addEventListener('click',()=>document.getElementById('file-input').click());
    document.getElementById('file-input').addEventListener('change',async e=>{ if(e.target.files.length){ try{ await dataLayer.importData(e.target.files[0]); alert('Đã nhập dữ liệu'); await renderAll(); }catch(err){ alert('Lỗi khi nhập dữ liệu: '+err.message); } } e.target.value=''; });

    window.addEventListener('keydown',e=>{ if(e.key==='Escape') closeModal(); });

    function genId(prefix){ const n = Math.floor(Math.random()*900+100); return prefix+(new Date().getTime().toString().slice(-4))+n; }

    // start
    setView('dashboard'); (async ()=>{ try{ if(!API_BASE){ // ensure local DB exists
        const local = await dataLayer.getAll(); if(!local) await dataLayer.seedLocal();
      } await renderAll(); }catch(e){ console.error(e); alert('Lỗi khi khởi tạo: '+e.message); } })();
  </script>
</body>
</html>
