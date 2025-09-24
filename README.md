<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Quản lý Cơ sở vật chất - TH Minh Khai (Demo: Auth + Asset)</title>

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
      --shadow: 0 8px 30px rgba(13,110,253,0.06);
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
    .small-muted{ color:var(--muted); font-size:0.9rem; }
    .role-badge { text-transform:uppercase; font-weight:600; font-size:0.75rem; padding:4px 8px; border-radius:999px; }
    .asset-preview { max-height:160px; object-fit:cover; border-radius:8px; }
    @media (max-width:900px){ .hero{flex-direction:column;text-align:center} .hero img{max-width:100%} }
    .pw-strength { height:8px; border-radius:6px; background:#e9ecef; overflow:hidden; }
    .pw-strength > i { display:block; height:100%; width:0%; background:linear-gradient(90deg,#ff6b6b,#ffd166,#06d6a0); transition:width .25s; }
  </style>
</head>
<body>
  <!-- TOP BAR -->
  <div class="container-fluid bg-white shadow-sm py-2">
    <div class="container d-flex justify-content-between align-items-center">
      <div class="d-flex align-items-center gap-3">
        <i class="fa-solid fa-school fa-2x text-primary"></i>
        <div>
          <h4 class="mb-0">Trường Tiểu học Minh Khai</h4>
          <div class="small-muted">Hệ thống quản lý cơ sở vật chất</div>
        </div>
      </div>
      <div class="d-flex align-items-center gap-2">
        <div id="current-user"></div>
        <button id="btn-open-login" class="btn btn-outline-primary btn-sm">Đăng nhập</button>
        <button id="btn-open-signup" class="btn btn-outline-success btn-sm">Đăng ký</button>
        <button id="btn-logout" class="btn btn-outline-danger btn-sm" style="display:none">Đăng xuất</button>
      </div>
    </div>
  </div>

  <div class="app-container">
    <!-- HERO -->
    <section class="hero my-4">
      <div style="flex:1">
        <h2 class="mb-1">Hệ thống Quản lý Cơ sở vật chất — TH Minh Khai</h2>
        <p class="small-muted mb-2">Dashboard, quản lý phòng, tài sản, kho vật tư và yêu cầu bảo trì. (Demo: auth client-side & lưu localStorage)</p>
        <div class="mt-2">
          <button id="btn-add-room" class="btn btn-primary me-2"><i class="fa-solid fa-door-open me-2"></i>Thêm phòng</button>
          <button id="btn-add-asset" class="btn btn-outline-primary me-2"><i class="fa-solid fa-box me-2"></i>Thêm thiết bị</button>
          <button id="btn-add-request" class="btn btn-outline-warning"><i class="fa-solid fa-wrench me-2"></i>Tạo yêu cầu</button>
          <button id="btn-export" class="btn btn-outline-secondary"><i class="fa-solid fa-file-export me-2"></i>Xuất dữ liệu</button>
          <button id="btn-import" class="btn btn-outline-secondary"><i class="fa-solid fa-file-import me-2"></i>Nhập dữ liệu</button>
          <input id="file-input" type="file" accept="application/json" style="display:none" />
        </div>
      </div>
      <img src="https://images.pexels.com/photos/256395/pexels-photo-256395.jpeg" alt="Banner phòng học" />
    </section>

    <!-- STATS -->
    <div class="row g-3 mb-4">
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div><div class="small-muted">Tổng phòng</div><h3 id="stat-rooms">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-door-closed fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div><div class="small-muted">Tổng tài sản</div><h3 id="stat-assets">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-boxes-stacked fa-2x"></i></div>
          </div>
        </div>
      </div>
      <div class="col-md-4">
        <div class="p-3 bg-white card-stat">
          <div class="d-flex justify-content-between align-items-center">
            <div><div class="small-muted">Yêu cầu mở</div><h3 id="stat-requests">0</h3></div>
            <div class="text-primary"><i class="fa-solid fa-wrench fa-2x"></i></div>
          </div>
        </div>
      </div>
    </div>

    <!-- MAIN -->
    <div class="row g-3 mb-4">
      <div class="col-lg-7">
        <div class="card">
          <div class="card-body">
            <ul class="nav nav-tabs" id="mainTabs">
              <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#tab-rooms">Phòng học</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-assets">Tài sản</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-inv">Kho vật tư</a></li>
              <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#tab-reqs">Yêu cầu</a></li>
            </ul>

            <div class="tab-content mt-3">
              <!-- Rooms -->
              <div class="tab-pane fade show active" id="tab-rooms">
                <div class="mb-2 d-flex justify-content-between">
                  <input id="search-rooms" class="form-control form-control-sm" placeholder="Tìm theo mã hoặc tên...">
                  <div>
                    <button id="seed-data" class="btn btn-sm btn-outline-secondary">Tạo dữ liệu mẫu</button>
                  </div>
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
                  <select id="filter-asset-status" class="form-select form-select-sm" style="max-width:160px"><option value="">Tất cả trạng thái</option><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option></select>
                  <input id="search-assets" class="form-control form-control-sm" placeholder="Tìm tài sản...">
                </div>
                <div class="table-responsive">
                  <table class="table table-hover">
                    <thead class="table-light"><tr><th>ID</th><th>Tên</th><th>Phòng</th><th>Trạng thái</th><th>Ngày mua</th><th>Ảnh</th><th>Hành động</th></tr></thead>
                    <tbody id="assets-tbody"></tbody>
                  </table>
                </div>
              </div>

              <!-- Inventory -->
              <div class="tab-pane fade" id="tab-inv">
                <div class="d-flex gap-2 mb-2"><input id="search-inv" class="form-control form-control-sm" placeholder="Tìm vật tư..."></div>
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

            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT -->
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
            <h6 class="mb-3">Thống kê trạng thái</h6>
            <canvas id="chartStatus" height="220"></canvas>
          </div>
        </div>
      </div>
    </div>

    <footer class="text-center small-muted mb-5">© 2025 Trường Tiểu học Minh Khai — Prototype (Frontend)</footer>
  </div>

  <!-- ===== Modals: Room / Asset / Inventory / Request / Login / Signup / View ===== -->
  <!-- Modal: Room -->
  <div class="modal fade" id="modalRoom" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-room" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa phòng</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã phòng</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tên phòng</label><input name="name" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Sức chứa</label><input name="capacity" type="number" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Ghi chú</label><textarea name="notes" class="form-control"></textarea></div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Asset (detailed) -->
  <div class="modal fade" id="modalAsset" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content"><form id="form-asset" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa tài sản</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="row g-2">
        <div class="col-md-4"><label class="form-label">ID</label><div class="input-group"><input name="id" class="form-control"><button id="gen-asset-id" type="button" class="btn btn-outline-secondary">Sinh</button></div></div>
        <div class="col-md-8"><label class="form-label">Tên</label><input name="name" class="form-control" required></div>
        <div class="col-md-4"><label class="form-label">Mã serial</label><input name="serial" class="form-control"></div>
        <div class="col-md-4"><label class="form-label">Loại</label><input name="category" class="form-control"></div>
        <div class="col-md-4"><label class="form-label">Phòng (mã)</label><input name="room" class="form-control"></div>
        <div class="col-md-4"><label class="form-label">Trạng thái</label><select name="status" class="form-select"><option value="normal">Bình thường</option><option value="broken">Hỏng</option><option value="maintenance">Bảo trì</option></select></div>
        <div class="col-md-4"><label class="form-label">Ngày mua</label><input name="purchased" type="date" class="form-control"></div>
        <div class="col-md-4"><label class="form-label">Giá (VND)</label><input name="price" type="number" class="form-control"></div>
        <div class="col-md-6"><label class="form-label">Nhà cung cấp</label><input name="vendor" class="form-control"></div>
        <div class="col-md-6"><label class="form-label">Bảo hành (tháng)</label><input name="warranty" type="number" class="form-control"></div>
        <div class="col-md-6"><label class="form-label">Ảnh (tùy chọn)</label><input name="image" type="file" accept="image/*" class="form-control form-control-sm"><div class="mt-2"><img id="asset-preview" class="asset-preview" src="" alt="" style="display:none"></div></div>
        <div class="col-12"><label class="form-label">Mô tả</label><textarea name="desc" class="form-control" rows="3"></textarea></div>
      </div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu tài sản</button></div>
  </form></div></div></div>

  <!-- Modal: Inventory -->
  <div class="modal fade" id="modalInv" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-inv" class="needs-validation" novalidate>
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
  <div class="modal fade" id="modalReq" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-req" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Tạo / Sửa yêu cầu bảo trì</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tiêu đề</label><input name="title" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Phòng (mã)</label><input name="room" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Người báo</label><input name="reporter" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Mô tả</label><textarea name="desc" class="form-control"></textarea></div>
      <div class="mb-2"><label class="form-label">Trạng thái</label><select name="status" class="form-select"><option value="open">Mới</option><option value="in_progress">Đang xử lý</option><option value="done">Hoàn tất</option></select></div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Login -->
  <div class="modal fade" id="modalLogin" tabindex="-1"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content"><form id="form-login" class="p-3">
    <h5>Đăng nhập</h5>
    <div class="mb-2"><label class="form-label">Tên đăng nhập</label><input id="login-username" class="form-control" required></div>
    <div class="mb-2"><label class="form-label">Mật khẩu</label><input id="login-password" type="password" class="form-control" required></div>
    <div class="d-flex justify-content-end"><button type="submit" class="btn btn-primary btn-sm">Đăng nhập</button></div>
    <div class="mt-2 small-muted">Demo accounts: admin/admin, teacher/teacher, tech/tech</div>
  </form></div></div></div>

  <!-- Modal: Signup -->
  <div class="modal fade" id="modalSignup" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-signup" class="p-3">
    <h5>Đăng ký</h5>
    <div class="row g-2">
      <div class="col-md-6"><label class="form-label">Tên đăng nhập *</label><input id="su-username" class="form-control" required></div>
      <div class="col-md-6"><label class="form-label">Tên hiển thị</label><input id="su-name" class="form-control"></div>
      <div class="col-md-6"><label class="form-label">Mật khẩu *</label><input id="su-password" type="password" class="form-control" required></div>
      <div class="col-md-6"><label class="form-label">Xác nhận mật khẩu *</label><input id="su-password2" type="password" class="form-control" required></div>
      <div class="col-md-6"><label class="form-label">Quyền</label><select id="su-role" class="form-select"><option value="teacher">Giáo viên</option><option value="technician">Kỹ thuật</option><option value="admin">Admin</option></select></div>
      <div class="col-12">
        <div class="pw-strength mt-2"><i id="pw-strength-bar"></i></div>
        <div class="form-text small-muted">Mật khẩu nên >= 8 ký tự, có chữ và số.</div>
      </div>
    </div>
    <div class="mt-3 d-flex justify-content-end"><button type="button" class="btn btn-outline-secondary btn-sm me-2" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-success btn-sm">Đăng ký</button></div>
  </form></div></div></div>

  <!-- Modal: View Request (detail) -->
  <div class="modal fade" id="modalViewReq" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content">
    <div class="modal-header"><h5 class="modal-title" id="view-req-title">Yêu cầu</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body" id="view-req-body"></div>
    <div class="modal-footer"><button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Đóng</button></div>
  </div></div></div>

  <!-- Bootstrap JS -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

  <!-- ================= App JS: dataLayer, auth (PBKDF2 demo), UI ================= -->
  <script>
  /*****************************************************************
   * CONFIG
   * API_BASE = '' => dùng localStorage demo.
   * IMPORTANT: Client-side auth & PBKDF2 here chỉ để demo.
   * For production: implement server-side auth (bcrypt + JWT) & DB.
   *****************************************************************/
  const API_BASE = '';
  const DB_KEY = 'minhkai_full_v1';

  /***************** WebCrypto helpers for PBKDF2 (demo hashing) *****************/
  async function genSalt(len=16){
    const arr = crypto.getRandomValues(new Uint8Array(len));
    return btoa(String.fromCharCode(...arr));
  }
  function b64ToArrayBuffer(b64){ const bin=atob(b64); const len=bin.length; const arr=new Uint8Array(len); for(let i=0;i<len;i++) arr[i]=bin.charCodeAt(i); return arr.buffer; }
  function arrayBufferToB64(buf){ const bytes=new Uint8Array(buf); let bin=''; for(let i=0;i<bytes.length;i++) bin+=String.fromCharCode(bytes[i]); return btoa(bin); }
  async function deriveKey(password, saltB64, iterations=150000, length=32){
    const enc = new TextEncoder();
    const key = await crypto.subtle.importKey('raw', enc.encode(password), {name:'PBKDF2'}, false, ['deriveBits']);
    const bits = await crypto.subtle.deriveBits({name:'PBKDF2', salt: b64ToArrayBuffer(saltB64), iterations, hash:'SHA-256'}, key, length*8);
    return arrayBufferToB64(bits);
  }

  /***************** Simple user store (localStorage) *****************/
  const USER_KEY = 'mk_users_full_v1';
  function readUsers(){ try{ return JSON.parse(localStorage.getItem(USER_KEY)) || {}; }catch(e){ return {}; } }
  function writeUsers(u){ localStorage.setItem(USER_KEY, JSON.stringify(u)); }
  function saveSession(u){ localStorage.setItem('mk_session', JSON.stringify({username:u.username,role:u.role,name:u.name,ts:Date.now()})); renderCurrentUser(); }
  function clearSession(){ localStorage.removeItem('mk_session'); renderCurrentUser(); }
  function getSession(){ try{ return JSON.parse(localStorage.getItem('mk_session')); }catch(e){ return null; } }

  async function registerUser(username, password, role='teacher', name=''){
    username = username.trim().toLowerCase();
    if(!username || !password) throw new Error('Username và password là bắt buộc');
    const users = readUsers();
    if(users[username]) throw new Error('Tên đăng nhập đã tồn tại');
    if(password.length < 6) throw new Error('Mật khẩu quá ngắn (>=6)');
    const salt = await genSalt();
    const hash = await deriveKey(password, salt);
    users[username] = { username, name: name||username, role, salt, hash, created: new Date().toISOString() };
    writeUsers(users);
    return users[username];
  }

  async function verifyUser(username, password){
    username = (username||'').trim().toLowerCase();
    const users = readUsers();
    const u = users[username];
    if(!u) throw new Error('Người dùng không tồn tại');
    const check = await deriveKey(password, u.salt);
    if(check !== u.hash) throw new Error('Mật khẩu không đúng');
    return u;
  }

  // seed demo users (non-blocking)
  (function seedUsers(){
    const users = readUsers();
    if(!users['admin']){ registerUser('admin','admin','admin','Quản trị').catch(()=>{}); registerUser('teacher','teacher','teacher','GV Nguyễn').catch(()=>{}); registerUser('tech','tech','technician','Kỹ thuật').catch(()=>{}); }
  })();

  /***************** Data layer (localStorage) *****************/
  function loadDB(){ try{ return JSON.parse(localStorage.getItem(DB_KEY)) || null; }catch(e){ return null; } }
  function saveDB(db){ localStorage.setItem(DB_KEY, JSON.stringify(db)); }

  const dataLayer = {
    async getAll(){ let db = loadDB(); return db; },
    async seed(){ const now=new Date().toISOString(); const db={ rooms:[{id:'R101',name:'Lớp 1A',capacity:30,notes:'Tầng 1'},{id:'R102',name:'Lớp 1B',capacity:28,notes:'Tầng 1'}], assets:[{id:'A001',name:'Máy chiếu Epson',room:'R101',status:'normal',purchased:'2021-08-15',image:''}], inventory:[{id:'I001',name:'Phấn viết bảng',qty:50,unit:'bịch'}], requests:[{id:'REQ001',title:'Máy chiếu không lên',room:'R101',reporter:'GV A',desc:'Máy không khởi động',status:'open',created:now}], log:[] }; saveDB(db); return db; },

    // rooms
    async getRooms(){ const db=loadDB(); return (db && db.rooms) ? db.rooms : []; },
    async addRoom(r){ const db=loadDB()||{rooms:[],assets:[],inventory:[],requests:[],log:[]}; if(db.rooms.find(x=>x.id===r.id)) throw new Error('Mã phòng đã tồn tại'); db.rooms.push(r); saveDB(db); return r; },
    async updateRoom(id,patch){ const db=loadDB(); const idx=db.rooms.findIndex(x=>x.id===id); if(idx<0) throw new Error('Không tìm thấy'); db.rooms[idx]=Object.assign(db.rooms[idx],patch); saveDB(db); return db.rooms[idx]; },
    async deleteRoom(id){ const db=loadDB(); db.rooms=db.rooms.filter(x=>x.id!==id); saveDB(db); return {ok:true}; },

    // assets
    async getAssets(){ const db=loadDB(); return (db && db.assets) ? db.assets : []; },
    async addAsset(a){ const db=loadDB()||{rooms:[],assets:[],inventory:[],requests:[],log:[]}; if(db.assets.find(x=>x.id===a.id)) throw new Error('ID đã tồn tại'); db.assets.unshift(a); saveDB(db); return a; },
    async updateAsset(id,patch){ const db=loadDB(); const idx=db.assets.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.assets[idx]=Object.assign(db.assets[idx],patch); saveDB(db); return db.assets[idx]; },
    async deleteAsset(id){ const db=loadDB(); db.assets=db.assets.filter(x=>x.id!==id); saveDB(db); return {ok:true}; },

    // inventory
    async getInventory(){ const db=loadDB(); return (db && db.inventory) ? db.inventory : []; },
    async addInv(it){ const db=loadDB()||{inventory:[]}; if(db.inventory.find(x=>x.id===it.id)) throw new Error('ID tồn tại'); db.inventory.unshift(it); saveDB(db); return it; },
    async updateInv(id,patch){ const db=loadDB(); const idx=db.inventory.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.inventory[idx]=Object.assign(db.inventory[idx],patch); saveDB(db); return db.inventory[idx]; },
    async deleteInv(id){ const db=loadDB(); db.inventory=db.inventory.filter(x=>x.id!==id); saveDB(db); return {ok:true}; },

    // requests
    async getRequests(){ const db=loadDB(); return (db && db.requests) ? db.requests : []; },
    async addRequest(r){ const db=loadDB()||{requests:[]}; if(db.requests.find(x=>x.id===r.id)) throw new Error('ID tồn tại'); r.created = new Date().toISOString(); db.requests.unshift(r); saveDB(db); return r; },
    async updateRequest(id,patch){ const db=loadDB(); const idx=db.requests.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.requests[idx]=Object.assign(db.requests[idx],patch); saveDB(db); return db.requests[idx]; },
    async deleteRequest(id){ const db=loadDB(); db.requests=db.requests.filter(x=>x.id!==id); saveDB(db); return {ok:true}; },

    // import/export
    async export(){ const db=loadDB()||{rooms:[],assets:[],inventory:[],requests:[]}; const blob=new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='minhkai_export.json'; a.click(); },
    async import(file){ const text = await file.text(); const data = JSON.parse(text); if(!data.rooms) throw new Error('Invalid file'); saveDB(data); return {ok:true}; }
  };

  /***************** UI & Auth handling *****************/
  const modalLogin = new bootstrap.Modal(document.getElementById('modalLogin'));
  const modalSignup = new bootstrap.Modal(document.getElementById('modalSignup'));
  const modalRoom = new bootstrap.Modal(document.getElementById('modalRoom'));
  const modalAsset = new bootstrap.Modal(document.getElementById('modalAsset'));
  const modalInv = new bootstrap.Modal(document.getElementById('modalInv'));
  const modalReq = new bootstrap.Modal(document.getElementById('modalReq'));
  const modalViewReq = new bootstrap.Modal(document.getElementById('modalViewReq'));

  // current user render
  function renderCurrentUser(){
    const s = getSession();
    const cuEl = document.getElementById('current-user');
    const loginBtn = document.getElementById('btn-open-login');
    const signupBtn = document.getElementById('btn-open-signup');
    const logoutBtn = document.getElementById('btn-logout');
    if(s){ cuEl.innerHTML = `<span class="role-badge bg-light border">${s.name} · ${s.role}</span>`; loginBtn.style.display='none'; signupBtn.style.display='none'; logoutBtn.style.display='inline-block'; } else { cuEl.innerHTML=''; loginBtn.style.display='inline-block'; signupBtn.style.display='inline-block'; logoutBtn.style.display='none'; }
    applyRoleUI();
  }
  function getSession(){ try{ return JSON.parse(localStorage.getItem('mk_session')); }catch(e){ return null; } }
  function setSession(u){ localStorage.setItem('mk_session', JSON.stringify({username:u.username,role:u.role,name:u.name,ts:Date.now()})); renderCurrentUser(); }
  function clearSessionLocal(){ localStorage.removeItem('mk_session'); renderCurrentUser(); }

  document.getElementById('btn-open-login').addEventListener('click', ()=> modalLogin.show());
  document.getElementById('btn-open-signup').addEventListener('click', ()=> modalSignup.show());
  document.getElementById('btn-logout').addEventListener('click', ()=> { if(confirm('Đăng xuất?')) { clearSessionLocal(); } });

  document.getElementById('form-login').addEventListener('submit', async e=>{
    e.preventDefault();
    const u = document.getElementById('login-username').value.trim();
    const p = document.getElementById('login-password').value;
    try{ const user = await verifyUser(u,p); setSession(user); modalLogin.hide(); alert('Đăng nhập thành công: '+user.role); await renderAll(); } catch(err){ alert(err.message); }
    e.target.reset();
  });

  document.getElementById('form-signup').addEventListener('submit', async e=>{
    e.preventDefault();
    const u=document.getElementById('su-username').value.trim();
    const p=document.getElementById('su-password').value;
    const p2=document.getElementById('su-password2').value;
    const role=document.getElementById('su-role').value;
    const name=document.getElementById('su-name').value.trim() || u;
    if(p!==p2){ alert('Mật khẩu xác nhận không khớp'); return; }
    try{ await registerUser(u,p,role,name); alert('Đăng ký thành công, vui lòng đăng nhập'); modalSignup.hide(); } catch(err){ alert('Đăng ký lỗi: '+err.message); }
    e.target.reset();
  });

  // password strength bar
  document.getElementById('su-password').addEventListener('input', e=>{
    const val = e.target.value; let score=0;
    if(val.length>=8) score+=34; if(/[A-Z]/.test(val)&&/[0-9]/.test(val)) score+=33; if(/[^A-Za-z0-9]/.test(val)) score+=33;
    document.getElementById('pw-strength-bar').style.width = Math.min(100,score) + '%';
  });

  function requireRole(roles){ const s = getSession(); if(!s) return false; return roles.includes(s.role); }
  function applyRoleUI(){
    // show/hide add buttons
    document.getElementById('btn-add-room').style.display = requireRole(['admin','teacher']) ? 'inline-block' : 'none';
    document.getElementById('btn-add-asset').style.display = requireRole(['admin','technician']) ? 'inline-block' : 'none';
    document.getElementById('btn-add-request').style.display = requireRole(['admin','teacher','technician']) ? 'inline-block' : 'none';
    // delete buttons shown per-row in render functions
  }

  /***************** Render / CRUD UI *****************/
  async function renderAll(){
    await renderRooms(); await renderAssets(); await renderInv(); await renderReqs(); await renderChart(); renderCurrentUser();
  }

  // Rooms
  async function renderRooms(){
    const rows = await dataLayer.getRooms();
    const q = document.getElementById('search-rooms').value.trim().toLowerCase();
    const filtered = rows.filter(r => !q || (r.id + r.name).toLowerCase().includes(q));
    document.getElementById('rooms-tbody').innerHTML = filtered.map(r=>`
      <tr>
        <td>${r.id}</td><td>${r.name}</td><td>${r.capacity||''}</td><td>${r.notes||''}</td>
        <td>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-room" data-id="${r.id}"><i class="fa-solid fa-pen"></i></button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-room" data-id="${r.id}"><i class="fa-solid fa-trash"></i></button>
        </td>
      </tr>
    `).join('');
    document.querySelectorAll('[data-act="edit-room"]').forEach(b=>b.addEventListener('click', e=> openRoomEdit(e.target.closest('button').dataset.id)));
    document.querySelectorAll('[data-act="del-room"]').forEach(b=>b.addEventListener('click', async e=>{ if(!requireRole(['admin'])) { alert('Chỉ Admin được xóa'); return; } if(!confirm('Xóa phòng?')) return; try{ await dataLayer.deleteRoom(e.target.closest('button').dataset.id); await renderAll(); }catch(err){ alert(err.message); }}));
    document.getElementById('stat-rooms').textContent = (rows||[]).length;
  }

  // Assets
  async function renderAssets(){
    const rows = await dataLayer.getAssets();
    const q = document.getElementById('search-assets').value.trim().toLowerCase();
    const statusFilter = document.getElementById('filter-asset-status').value;
    const filtered = rows.filter(a=> (!q || (a.id + a.name + (a.room||'')).toLowerCase().includes(q)) && (!statusFilter || a.status===statusFilter));
    document.getElementById('assets-tbody').innerHTML = filtered.map(a=>`
      <tr>
        <td>${a.id}</td><td>${a.name}</td><td>${a.room||''}</td><td>${a.status||''}</td><td>${a.purchased||''}</td>
        <td>${a.image? '<img src="'+a.image+'" style="height:50px;border-radius:6px;">' : ''}</td>
        <td>
          <button class="btn btn-sm btn-outline-info me-1" data-act="view-asset" data-id="${a.id}"><i class="fa-solid fa-eye"></i></button>
          <button class="btn btn-sm btn-outline-primary me-1" data-act="edit-asset" data-id="${a.id}"><i class="fa-solid fa-pen"></i></button>
          <button class="btn btn-sm btn-outline-danger" data-act="del-asset" data-id="${a.id}"><i class="fa-solid fa-trash"></i></button>
        </td>
      </tr>
    `).join('');
    document.querySelectorAll('[data-act="view-asset"]').forEach(b=>b.addEventListener('click', e=> viewAsset(e.target.closest('button').dataset.id)));
    document.querySelectorAll('[data-act="edit-asset"]').forEach(b=>b.addEventListener('click', e=> editAsset(e.target.closest('button').dataset.id)));
    document.querySelectorAll('[data-act="del-asset"]').forEach(b=>b.addEventListener('click', async e=>{
      const id=e.target.closest('button').dataset.id; if(!requireRole(['admin'])){ alert('Chỉ Admin được xóa'); return; } if(!confirm('Xóa tài sản '+id+' ?')) return;
      await dataLayer.deleteAsset(id); await renderAll();
    }));
    document.getElementById('stat-assets').textContent = (rows||[]).length;
  }

  async function viewAsset(id){
    const rows = await dataLayer.getAssets(); const a = rows.find(x=>x.id===id); if(!a) return alert('Không tìm thấy');
    const html = `<div style="display:flex;gap:12px">
      <div style="flex:1"><h5>${a.name}</h5><p class="small-muted">ID: ${a.id} · Phòng: ${a.room||''}</p><p>${a.desc||''}</p></div>
      <div style="width:220px">${a.image? '<img src="'+a.image+'" style="width:100%;border-radius:8px">' : ''}<p class="small-muted mt-2">Ngày mua: ${a.purchased||''}<br>Giá: ${a.price? a.price.toLocaleString():''}</p></div>
    </div>`;
    document.getElementById('view-req-title').textContent = 'Chi tiết tài sản'; document.getElementById('view-req-body').innerHTML = html; modalViewReq.show();
  }

  async function editAsset(id){
    const arr = await dataLayer.getAssets(); const a = arr.find(x=>x.id===id); if(!a) return alert('Không tìm thấy'); if(!requireRole(['admin','technician'])) { alert('Bạn không có quyền chỉnh sửa'); return; }
    // populate form
    document.querySelector('#form-asset [name=id]').value = a.id; document.querySelector('#form-asset [name=id]').setAttribute('readonly','readonly');
    document.querySelector('#form-asset [name=name]').value = a.name; document.querySelector('#form-asset [name=serial]').value = a.serial||'';
    document.querySelector('#form-asset [name=category]').value = a.category||''; document.querySelector('#form-asset [name=room]').value = a.room||'';
    document.querySelector('#form-asset [name=status]').value = a.status||'normal'; document.querySelector('#form-asset [name=purchased]').value = a.purchased||'';
    document.querySelector('#form-asset [name=price]').value = a.price||''; document.querySelector('#form-asset [name=vendor]').value = a.vendor||'';
    document.querySelector('#form-asset [name=warranty]').value = a.warranty||''; document.querySelector('#form-asset [name=desc]').value = a.desc||'';
    if(a.image){ const pre = document.getElementById('asset-preview'); pre.src = a.image; pre.style.display='block'; } else { document.getElementById('asset-preview').style.display='none'; }
    modalAsset.show();
  }

  // Inventory
  async function renderInv(){
    const rows = await dataLayer.getInventory();
    document.getElementById('inv-tbody').innerHTML = (rows||[]).map(i=>`<tr><td>${i.id}</td><td>${i.name}</td><td>${i.qty}</td><td>${i.unit||''}</td><td><button class="btn btn-sm btn-outline-primary" data-act="edit-inv" data-id="${i.id}">Sửa</button></td></tr>`).join('');
  }

  // Requests
  async function renderReqs(){
    const rows = await dataLayer.getRequests();
    const q = document.getElementById('search-req').value.trim().toLowerCase(); const statusFilter = document.getElementById('filter-req-status').value;
    const filtered = rows.filter(r=> (!q || (r.id+r.title+r.reporter).toLowerCase().includes(q)) && (!statusFilter || r.status===statusFilter));
    document.getElementById('reqs-tbody').innerHTML = filtered.map(r=>`
      <tr><td>${r.id}</td><td>${r.title}</td><td>${r.room||''}</td><td>${r.reporter||''}</td><td>${r.status}</td>
      <td><button class="btn btn-sm btn-outline-info me-1" data-act="view-req" data-id="${r.id}"><i class="fa-solid fa-eye"></i></button><button class="btn btn-sm btn-outline-primary" data-act="edit-req" data-id="${r.id}">Sửa</button></td></tr>
    `).join('');
    document.querySelectorAll('[data-act="view-req"]').forEach(b=>b.addEventListener('click', e=> viewRequest(e.target.closest('button').dataset.id)));
    document.getElementById('stat-requests').textContent = (rows||[]).filter(r=>r.status==='open').length;
  }

  async function viewRequest(id){
    const rows = await dataLayer.getRequests(); const r = rows.find(x=>x.id===id); if(!r) return alert('Không tìm thấy');
    const html = `<p><strong>Mã:</strong> ${r.id}</p><p><strong>Tiêu đề:</strong> ${r.title}</p><p><strong>Phòng:</strong> ${r.room||''}</p><p><strong>Người báo:</strong> ${r.reporter||''}</p><p><strong>Trạng thái:</strong> ${r.status}</p><p><strong>Mô tả:</strong><br>${r.desc||''}</p><p class="small-muted"><strong>Ngày tạo:</strong> ${r.created||''}</p>`;
    document.getElementById('view-req-title').textContent = 'Chi tiết yêu cầu'; document.getElementById('view-req-body').innerHTML = html; modalViewReq.show();
  }

  /***************** Forms: handle submits *****************/
  document.getElementById('form-room').addEventListener('submit', async e=>{ e.preventDefault(); const fd=new FormData(e.target); const obj={id:fd.get('id').trim(), name:fd.get('name').trim(), capacity:parseInt(fd.get('capacity'))||null, notes:fd.get('notes').trim()}; try{ if(document.querySelector('#form-room [name=id]').hasAttribute('readonly')) await dataLayer.updateRoom(obj.id,obj); else await dataLayer.addRoom(obj); modalRoom.hide(); await renderAll(); }catch(err){ alert(err.message);} });

  // Asset form: image preview handling
  document.querySelector('#form-asset [name=image]').addEventListener('change', e=> {
    const f = e.target.files && e.target.files[0]; const pre = document.getElementById('asset-preview');
    if(!f){ pre.style.display='none'; pre.src=''; return; }
    const r = new FileReader(); r.onload = ()=> { pre.src = r.result; pre.style.display = 'block'; }; r.readAsDataURL(f);
  });

  document.getElementById('gen-asset-id').addEventListener('click', ()=> document.querySelector('#form-asset [name=id]').value = genId('AS-'));

  document.getElementById('form-asset').addEventListener('submit', async e=>{
    e.preventDefault();
    if(!requireRole(['admin','technician'])) { alert('Bạn không có quyền.'); return; }
    const fd=new FormData(e.target);
    const id = (fd.get('id')||'').trim() || genId('AS-');
    const obj = { id, name:fd.get('name').trim(), serial:fd.get('serial').trim(), category:fd.get('category').trim(), room:fd.get('room').trim(), status:fd.get('status'), purchased:fd.get('purchased')||'', price: Number(fd.get('price')||0), vendor:fd.get('vendor')||'', warranty: Number(fd.get('warranty')||0), desc:fd.get('desc')||'', updatedAt: new Date().toISOString() };
    // image to dataURL
    const imgFile = e.target.querySelector('[name=image]').files[0];
    if(imgFile){ obj.image = await fileToDataURL(imgFile); } else { obj.image = ''; }
    try{ const existing = (await dataLayer.getAssets()).find(x=>x.id===id); if(existing) await dataLayer.updateAsset(id,obj); else await dataLayer.addAsset(obj); modalAsset.hide(); await renderAll(); } catch(err){ alert(err.message); }
  });

  document.getElementById('form-inv').addEventListener('submit', async e=>{ e.preventDefault(); const fd=new FormData(e.target); const obj={id:fd.get('id').trim(), name:fd.get('name').trim(), qty:parseInt(fd.get('qty'))||0, unit:fd.get('unit').trim()}; try{ const existing = (await dataLayer.getInventory()).find(x=>x.id===obj.id); if(existing) await dataLayer.updateInv(obj.id,obj); else await dataLayer.addInv(obj); modalInv.hide(); await renderAll(); }catch(err){ alert(err.message);} });

  document.getElementById('form-req').addEventListener('submit', async e=>{ e.preventDefault(); const fd=new FormData(e.target); const obj={id:fd.get('id').trim(), title:fd.get('title').trim(), room:fd.get('room').trim(), reporter:fd.get('reporter').trim(), desc:fd.get('desc').trim(), status:fd.get('status')}; try{ const existing = (await dataLayer.getRequests()).find(x=>x.id===obj.id); if(existing) await dataLayer.updateRequest(obj.id,obj); else await dataLayer.addRequest(obj); modalReq.hide(); await renderAll(); }catch(err){ alert(err.message);} });

  // Open modals
  document.getElementById('btn-add-room').addEventListener('click', ()=> { document.getElementById('form-room').reset(); document.querySelector('#form-room [name=id]').removeAttribute('readonly'); modalRoom.show(); });
  document.getElementById('btn-add-asset').addEventListener('click', ()=> { if(!requireRole(['admin','technician'])){ alert('Bạn cần đăng nhập với quyền admin hoặc technician.'); return; } document.getElementById('form-asset').reset(); document.querySelector('#form-asset [name=id]').removeAttribute('readonly'); document.getElementById('asset-preview').style.display='none'; modalAsset.show(); });
  document.getElementById('btn-add-request').addEventListener('click', ()=> { if(!requireRole(['admin','teacher','technician'])){ alert('Bạn cần đăng nhập.'); return; } document.getElementById('form-req').reset(); document.querySelector('#form-req [name=id]').removeAttribute('readonly'); modalReq.show(); });

  // search / filters
  document.getElementById('search-rooms').addEventListener('input', debounce(()=>renderRooms(),200));
  document.getElementById('search-assets').addEventListener('input', debounce(()=>renderAssets(),200));
  document.getElementById('filter-asset-status').addEventListener('change', ()=>renderAssets());
  document.getElementById('search-inv').addEventListener('input', debounce(()=>renderInv(),200));
  document.getElementById('search-req').addEventListener('input', debounce(()=>renderReqs(),200));
  document.getElementById('filter-req-status').addEventListener('change', ()=>renderReqs());

  // import/export
  document.getElementById('btn-export').addEventListener('click', ()=> dataLayer.export());
  document.getElementById('btn-import').addEventListener('click', ()=> document.getElementById('file-input').click());
  document.getElementById('file-input').addEventListener('change', async e=>{ if(!e.target.files.length) return; try{ await dataLayer.import(e.target.files[0]); alert('Import thành công'); await renderAll(); } catch(err){ alert('Import lỗi: '+err.message);} e.target.value=''; });

  document.getElementById('seed-data').addEventListener('click', async ()=> { if(confirm('Tạo dữ liệu mẫu (ghi đè)?')){ await dataLayer.seed(); await renderAll(); } });

  // CSV import assets (simple)
  // Add button if needed - here reuse file input from user side if implemented

  // helper: convert file to dataURL
  function fileToDataURL(file){ return new Promise((res,rej)=>{ const r=new FileReader(); r.onload=()=>res(r.result); r.onerror=()=>rej('File read error'); r.readAsDataURL(file); }); }

  // utils
  function genId(prefix='X'){ const n = Math.floor(Math.random()*900+100); return prefix + Date.now().toString().slice(-6) + n; }
  function debounce(fn, ms=200){ let t; return (...args)=>{ clearTimeout(t); t=setTimeout(()=>fn(...args), ms); }; }

  // Chart
  let chart = null;
  async function renderChart(){
    const ctx = document.getElementById('chartStatus').getContext('2d');
    const reqs = await dataLayer.getRequests(); const assets = await dataLayer.getAssets();
    const statusCounts = { open:0, in_progress:0, done:0 }; (reqs||[]).forEach(r=> statusCounts[r.status] = (statusCounts[r.status]||0)+1 );
    const data = [statusCounts.open, statusCounts.in_progress, statusCounts.done];
    if(chart) chart.destroy();
    chart = new Chart(ctx, { type:'doughnut', data:{ labels:['Mới','Đang xử lý','Hoàn tất'], datasets:[{ data, backgroundColor:['#ffc107','#0d6efd','#198754'] }] }, options:{responsive:true, plugins:{legend:{position:'bottom'}}} });
  }

  // initial render & seed if empty
  (async ()=>{ try{ const db = await dataLayer.getAll(); if(!db) await dataLayer.seed(); renderCurrentUser(); await renderAll(); } catch(err){ console.error(err); alert('Lỗi khởi tạo: '+err.message); } })();

  </script>
</body>
</html>
