<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8">
  <title>Quản lý Cơ sở vật chất — Local (THCS Hàn Thuyên)</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <!-- Bootstrap + Icons -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" rel="stylesheet">
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <!-- XLSX & FileSaver -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

  <style>
    :root{
      --bg:#f6f8fb; --card:#ffffff; --muted:#6c757d; --accent:#0d6efd;
    }
    body{ background:var(--bg); font-family:Inter,system-ui,Arial,sans-serif; color:#222; }
    .container-app{ max-width:1200px; margin:28px auto; padding:18px; }
    .brand { display:flex; align-items:center; gap:12px; }
    .hero { background:linear-gradient(90deg, rgba(13,110,253,0.06), rgba(13,110,253,0.02)); padding:20px; border-radius:14px; display:flex; gap:18px; align-items:center; }
    .hero img{ width:180px; border-radius:10px; object-fit:cover; }
    .small-muted{ color:var(--muted); }
    .card-soft{ background:var(--card); border-radius:12px; padding:12px; box-shadow: 0 6px 20px rgba(12,24,60,0.04); }
    .asset-thumb{ height:46px; width:64px; object-fit:cover; border-radius:6px; }
    .pw-strength{ height:8px; border-radius:6px; background:#e9ecef; overflow:hidden; }
    .pw-strength > i{ display:block; height:100%; width:0%; background:linear-gradient(90deg,#ff6b6b,#ffd166,#06d6a0); transition:width .25s; }
    footer { opacity:0.8; font-size:0.9rem; text-align:center; margin-top:24px; }
    .top-controls .btn { min-width:120px; }
    .zebra-row{ background:#fafafa; }
  </style>
</head>
<body>
  <div class="container-fluid bg-white shadow-sm py-2 mb-3">
    <div class="container d-flex justify-content-between align-items-center">
      <div class="brand">
        <i class="fa-solid fa-school fa-2x text-primary"></i>
        <div>
          <h4 class="mb-0">THCS Hàn Thuyên</h4>
          <small class="small-muted">Hệ thống Quản lý Cơ sở vật chất — Local</small>
        </div>
      </div>
      <div class="d-flex gap-2 align-items-center">
        <div id="sync-indicator" class="me-2"><span class="badge bg-secondary">Local</span></div>
        <div id="current-user"></div>
        <button id="btn-open-login" class="btn btn-outline-primary btn-sm">Đăng nhập</button>
        <button id="btn-open-signup" class="btn btn-outline-success btn-sm">Đăng ký</button>
        <button id="btn-logout" class="btn btn-outline-danger btn-sm" style="display:none">Đăng xuất</button>
      </div>
    </div>
  </div>

  <div class="container-app">
    <section class="hero mb-4 card-soft">
      <div style="flex:1">
        <h2 class="mb-1">Quản lý Cơ sở vật chất</h2>
        <p class="small-muted mb-2">Chạy offline bằng LocalStorage — Admin / Technician / Teacher</p>

        <div class="top-controls d-flex gap-2">
          <button id="btn-add-room" class="btn btn-primary"><i class="fa-solid fa-door-open me-2"></i>Thêm phòng</button>
          <button id="btn-add-asset" class="btn btn-outline-primary"><i class="fa-solid fa-box me-2"></i>Thêm thiết bị</button>
          <button id="btn-add-req" class="btn btn-outline-warning"><i class="fa-solid fa-wrench me-2"></i>Tạo yêu cầu</button>
          <div class="ms-auto btn-group">
            <button id="btn-export-excel" class="btn btn-success"><i class="fa-solid fa-file-excel me-1"></i>Xuất Excel</button>
            <button id="btn-clear-data" class="btn btn-outline-danger" title="Xoá toàn bộ dữ liệu (demo)"><i class="fa-solid fa-trash"></i></button>
          </div>
        </div>
      </div>
      <img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" alt="Banner">
    </section>

    <!-- stats -->
    <div class="row g-3 mb-4">
      <div class="col-md-4"><div class="card-soft h-100"><small class="small-muted">Tổng phòng</small><h3 id="stat-rooms">0</h3></div></div>
      <div class="col-md-4"><div class="card-soft h-100"><small class="small-muted">Tổng tài sản</small><h3 id="stat-assets">0</h3></div></div>
      <div class="col-md-4"><div class="card-soft h-100"><small class="small-muted">Yêu cầu đang mở</small><h3 id="stat-requests">0</h3></div></div>
    </div>

    <div class="row g-3">
      <div class="col-lg-7">
        <div class="card-soft">
          <ul class="nav nav-tabs mb-3" id="mainTabs">
            <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#tab-rooms">Phòng</a></li>
            <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-assets">Tài sản</a></li>
            <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-inv">Kho</a></li>
            <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-reqs">Yêu cầu</a></li>
            <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-logs">Lịch sử</a></li>
          </ul>

          <div class="tab-content">
            <!-- Rooms -->
            <div class="tab-pane fade show active" id="tab-rooms">
              <div class="d-flex mb-2 gap-2">
                <input id="search-rooms" class="form-control form-control-sm" placeholder="Tìm phòng...">
                <button id="btn-refresh-rooms" class="btn btn-sm btn-outline-secondary">Làm mới</button>
              </div>
              <div class="table-responsive">
                <table class="table table-hover">
                  <thead><tr><th>Mã</th><th>Tên</th><th>Sức chứa</th><th>Ghi chú</th><th>Hành động</th></tr></thead>
                  <tbody id="rooms-tbody"></tbody>
                </table>
              </div>
            </div>

            <!-- Assets -->
            <div class="tab-pane fade" id="tab-assets">
              <div class="d-flex mb-2 gap-2">
                <select id="filter-asset-status" class="form-select form-select-sm" style="max-width:160px">
                  <option value="">Tất cả</option><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option>
                </select>
                <input id="search-assets" class="form-control form-control-sm" placeholder="Tìm tài sản...">
              </div>
              <div class="table-responsive">
                <table class="table table-hover align-middle">
                  <thead><tr><th>ID</th><th>Tên</th><th>Phòng</th><th>Trạng thái</th><th>Ảnh</th><th>Hành động</th></tr></thead>
                  <tbody id="assets-tbody"></tbody>
                </table>
              </div>
            </div>

            <!-- Inventory -->
            <div class="tab-pane fade" id="tab-inv">
              <div class="mb-2"><input id="search-inv" class="form-control form-control-sm" placeholder="Tìm vật tư..."></div>
              <div class="table-responsive">
                <table class="table table-hover">
                  <thead><tr><th>Mã</th><th>Tên</th><th>Số lượng</th><th>Đơn vị</th><th>Hành động</th></tr></thead>
                  <tbody id="inv-tbody"></tbody>
                </table>
              </div>
            </div>

            <!-- Requests -->
            <div class="tab-pane fade" id="tab-reqs">
              <div class="d-flex mb-2 gap-2">
                <select id="filter-req-status" class="form-select form-select-sm" style="max-width:160px">
                  <option value="">Tất cả</option><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option>
                </select>
                <input id="search-req" class="form-control form-control-sm" placeholder="Tìm yêu cầu...">
              </div>
              <div class="table-responsive">
                <table class="table table-hover">
                  <thead><tr><th>Mã</th><th>Tiêu đề</th><th>Phòng</th><th>Người báo</th><th>Trạng thái</th><th>Hành động</th></tr></thead>
                  <tbody id="reqs-tbody"></tbody>
                </table>
              </div>
            </div>

            <!-- Logs -->
            <div class="tab-pane fade" id="tab-logs">
              <div class="table-responsive">
                <table class="table table-sm">
                  <thead><tr><th>Thời gian</th><th>Người</th><th>Hành động</th><th>Mô tả</th></tr></thead>
                  <tbody id="logs-tbody"></tbody>
                </table>
              </div>
            </div>

          </div>
        </div>
      </div>

      <!-- Right column -->
      <div class="col-lg-5">
        <div class="card-soft">
          <div class="row g-2 mb-3">
            <div class="col-6"><img src="https://images.pexels.com/photos/301926/pexels-photo-301926.jpeg" class="img-fluid rounded" alt=""></div>
            <div class="col-6"><img src="https://images.pexels.com/photos/256455/pexels-photo-256455.jpeg" class="img-fluid rounded" alt=""></div>
            <div class="col-6"><img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" class="img-fluid rounded" alt=""></div>
            <div class="col-6"><img src="https://images.pexels.com/photos/256401/pexels-photo-256401.jpeg" class="img-fluid rounded" alt=""></div>
          </div>
          <canvas id="chartStatus" height="260"></canvas>
        </div>
      </div>
    </div>

    <footer>© <span id="year"></span> THCS Hàn Thuyên — Demo LocalStorage</footer>
  </div>

  <!-- Modals -->
  <!-- Room Modal -->
  <div class="modal fade" id="modalRoom" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content">
    <form id="form-room" class="p-3">
      <h5>Thêm / Sửa phòng</h5>
      <input name="id" class="form-control mb-2" placeholder="Mã phòng (vd: R101)" required>
      <input name="name" class="form-control mb-2" placeholder="Tên phòng" required>
      <input name="capacity" type="number" class="form-control mb-2" placeholder="Sức chứa">
      <textarea name="notes" class="form-control mb-2" placeholder="Ghi chú"></textarea>
      <div class="d-flex justify-content-end gap-2"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
    </form>
  </div></div></div>

  <!-- Asset Modal -->
  <div class="modal fade" id="modalAsset" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content">
    <form id="form-asset" class="p-3">
      <h5>Thêm / Sửa tài sản</h5>
      <div class="row g-2">
        <div class="col-md-4"><input name="id" class="form-control" placeholder="ID"></div>
        <div class="col-md-8"><input name="name" class="form-control" placeholder="Tên"></div>
        <div class="col-md-4"><input name="serial" class="form-control" placeholder="Serial"></div>
        <div class="col-md-4"><input name="category" class="form-control" placeholder="Loại"></div>
        <div class="col-md-4"><input name="room" class="form-control" placeholder="Mã phòng"></div>
        <div class="col-md-4"><select name="status" class="form-select"><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option></select></div>
        <div class="col-md-4"><input name="purchased" type="date" class="form-control"></div>
        <div class="col-md-4"><input name="price" type="number" class="form-control" placeholder="Giá (VND)"></div>
        <div class="col-md-6"><input name="vendor" class="form-control" placeholder="Nhà cung cấp"></div>
        <div class="col-md-6"><input name="warranty" type="number" class="form-control" placeholder="Bảo hành (tháng)"></div>
        <div class="col-md-6"><input name="image" type="file" accept="image/*" class="form-control form-control-sm"></div>
        <div class="col-12"><textarea name="desc" class="form-control" rows="3" placeholder="Mô tả"></textarea></div>
      </div>
      <div class="mt-2 text-end"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu tài sản</button></div>
    </form>
  </div></div></div>

  <!-- Request Modal -->
  <div class="modal fade" id="modalReq" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content">
    <form id="form-req" class="p-3">
      <h5>Tạo / Sửa yêu cầu bảo trì</h5>
      <input name="id" class="form-control mb-2" placeholder="Mã yêu cầu (vd: REQ-001)" required>
      <input name="title" class="form-control mb-2" placeholder="Tiêu đề" required>
      <input name="room" class="form-control mb-2" placeholder="Phòng">
      <input name="reporter" class="form-control mb-2" placeholder="Người báo">
      <textarea name="desc" class="form-control mb-2" placeholder="Mô tả"></textarea>
      <select name="status" class="form-select mb-2"><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select>
      <div class="d-flex justify-content-end gap-2"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
    </form>
  </div></div></div>

  <!-- Login / Signup modals -->
  <div class="modal fade" id="modalLogin" tabindex="-1"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content p-3">
    <h5>Đăng nhập</h5>
    <input id="login-email" class="form-control mb-2" placeholder="Email">
    <input id="login-password" type="password" class="form-control mb-2" placeholder="Mật khẩu">
    <div class="d-flex justify-content-end gap-2"><button id="login-btn" class="btn btn-primary">Đăng nhập</button><button class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button></div>
    <div class="mt-2 small-muted">Tạo tài khoản để dùng (admin / teacher / technician)</div>
  </div></div></div>

  <div class="modal fade" id="modalSignup" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content p-3">
    <h5>Đăng ký</h5>
    <input id="su-email" class="form-control mb-2" placeholder="Email">
    <input id="su-password" type="password" class="form-control mb-2" placeholder="Mật khẩu">
    <input id="su-name" class="form-control mb-2" placeholder="Tên hiển thị">
    <select id="su-role" class="form-select mb-2"><option value="teacher">Giáo viên</option><option value="technician">Kỹ thuật viên</option><option value="admin">Admin</option></select>
    <div class="pw-strength mb-2"><i id="pw-strength-bar"></i></div>
    <div class="d-flex justify-content-end gap-2"><button id="su-btn" class="btn btn-success">Đăng ký</button><button class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button></div>
  </div></div></div>

  <!-- View modal -->
  <div class="modal fade" id="modalView" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><div class="modal-body" id="modalViewBody"></div></div></div></div>

  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

  <script>
  /******************************
   * LocalStorage "backend"
   ******************************/
  const LS = {
    users: 'kt_users_v1',
    current: 'kt_current_v1',
    rooms: 'kt_rooms_v1',
    assets: 'kt_assets_v1',
    inventory: 'kt_inventory_v1',
    requests: 'kt_requests_v1',
    logs: 'kt_logs_v1'
  };

  // Helpers
  const el = id => document.getElementById(id);
  const nowIso = () => new Date().toISOString();
  const save = (key, data) => localStorage.setItem(key, JSON.stringify(data));
  const load = (key, def=[]) => {
    try { const t = localStorage.getItem(key); return t ? JSON.parse(t) : def } catch(e){ return def; }
  };
  const uidGen = (p='X') => p + Date.now().toString().slice(-6) + Math.floor(Math.random()*900+100);
  const notify = msg => { alert(msg); };

  // Seed default users & keys if empty
  (function seed(){
    if(!localStorage.getItem(LS.users)){
      const u = [
        { id: uidGen('U'), email:'admin@local', password:'admin123', displayName:'Admin', role:'admin', username:'admin' },
        { id: uidGen('U'), email:'teacher@local', password:'teacher123', displayName:'Teacher', role:'teacher', username:'teacher' },
        { id: uidGen('U'), email:'tech@local', password:'tech123', displayName:'Technician', role:'technician', username:'technician' }
      ];
      save(LS.users, u);
    }
    if(!localStorage.getItem(LS.rooms)) save(LS.rooms, []);
    if(!localStorage.getItem(LS.assets)) save(LS.assets, []);
    if(!localStorage.getItem(LS.inventory)) save(LS.inventory, []);
    if(!localStorage.getItem(LS.requests)) save(LS.requests, []);
    if(!localStorage.getItem(LS.logs)) save(LS.logs, []);
  })();

  // Current user
  let currentUserMeta = load(LS.current, null);

  // Render user area
  function renderCurrentUser(user){
    const cu = el('current-user');
    const loginBtn = el('btn-open-login'), signupBtn = el('btn-open-signup'), logoutBtn = el('btn-logout');
    if(user){
      cu.innerHTML = `<span class="badge bg-light text-dark">${user.displayName || user.username} · ${user.role}</span>`;
      loginBtn.style.display='none'; signupBtn.style.display='none'; logoutBtn.style.display='inline-block';
    } else {
      cu.innerHTML = '';
      loginBtn.style.display='inline-block'; signupBtn.style.display='inline-block'; logoutBtn.style.display='none';
    }
  }

  function applyRoleUI(role){
    // ADMIN: full
    // TECHNICIAN: add/edit assets
    // TEACHER: view, create request, edit room numeric fields (capacity)
    el('btn-add-room').style.display = ['admin','teacher'].includes(role) ? 'inline-block' : 'none';
    el('btn-add-asset').style.display = ['admin','technician'].includes(role) ? 'inline-block' : 'none';
    el('btn-add-req').style.display = role ? 'inline-block' : 'none';
  }

  // Log action
  function logAction(action, desc){
    const logs = load(LS.logs, []);
    logs.push({ ts: nowIso(), user: currentUserMeta ? currentUserMeta.username : 'anonymous', action, desc });
    save(LS.logs, logs);
  }

  /**************** CRUD wrappers for LocalStorage ****************/
  function addRoom(room){ const rows = load(LS.rooms, []); rows.push(room); save(LS.rooms, rows); logAction('add_room', `Thêm phòng ${room.id}`); }
  function updateRoom(id, patch){ const rows = load(LS.rooms, []); const i = rows.findIndex(r=>r.id===id); if(i>-1){ rows[i] = {...rows[i], ...patch}; save(LS.rooms, rows); logAction('update_room', `Sửa phòng ${id}`); } }
  function deleteRoom(id){ let rows = load(LS.rooms, []); rows = rows.filter(r=>r.id!==id); save(LS.rooms, rows); logAction('delete_room', `Xóa phòng ${id}`); }

  function addAsset(asset){ const rows = load(LS.assets, []); rows.push(asset); save(LS.assets, rows); logAction('add_asset', `Thêm tài sản ${asset.id}`); }
  function updateAsset(id, patch){ const rows = load(LS.assets, []); const i = rows.findIndex(r=>r.id===id); if(i>-1){ rows[i] = {...rows[i], ...patch}; save(LS.assets, rows); logAction('update_asset', `Sửa tài sản ${id}`); } }
  function deleteAsset(id){ let rows = load(LS.assets, []); rows = rows.filter(r=>r.id!==id); save(LS.assets, rows); logAction('delete_asset', `Xóa tài sản ${id}`); }

  function addRequest(r){ const rows = load(LS.requests, []); rows.push({...r, created: nowIso()}); save(LS.requests, rows); logAction('add_request', `Tạo yêu cầu ${r.id}`); }
  function updateRequest(id, patch){ const rows = load(LS.requests, []); const i = rows.findIndex(r=>r.id===id); if(i>-1){ rows[i] = {...rows[i], ...patch}; save(LS.requests, rows); logAction('update_request', `Sửa yêu cầu ${id}`); } }

  /**************** Rendering functions ****************/
  function renderAll(){
    renderRooms(); renderAssets(); renderInv(); renderReqs(); renderLogs(); renderChart();
    el('year').textContent = new Date().getFullYear();
  }

  function renderRooms(){
    const rows = load(LS.rooms, []);
    const q = el('search-rooms').value.trim().toLowerCase();
    const filtered = rows.filter(r=>!q || (r.id + r.name).toLowerCase().includes(q));
    el('rooms-tbody').innerHTML = filtered.map((r,idx)=>`
      <tr class="${idx%2? 'zebra-row':''}">
        <td>${r.id}</td>
        <td>${r.name}</td>
        <td>${r.capacity||''}</td>
        <td>${r.notes||''}</td>
        <td>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-room" data-id="${r.id}">Sửa</button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-room" data-id="${r.id}">Xóa</button>
        </td>
      </tr>`).join('');
    el('stat-rooms').textContent = rows.length;
  }

  function renderAssets(){
    const rows = load(LS.assets, []);
    const q = el('search-assets').value.trim().toLowerCase();
    const statusFilter = el('filter-asset-status').value;
    const filtered = rows.filter(a=> (!q || (a.id+a.name+(a.room||'')).toLowerCase().includes(q)) && (!statusFilter || a.status===statusFilter));
    el('assets-tbody').innerHTML = filtered.map((a, idx)=>`
      <tr class="${idx%2? 'zebra-row':''}">
        <td>${a.id}</td><td>${a.name}</td><td>${a.room||''}</td><td>${a.status||''}</td>
        <td>${a.image? '<img src="'+a.image+'" class="asset-thumb">' : ''}</td>
        <td>
          <button class="btn btn-sm btn-outline-info me-1" data-act="view-asset" data-id="${a.id}">Xem</button>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-asset" data-id="${a.id}">Sửa</button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-asset" data-id="${a.id}">Xóa</button>
        </td>
      </tr>`).join('');
    el('stat-assets').textContent = rows.length;
  }

  function renderInv(){
    const rows = load(LS.inventory, []);
    el('inv-tbody').innerHTML = rows.map((i, idx)=>`<tr class="${idx%2? 'zebra-row':''}"><td>${i.id}</td><td>${i.name}</td><td>${i.qty}</td><td>${i.unit||''}</td><td><button class="btn btn-sm btn-outline-primary" data-act="edit-inv" data-id="${i.id}">Sửa</button></td></tr>`).join('');
  }

  function renderReqs(){
    const rows = load(LS.requests, []);
    const q = el('search-req').value.trim().toLowerCase(); const statusFilter = el('filter-req-status').value;
    const filtered = rows.filter(r=> (!q || (r.id+r.title+r.reporter).toLowerCase().includes(q)) && (!statusFilter || r.status===statusFilter));
    el('reqs-tbody').innerHTML = filtered.map((r,idx)=>`<tr class="${idx%2? 'zebra-row':''}"><td>${r.id}</td><td>${r.title}</td><td>${r.room||''}</td><td>${r.reporter||''}</td><td>${r.status}</td><td><button class="btn btn-sm btn-outline-info me-1" data-act="view-req" data-id="${r.id}">Xem</button><button class="btn btn-sm btn-outline-primary" data-act="edit-req" data-id="${r.id}">Sửa</button></td></tr>`).join('');
    const openCount = rows.filter(x=>x.status==='open').length;
    el('stat-requests').textContent = openCount;
  }

  function viewRequest(id){
    const rows = load(LS.requests, []);
    const r = rows.find(x=>x.id===id);
    if(!r) return notify('Không tìm thấy yêu cầu');
    const html = `<p><strong>Mã:</strong> ${r.id}</p><p><strong>Tiêu đề:</strong> ${r.title}</p><p><strong>Phòng:</strong> ${r.room||''}</p><p><strong>Người báo:</strong> ${r.reporter||''}</p><p><strong>Trạng thái:</strong> ${r.status}</p><p><strong>Mô tả:</strong><br>${r.desc||''}</p><p class="small-muted"><strong>Ngày tạo:</strong> ${r.created||''}</p>`;
    el('modalViewBody').innerHTML = html; new bootstrap.Modal(el('modalView')).show();
  }

  function renderLogs(){
    const rows = load(LS.logs, []).slice().reverse();
    el('logs-tbody').innerHTML = rows.map(l=>`<tr><td>${l.ts}</td><td>${l.user}</td><td>${l.action}</td><td>${l.desc}</td></tr>`).join('');
  }

  // Chart
  let chart = null;
  function renderChart(){
    const rows = load(LS.requests, []);
    const counts = { open:0, in_progress:0, done:0 };
    rows.forEach(r=> counts[r.status] = (counts[r.status]||0) + 1);
    const ctx = el('chartStatus').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, { type:'doughnut', data:{ labels:['Mới','Đang xử lý','Hoàn tất'], datasets:[{ data:[counts.open, counts.in_progress, counts.done], backgroundColor:['#ffc107','#0d6efd','#198754'] }] }, options:{responsive:true, plugins:{legend:{position:'bottom'}}} });
  }

  /**************** Forms handling ****************/
  // Room
  el('form-room').addEventListener('submit', (e)=>{
    e.preventDefault();
    const f = e.target;
    const room = { id: f['id'].value.trim(), name: f['name'].value.trim(), capacity: Number(f['capacity'].value)||0, notes: f['notes'].value.trim() };
    const rooms = load(LS.rooms, []);
    const exists = rooms.some(r=>r.id===room.id);
    if(exists){
      // Only allow teacher to edit numeric capacity and notes (as requested)
      if(currentUserMeta && currentUserMeta.role==='teacher'){
        // teacher can only change capacity and notes for a room they edit
        updateRoom(room.id, { capacity: room.capacity, notes: room.notes });
      } else {
        updateRoom(room.id, room);
      }
    } else {
      addRoom(room);
    }
    new bootstrap.Modal(el('modalRoom')).hide();
    renderAll();
  });

  // Asset image helper
  function fileToBase64(file){ return new Promise((res, rej)=>{ const fr = new FileReader(); fr.onload = ()=>res(fr.result); fr.onerror = rej; fr.readAsDataURL(file); }); }

  // Asset
  el('form-asset').addEventListener('submit', async (e)=>{
    e.preventDefault();
    if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Bạn không có quyền.'); return; }
    const f = e.target;
    const id = f['id'].value.trim() || ('AS-'+Date.now().toString().slice(-6));
    const asset = { id, name: f['name'].value.trim(), serial: f['serial'].value.trim(), category: f['category'].value.trim(), room: f['room'].value.trim(), status: f['status'].value, purchased: f['purchased'].value||'', price: Number(f['price'].value)||0, vendor: f['vendor'].value||'', warranty: Number(f['warranty'].value)||0, desc: f['desc'].value||'', updatedAt: nowIso() };
    const file = f['image'].files[0];
    if(file){
      try{ asset.image = await fileToBase64(file); } catch(e){}
    } else {
      const existing = load(LS.assets, []).find(a=>a.id===id);
      if(existing) asset.image = existing.image || '';
    }
    const exists = load(LS.assets, []).some(a=>a.id===id);
    if(exists) updateAsset(id, asset); else addAsset(asset);
    new bootstrap.Modal(el('modalAsset')).hide();
    renderAll();
  });

  // Request
  el('form-req').addEventListener('submit', (e)=>{
    e.preventDefault();
    if(!currentUserMeta){ notify('Cần đăng nhập để tạo yêu cầu'); return; }
    const f = e.target;
    const obj = { id: f['id'].value.trim(), title: f['title'].value.trim(), room: f['room'].value.trim(), reporter: f['reporter'].value || currentUserMeta.username, desc: f['desc'].value.trim(), status: f['status'].value };
    const exists = load(LS.requests, []).some(r=>r.id===obj.id);
    if(exists) updateRequest(obj.id, obj); else addRequest(obj);
    new bootstrap.Modal(el('modalReq')).hide();
    renderAll();
  });

  // Top buttons
  el('btn-add-room').addEventListener('click', ()=>{ el('form-room').reset(); el('form-room')['id'].removeAttribute('readonly'); new bootstrap.Modal(el('modalRoom')).show(); });
  el('btn-add-asset').addEventListener('click', ()=>{ if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Cần đăng nhập với quyền admin hoặc technician'); return; } el('form-asset').reset(); el('form-asset')['id'].removeAttribute('readonly'); new bootstrap.Modal(el('modalAsset')).show(); });
  el('btn-add-req').addEventListener('click', ()=>{ if(!currentUserMeta){ notify('Cần đăng nhập'); return; } el('form-req').reset(); el('form-req')['reporter'].value = currentUserMeta.username; new bootstrap.Modal(el('modalReq')).show(); });

  // Search/filter events
  el('search-rooms').addEventListener('input', debounce(()=>renderRooms(),200));
  el('search-assets').addEventListener('input', debounce(()=>renderAssets(),200));
  el('filter-asset-status').addEventListener('change', ()=>renderAssets());
  el('search-inv').addEventListener('input', debounce(()=>renderInv(),200));
  el('search-req').addEventListener('input', debounce(()=>renderReqs(),200));
  el('filter-req-status').addEventListener('change', ()=>renderReqs());

  /**************** Export Excel (Professional - Option B) ****************/
  function styleSheetProfessional(ws, sheetTitle, headerRowCount=1){
    // Add merge for title row (A1:E1) and style title
    const range = XLSX.utils.decode_range(ws['!ref']);
    // Merge first row across all columns for main title (we will add a title row before data)
    // Note: we'll assume data starts at row 2 (after inserted title)
    // Styles
    // Apply header (row headerRowCount) styling: dark blue bg, white bold, center
    for(let C = range.s.c; C <= range.e.c; ++C){
      const cellAddr = XLSX.utils.encode_cell({r: headerRowCount-1, c: C});
      const cell = ws[cellAddr];
      if(!cell) continue;
      cell.s = cell.s || {};
      cell.s.font = { name: "Times New Roman", sz: 12, bold: true, color: { rgb: "FFFFFFFF" } };
      cell.s.fill = { fgColor: { rgb: "1F4E78" } }; // dark blue
      cell.s.alignment = { horizontal: "center", vertical: "center" };
      cell.s.border = {
        top: { style: "thin", color: { rgb:"000000" } },
        bottom: { style: "thin", color: { rgb:"000000" } },
        left: { style: "thin", color: { rgb:"000000" } },
        right: { style: "thin", color: { rgb:"000000" } }
      };
    }

    // Apply zebra rows and inner borders & font for data rows
    for(let R = headerRowCount; R <= range.e.r; ++R){
      for(let C = range.s.c; C <= range.e.c; ++C){
        const cellAddr = XLSX.utils.encode_cell({r: R, c: C});
        const cell = ws[cellAddr];
        if(!cell) continue;
        cell.s = cell.s || {};
        cell.s.font = { name: "Times New Roman", sz: 11, color: { rgb: "000000" } };
        cell.s.alignment = cell.s.alignment || { horizontal: "left", vertical: "center" };
        cell.s.border = {
          top: { style: "thin", color: { rgb:"000000" } },
          bottom: { style: "thin", color: { rgb:"000000" } },
          left: { style: "thin", color: { rgb:"000000" } },
          right: { style: "thin", color: { rgb:"000000" } }
        };
      }
      // zebra: light fill for odd rows (visual)
      if((R - headerRowCount) % 2 === 1){
        for(let C = range.s.c; C <= range.e.c; ++C){
          const cellAddr = XLSX.utils.encode_cell({r: R, c: C});
          const cell = ws[cellAddr];
          if(!cell) continue;
          cell.s.fill = cell.s.fill || { fgColor: { rgb: "F7F7F7" } };
        }
      }
    }

    // Auto column widths (approximate by character length)
    const cols = [];
    for(let C = range.s.c; C <= range.e.c; ++C){
      let maxlen = 10;
      for(let R = range.s.r; R <= range.e.r; ++R){
        const cell = ws[XLSX.utils.encode_cell({r:R,c:C})];
        if(cell && cell.v){
          const v = String(cell.v);
          if(v.length > maxlen) maxlen = v.length;
        }
      }
      cols.push({ wch: Math.min(Math.max(maxlen + 2, 10), 40) });
    }
    ws['!cols'] = cols;
  }

  async function exportExcel(){
    // We'll produce multiple sheets: Rooms, Assets, Inventory, Requests, Metadata
    const rooms = load(LS.rooms, []);
    const assets = load(LS.assets, []);
    const invs = load(LS.inventory, []);
    const reqs = load(LS.requests, []);

    const wb = XLSX.utils.book_new();

    // Helper to prepend title row
    function sheetWithTitle(arr, title){
      // Convert JSON to sheet starting at row 2 (so we can insert title at row1)
      const ws = XLSX.utils.json_to_sheet(arr, { origin: 1 });
      // Insert title in A1
      const colsCount = Object.keys(arr[0] || {}).length || 1;
      const titleCell = 'A1';
      ws[titleCell] = { v: title };
      // Merge A1:... based on colsCount
      ws['!merges'] = ws['!merges'] || [];
      ws['!merges'].push({ s: { r:0, c:0 }, e: { r:0, c: colsCount - 1 } });
      // Put header row style etc later
      // Create header row (row 2 already created by json_to_sheet)
      // Adjust ref
      ws['!ref'] = XLSX.utils.encode_range(XLSX.utils.decode_range(ws['!ref']));
      return ws;
    }

    // Rooms sheet
    if(rooms.length){
      const rowsOrdered = rooms.map(r=>({Mã:r.id, Tên:r.name, Sức_chứa:r.capacity, Ghi_chú:r.notes||''}));
      const ws = sheetWithTitle(rowsOrdered, 'Danh sách phòng');
      styleSheetProfessional(ws, 'Rooms', 2);
      XLSX.utils.book_append_sheet(wb, ws, 'Rooms');
    } else {
      // empty sheet with title
      const wsEmpty = XLSX.utils.aoa_to_sheet([['Danh sách phòng']]);
      wsEmpty['!ref'] = 'A1:A1';
      styleSheetProfessional(wsEmpty, 'Rooms', 1);
      XLSX.utils.book_append_sheet(wb, wsEmpty, 'Rooms');
    }

    // Assets sheet (with currency format)
    if(assets.length){
      const assetsOrdered = assets.map(a=>({ID:a.id, Tên:a.name, Phòng:a.room||'', Trạng_thái:a.status||'', Ngày_mua:a.purchased||'', Giá:a.price||0}));
      const ws = sheetWithTitle(assetsOrdered, 'Danh sách tài sản');
      // Format price column (find index of Giá column)
      const range = XLSX.utils.decode_range(ws['!ref']);
      for(let R = 1; R <= range.e.r; ++R){
        for(let C = range.s.c; C <= range.e.c; ++C){
          const addr = XLSX.utils.encode_cell({r:R,c:C});
          const hdr = XLSX.utils.encode_cell({r:1,c:C});
          const headerCell = ws[hdr];
          if(headerCell && headerCell.v === 'Giá'){
            const priceCell = ws[addr];
            if(priceCell && typeof priceCell.v === 'number'){
              priceCell.z = '"VND" #,##0;@';
            }
          }
        }
      }
      styleSheetProfessional(ws, 'Assets', 2);
      XLSX.utils.book_append_sheet(wb, ws, 'Assets');
    } else {
      const wsEmpty = XLSX.utils.aoa_to_sheet([['Danh sách tài sản']]);
      styleSheetProfessional(wsEmpty, 'Assets', 1);
      XLSX.utils.book_append_sheet(wb, wsEmpty, 'Assets');
    }

    // Inventory sheet
    if(invs.length){
      const invOrdered = invs.map(i=>({Mã:i.id, Tên:i.name, Số_lượng:i.qty, Đơn_vị:i.unit||''}));
      const ws = sheetWithTitle(invOrdered, 'Kho vật tư');
      styleSheetProfessional(ws, 'Inventory', 2);
      XLSX.utils.book_append_sheet(wb, ws, 'Inventory');
    } else {
      const wsEmpty = XLSX.utils.aoa_to_sheet([['Kho vật tư']]);
      styleSheetProfessional(wsEmpty, 'Inventory', 1);
      XLSX.utils.book_append_sheet(wb, wsEmpty, 'Inventory');
    }

    // Requests sheet
    if(reqs.length){
      const reqOrdered = reqs.map(r=>({Mã:r.id, Tiêu_đề:r.title, Phòng:r.room||'', Người_báo:r.reporter||'', Trạng_thái:r.status||'', Ngày_tạo:r.created||''}));
      const ws = sheetWithTitle(reqOrdered, 'Yêu cầu bảo trì');
      styleSheetProfessional(ws, 'Requests', 2);
      XLSX.utils.book_append_sheet(wb, ws, 'Requests');
    } else {
      const wsEmpty = XLSX.utils.aoa_to_sheet([['Yêu cầu bảo trì']]);
      styleSheetProfessional(wsEmpty, 'Requests', 1);
      XLSX.utils.book_append_sheet(wb, wsEmpty, 'Requests');
    }

    // Metadata
    const meta = [{Key:"ExportedAt", Value: new Date().toLocaleString()}, {Key:"Rooms", Value: rooms.length}, {Key:"Assets", Value: assets.length}, {Key:"Inventory", Value: invs.length}, {Key:"Requests", Value: reqs.length}];
    const wsMeta = XLSX.utils.json_to_sheet(meta);
    styleSheetProfessional(wsMeta, 'Metadata', 1);
    XLSX.utils.book_append_sheet(wb, wsMeta, 'Metadata');

    // Write file
    const wbout = XLSX.write(wb, {bookType:'xlsx', type:'array', cellStyles:true});
    saveAs(new Blob([wbout],{type:'application/octet-stream'}), `BaoCao_CoSoVatChat_${(new Date()).toISOString().slice(0,10)}.xlsx`);
    logAction('export_excel','Xuất báo cáo Excel (professional)');
    notify('Đã xuất Excel (mẫu chuyên nghiệp).');
  }
  el('btn-export-excel').addEventListener('click', exportExcel);

  // Clear demo data (except users)
  el('btn-clear-data').addEventListener('click', ()=>{
    if(!confirm('Xoá toàn bộ data (phòng, tài sản, kho, yêu cầu, log)?')) return;
    save(LS.rooms, []); save(LS.assets, []); save(LS.inventory, []); save(LS.requests, []); save(LS.logs, []);
    renderAll();
  });

  /**************** Auth (LocalStorage) ****************/
  function loadUsers(){ return load(LS.users, []); }
  function saveUsers(u){ save(LS.users, u); }
  function loadCurrentUser(){ return load(LS.current, null); }
  function saveCurrentUser(u){ save(LS.current, u); }
  function removeCurrentUser(){ localStorage.removeItem(LS.current); }

  function doSignup(email, password, name, role){
    const users = loadUsers();
    if(users.some(x=>x.email===email)) return {ok:false, reason:'exists'};
    const user = { id: uidGen('U'), email, password, displayName: name || email.split('@')[0], role, username: email.split('@')[0], created: nowIso() };
    users.push(user); saveUsers(users);
    logAction('signup', `Đăng ký ${email}`);
    return {ok:true, user};
  }

  function doLogin(email, password){
    const users = loadUsers();
    const u = users.find(x=>x.email===email && x.password===password);
    if(!u) return null;
    saveCurrentUser(u);
    logAction('login', `Đăng nhập ${email}`);
    return u;
  }

  // UI handlers for auth
  const modalLogin = new bootstrap.Modal(el('modalLogin'));
  const modalSignup = new bootstrap.Modal(el('modalSignup'));

  el('btn-open-login').addEventListener('click', ()=> modalLogin.show());
  el('btn-open-signup').addEventListener('click', ()=> modalSignup.show());
  el('btn-logout').addEventListener('click', ()=> {
    if(!confirm('Đăng xuất?')) return;
    removeCurrentUser();
    currentUserMeta = null;
    renderCurrentUser(null);
    applyRoleUI(null);
    renderAll();
  });

  el('login-btn').addEventListener('click', ()=>{
    const email = el('login-email').value.trim();
    const pass = el('login-password').value;
    const u = doLogin(email, pass);
    if(!u) return notify('Sai email hoặc mật khẩu');
    currentUserMeta = u;
    renderCurrentUser(u);
    applyRoleUI(u.role);
    modalLogin.hide();
    el('login-email').value=''; el('login-password').value='';
    renderAll();
  });

  el('su-btn').addEventListener('click', ()=>{
    const email = el('su-email').value.trim();
    const pass = el('su-password').value;
    const name = el('su-name').value.trim();
    const role = el('su-role').value;
    if(!email || !pass){ notify('Nhập email và mật khẩu'); return; }
    const res = doSignup(email, pass, name, role);
    if(!res.ok) return notify('Email đã tồn tại');
    modalSignup.hide();
    el('su-email').value=''; el('su-password').value=''; el('su-name').value='';
    notify('Đăng ký thành công. Vui lòng đăng nhập.');
  });

  el('su-password').addEventListener('input', (e)=>{
    const val = e.target.value; let score=0;
    if(val.length>=8) score+=40; if(/[A-Z]/.test(val) && /[0-9]/.test(val)) score+=30; if(/[^A-Za-z0-9]/.test(val)) score+=30;
    el('pw-strength-bar').style.width = Math.min(score,100) + '%';
  });

  // Try load current user on start
  (function initAuth(){
    const u = loadCurrentUser();
    if(u){ currentUserMeta = u; renderCurrentUser(u); applyRoleUI(u.role); }
    else { renderCurrentUser(null); applyRoleUI(null); }
  })();

  /**************** Event delegation for table buttons ****************/
  document.body.addEventListener('click', (e)=>{
    const btn = e.target.closest('button'); if(!btn) return;
    const act = btn.dataset.act, id = btn.dataset.id;
    if(!act) return;
    try{
      if(act==='edit-room'){ const r = load(LS.rooms, []).find(x=>x.id===id); if(!r) return notify('Không tìm thấy'); const f = el('form-room'); f['id'].value=r.id; f['id'].setAttribute('readonly','readonly'); f['name'].value=r.name; f['capacity'].value=r.capacity||''; f['notes'].value=r.notes||''; new bootstrap.Modal(el('modalRoom')).show(); }
      if(act==='del-room'){ if(!currentUserMeta || currentUserMeta.role!=='admin'){ notify('Chỉ Admin được xóa'); return; } if(!confirm('Xóa phòng?')) return; deleteRoom(id); renderAll(); }
      if(act==='view-asset'){ const a = load(LS.assets, []).find(x=>x.id===id); if(!a) return notify('Không tìm thấy'); const html = `<h5>${a.name}</h5><p>ID: ${a.id}<br>Phòng: ${a.room||''}<br>Trạng thái: ${a.status||''}</p>${a.image? '<img src="'+a.image+'" style="width:100%;border-radius:8px">' : ''}<p class="small-muted">Ngày mua: ${a.purchased||''} · Giá: ${a.price? a.price.toLocaleString() : ''}</p>`; el('modalViewBody').innerHTML = html; new bootstrap.Modal(el('modalView')).show(); }
      if(act==='edit-asset'){ const a = load(LS.assets, []).find(x=>x.id===id); if(!a) return notify('Không tìm thấy'); if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Bạn không có quyền chỉnh sửa'); return; } const f = el('form-asset'); f['id'].value=a.id; f['id'].setAttribute('readonly','readonly'); f['name'].value=a.name; f['serial'].value=a.serial||''; f['category'].value=a.category||''; f['room'].value=a.room||''; f['status'].value=a.status||'normal'; f['purchased'].value=a.purchased||''; f['price'].value=a.price||''; f['vendor'].value=a.vendor||''; f['warranty'].value=a.warranty||''; f['desc'].value=a.desc||''; new bootstrap.Modal(el('modalAsset')).show(); }
      if(act==='del-asset'){ if(!currentUserMeta || currentUserMeta.role!=='admin'){ notify('Chỉ Admin được xóa'); return; } if(!confirm('Xóa tài sản?')) return; deleteAsset(id); renderAll(); }
      if(act==='view-req'){ viewRequest(id); }
      if(act==='edit-req'){ const r = load(LS.requests, []).find(x=>x.id===id); if(!r) return notify('Không tìm thấy'); const f = el('form-req'); f['id'].value=r.id; f['id'].setAttribute('readonly','readonly'); f['title'].value=r.title; f['room'].value=r.room||''; f['reporter'].value=r.reporter||''; f['desc'].value=r.desc||''; f['status'].value=r.status||'open'; new bootstrap.Modal(el('modalReq')).show(); }
    }catch(err){ notify(err.message); }
  });

  /**************** Utilities ****************/
  function debounce(fn, ms=200){ let t; return (...args)=>{ clearTimeout(t); t=setTimeout(()=>fn(...args), ms); }; }

  // Initial render
  renderAll();

  </script>
</body>
</html>
