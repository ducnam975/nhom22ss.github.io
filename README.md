<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Quản lý Cơ sở vật chất - TH Minh Khai</title>

  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <!-- FontAwesome -->
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" rel="stylesheet">
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

  <style>
    :root{
      --brand:#0d6efd;
      --muted:#6c757d;
      --card-radius:12px;
      --shadow: 0 8px 30px rgba(13,110,253,0.08);
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    body { background:#f4f6fb; }
    .app-container{ max-width:1200px; margin:20px auto; padding:0 12px; }
    .hero { background:linear-gradient(90deg, rgba(13,110,253,0.06), rgba(13,110,253,0.02)); border-radius:var(--card-radius); padding:18px; display:flex; gap:16px; align-items:center; box-shadow:var(--shadow); }
    .hero img{ width:260px; border-radius:10px; object-fit:cover; }
    .card-stat{ border-radius:12px; }
    .table-actions button{ margin-right:6px; }
    .gallery img{ border-radius:10px; object-fit:cover; height:140px; width:100%; transition:transform .25s; }
    .gallery img:hover{ transform:scale(1.05); }
    /* sidebar small */
    @media (max-width:900px){ .hero{flex-direction:column;text-align:center} .hero img{max-width:100%} }
    /* small polish */
    .small-muted{ color:var(--muted); font-size:0.9rem; }
    .role-badge { text-transform:uppercase; font-weight:600; font-size:0.75rem; padding:4px 8px; border-radius:999px; }
  </style>
</head>
<body>
  <div class="container-fluid bg-white shadow-sm py-2">
    <div class="container d-flex justify-content-between align-items-center">
      <div class="d-flex align-items-center gap-3">
        <i class="fa-solid fa-school fa-2x text-primary"></i>
        <div>
          <h4 class="mb-0">Trường Tiểu học Minh Khai</h4>
          <div class="small-muted">Hệ thống quản lý cơ sở vật chất</div>
        </div>
      </div>
      <div class="d-flex align-items-center gap-3">
        <div id="current-user" class="me-2"></div>
        <button id="btn-login" class="btn btn-outline-primary btn-sm">Đăng nhập</button>
        <button id="btn-logout" class="btn btn-outline-danger btn-sm" style="display:none">Đăng xuất</button>
      </div>
    </div>
  </div>

  <div class="app-container">
    <!-- HERO -->
    <section class="hero my-4">
      <div style="flex:1">
        <h2 class="mb-1">Hệ thống Quản lý Cơ sở vật chất</h2>
        <p class="small-muted mb-2">Quản lý phòng, tài sản, kho vật tư và yêu cầu bảo trì — trực quan, dễ dùng.</p>
        <div class="d-flex gap-2 flex-wrap">
          <button id="open-add-room" class="btn btn-primary"><i class="fa-solid fa-door-open me-2"></i>Thêm phòng</button>
          <button id="open-add-asset" class="btn btn-outline-primary"><i class="fa-solid fa-box me-2"></i>Thêm tài sản</button>
          <button id="open-add-request" class="btn btn-outline-warning"><i class="fa-solid fa-wrench me-2"></i>Tạo yêu cầu</button>
          <button id="btn-export" class="btn btn-outline-secondary"><i class="fa-solid fa-file-export me-2"></i>Xuất dữ liệu</button>
          <button id="btn-import" class="btn btn-outline-secondary"><i class="fa-solid fa-file-import me-2"></i>Nhập dữ liệu</button>
          <input id="file-input" type="file" accept="application/json" style="display:none" />
        </div>
      </div>
      <img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" alt="Banner lớp học">
    </section>

    <!-- STATS & CHART -->
    <div class="row g-3 mb-4">
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div>
              <div class="small-muted">Tổng phòng</div>
              <h3 id="stat-rooms">0</h3>
            </div>
            <div class="text-primary"><i class="fa-solid fa-door-closed fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div>
              <div class="small-muted">Tổng tài sản</div>
              <h3 id="stat-assets">0</h3>
            </div>
            <div class="text-primary"><i class="fa-solid fa-boxes-stacked fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div>
              <div class="small-muted">Yêu cầu mở</div>
              <h3 id="stat-requests">0</h3>
            </div>
            <div class="text-primary"><i class="fa-solid fa-wrench fa-2x"></i></div>
          </div>
        </div>
      </div>
    </div>

    <div class="row g-3 mb-4">
      <div class="col-lg-7">
        <!-- Table tabs -->
        <div class="card">
          <div class="card-body">
            <ul class="nav nav-tabs" id="mainTabs">
              <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#tab-rooms">Phòng học</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-assets">Tài sản</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-inv">Kho vật tư</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-reqs">Yêu cầu</a></li>
            </ul>

            <div class="tab-content mt-3">
              <!-- ROOMS -->
              <div class="tab-pane fade show active" id="tab-rooms">
                <div class="mb-2 d-flex justify-content-between">
                  <div>
                    <input id="search-rooms" class="form-control form-control-sm" placeholder="Tìm theo mã hoặc tên...">
                  </div>
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tên</th><th>Sức chứa</th><th>Ghi chú</th><th>Hành động</th></tr></thead>
                    <tbody id="rooms-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- ASSETS -->
              <div class="tab-pane fade" id="tab-assets">
                <div class="mb-2 d-flex gap-2">
                  <select id="filter-asset-status" class="form-select form-select-sm" style="max-width:180px">
                    <option value="">Tất cả trạng thái</option>
                    <option value="normal">Bình thường</option>
                    <option value="broken">Hỏng</option>
                    <option value="maintenance">Bảo trì</option>
                  </select>
                  <input id="search-assets" class="form-control form-control-sm" placeholder="Tìm tài sản...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>ID</th><th>Tên</th><th>Phòng</th><th>Trạng thái</th><th>Ngày mua</th><th>Hành động</th></tr></thead>
                    <tbody id="assets-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- INVENTORY -->
              <div class="tab-pane fade" id="tab-inv">
                <div class="d-flex gap-2 mb-2">
                  <input id="search-inv" class="form-control form-control-sm" placeholder="Tìm vật tư...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tên</th><th>Số lượng</th><th>Đơn vị</th><th>Hành động</th></tr></thead>
                    <tbody id="inv-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- REQUESTS -->
              <div class="tab-pane fade" id="tab-reqs">
                <div class="mb-2 d-flex gap-2">
                  <select id="filter-req-status" class="form-select form-select-sm" style="max-width:200px">
                    <option value="">Tất cả</option>
                    <option value="open">Mới</option>
                    <option value="in_progress">Đang xử lý</option>
                    <option value="done">Hoàn tất</option>
                  </select>
                  <input id="search-req" class="form-control form-control-sm" placeholder="Tìm yêu cầu...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tiêu đề</th><th>Phòng</th><th>Người báo</th><th>Trạng thái</th><th>Hành động</th></tr></thead>
                    <tbody id="reqs-tbody"></tbody>
                  </table>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Gallery & Chart -->
      <div class="col-lg-5">
        <div class="card mb-3">
          <div class="card-body">
            <h6 class="mb-3">Hình ảnh nổi bật</h6>
            <div class="row g-2">
              <div class="col-6"><img src="https://images.pexels.com/photos/301926/pexels-photo-301926.jpeg" class="img-fluid gallery" alt="Học sinh"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256455/pexels-photo-256455.jpeg" class="img-fluid gallery" alt="Sân trường"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" class="img-fluid gallery" alt="Lớp học"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256401/pexels-photo-256401.jpeg" class="img-fluid gallery" alt="Thiết bị"></div>
            </div>
          </div>
        </div>

        <div class="card">
          <div class="card-body">
            <h6 class="mb-3">Thống kê nhanh</h6>
            <canvas id="chartStatus" height="220"></canvas>
          </div>
        </div>
      </div>
    </div>

    <footer class="text-center small-muted mb-5">© 2025 Trường Tiểu học Minh Khai — Prototype chuyên nghiệp (Frontend)</footer>

  </div>

  <!-- ===== Modals (Room/Asset/Inventory/Request/Login/View) ===== -->
  <!-- Modal: Room -->
  <div class="modal fade" id="modalRoom" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-room" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa phòng</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã phòng</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tên phòng</label><input name="name" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Sức chứa</label><input name="capacity" type="number" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Ghi chú</label><textarea name="notes" class="form-control"></textarea></div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Asset -->
  <div class="modal fade" id="modalAsset" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-asset" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa tài sản</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">ID</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tên</label><input name="name" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Phòng (mã)</label><input name="room" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Trạng thái</label>
        <select name="status" class="form-select">
          <option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option>
        </select>
      </div>
      <div class="mb-2"><label class="form-label">Ngày mua</label><input name="purchased" type="date" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Ảnh (tùy chọn)</label><input name="image" type="file" accept="image/*" class="form-control form-control-sm"></div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Inventory -->
  <div class="modal fade" id="modalInv" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-inv" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa vật tư</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tên</label><input name="name" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Số lượng</label><input name="qty" type="number" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Đơn vị</label><input name="unit" class="form-control"></div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Request -->
  <div class="modal fade" id="modalReq" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-req" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Tạo / Sửa yêu cầu bảo trì</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tiêu đề</label><input name="title" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Phòng (mã)</label><input name="room" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Người báo</label><input name="reporter" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Mô tả</label><textarea name="desc" class="form-control"></textarea></div>
      <div class="mb-2"><label class="form-label">Trạng thái</label>
        <select name="status" class="form-select">
          <option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option>
        </select>
      </div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Login -->
  <div class="modal fade" id="modalLogin" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content"><form id="form-login">
    <div class="modal-header"><h5 class="modal-title">Đăng nhập (Demo)</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Tên đăng nhập</label><input id="login-username" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Mật khẩu</label><input id="login-password" type="password" class="form-control" required></div>
      <div class="small-muted">Tài khoản demo: admin/admin, teacher/teacher, tech/tech</div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Đăng nhập</button></div>
  </form></div></div></div>

  <!-- Modal: View Request (detail) -->
  <div class="modal fade" id="modalViewReq" tabindex="-1" aria-hidden="true"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content">
    <div class="modal-header"><h5 class="modal-title" id="view-req-title">Yêu cầu</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body" id="view-req-body"></div>
    <div class="modal-footer"><button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Đóng</button></div>
  </div></div></div>

  <!-- Bootstrap JS bundle -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

  <!-- ==================== App JS ==================== -->
  <script>
  /***************************************************************
   * CONFIG
   * - API_BASE: đặt URL backend nếu bạn có server (ví dụ: 'http://localhost:3000/api')
   * - Nếu API_BASE = '' -> chạy localStorage mode (demo)
   ***************************************************************/
  const API_BASE = ''; // <-- set backend URL to enable server mode (must implement REST API)
  const DB_KEY = 'minhkai_prod_v1';

  /***************** Simple auth demo (no real security) *****************/
  const demoUsers = {
    "admin": { username:'admin', role:'admin', name:'Admin' },
    "teacher": { username:'teacher', role:'teacher', name:'GV. Nguyễn' },
    "tech": { username:'tech', role:'technician', name:'Kỹ thuật' }
  };
  const demoPasswords = { admin:'admin', teacher:'teacher', tech:'tech' };

  function saveCurrentUser(u){ localStorage.setItem('mk_user', JSON.stringify(u)); renderCurrentUser(); }
  function clearCurrentUser(){ localStorage.removeItem('mk_user'); renderCurrentUser(); }
  function getCurrentUser(){ try{ return JSON.parse(localStorage.getItem('mk_user')); }catch(e){ return null; } }

  function renderCurrentUser(){
    const cu = getCurrentUser();
    const el = document.getElementById('current-user');
    const loginBtn = document.getElementById('btn-login');
    const logoutBtn = document.getElementById('btn-logout');
    if(cu){ el.innerHTML = `<span class="role-badge bg-light border">${cu.name} · ${cu.role}</span>`; loginBtn.style.display='none'; logoutBtn.style.display='inline-block'; }
    else { el.innerHTML = ''; loginBtn.style.display='inline-block'; logoutBtn.style.display='none'; }
    applyRoleUI();
  }

  document.getElementById('btn-login').addEventListener('click', ()=> new bootstrap.Modal(document.getElementById('modalLogin')).show());
  document.getElementById('btn-logout').addEventListener('click', ()=> { if(confirm('Đăng xuất?')){ clearCurrentUser(); } });

  document.getElementById('form-login').addEventListener('submit', e=>{
    e.preventDefault();
    const u = document.getElementById('login-username').value.trim();
    const p = document.getElementById('login-password').value;
    if(demoUsers[u] && demoPasswords[u] === p){ saveCurrentUser(demoUsers[u]); bootstrap.Modal.getInstance(document.getElementById('modalLogin')).hide(); alert('Đăng nhập thành công: '+demoUsers[u].role); } else alert('Sai thông tin đăng nhập (demo).');
    document.getElementById('form-login').reset();
  });

  function requireRole(roles){ const cu = getCurrentUser(); if(!cu) return false; return roles.includes(cu.role); }
  function applyRoleUI(){
    // Example: only admin can delete resources
    const deleteButtons = document.querySelectorAll('[data-act="del"]');
    deleteButtons.forEach(b=> b.style.display = requireRole(['admin']) ? 'inline-block' : 'none');
    // show add buttons only for admin + teacher
    document.getElementById('open-add-room').style.display = requireRole(['admin','teacher']) ? 'inline-block' : 'none';
    document.getElementById('open-add-asset').style.display = requireRole(['admin']) ? 'inline-block' : 'none';
    // open-add-request available for teacher & admin & technician
    document.getElementById('open-add-request').style.display = requireRole(['admin','teacher','technician']) ? 'inline-block' : 'none';
  }

  /***************** Data layer (localStorage / optional REST) *****************/
  const dataLayer = (function(){
    const isServer = !!API_BASE;
    async function localLoad(){ try{ return JSON.parse(localStorage.getItem(DB_KEY)); } catch(e){ return null; } }
    async function localSave(db){ localStorage.setItem(DB_KEY, JSON.stringify(db)); }
    async function srvFetch(path, opts){ const res = await fetch(API_BASE + path, opts); if(!res.ok) throw new Error('Server error '+res.status); return await res.json(); }

    return {
      // full DB
      async getAll(){ if(isServer) return await srvFetch('/db'); return await localLoad(); },
      async export(){ if(isServer){ window.open(API_BASE + '/export','_blank'); return; } const db = await localLoad() || {rooms:[],assets:[],inventory:[],requests:[],log:[]}; const blob=new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='minhkai_export.json'; a.click(); },
      async import(file){ if(isServer){ const fd=new FormData(); fd.append('file', file); const res=await fetch(API_BASE + '/import',{method:'POST',body:fd}); if(!res.ok) throw new Error('Import failed'); return await res.json(); } const text = await file.text(); const data = JSON.parse(text); if(!data.rooms) throw new Error('Invalid file'); await localSave(data); return {ok:true}; },
      async seed(){ const now=new Date().toISOString(); const db={rooms:[{id:'R101',name:'Lớp 1A',capacity:30,notes:'Tầng 1'},{id:'R102',name:'Lớp 1B',capacity:28,notes:'Tầng 1'}],assets:[{id:'A001',name:'Máy chiếu',room:'R101',status:'normal',purchased:'2021-08-15',image:''}],inventory:[{id:'I001',name:'Phấn viết bảng',qty:50,unit:'bịch'}],requests:[{id:'REQ001',title:'Máy chiếu không lên',room:'R101',reporter:'GV. A',desc:'Máy không khởi động',status:'open',created:now}],log:[]}; if(isServer){ const blob=new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const file=new File([blob],'seed.json',{type:'application/json'}); await this.import(file); return db; } await localSave(db); return db; },

      // ROOMS
      async getRooms(){ if(isServer) return await srvFetch('/rooms'); const db = await localLoad() || {rooms:[]}; return db.rooms; },
      async addRoom(r){ if(isServer) return await srvFetch('/rooms',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(r)}); const db = await localLoad() || {rooms:[],assets:[],inventory:[],requests:[],log:[]}; if(db.rooms.find(x=>x.id===r.id)) throw new Error('Mã phòng đã tồn tại'); db.rooms.push(r); await localSave(db); return r; },
      async updateRoom(id,patch){ if(isServer) return await srvFetch('/rooms/'+id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db = await localLoad(); const idx = db.rooms.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.rooms[idx] = {...db.rooms[idx],...patch}; await localSave(db); return db.rooms[idx]; },
      async deleteRoom(id){ if(isServer) return await srvFetch('/rooms/'+id,{method:'DELETE'}); const db = await localLoad(); db.rooms = db.rooms.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

      // ASSETS
      async getAssets(){ if(isServer) return await srvFetch('/assets'); const db = await localLoad() || {assets:[]}; return db.assets; },
      async addAsset(a){ if(isServer) return await srvFetch('/assets',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(a)}); const db = await localLoad()||{assets:[]}; if(db.assets.find(x=>x.id===a.id)) throw new Error('ID đã tồn tại'); db.assets.push(a); await localSave(db); return a; },
      async updateAsset(id,patch){ if(isServer) return await srvFetch('/assets/'+id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db = await localLoad(); const idx=db.assets.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.assets[idx] = {...db.assets[idx],...patch}; await localSave(db); return db.assets[idx]; },
      async deleteAsset(id){ if(isServer) return await srvFetch('/assets/'+id,{method:'DELETE'}); const db = await localLoad(); db.assets = db.assets.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

      // INVENTORY
      async getInventory(){ if(isServer) return await srvFetch('/inventory'); const db = await localLoad()||{inventory:[]}; return db.inventory; },
      async addInv(it){ if(isServer) return await srvFetch('/inventory',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(it)}); const db = await localLoad()||{inventory:[]}; if(db.inventory.find(x=>x.id===it.id)) throw new Error('ID đã tồn tại'); db.inventory.push(it); await localSave(db); return it; },
      async updateInv(id,patch){ if(isServer) return await srvFetch('/inventory/'+id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db = await localLoad(); const idx=db.inventory.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.inventory[idx] = {...db.inventory[idx],...patch}; await localSave(db); return db.inventory[idx]; },
      async deleteInv(id){ if(isServer) return await srvFetch('/inventory/'+id,{method:'DELETE'}); const db = await localLoad(); db.inventory = db.inventory.filter(x=>x.id!==id); await localSave(db); return {ok:true}; },

      // REQUESTS
      async getRequests(){ if(isServer) return await srvFetch('/requests'); const db = await localLoad()||{requests:[]}; return db.requests; },
      async addRequest(r){ if(isServer) return await srvFetch('/requests',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(r)}); const db = await localLoad()||{requests:[]}; if(db.requests.find(x=>x.id===r.id)) throw new Error('ID đã tồn tại'); r.created = new Date().toISOString(); db.requests.push(r); await localSave(db); return r; },
      async updateRequest(id,patch){ if(isServer) return await srvFetch('/requests/'+id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(patch)}); const db = await localLoad(); const idx=db.requests.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.requests[idx] = {...db.requests[idx],...patch}; await localSave(db); return db.requests[idx]; },
      async deleteRequest(id){ if(isServer) return await srvFetch('/requests/'+id,{method:'DELETE'}); const db = await localLoad(); db.requests = db.requests.filter(x=>x.id!==id); await localSave(db); return {ok:true}; }
    };
  })();

  /***************** UI & Logic *****************/
  const modalRoom = new bootstrap.Modal(document.getElementById('modalRoom'));
  const modalAsset = new bootstrap.Modal(document.getElementById('modalAsset'));
  const modalInv = new bootstrap.Modal(document.getElementById('modalInv'));
  const modalReq = new bootstrap.Modal(document.getElementById('modalReq'));
  const modalLogin = new bootstrap.Modal(document.getElementById('modalLogin'));
  const modalViewReq = new bootstrap.Modal(document.getElementById('modalViewReq'));

  // Helpers
  function genId(prefix='X'){ return prefix + Date.now().toString().slice(-6) + Math.floor(Math.random()*90+10); }
  function showAlert(msg){ alert(msg); }
  function getInputFileDataUrl(input){ return new Promise((res,rej)=>{ const f = input.files && input.files[0]; if(!f) return res(''); const r=new FileReader(); r.onload=()=>res(r.result); r.onerror=()=>rej('Read error'); r.readAsDataURL(f); }); }

  // Render functions
  async function renderAll(){
    await renderRooms();
    await renderAssets();
    await renderInv();
    await renderReqs();
    await renderStats();
    renderCurrentUser();
    applyRoleUI();
    renderChart();
  }

  // ROOMS
  async function renderRooms(){
    try{
      const list = await dataLayer.getRooms() || [];
      const q = document.getElementById('search-rooms').value.trim().toLowerCase();
      const filtered = list.filter(r=> !q || (r.id+r.name).toLowerCase().includes(q) );
      document.getElementById('rooms-tbody').innerHTML = filtered.map(r=>`
        <tr>
          <td>${r.id}</td>
          <td>${r.name}</td>
          <td>${r.capacity||''}</td>
          <td>${r.notes||''}</td>
          <td>
            <button class="btn btn-sm btn-outline-primary me-1" data-id="${r.id}" data-act="edit-room"><i class="fa-solid fa-pen"></i></button>
            <button class="btn btn-sm btn-outline-danger" data-id="${r.id}" data-act="del-room"><i class="fa-solid fa-trash"></i></button>
          </td>
        </tr>
      `).join('');
      // attach events
      document.querySelectorAll('[data-act="edit-room"]').forEach(b=>b.addEventListener('click', e=> openRoomEdit(e.target.closest('button').dataset.id)));
      document.querySelectorAll('[data-act="del-room"]').forEach(b=>b.addEventListener('click', async e=>{ if(!confirm('Xóa phòng?')) return; try{ await dataLayer.deleteRoom(e.target.closest('button').dataset.id); await renderAll(); }catch(err){ alert(err.message); } }));
      document.getElementById('stat-rooms').textContent = list.length;
    }catch(err){ console.error(err); showAlert('Lỗi load phòng: '+err.message); }
  }

  // ASSETS
  async function renderAssets(){
    try{
      const list = await dataLayer.getAssets() || [];
      const q = document.getElementById('search-assets').value.trim().toLowerCase();
      const statusFilter = document.getElementById('filter-asset-status').value;
      const filtered = list.filter(a=> (!q || (a.id+a.name+(a.room||'')).toLowerCase().includes(q)) && (!statusFilter || a.status===statusFilter) );
      document.getElementById('assets-tbody').innerHTML = filtered.map(a=>`
        <tr>
          <td>${a.id}</td>
          <td>${a.name}</td>
          <td>${a.room||''}</td>
          <td>${a.status||''}</td>
          <td>${a.purchased||''}</td>
          <td>
            <button class="btn btn-sm btn-outline-primary me-1" data-id="${a.id}" data-act="edit-asset"><i class="fa-solid fa-pen"></i></button>
            <button class="btn btn-sm btn-outline-danger" data-id="${a.id}" data-act="del-asset"><i class="fa-solid fa-trash"></i></button>
          </td>
        </tr>
      `).join('');
      document.querySelectorAll('[data-act="edit-asset"]').forEach(b=>b.addEventListener('click', e=> openAssetEdit(e.target.closest('button').dataset.id)));
      document.querySelectorAll('[data-act="del-asset"]').forEach(b=>b.addEventListener('click', async e=>{ if(!confirm('Xóa tài sản?')) return; try{ await dataLayer.deleteAsset(e.target.closest('button').dataset.id); await renderAll(); }catch(err){ alert(err.message); } }));
      document.getElementById('stat-assets').textContent = list.length;
    }catch(err){ console.error(err); showAlert('Lỗi load assets: '+err.message); }
  }

  // INVENTORY
  async function renderInv(){
    try{
      const list = await dataLayer.getInventory() || [];
      const q = document.getElementById('search-inv').value.trim().toLowerCase();
      const filtered = list.filter(i=> !q || (i.id+i.name).toLowerCase().includes(q));
      document.getElementById('inv-tbody').innerHTML = filtered.map(i=>`
        <tr>
          <td>${i.id}</td>
          <td>${i.name}</td>
          <td>${i.qty}</td>
          <td>${i.unit||''}</td>
          <td>
            <button class="btn btn-sm btn-outline-primary me-1" data-id="${i.id}" data-act="edit-inv"><i class="fa-solid fa-pen"></i></button>
            <button class="btn btn-sm btn-outline-danger" data-id="${i.id}" data-act="del-inv"><i class="fa-solid fa-trash"></i></button>
          </td>
        </tr>
      `).join('');
      document.querySelectorAll('[data-act="edit-inv"]').forEach(b=>b.addEventListener('click', e=> openInvEdit(e.target.closest('button').dataset.id)));
      document.querySelectorAll('[data-act="del-inv"]').forEach(b=>b.addEventListener('click', async e=>{ if(!confirm('Xóa vật tư?')) return; try{ await dataLayer.deleteInv(e.target.closest('button').dataset.id); await renderAll(); }catch(err){ alert(err.message); } }));
    }catch(err){ console.error(err); showAlert('Lỗi load inventory: '+err.message); }
  }

  // REQUESTS
  async function renderReqs(){
    try{
      const list = await dataLayer.getRequests() || [];
      const q = document.getElementById('search-req').value.trim().toLowerCase();
      const statusFilter = document.getElementById('filter-req-status').value;
      const filtered = list.filter(r=> (!q || (r.id+r.title+r.reporter).toLowerCase().includes(q)) && (!statusFilter || r.status===statusFilter) );
      document.getElementById('reqs-tbody').innerHTML = filtered.map(r=>`
        <tr>
          <td>${r.id}</td>
          <td>${r.title}</td>
          <td>${r.room||''}</td>
          <td>${r.reporter||''}</td>
          <td>${r.status}</td>
          <td>
            <button class="btn btn-sm btn-outline-info me-1" data-id="${r.id}" data-act="view-req"><i class="fa-solid fa-eye"></i></button>
            <button class="btn btn-sm btn-outline-primary me-1" data-id="${r.id}" data-act="edit-req"><i class="fa-solid fa-pen"></i></button>
            <button class="btn btn-sm btn-outline-danger" data-id="${r.id}" data-act="del-req"><i class="fa-solid fa-trash"></i></button>
          </td>
        </tr>
      `).join('');
      document.querySelectorAll('[data-act="view-req"]').forEach(b=>b.addEventListener('click', e=> viewRequest(e.target.closest('button').dataset.id)));
      document.querySelectorAll('[data-act="edit-req"]').forEach(b=>b.addEventListener('click', e=> openReqEdit(e.target.closest('button').dataset.id)));
      document.querySelectorAll('[data-act="del-req"]').forEach(b=>b.addEventListener('click', async e=>{ if(!confirm('Xóa yêu cầu?')) return; try{ await dataLayer.deleteRequest(e.target.closest('button').dataset.id); await renderAll(); }catch(err){ alert(err.message); } }));
      document.getElementById('stat-requests').textContent = list.filter(r=>r.status==='open').length;
    }catch(err){ console.error(err); showAlert('Lỗi load requests: '+err.message); }
  }

  async function viewRequest(id){
    const list = await dataLayer.getRequests();
    const r = (list||[]).find(x=>x.id===id);
    if(!r) return alert('Không tìm thấy yêu cầu');
    document.getElementById('view-req-title').textContent = r.title;
    document.getElementById('view-req-body').innerHTML = `<p><strong>Mã:</strong> ${r.id}</p>
      <p><strong>Phòng:</strong> ${r.room||''}</p>
      <p><strong>Người báo:</strong> ${r.reporter||''}</p>
      <p><strong>Trạng thái:</strong> ${r.status}</p>
      <p><strong>Mô tả:</strong><br>${(r.desc||'')}</p>
      <p class="small-muted"><strong>Ngày tạo:</strong> ${r.created||''}</p>`;
    modalViewReq.show();
  }

  /***************** Open forms *****************/
  document.getElementById('open-add-room').addEventListener('click', ()=>{ document.getElementById('form-room').reset(); document.querySelector('#form-room [name=id]').removeAttribute('readonly'); modalRoom.show(); });
  document.getElementById('open-add-asset').addEventListener('click', ()=>{ document.getElementById('form-asset').reset(); document.querySelector('#form-asset [name=id]').removeAttribute('readonly'); modalAsset.show(); });
  document.getElementById('open-add-request').addEventListener('click', ()=>{ document.getElementById('form-req').reset(); document.querySelector('#form-req [name=id]').removeAttribute('readonly'); modalReq.show(); });
  document.getElementById('open-add-asset').style.display = 'inline-block';

  // FORM SUBMITS
  document.getElementById('form-room').addEventListener('submit', async e=>{
    e.preventDefault();
    const fd = new FormData(e.target);
    const obj = { id: fd.get('id').trim(), name: fd.get('name').trim(), capacity: parseInt(fd.get('capacity'))||null, notes: fd.get('notes').trim() };
    try{ if(document.querySelector('#form-room [name=id]').hasAttribute('readonly')) { await dataLayer.updateRoom(obj.id,obj); } else await dataLayer.addRoom(obj); modalRoom.hide(); await renderAll(); } catch(err){ alert(err.message); }
  });

  document.getElementById('form-asset').addEventListener('submit', async e=>{
    e.preventDefault();
    const fd = new FormData(e.target);
    const imageInput = e.target.querySelector('[name=image]');
    const imageData = await getInputFileDataUrl(imageInput);
    const obj = { id: fd.get('id').trim(), name: fd.get('name').trim(), room: fd.get('room').trim(), status: fd.get('status'), purchased: fd.get('purchased')||'', image: imageData };
    try{ if(document.querySelector('#form-asset [name=id]').hasAttribute('readonly')) { await dataLayer.updateAsset(obj.id, obj); } else await dataLayer.addAsset(obj); modalAsset.hide(); await renderAll(); } catch(err){ alert(err.message); }
  });

  document.getElementById('form-inv').addEventListener('submit', async e=>{
    e.preventDefault();
    const fd = new FormData(e.target);
    const obj = { id: fd.get('id').trim(), name: fd.get('name').trim(), qty: parseInt(fd.get('qty'))||0, unit: fd.get('unit').trim() };
    try{ if(document.querySelector('#form-inv [name=id]').hasAttribute('readonly')) { await dataLayer.updateInv(obj.id,obj); } else await dataLayer.addInv(obj); modalInv.hide(); await renderAll(); } catch(err){ alert(err.message); }
  });

  document.getElementById('form-req').addEventListener('submit', async e=>{
    e.preventDefault();
    const fd = new FormData(e.target);
    const obj = { id: fd.get('id').trim(), title: fd.get('title').trim(), room: fd.get('room').trim(), reporter: fd.get('reporter').trim(), desc: fd.get('desc').trim(), status: fd.get('status') };
    try{ if(document.querySelector('#form-req [name=id]').hasAttribute('readonly')) { await dataLayer.updateRequest(obj.id,obj); } else await dataLayer.addRequest(obj); modalReq.hide(); await renderAll(); } catch(err){ alert(err.message); }
  });

  // handlers for edit opens
  async function openRoomEdit(id){ const rooms = await dataLayer.getRooms(); const r = rooms.find(x=>x.id===id); if(!r) return alert('Không tìm thấy'); document.getElementById('form-room').reset(); document.querySelector('#form-room [name=id]').value = r.id; document.querySelector('#form-room [name=name]').value = r.name; document.querySelector('#form-room [name=capacity]').value = r.capacity||''; document.querySelector('#form-room [name=notes]').value = r.notes||''; document.querySelector('#form-room [name=id]').setAttribute('readonly','readonly'); modalRoom.show();}
  async function openAssetEdit(id){ const arr = await dataLayer.getAssets(); const a = arr.find(x=>x.id===id); if(!a) return alert('Không tìm thấy'); document.getElementById('form-asset').reset(); document.querySelector('#form-asset [name=id]').value = a.id; document.querySelector('#form-asset [name=name]').value = a.name; document.querySelector('#form-asset [name=room]').value = a.room||''; document.querySelector('#form-asset [name=status]').value = a.status||'normal'; document.querySelector('#form-asset [name=purchased]').value = a.purchased||''; document.querySelector('#form-asset [name=id]').setAttribute('readonly','readonly'); modalAsset.show(); }
  async function openInvEdit(id){ const arr = await dataLayer.getInventory(); const it = arr.find(x=>x.id===id); if(!it) return alert('Không tìm thấy'); document.getElementById('form-inv').reset(); document.querySelector('#form-inv [name=id]').value = it.id; document.querySelector('#form-inv [name=name]').value = it.name; document.querySelector('#form-inv [name=qty]').value = it.qty||0; document.querySelector('#form-inv [name=unit]').value = it.unit||''; document.querySelector('#form-inv [name=id]').setAttribute('readonly','readonly'); modalInv.show(); }
  async function openReqEdit(id){ const arr = await dataLayer.getRequests(); const r = arr.find(x=>x.id===id); if(!r) return alert('Không tìm thấy'); document.getElementById('form-req').reset(); document.querySelector('#form-req [name=id]').value = r.id; document.querySelector('#form-req [name=title]').value = r.title; document.querySelector('#form-req [name=room]').value = r.room||''; document.querySelector('#form-req [name=reporter]').value = r.reporter||''; document.querySelector('#form-req [name=desc]').value = r.desc||''; document.querySelector('#form-req [name=status]').value = r.status||'open'; document.querySelector('#form-req [name=id]').setAttribute('readonly','readonly'); modalReq.show(); }

  // search / filter listeners
  document.getElementById('search-rooms').addEventListener('input', debounce(()=>renderRooms(), 200));
  document.getElementById('search-assets').addEventListener('input', debounce(()=>renderAssets(), 200));
  document.getElementById('filter-asset-status').addEventListener('change', ()=>renderAssets());
  document.getElementById('search-inv').addEventListener('input', debounce(()=>renderInv(), 200));
  document.getElementById('search-req').addEventListener('input', debounce(()=>renderReqs(), 200));
  document.getElementById('filter-req-status').addEventListener('change', ()=>renderReqs());

  // import/export
  document.getElementById('btn-export').addEventListener('click', ()=> dataLayer.export());
  document.getElementById('btn-import').addEventListener('click', ()=> document.getElementById('file-input').click());
  document.getElementById('file-input').addEventListener('change', async e=>{ if(!e.target.files.length) return; try{ await dataLayer.import(e.target.files[0]); alert('Import thành công'); await renderAll(); } catch(err){ alert('Import lỗi: '+err.message);} e.target.value=''; });

  // seed if empty
  (async ()=>{ try{ const rooms = await dataLayer.getRooms(); if(!rooms || rooms.length===0) await dataLayer.seed(); await renderAll(); } catch(err){ console.error(err); alert('Lỗi khởi tạo: '+err.message);} })();

  // logout handler
  document.getElementById('btn-logout').addEventListener('click', ()=>{ clearCurrentUser(); });

  // debouncer
  function debounce(fn, ms=200){ let t; return (...args)=>{ clearTimeout(t); t=setTimeout(()=>fn(...args), ms); }; }

  // Chart
  let chartInstance = null;
  async function renderChart(){
    const ctx = document.getElementById('chartStatus').getContext('2d');
    const reqs = await dataLayer.getRequests() || [];
    const assets = await dataLayer.getAssets() || [];
    const statusCounts = { open:0, in_progress:0, done:0 };
    reqs.forEach(r=> statusCounts[r.status] = (statusCounts[r.status]||0)+1 );
    const assetByStatus = ['normal','broken','maintenance'].map(s=> assets.filter(a=>a.status===s).length );
    const labels = ['Yêu cầu Mới','Đang xử lý','Hoàn tất'];
    const data = [statusCounts.open, statusCounts.in_progress, statusCounts.done];
    if(chartInstance) chartInstance.destroy();
    chartInstance = new Chart(ctx, {
      type: 'doughnut',
      data: { labels, datasets:[{ label:'Yêu cầu', data, backgroundColor:['#ffc107','#0d6efd','#198754'] }]},
      options:{ responsive:true, plugins:{ legend:{position:'bottom'} } }
    });
  }

  // stats (update top cards)
  async function renderStats(){
    // handled in renderRooms/renderAssets/renderReqs
  }

  // initial role UI
  renderCurrentUser();

  </script>
</body>
</html>
