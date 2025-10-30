<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <title>Quản lý Cơ sở vật chất - THCS Hàn Thuyên (No PDF, No Seed)</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <!-- Bootstrap + FontAwesome -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"/>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" rel="stylesheet"/>
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <!-- Firebase (compat build) -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>
  <!-- Export libs: XLSX only -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

  <style>
    :root{ --brand:#0d6efd; --muted:#6c757d; --card-radius:12px; }
    body{ background:#f4f6fb; font-family:Inter,system-ui,Arial; }
    .app-container{ max-width:1200px; margin:20px auto; padding:12px; }
    .hero{ background:linear-gradient(90deg, rgba(13,110,253,0.06), rgba(13,110,253,0.02)); padding:18px; border-radius:12px; display:flex; gap:16px; align-items:center; }
    .hero img{ width:220px; border-radius:8px; object-fit:cover; }
    .small-muted{ color:var(--muted); }
    .pw-strength{ height:8px; border-radius:6px; background:#e9ecef; overflow:hidden; }
    .pw-strength > i{ display:block; height:100%; width:0%; background:linear-gradient(90deg,#ff6b6b,#ffd166,#06d6a0); transition:width .25s; }
    footer{ opacity:0.8; font-size:0.9rem; }
    .sync-dot{ width:10px; height:10px; border-radius:50%; display:inline-block; margin-right:6px; }
    .sync-online{ background:#28a745; box-shadow:0 0 6px rgba(40,167,69,0.4); }
    .sync-offline{ background:#dc3545; box-shadow:0 0 6px rgba(220,53,69,0.3); }
    .table-hover tbody tr:hover{ background:#f8f9fa; }
    .btn-export-group .btn{ min-width:120px; }
  </style>
</head>
<body>
  <div class="container-fluid bg-white shadow-sm py-2 mb-3">
    <div class="container d-flex justify-content-between align-items-center">
      <div class="d-flex gap-3 align-items-center">
        <i class="fa-solid fa-school fa-2x text-primary"></i>
        <div>
          <h4 class="mb-0">THCS Hàn Thuyên</h4>
          <div class="small-muted">Hệ thống Quản lý Cơ sở vật chất (Realtime + Export Excel)</div>
        </div>
      </div>
      <div class="d-flex gap-2 align-items-center">
        <div id="sync-indicator" class="me-2"><span class="sync-dot sync-offline"></span><small class="small-muted">Chưa kết nối</small></div>
        <div id="current-user"></div>
        <button id="btn-open-login" class="btn btn-outline-primary btn-sm">Đăng nhập</button>
        <button id="btn-open-signup" class="btn btn-outline-success btn-sm">Đăng ký</button>
        <button id="btn-logout" class="btn btn-outline-danger btn-sm" style="display:none">Đăng xuất</button>
      </div>
    </div>
  </div>

  <div class="app-container">
    <section class="hero mb-4">
      <div style="flex:1">
        <h2 class="mb-1">Hệ thống Quản lý Cơ sở vật chất — THCS Hàn Thuyên</h2>
        <p class="small-muted">Realtime sync bằng Firebase, upload ảnh, phân quyền, xuất Excel.</p>
        <div class="mt-2">
          <button id="btn-add-room" class="btn btn-primary me-2"><i class="fa-solid fa-door-open me-2"></i>Thêm phòng</button>
          <button id="btn-add-asset" class="btn btn-outline-primary me-2"><i class="fa-solid fa-box me-2"></i>Thêm thiết bị</button>
          <div class="btn-group btn-export-group me-2" role="group" aria-label="Export buttons">
            <button id="btn-export-excel" class="btn btn-outline-success"><i class="fa-solid fa-file-excel me-1"></i>Excel</button>
          </div>
        </div>
      </div>
      <img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" alt="Banner"/>
    </section>

    <!-- STATS -->
    <div class="row g-3 mb-4">
      <div class="col-md-4">
        <div class="p-3 bg-white rounded shadow-sm">
          <div class="d-flex justify-content-between">
            <div><div class="small-muted">Tổng phòng</div><h3 id="stat-rooms">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-door-closed fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white rounded shadow-sm">
          <div class="d-flex justify-content-between">
            <div><div class="small-muted">Tổng tài sản</div><h3 id="stat-assets">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-boxes-stacked fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white rounded shadow-sm">
          <div class="d-flex justify-content-between">
            <div><div class="small-muted">Yêu cầu mở</div><h3 id="stat-requests">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-wrench fa-2x"></i></div>
          </div>
        </div>
      </div>
    </div>

    <!-- MAIN (same structure as before) -->
    <div class="row g-3">
      <div class="col-lg-7">
        <div class="card mb-3">
          <div class="card-body">
            <ul class="nav nav-tabs" id="mainTabs">
              <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#tab-rooms">Phòng</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-assets">Tài sản</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-inv">Kho</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-reqs">Yêu cầu</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-logs">Lịch sử</a></li>
            </ul>

            <div class="tab-content mt-3">
              <!-- Rooms -->
              <div class="tab-pane fade show active" id="tab-rooms">
                <div class="d-flex mb-2">
                  <input id="search-rooms" class="form-control form-control-sm me-2" placeholder="Tìm...">
                  <button id="btn-refresh-rooms" class="btn btn-sm btn-outline-secondary">Làm mới</button>
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tên</th><th>Sức chứa</th><th>Ghi chú</th><th>Hành động</th></tr></thead>
                    <tbody id="rooms-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- Assets -->
              <div class="tab-pane fade" id="tab-assets">
                <div class="mb-2 d-flex gap-2">
                  <select id="filter-asset-status" class="form-select form-select-sm" style="max-width:160px"><option value="">Tất cả</option><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option></select>
                  <input id="search-assets" class="form-control form-control-sm" placeholder="Tìm...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>ID</th><th>Tên</th><th>Phòng</th><th>Trạng thái</th><th>Ảnh</th><th>Hành động</th></tr></thead>
                    <tbody id="assets-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- Inventory -->
              <div class="tab-pane fade" id="tab-inv">
                <div class="mb-2"><input id="search-inv" class="form-control form-control-sm" placeholder="Tìm vật tư..."></div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tên</th><th>Số lượng</th><th>Đơn vị</th><th>Hành động</th></tr></thead>
                    <tbody id="inv-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- Requests -->
              <div class="tab-pane fade" id="tab-reqs">
                <div class="mb-2 d-flex gap-2">
                  <select id="filter-req-status" class="form-select form-select-sm" style="max-width:160px"><option value="">Tất cả</option><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select>
                  <input id="search-req" class="form-control form-control-sm" placeholder="Tìm yêu cầu...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>Mã</th><th>Tiêu đề</th><th>Phòng</th><th>Người báo</th><th>Trạng thái</th><th>Hành động</th></tr></thead>
                    <tbody id="reqs-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- Logs -->
              <div class="tab-pane fade" id="tab-logs">
                <div class="table-responsive">
                  <table class="table table-sm">
                    <thead class="table-light"><tr><th>Thời gian</th><th>Người</th><th>Hành động</th><th>Mô tả</th></tr></thead>
                    <tbody id="logs-tbody"></tbody>
                  </table>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT -->
      <div class="col-lg-5">
        <div class="card mb-3 p-3">
          <div class="row g-2 mb-3">
           <div class="col-6"><img src="https://images.pexels.com/photos/301926/pexels-photo-301926.jpeg" class="img-fluid gallery" alt="Học sinh"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256455/pexels-photo-256455.jpeg" class="img-fluid gallery" alt="Sân trường"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" class="img-fluid gallery" alt="Lớp học"></div>
              <div class="col-6"><img src="https://images.pexels.com/photos/256401/pexels-photo-256401.jpeg" class="img-fluid gallery" alt="Thiết bị"></div>
          </div>
          <canvas id="chartStatus" height="220"></canvas>
        </div>
      </div>
    </div>

    <footer class="text-center small-muted mt-4 mb-4">© <span id="year"></span> THCS Hàn Thuyên — Prototype (Frontend + Firebase)</footer>
  </div>

  <!-- Modals (full) -->
  <!-- Room Modal -->
  <div class="modal fade" id="modalRoom" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-room" class="p-3">
    <h5 class="mb-3">Thêm / Sửa phòng</h5>
    <input name="id" class="form-control mb-2" placeholder="Mã phòng (vd: R101)" required>
    <input name="name" class="form-control mb-2" placeholder="Tên phòng" required>
    <input name="capacity" type="number" class="form-control mb-2" placeholder="Sức chứa">
    <textarea name="notes" class="form-control mb-2" placeholder="Ghi chú"></textarea>
    <div class="d-flex justify-content-end gap-2"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Asset Modal -->
  <div class="modal fade" id="modalAsset" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content"><form id="form-asset" class="p-3">
    <h5 class="mb-3">Thêm / Sửa tài sản</h5>
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
  </form></div></div></div>

  <!-- Request Modal -->
  <div class="modal fade" id="modalReq" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-req" class="p-3">
    <h5 class="mb-3">Tạo / Sửa yêu cầu bảo trì</h5>
    <input name="id" class="form-control mb-2" placeholder="Mã yêu cầu" required>
    <input name="title" class="form-control mb-2" placeholder="Tiêu đề" required>
    <input name="room" class="form-control mb-2" placeholder="Phòng">
    <input name="reporter" class="form-control mb-2" placeholder="Người báo">
    <textarea name="desc" class="form-control mb-2" placeholder="Mô tả"></textarea>
    <select name="status" class="form-select mb-2"><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select>
    <div class="d-flex justify-content-end gap-2"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Login / Signup modals -->
  <div class="modal fade" id="modalLogin" tabindex="-1"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content p-3">
    <h5>Đăng nhập</h5>
    <input id="login-email" class="form-control mb-2" placeholder="Email">
    <input id="login-password" type="password" class="form-control mb-2" placeholder="Mật khẩu">
    <div class="d-flex justify-content-end gap-2"><button id="login-btn" class="btn btn-primary">Đăng nhập</button><button class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button></div>
    <div class="mt-2 small-muted">Đăng ký tài khoản để dùng (admin/teacher/technician)</div>
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
   * CONFIG - REPLACE with your Firebase project config
   ******************************/
  const firebaseConfig = {  
    apiKey: "AIzaSyCV4yZpNC9dIchdN02SaP3LISpFzz2PUrc",
  authDomain: "nhom22-37f68.firebaseapp.com",
  projectId: "nhom22-37f68",
  storageBucket: "nhom22-37f68.firebasestorage.app",
  messagingSenderId: "843643018659",
  appId: "1:843643018659:web:33d6717d1c104f89f20e16",
  measurementId: "G-JX4P1WGVJP"
  };
  // Initialize Firebase
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.database();
  const storage = firebase.storage();

  // Helpers
  function nowIso(){ return new Date().toISOString(); }
  function uid(){ return auth.currentUser ? auth.currentUser.uid : 'anonymous'; }
  function notify(msg){ alert(msg); }
  function el(id){ return document.getElementById(id); }

  // Bootstrap modals
  const modalLogin = new bootstrap.Modal(el('modalLogin'));
  const modalSignup = new bootstrap.Modal(el('modalSignup'));
  const modalRoom = new bootstrap.Modal(el('modalRoom'));
  const modalAsset = new bootstrap.Modal(el('modalAsset'));
  const modalReq = new bootstrap.Modal(el('modalReq'));
  const modalView = new bootstrap.Modal(el('modalView'));

  // Render current user
  function renderCurrentUser(userMeta){
    const cu = el('current-user');
    const loginBtn = el('btn-open-login'), signupBtn = el('btn-open-signup'), logoutBtn = el('btn-logout');
    if(userMeta){
      cu.innerHTML = `<span class="badge bg-light text-dark">${userMeta.displayName || userMeta.username} · ${userMeta.role}</span>`;
      loginBtn.style.display='none'; signupBtn.style.display='none'; logoutBtn.style.display='inline-block';
    } else {
      cu.innerHTML = '';
      loginBtn.style.display='inline-block'; signupBtn.style.display='inline-block'; logoutBtn.style.display='none';
    }
  }

  // Apply role UI
  function applyRoleUI(role){
    el('btn-add-room').style.display = ['admin','teacher'].includes(role) ? 'inline-block' : 'none';
    el('btn-add-asset').style.display = ['admin','technician'].includes(role) ? 'inline-block' : 'none';
  }

  // Sync indicator
  function setSyncIndicator(online){
    const indicator = el('sync-indicator');
    indicator.innerHTML = `<span class="sync-dot ${online ? 'sync-online' : 'sync-offline'}"></span><small class="small-muted">${online ? 'Đã kết nối (Realtime)' : 'Chưa kết nối'}</small>`;
  }

  // AUTH handlers (same logic as before)
  el('btn-open-login').addEventListener('click', ()=> modalLogin.show());
  el('btn-open-signup').addEventListener('click', ()=> modalSignup.show());
  el('btn-logout').addEventListener('click', async ()=> { if(confirm('Đăng xuất?')) await auth.signOut(); });

  el('login-btn').addEventListener('click', async ()=> {
    try{
      const email = el('login-email').value.trim();
      const pass = el('login-password').value;
      await auth.signInWithEmailAndPassword(email, pass);
      modalLogin.hide();
      el('login-email').value=''; el('login-password').value='';
    }catch(e){ notify('Đăng nhập lỗi: '+e.message); }
  });

  el('su-btn').addEventListener('click', async ()=> {
    try{
      const email = el('su-email').value.trim();
      const pass = el('su-password').value;
      const name = el('su-name').value.trim() || email.split('@')[0];
      const role = el('su-role').value;
      const uc = await auth.createUserWithEmailAndPassword(email, pass);
      const uidv = uc.user.uid;
      // save metadata
      await db.ref('users/'+uidv).set({ username: email.split('@')[0], displayName: name, role, created: nowIso() });
      modalSignup.hide();
      el('su-email').value=''; el('su-password').value=''; el('su-name').value='';
      notify('Đăng ký thành công. Vui lòng đăng nhập.');
    }catch(e){ notify('Đăng ký lỗi: '+e.message); }
  });

  // Password strength UI
  el('su-password').addEventListener('input', e=> {
    const val = e.target.value; let score=0;
    if(val.length>=8) score+=40; if(/[A-Z]/.test(val)&&/[0-9]/.test(val)) score+=30; if(/[^A-Za-z0-9]/.test(val)) score+=30;
    el('pw-strength-bar').style.width = Math.min(100,score)+'%';
  });

  // Auth state sync -> load user meta & UI update
  let currentUserMeta = null;
  auth.onAuthStateChanged(async (u)=> {
    if(u){
      // read user meta
      const snap = await db.ref('users/'+u.uid).get();
      const meta = snap.exists() ? snap.val() : { username: u.email.split('@')[0], displayName:u.email.split('@')[0], role:'teacher' };
      currentUserMeta = { uid: u.uid, ...meta };
      renderCurrentUser(currentUserMeta);
      applyRoleUI(currentUserMeta.role);
      await renderAll(); // initial render
    } else {
      currentUserMeta = null;
      renderCurrentUser(null);
      applyRoleUI(null);
      await renderAll();
    }
  });

  /**************** DATA LAYER (Realtime DB wrappers + logs) ****************/
  async function logAction(action, desc){
    const entry = { ts: nowIso(), user: (currentUserMeta? currentUserMeta.username: 'anonymous'), action, desc };
    await db.ref('logs').push(entry);
  }

  // CRUD wrappers (same as before)
  async function addRoom(room){ await db.ref('rooms/'+room.id).set(room); await logAction('add_room', `Thêm phòng ${room.id}`); }
  async function updateRoom(id, patch){ await db.ref('rooms/'+id).update(patch); await logAction('update_room', `Sửa phòng ${id}`); }
  async function deleteRoom(id){ await db.ref('rooms/'+id).remove(); await logAction('delete_room', `Xóa phòng ${id}`); }

  async function addAsset(asset){ await db.ref('assets/'+asset.id).set(asset); await logAction('add_asset', `Thêm tài sản ${asset.id}`); }
  async function updateAsset(id, patch){ await db.ref('assets/'+id).update(patch); await logAction('update_asset', `Sửa tài sản ${id}`); }
  async function deleteAsset(id){ await db.ref('assets/'+id).remove(); await logAction('delete_asset', `Xóa tài sản ${id}`); }

  async function addRequest(r){ await db.ref('requests/'+r.id).set({...r,created:nowIso()}); await logAction('add_request', `Tạo yêu cầu ${r.id}`); }
  async function updateRequest(id,patch){ await db.ref('requests/'+id).update(patch); await logAction('update_request', `Sửa yêu cầu ${id}`); }

  /***************** RENDER & UI HANDLERS (same as before) *****************/
  async function renderAll(){
    await Promise.all([renderRooms(), renderAssets(), renderInv(), renderReqs(), renderLogs(), renderChart()]);
    el('year').textContent = new Date().getFullYear();
  }

  async function renderRooms(){
    const snap = await db.ref('rooms').get();
    const rows = snap.exists() ? Object.values(snap.val()) : [];
    const q = el('search-rooms').value.trim().toLowerCase();
    const filtered = rows.filter(r=> !q || (r.id + r.name).toLowerCase().includes(q));
    el('rooms-tbody').innerHTML = filtered.map(r=>`
      <tr>
        <td>${r.id}</td><td>${r.name}</td><td>${r.capacity||''}</td><td>${r.notes||''}</td>
        <td>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-room" data-id="${r.id}">Sửa</button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-room" data-id="${r.id}">Xóa</button>
        </td>
      </tr>`).join('');
    el('stat-rooms').textContent = rows.length;
  }

  async function renderAssets(){
    const snap = await db.ref('assets').get();
    const rows = snap.exists() ? Object.values(snap.val()) : [];
    const q = el('search-assets').value.trim().toLowerCase();
    const statusFilter = el('filter-asset-status').value;
    const filtered = rows.filter(a=> (!q || (a.id+a.name+(a.room||'')).toLowerCase().includes(q)) && (!statusFilter || a.status===statusFilter));
    el('assets-tbody').innerHTML = filtered.map(a=>`
      <tr>
        <td>${a.id}</td><td>${a.name}</td><td>${a.room||''}</td><td>${a.status||''}</td>
        <td>${a.image? '<img src="'+a.image+'" style="height:50px;border-radius:6px">' : ''}</td>
        <td>
          <button class="btn btn-sm btn-outline-info me-1" data-act="view-asset" data-id="${a.id}">Xem</button>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-asset" data-id="${a.id}">Sửa</button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-asset" data-id="${a.id}">Xóa</button>
        </td>
      </tr>`).join('');
    el('stat-assets').textContent = rows.length;
  }

  async function renderInv(){
    const snap = await db.ref('inventory').get();
    const rows = snap.exists() ? Object.values(snap.val()) : [];
    el('inv-tbody').innerHTML = rows.map(i=>`<tr><td>${i.id}</td><td>${i.name}</td><td>${i.qty}</td><td>${i.unit||''}</td><td><button class="btn btn-sm btn-outline-primary" data-id="${i.id}" data-act="edit-inv">Sửa</button></td></tr>`).join('');
  }

  async function renderReqs(){
    const snap = await db.ref('requests').get();
    const rows = snap.exists() ? Object.values(snap.val()) : [];
    const q = el('search-req').value.trim().toLowerCase(); const statusFilter = el('filter-req-status').value;
    const filtered = rows.filter(r=> (!q || (r.id+r.title+r.reporter).toLowerCase().includes(q)) && (!statusFilter || r.status===statusFilter));
    el('reqs-tbody').innerHTML = filtered.map(r=>`<tr><td>${r.id}</td><td>${r.title}</td><td>${r.room||''}</td><td>${r.reporter||''}</td><td>${r.status}</td><td><button class="btn btn-sm btn-outline-info me-1" data-act="view-req" data-id="${r.id}">Xem</button><button class="btn btn-sm btn-outline-primary" data-act="edit-req" data-id="${r.id}">Sửa</button></td></tr>`).join('');
  }

  async function viewRequest(id){
    const snap = await db.ref('requests/'+id).get(); if(!snap.exists()) return notify('Không tìm thấy');
    const r = snap.val();
    const html = `<p><strong>Mã:</strong> ${r.id}</p><p><strong>Tiêu đề:</strong> ${r.title}</p><p><strong>Phòng:</strong> ${r.room||''}</p><p><strong>Người báo:</strong> ${r.reporter||''}</p><p><strong>Trạng thái:</strong> ${r.status}</p><p><strong>Mô tả:</strong><br>${r.desc||''}</p><p class="small-muted"><strong>Ngày tạo:</strong> ${r.created||''}</p>`;
    el('modalViewBody').innerHTML = html; modalView.show();
  }

  async function renderLogs(){
    const snap = await db.ref('logs').orderByChild('ts').limitToLast(200).get();
    const rows = snap.exists() ? Object.values(snap.val()).reverse() : [];
    el('logs-tbody').innerHTML = rows.map(l=>`<tr><td>${l.ts}</td><td>${l.user}</td><td>${l.action}</td><td>${l.desc}</td></tr>`).join('');
  }

  let chart = null;
  async function renderChart(){
    const snap = await db.ref('requests').get();
    const rows = snap.exists() ? Object.values(snap.val()) : [];
    const counts = { open:0, in_progress:0, done:0 }; rows.forEach(r=> counts[r.status] = (counts[r.status]||0)+1);
    const data = [counts.open, counts.in_progress, counts.done];
    const ctx = el('chartStatus').getContext('2d');
    if(chart) chart.destroy();
    chart = new Chart(ctx, { type:'doughnut', data:{ labels:['Mới','Đang xử lý','Hoàn tất'], datasets:[{ data, backgroundColor:['#ffc107','#0d6efd','#198754'] }] }, options:{responsive:true, plugins:{legend:{position:'bottom'}}}});
    el('stat-requests').textContent = counts.open;
  }

  /**************** Forms handling ****************/
  el('form-room').addEventListener('submit', async (e)=> {
    e.preventDefault();
    const f = e.target;
    const room = { id: f['id'].value.trim(), name: f['name'].value.trim(), capacity: Number(f['capacity'].value)||0, notes: f['notes'].value.trim() };
    try{
      const exists = (await db.ref('rooms/'+room.id).get()).exists();
      if(exists) await updateRoom(room.id, room); else await addRoom(room);
      modalRoom.hide();
    }catch(err){ notify(err.message); }
  });

  el('form-asset').querySelector('input[name=image]').addEventListener('change', function(){ /* preview optional */ });

  el('form-asset').addEventListener('submit', async (e)=> {
    e.preventDefault();
    if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Bạn không có quyền.'); return; }
    const f = e.target;
    const id = (f['id'].value.trim()) || ('AS-'+Date.now().toString().slice(-6));
    const asset = { id, name: f['name'].value.trim(), serial: f['serial'].value.trim(), category: f['category'].value.trim(), room: f['room'].value.trim(), status: f['status'].value, purchased: f['purchased'].value||'', price: Number(f['price'].value)||0, vendor: f['vendor'].value||'', warranty: Number(f['warranty'].value)||0, desc: f['desc'].value||'', updatedAt: nowIso() };
    const file = f['image'].files[0];
    if(file){
      const ref = storage.ref('asset_images/'+id+'_'+file.name);
      const snap = await ref.put(file);
      const url = await snap.ref.getDownloadURL();
      asset.image = url;
    } else {
      const existingSnap = await db.ref('assets/'+id).get();
      if(existingSnap.exists()) asset.image = existingSnap.val().image || '';
    }
    const exists = (await db.ref('assets/'+id).get()).exists();
    try{ if(exists) await updateAsset(id, asset); else await addAsset(asset); modalAsset.hide(); } catch(err){ notify(err.message); }
  });

  el('form-req').addEventListener('submit', async e=> {
    e.preventDefault();
    if(!currentUserMeta){ notify('Cần đăng nhập để tạo yêu cầu'); return; }
    const f = e.target; const obj = { id: f['id'].value.trim(), title: f['title'].value.trim(), room: f['room'].value.trim(), reporter: f['reporter'].value || currentUserMeta.username, desc: f['desc'].value.trim(), status: f['status'].value };
    const exists = (await db.ref('requests/'+obj.id).get()).exists();
    try{ if(exists) await updateRequest(obj.id, obj); else await addRequest(obj); modalReq.hide(); }catch(err){ notify(err.message); }
  });

  el('btn-add-room').addEventListener('click', ()=> { el('form-room').reset(); el('form-room')['id'].removeAttribute('readonly'); modalRoom.show(); });
  el('btn-add-asset').addEventListener('click', ()=> { if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Cần đăng nhập với quyền admin hoặc technician'); return; } el('form-asset').reset(); el('form-asset')['id'].removeAttribute('readonly'); modalAsset.show(); });

  el('search-rooms').addEventListener('input', debounce(()=>renderRooms(),200));
  el('search-assets').addEventListener('input', debounce(()=>renderAssets(),200));
  el('filter-asset-status').addEventListener('change', ()=>renderAssets());
  el('search-inv').addEventListener('input', debounce(()=>renderInv(),200));
  el('search-req').addEventListener('input', debounce(()=>renderReqs(),200));
  el('filter-req-status').addEventListener('change', ()=>renderReqs());

  /**************** EXPORT FUNCTIONS (Excel only) ****************/
  async function exportExcel(){
    const [roomsSnap, assetsSnap, invSnap, reqSnap] = await Promise.all([db.ref('rooms').get(), db.ref('assets').get(), db.ref('inventory').get(), db.ref('requests').get()]);
    const rooms = roomsSnap.exists()? Object.values(roomsSnap.val()) : [];
    const assets = assetsSnap.exists()? Object.values(assetsSnap.val()) : [];
    const invs = invSnap.exists()? Object.values(invSnap.val()) : [];
    const reqs = reqSnap.exists()? Object.values(reqSnap.val()) : [];

    const wb = XLSX.utils.book_new();

    if(rooms.length){
      const roomsOrdered = rooms.map(r=>({Mã:r.id, Tên:r.name, Sức_chứa:r.capacity, Ghi_chú:r.notes||''}));
      const ws = XLSX.utils.json_to_sheet(roomsOrdered, {header:["Mã","Tên","Sức_chứa","Ghi_chú"]});
      ws['!cols'] = [{wpx:80},{wpx:180},{wpx:80},{wpx:220}];
      XLSX.utils.book_append_sheet(wb, ws, 'Rooms');
    }
    if(assets.length){
      const assetsOrdered = assets.map(a=>({ID:a.id, Tên:a.name, Phòng:a.room||'', Trạng_thái:a.status||'', Ngày_mua:a.purchased||'', Giá:a.price||0}));
      const ws = XLSX.utils.json_to_sheet(assetsOrdered, {header:["ID","Tên","Phòng","Trạng_thái","Ngày_mua","Giá"]});
      ws['!cols'] = [{wpx:80},{wpx:180},{wpx:80},{wpx:100},{wpx:110},{wpx:120}];
      XLSX.utils.book_append_sheet(wb, ws, 'Assets');
    }
    if(invs.length){
      const invOrdered = invs.map(i=>({Mã:i.id, Tên:i.name, Số_lượng:i.qty, Đơn_vị:i.unit||''}));
      const ws = XLSX.utils.json_to_sheet(invOrdered, {header:["Mã","Tên","Số_lượng","Đơn_vị"]});
      ws['!cols'] = [{wpx:80},{wpx:180},{wpx:80},{wpx:100}];
      XLSX.utils.book_append_sheet(wb, ws, 'Inventory');
    }
    if(reqs.length){
      const reqOrdered = reqs.map(r=>({Mã:r.id, Tiêu_đề:r.title, Phòng:r.room||'', Người_báo:r.reporter||'', Trạng_thái:r.status||'', Ngày_tạo:r.created||''}));
      const ws = XLSX.utils.json_to_sheet(reqOrdered, {header:["Mã","Tiêu_đề","Phòng","Người_báo","Trạng_thái","Ngày_tạo"]});
      ws['!cols'] = [{wpx:100},{wpx:240},{wpx:80},{wpx:120},{wpx:100},{wpx:120}];
      XLSX.utils.book_append_sheet(wb, ws, 'Requests');
    }

    const meta = [{Key:"ExportedAt", Value: new Date().toLocaleString()}, {Key:"Rooms", Value: rooms.length}, {Key:"Assets", Value: assets.length}, {Key:"Inventory", Value: invs.length}, {Key:"Requests", Value: reqs.length}];
    const wsMeta = XLSX.utils.json_to_sheet(meta);
    wsMeta['!cols'] = [{wpx:180},{wpx:120}];
    XLSX.utils.book_append_sheet(wb, wsMeta, 'Metadata');

    const wbout = XLSX.write(wb, {bookType:'xlsx', type:'array'});
    saveAs(new Blob([wbout],{type:'application/octet-stream'}), `BaoCao_CoSoVatChat_${(new Date()).toISOString().slice(0,10)}.xlsx`);
    await logAction('export_excel','Xuất báo cáo Excel');
    notify('Đã xuất Excel (có sheet Metadata)');
  }

  // Wire export button
  el('btn-export-excel').addEventListener('click', async ()=> {
    try{ await exportExcel(); } catch(e){ notify('Export Excel lỗi: '+e.message); }
  });

  // Utility functions
  function debounce(fn, ms=200){ let t; return (...args)=>{ clearTimeout(t); t=setTimeout(()=>fn(...args), ms); }; }
  function genId(prefix='X'){ return prefix + Date.now().toString().slice(-6) + Math.floor(Math.random()*900+100); }

  /**************** Realtime listeners (Realtime sync) ****************/
  setSyncIndicator(false);

  function attachRealtimeListeners(){
    try{
      db.ref('rooms').on('value', snapshot => { renderRooms(); setSyncIndicator(true); });
      db.ref('assets').on('value', snapshot => { renderAssets(); setSyncIndicator(true); });
      db.ref('inventory').on('value', snapshot => { renderInv(); setSyncIndicator(true); });
      db.ref('requests').on('value', snapshot => { renderReqs(); renderChart(); setSyncIndicator(true); });
      db.ref('logs').on('value', snapshot => { renderLogs(); setSyncIndicator(true); });
      db.ref('.info/connected').on('value', snap => { const connected = snap.val() === true; setSyncIndicator(connected); });
    }catch(e){
      console.warn('Realtime listener attach error', e);
      setSyncIndicator(false);
    }
  }
  attachRealtimeListeners();

  /**************** Event delegation for table buttons ****************/
  document.body.addEventListener('click', async (e)=>{
    const btn = e.target.closest('button'); if(!btn) return;
    const act = btn.dataset.act; const id = btn.dataset.id;
    if(!act) return;
    try{
      if(act==='edit-room'){ const snap = await db.ref('rooms/'+id).get(); if(!snap.exists()) return notify('Không tìm thấy'); const r = snap.val(); const f = el('form-room'); f['id'].value = r.id; f['id'].setAttribute('readonly','readonly'); f['name'].value=r.name; f['capacity'].value=r.capacity||''; f['notes'].value=r.notes||''; modalRoom.show(); }
      if(act==='del-room'){ if(!currentUserMeta || currentUserMeta.role!=='admin'){ notify('Chỉ Admin được xóa'); return; } if(!confirm('Xóa phòng?')) return; await deleteRoom(id); }
      if(act==='view-asset'){ const snap = await db.ref('assets/'+id).get(); const a = snap.val(); const html = `<h5>${a.name}</h5><p>ID: ${a.id}<br>Phòng: ${a.room||''}<br>Trạng thái: ${a.status||''}</p>${a.image? '<img src="'+a.image+'" style="width:100%;border-radius:8px">':''}<p class="small-muted">Ngày mua: ${a.purchased||''} · Giá: ${a.price? a.price.toLocaleString() : ''}</p>`; el('modalViewBody').innerHTML = html; modalView.show(); }
      if(act==='edit-asset'){ const snap = await db.ref('assets/'+id).get(); if(!snap.exists()) return notify('Không tìm thấy'); const a = snap.val(); if(!currentUserMeta || !['admin','technician'].includes(currentUserMeta.role)){ notify('Bạn không có quyền chỉnh sửa'); return; } const f = el('form-asset'); f['id'].value=a.id; f['id'].setAttribute('readonly','readonly'); f['name'].value=a.name; f['serial'].value=a.serial||''; f['category'].value=a.category||''; f['room'].value=a.room||''; f['status'].value=a.status||'normal'; f['purchased'].value=a.purchased||''; f['price'].value=a.price||''; f['vendor'].value=a.vendor||''; f['warranty'].value=a.warranty||''; f['desc'].value=a.desc||''; modalAsset.show(); }
      if(act==='del-asset'){ if(!currentUserMeta || currentUserMeta.role!=='admin'){ notify('Chỉ Admin được xóa'); return; } if(!confirm('Xóa tài sản?')) return; await deleteAsset(id); }
      if(act==='view-req'){ viewRequest(id); }
      if(act==='edit-req'){ const snap = await db.ref('requests/'+id).get(); if(!snap.exists()) return notify('Không tìm thấy'); const r = snap.val(); const f = el('form-req'); f['id'].value=r.id; f['id'].setAttribute('readonly','readonly'); f['title'].value=r.title; f['room'].value=r.room||''; f['reporter'].value=r.reporter||''; f['desc'].value=r.desc||''; f['status'].value=r.status||'open'; modalReq.show(); }
    }catch(err){ notify(err.message); }
  });

  /**************** Init ****************/
  (async ()=>{ try{ await renderAll(); }catch(e){ console.error(e); } })();
  el('year').textContent = new Date().getFullYear();

  </script>
</body>
</html>
