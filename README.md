<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Quản Lý Cơ Sở Vật Chất - THCS Hàn Thuyên </title>

  <!-- Bootstrap 5 -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <!-- FontAwesome -->
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" rel="stylesheet">
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>

  <style>
    :root{ --brand:#0d6efd; --muted:#6c757d; --card-radius:12px; --shadow: 0 8px 30px rgba(13,110,253,0.06); font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
    body { background:#f4f6fb; }
    .app-container{ max-width:1200px; margin:20px auto; padding:0 12px; }
    .hero { background:linear-gradient(90deg, rgba(13,110,253,0.06), rgba(13,110,253,0.02)); border-radius:var(--card-radius); padding:18px; display:flex; gap:16px; align-items:center; box-shadow:var(--shadow); }
    .hero img{ width:260px; border-radius:10px; object-fit:cover; }
    .card-stat{ border-radius:12px; }
    .gallery img{ border-radius:10px; object-fit:cover; height:140px; width:100%; transition:transform .25s; }
    .gallery img:hover{ transform:scale(1.05); }
    .small-muted{ color:var(--muted); font-size:0.9rem; }
    .role-badge { text-transform:uppercase; font-weight:600; font-size:0.75rem; padding:4px 8px; border-radius:999px; }
    .asset-preview{ max-height:160px; object-fit:cover; border-radius:8px; }
    .pw-strength { height:8px; border-radius:6px; background:#e9ecef; overflow:hidden; }
    .pw-strength > i { display:block; height:100%; width:0%; background:linear-gradient(90deg,#ff6b6b,#ffd166,#06d6a0); transition:width .25s; }
    .muted-small { color:#6c757d; font-size:0.85rem; }
    @media (max-width:900px){ .hero{flex-direction:column;text-align:center} .hero img{max-width:100%} }
    .toast-container{ position: fixed; right: 20px; top: 80px; z-index: 1080; }
  </style>
</head>
<body>
  <!-- TOP BAR -->
  <div class="container-fluid bg-white shadow-sm py-2">
    <div class="container d-flex justify-content-between align-items-center">
      <div class="d-flex align-items-center gap-3">
        <i class="fa-solid fa-school fa-2x text-primary"></i>
        <div>
          <h4 class="mb-0">Trường THCS Hàn Thuyên</h4>
          <div class="small-muted">Hệ thống quản lý cơ sở vật chất</div>
        </div>
      </div>
      <div class="d-flex align-items-center gap-2">
        <div id="current-user"></div>
        <button id="btn-open-sync" class="btn btn-outline-info btn-sm">Bật đồng bộ</button>
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
        <h2 class="mb-1">Hệ Thống Quản Lý Cơ Sở Vật Chất — THCS Hàn Thuyên</h2>
        <p class="small-muted mb-2">Dashboard, quản lý phòng, tài sản, kho vật tư và yêu cầu bảo trì.</p>
        <div class="mt-2">
          <button id="btn-add-room" class="btn btn-primary me-2"><i class="fa-solid fa-door-open me-2"></i>Thêm phòng</button>
          <button id="btn-add-asset" class="btn btn-outline-primary me-2"><i class="fa-solid fa-box me-2"></i>Thêm thiết bị</button>
          <button id="btn-add-request" class="btn btn-outline-warning me-2"><i class="fa-solid fa-wrench me-2"></i>Tạo yêu cầu</button>
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
            <h6 class="mb-3"></h6>
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

    <footer class="text-center small-muted mb-5">© 2025 Trường THCS Hàn Thuyên — Prototype (Frontend)</footer>
  </div>

  <!-- ===== Modals: Room / Asset / Inventory / Request / Login / Signup / View / SyncConfig ===== -->
  <!-- Modal: Room -->
  <div class="modal fade" id="modalRoom" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content"><form id="form-room" class="needs-validation" novalidate>
    <div class="modal-header"><h5 class="modal-title">Thêm / Sửa phòng</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <div class="mb-2"><label class="form-label">Mã phòng</label><input name="id" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Tên phòng</label><input name="name" class="form-control" required></div>
      <div class="mb-2"><label class="form-label">Sức chứa</label><input name="capacity" type="number" class="form-control"></div>
      <div class="mb-2"><label class="form-label">Ghi chú</label><textarea name="notes" class="form-control"></textarea></div>
      <div class="muted-small mt-2">Ghi chú quyền: Giáo viên có thể chỉnh tên, sức chứa, ghi chú; Admin chỉnh mọi trường; Kỹ thuật viên không chỉnh phòng.</div>
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
      <div class="muted-small mt-2">Ghi chú quyền: Kỹ thuật viên có thể chỉnh trạng thái, mô tả, phòng, nhà cung cấp, bảo hành; Giáo viên chỉ xem; Admin toàn quyền.</div>
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
      <div class="muted-small mt-2">Ghi chú quyền: Kỹ thuật viên chỉnh số lượng/đơn vị; Giáo viên chỉ xem; Admin toàn quyền.</div>
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
      <div class="muted-small mt-2">Ghi chú quyền: Giáo viên tạo/sửa yêu cầu của chính họ; Kỹ thuật viên cập nhật trạng thái; Admin toàn quyền.</div>
    </div>
    <div class="modal-footer"><button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Hủy</button><button type="submit" class="btn btn-primary">Lưu</button></div>
  </form></div></div></div>

  <!-- Modal: Login -->
  <div class="modal fade" id="modalLogin" tabindex="-1"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content"><form id="form-login" class="p-3">
    <h5>Đăng nhập</h5>
    <div class="mb-2"><label class="form-label">Tên đăng nhập / Email</label><input id="login-username" class="form-control" required></div>
    <div class="mb-2"><label class="form-label">Mật khẩu</label><input id="login-password" type="password" class="form-control" required></div>
    <div class="d-flex justify-content-end"><button type="submit" class="btn btn-primary btn-sm">Đăng nhập</button></div>
    <div class="mt-2 small-muted"></div>
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

  <!-- Modal: View Request / Generic Detail -->
  <div class="modal fade" id="modalViewReq" tabindex="-1"><div class="modal-dialog modal-md modal-dialog-centered"><div class="modal-content">
    <div class="modal-header"><h5 class="modal-title" id="view-req-title">Chi tiết</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body" id="view-req-body"></div>
    <div class="modal-footer"><button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Đóng</button></div>
  </div></div></div>

  <!-- Modal: Sync config (user paste firebaseConfig JSON here) -->
  <div class="modal fade" id="modalSync" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content">
    <div class="modal-header"><h5 class="modal-title">Bật đồng bộ (Firebase)</h5><button type="button" class="btn-close" data-bs-dismiss="modal"></button></div>
    <div class="modal-body">
      <p>Paste object <code>firebaseConfig</code> (JSON) từ Firebase Console vào ô dưới, rồi bấm <strong>Kích hoạt đồng bộ</strong>.</p>
      <textarea id="firebase-config-input" rows="10" class="form-control" placeholder='Ví dụ: {"apiKey":"...","authDomain":"...","projectId":"...","storageBucket":"...","messagingSenderId":"...","appId":"..."}'></textarea>
      <div class="form-text mt-2 muted-small">Lưu ý: bạn cần bật Email/Password sign-in nếu muốn sử dụng Firebase Auth.</div>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-outline-secondary" data-bs-dismiss="modal">Đóng</button>
      <button id="btn-start-sync" type="button" class="btn btn-info">Kích hoạt đồng bộ</button>
    </div>
  </div></div></div>

  <!-- Toast container -->
  <div class="toast-container" id="toastContainer"></div>

  <!-- Bootstrap JS -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

  <!-- ================= App JS: localStorage dataLayer, auth, UI ================= -->
  <script>
  /* (To save space here we keep the same app logic as provided previously: 
     - localDataLayer, users logic, UI handlers, CRUD, renderAll, startRealtimeSync function, etc.
     - For clarity and to keep this snippet focused on sync UI we include the complete logic below.)
     The full app code (unchanged) is included after this comment block.
  */
  </script>

  <!-- Insert the full app JS from previous message here (same as before, including startRealtimeSync definition).
       For brevity in this reply I will now paste the full JS exactly as in the code you've previously accepted,
       but with an extra block at the end that wires the "Bật đồng bộ" modal to call startRealtimeSync.
  -->

  <!-- Full app JS START -->
  <script>
  /******************************************************************
   * LOCAL dataLayer (localStorage) - same API as cloud layer later
   ******************************************************************/
  const DB_KEY = 'hanthuyen_full_v1';
  function loadDBLocal(){ try{ return JSON.parse(localStorage.getItem(DB_KEY)) || { rooms:[], assets:[], inventory:[], requests:[], users:{} } }catch(e){ return { rooms:[], assets:[], inventory:[], requests:[], users:{} } } }
  function saveDBLocal(db){ localStorage.setItem(DB_KEY, JSON.stringify(db)); }

  // Local user store (for demo). When cloud sync is enabled, we will sync users to Firestore as well.
  const USER_KEY = 'ht_users_v1';
  function readUsersLocal(){ try{ return JSON.parse(localStorage.getItem(USER_KEY)) || {}; }catch(e){ return {}; } }
  function writeUsersLocal(u){ localStorage.setItem(USER_KEY, JSON.stringify(u)); }

  // Simple PBKDF2 demo hashing (client-side only)
  async function genSalt(len=16){ const arr = crypto.getRandomValues(new Uint8Array(len)); return btoa(String.fromCharCode(...arr)); }
  function b64ToArrayBuffer(b64){ const bin=atob(b64); const len=bin.length; const arr=new Uint8Array(len); for(let i=0;i<len;i++) arr[i]=bin.charCodeAt(i); return arr.buffer; }
  function arrayBufferToB64(buf){ const bytes=new Uint8Array(buf); let bin=''; for(let i=0;i<bytes.length;i++) bin+=String.fromCharCode(bytes[i]); return btoa(bin); }
  async function deriveKey(password, saltB64, iterations=120000, length=32){ const enc = new TextEncoder(); const key = await crypto.subtle.importKey('raw', enc.encode(password), {name:'PBKDF2'}, false, ['deriveBits']); const bits = await crypto.subtle.deriveBits({name:'PBKDF2', salt: b64ToArrayBuffer(saltB64), iterations, hash:'SHA-256'}, key, length*8); return arrayBufferToB64(bits); }

  // Local register/verify
  async function registerUserLocal(username, password, role='teacher', name=''){
    username = username.trim().toLowerCase();
    if(!username || !password) throw new Error('Username và password là bắt buộc');
    const users = readUsersLocal();
    if(users[username]) throw new Error('Tên đăng nhập đã tồn tại');
    if(password.length < 6) throw new Error('Mật khẩu quá ngắn (>=6)');
    const salt = await genSalt();
    const hash = await deriveKey(password, salt);
    users[username] = { username, name: name||username, role, salt, hash, created: new Date().toISOString() };
    writeUsersLocal(users);
    return users[username];
  }
  async function verifyUserLocal(username, password){
    username = (username||'').trim().toLowerCase();
    const users = readUsersLocal();
    const u = users[username];
    if(!u) throw new Error('Người dùng không tồn tại');
    const check = await deriveKey(password, u.salt);
    if(check !== u.hash) throw new Error('Mật khẩu không đúng');
    return u;
  }

  // seed demo users if none
  (async ()=>{
    const users = readUsersLocal();
    if(!users['admin']){
      try{ await registerUserLocal('admin','admin','admin','Quản trị'); await registerUserLocal('teacher','teacher','teacher','GV Nguyễn'); await registerUserLocal('tech','tech','technician','Kỹ thuật'); }catch(e){}
    }
  })();

  /******************************************************************
   * LOCAL dataLayer (CRUD) - used by default
   ******************************************************************/
  const localDataLayer = {
    async getAll(){ return loadDBLocal(); },
    async seed(){
      const now = new Date().toISOString();
      const db = {
        rooms:[
          {id:'R101',name:'Lớp 6A',capacity:32,notes:'Tầng 1', updatedAt: now},
          {id:'R102',name:'Lớp 7B',capacity:30,notes:'Tầng 2', updatedAt: now}
        ],
        assets:[
          {id:'A001',name:'Máy chiếu Epson',room:'R101',status:'normal',purchased:'2021-08-15',image:'',desc:'Máy chiếu phòng 6A', updatedAt: now}
        ],
        inventory:[
          {id:'I001',name:'Phấn viết bảng',qty:50,unit:'bịch', updatedAt: now}
        ],
        requests:[
          {id:'REQ001',title:'Máy chiếu không lên',room:'R101',reporter:'GV A',desc:'Máy không khởi động',status:'open',created:now, updatedAt: now}
        ],
        users: readUsersLocal()
      };
      saveDBLocal(db); return db;
    },

    // rooms
    async getRooms(){ const db = loadDBLocal(); return db.rooms || []; },
    async addRoom(r){ const db = loadDBLocal(); if(db.rooms.find(x=>x.id===r.id)) throw new Error('Mã phòng đã tồn tại'); db.rooms.push(r); saveDBLocal(db); return r; },
    async updateRoom(id,patch){ const db = loadDBLocal(); const idx = db.rooms.findIndex(x=>x.id===id); if(idx<0) throw new Error('Không tìm thấy'); db.rooms[idx] = Object.assign(db.rooms[idx], patch); saveDBLocal(db); return db.rooms[idx]; },
    async deleteRoom(id){ const db = loadDBLocal(); db.rooms = db.rooms.filter(x=>x.id!==id); saveDBLocal(db); return {ok:true}; },

    // assets
    async getAssets(){ return (loadDBLocal().assets || []); },
    async addAsset(a){ const db = loadDBLocal(); if(db.assets.find(x=>x.id===a.id)) throw new Error('ID đã tồn tại'); db.assets.unshift(a); saveDBLocal(db); return a; },
    async updateAsset(id,patch){ const db = loadDBLocal(); const idx = db.assets.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.assets[idx] = Object.assign(db.assets[idx], patch); saveDBLocal(db); return db.assets[idx]; },
    async deleteAsset(id){ const db = loadDBLocal(); db.assets = db.assets.filter(x=>x.id!==id); saveDBLocal(db); return {ok:true}; },

    // inventory
    async getInventory(){ return (loadDBLocal().inventory || []); },
    async addInv(it){ const db = loadDBLocal(); if(db.inventory.find(x=>x.id===it.id)) throw new Error('ID tồn tại'); db.inventory.unshift(it); saveDBLocal(db); return it; },
    async updateInv(id,patch){ const db = loadDBLocal(); const idx = db.inventory.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.inventory[idx] = Object.assign(db.inventory[idx], patch); saveDBLocal(db); return db.inventory[idx]; },
    async deleteInv(id){ const db = loadDBLocal(); db.inventory = db.inventory.filter(x=>x.id!==id); saveDBLocal(db); return {ok:true}; },

    // requests
    async getRequests(){ return (loadDBLocal().requests || []); },
    async addRequest(r){ const db = loadDBLocal(); if(db.requests.find(x=>x.id===r.id)) throw new Error('ID tồn tại'); r.created = r.created || new Date().toISOString(); db.requests.unshift(r); saveDBLocal(db); return r; },
    async updateRequest(id,patch){ const db = loadDBLocal(); const idx = db.requests.findIndex(x=>x.id===id); if(idx<0) throw new Error('Not found'); db.requests[idx] = Object.assign(db.requests[idx], patch); saveDBLocal(db); return db.requests[idx]; },
    async deleteRequest(id){ const db = loadDBLocal(); db.requests = db.requests.filter(x=>x.id!==id); saveDBLocal(db); return {ok:true}; },

    // users (for migration)
    async getUsers(){ return readUsersLocal(); },
    async addUser(u){ const users = readUsersLocal(); users[u.username] = u; writeUsersLocal(users); return u; }
  };

  // Start with local layer; when cloud sync is started, we will replace dataLayer with cloud layer
  let dataLayer = localDataLayer;
  let usersLayer = { register: registerUserLocal, verify: verifyUserLocal, getAll: readUsersLocal, add: writeUsersLocal };

  /******************************************************************
   * UI & Auth handling (uses dataLayer and usersLayer)
   ******************************************************************/
  const modalLogin = new bootstrap.Modal(document.getElementById('modalLogin'));
  const modalSignup = new bootstrap.Modal(document.getElementById('modalSignup'));
  const modalRoom = new bootstrap.Modal(document.getElementById('modalRoom'));
  const modalAsset = new bootstrap.Modal(document.getElementById('modalAsset'));
  const modalInv = new bootstrap.Modal(document.getElementById('modalInv'));
  const modalReq = new bootstrap.Modal(document.getElementById('modalReq'));
  const modalViewReq = new bootstrap.Modal(document.getElementById('modalViewReq'));
  const modalSync = new bootstrap.Modal(document.getElementById('modalSync'));

  function getSession(){ try{ return JSON.parse(localStorage.getItem('ht_session')); }catch(e){ return null; } }
  function setSession(u){ localStorage.setItem('ht_session', JSON.stringify({username:u.username,role:u.role,name:u.name,ts:Date.now()})); renderCurrentUser(); }
  function clearSession(){ localStorage.removeItem('ht_session'); renderCurrentUser(); }

  function renderCurrentUser(){
    const s = getSession();
    const cuEl = document.getElementById('current-user');
    const loginBtn = document.getElementById('btn-open-login');
    const signupBtn = document.getElementById('btn-open-signup');
    const logoutBtn = document.getElementById('btn-logout');
    if(s){ cuEl.innerHTML = `<span class="role-badge bg-light border">${s.name} · ${s.role}</span>`; loginBtn.style.display='none'; signupBtn.style.display='none'; logoutBtn.style.display='inline-block'; }
    else { cuEl.innerHTML=''; loginBtn.style.display='inline-block'; signupBtn.style.display='inline-block'; logoutBtn.style.display='none'; }
    applyRoleUI();
  }

  function requireRole(roles){ const s = getSession(); if(!s) return false; return roles.includes(s.role); }

  function applyRoleUI(){
    document.getElementById('btn-add-room').style.display = requireRole(['admin','teacher']) ? 'inline-block' : 'none';
    document.getElementById('btn-add-asset').style.display = requireRole(['admin','technician']) ? 'inline-block' : 'none';
    document.getElementById('btn-add-request').style.display = requireRole(['admin','teacher','technician']) ? 'inline-block' : 'none';
  }

  // Fix: delegate click events so inner icons don't break target
  document.addEventListener('click', async function(e){
    const el = e.target.closest('[data-act]');
    if(!el) return;
    const act = el.dataset.act;
    const id = el.dataset.id;
    // Route actions
    if(act === 'edit-room') openRoomEdit(id);
    else if(act === 'del-room') {
      if(!requireRole(['admin'])) return alert('Chỉ Admin được xóa');
      if(!confirm('Xóa phòng?')) return;
      await dataLayer.deleteRoom(id); await renderAll();
    }
    else if(act === 'view-asset') viewAsset(id);
    else if(act === 'edit-asset') editAsset(id);
    else if(act === 'del-asset') {
      if(!requireRole(['admin'])) return alert('Chỉ Admin được xóa');
      if(!confirm('Xóa tài sản '+id+' ?')) return;
      await dataLayer.deleteAsset(id); await renderAll();
    }
    else if(act === 'edit-inv') editInv(id);
    else if(act === 'view-req') viewRequest(id);
    else if(act === 'edit-req') editReq(id);
  });

  // Login / Signup handlers
  document.getElementById('btn-open-login').addEventListener('click', ()=> modalLogin.show());
  document.getElementById('btn-open-signup').addEventListener('click', ()=> modalSignup.show());
  document.getElementById('btn-logout').addEventListener('click', ()=> { if(confirm('Đăng xuất?')) clearSession(); });

  document.getElementById('form-login').addEventListener('submit', async e=>{
    e.preventDefault();
    const u = document.getElementById('login-username').value.trim();
    const p = document.getElementById('login-password').value;
    try{
      // try cloud verify first if usersLayer supports verifyCloud
      let user = null;
      if(typeof usersLayer.verifyCloud === 'function'){
        user = await usersLayer.verifyCloud(u,p).catch(()=>null);
      }
      if(!user){
        user = await usersLayer.verify(u,p);
      }
      setSession(user);
      try{ modalLogin.hide(); }catch(err){}
      await renderAll();
      toast('Đăng nhập thành công: '+user.role);
    } catch(err){ alert(err.message); }
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
    try{
      const lu = await usersLayer.register(u,p,role,name).catch(()=>null);
      if(typeof usersLayer.addCloud === 'function'){
        await usersLayer.addCloud({ username:u, name, role, created: new Date().toISOString() });
      }
      alert('Đăng ký thành công, vui lòng đăng nhập');
      modalSignup.hide();
    } catch(err){ alert('Đăng ký lỗi: '+err.message); }
    e.target.reset();
  });

  // password strength bar
  document.getElementById('su-password').addEventListener('input', e=>{
    const val = e.target.value; let score=0;
    if(val.length>=8) score+=34; if(/[A-Z]/.test(val)&&/[0-9]/.test(val)) score+=33; if(/[^A-Za-z0-9]/.test(val)) score+=33;
    document.getElementById('pw-strength-bar').style.width = Math.min(100,score) + '%';
  });

  // toast helper
  function toast(msg, timeout=2500){
    const cont = document.getElementById('toastContainer');
    const id = 't'+Date.now();
    const el = document.createElement('div');
    el.className = 'toast show align-items-center text-bg-primary border-0';
    el.style.minWidth = '220px';
    el.id = id;
    el.innerHTML = `<div class="d-flex"><div class="toast-body">${msg}</div><button class="btn-close btn-close-white me-2 m-auto" data-bs-dismiss="toast"></button></div>`;
    cont.appendChild(el);
    setTimeout(()=>{ try{ el.remove(); }catch(e){} }, timeout);
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
    document.getElementById('rooms-tbody').innerHTML = filtered.map(r=>{
      const canEdit = requireRole(['admin','teacher']);
      const canDelete = requireRole(['admin']);
      return `
      <tr>
        <td>${r.id}</td><td>${r.name}</td><td>${r.capacity||''}</td><td>${r.notes||''}</td>
        <td>
          <button class="btn btn-sm btn-outline-info me-1" data-act="view-room" data-id="${r.id}" title="Xem"><i class="fa-solid fa-eye"></i></button>
          ${canEdit? `<button class="btn btn-sm btn-outline-primary me-1" data-act="edit-room" data-id="${r.id}" title="Sửa"><i class="fa-solid fa-pen"></i></button>` : ''}
          ${canDelete? `<button class="btn btn-sm btn-outline-danger" data-act="del-room" data-id="${r.id}" title="Xóa"><i class="fa-solid fa-trash"></i></button>`: ''}
        </td>
      </tr>`;
    }).join('');
    document.querySelectorAll('[data-act="view-room"]').forEach(b=> b.addEventListener('click', e => {
      const id = e.target.closest('[data-id]').dataset.id;
      viewRoom(id);
    }));
    document.getElementById('stat-rooms').textContent = (rows||[]).length;
  }

  function viewRoom(id){
    (async ()=>{
      const rows = await dataLayer.getRooms(); const r = rows.find(x=>x.id===id); if(!r) return alert('Không tìm thấy');
      document.getElementById('view-req-title').textContent = 'Chi tiết phòng'; document.getElementById('view-req-body').innerHTML = `<p><strong>Mã:</strong> ${r.id}</p><p><strong>Tên:</strong> ${r.name}</p><p><strong>Sức chứa:</strong> ${r.capacity||''}</p><p><strong>Ghi chú:</strong> ${r.notes||''}</p>`;
      modalViewReq.show();
    })();
  }

  // Assets
  async function renderAssets(){
    const rows = await dataLayer.getAssets();
    const q = document.getElementById('search-assets').value.trim().toLowerCase();
    const statusFilter = document.getElementById('filter-asset-status').value;
    const filtered = rows.filter(a=> (!q || (a.id + a.name + (a.room||'')).toLowerCase().includes(q)) && (!statusFilter || a.status===statusFilter));
    document.getElementById('assets-tbody').innerHTML = filtered.map(a=>{
      const canEdit = requireRole(['admin','technician']);
      const canDelete = requireRole(['admin']);
      return `
      <tr>
        <td>${a.id}</td><td>${a.name}</td><td>${a.room||''}</td><td>${a.status||''}</td><td>${a.purchased||''}</td>
        <td>${a.image? '<img src="'+a.image+'" style="height:50px;border-radius:6px;">' : ''}</td>
        <td>
          <button class="btn btn-sm btn-outline-info me-1" data-act="view-asset" data-id="${a.id}"><i class="fa-solid fa-eye"></i></button>
          ${canEdit? `<button class="btn btn-sm btn-outline-primary me-1" data-act="edit-asset" data-id="${a.id}"><i class="fa-solid fa-pen"></i></button>`: ''}
          ${canDelete? `<button class="btn btn-sm btn-outline-danger" data-act="del-asset" data-id="${a.id}"><i class="fa-solid fa-trash"></i></button>`: ''}
        </td>
      </tr>`;
    }).join('');
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

  // Inventory
  async function renderInv(){
    const rows = await dataLayer.getInventory();
    document.getElementById('inv-tbody').innerHTML = (rows||[]).map(i=>{
      const canEdit = requireRole(['admin','technician']);
      return `<tr><td>${i.id}</td><td>${i.name}</td><td>${i.qty}</td><td>${i.unit||''}</td><td>${canEdit? `<button class="btn btn-sm btn-outline-primary" data-act="edit-inv" data-id="${i.id}">Sửa</button>` : ''}</td></tr>`;
    }).join('');
  }

  // Requests
  async function renderReqs(){
    const rows = await dataLayer.getRequests();
    const q = document.getElementById('search-req').value.trim().toLowerCase(); const statusFilter = document.getElementById('filter-req-status').value;
    const filtered = rows.filter(r=> (!q || (r.id+r.title+r.reporter).toLowerCase().includes(q)) && (!statusFilter || r.status===statusFilter));
    document.getElementById('reqs-tbody').innerHTML = filtered.map(r=>{
      const s = getSession();
      const canEdit = s && (s.role==='admin' || s.username=== (r.reporter && r.reporter.toLowerCase()) || s.role==='technician');
      return `<tr><td>${r.id}</td><td>${r.title}</td><td>${r.room||''}</td><td>${r.reporter||''}</td><td>${r.status}</td>
      <td><button class="btn btn-sm btn-outline-info me-1" data-act="view-req" data-id="${r.id}"><i class="fa-solid fa-eye"></i></button>${canEdit? `<button class="btn btn-sm btn-outline-primary" data-act="edit-req" data-id="${r.id}">Sửa</button>`: ''}</td></tr>`;
    }).join('');
    document.getElementById('stat-requests').textContent = (rows||[]).filter(r=>r.status==='open').length;
  }

  async function viewRequest(id){
    const rows = await dataLayer.getRequests(); const r = rows.find(x=>x.id===id); if(!r) return alert('Không tìm thấy');
    const html = `<p><strong>Mã:</strong> ${r.id}</p><p><strong>Tiêu đề:</strong> ${r.title}</p><p><strong>Phòng:</strong> ${r.room||''}</p><p><strong>Người báo:</strong> ${r.reporter||''}</p><p><strong>Trạng thái:</strong> ${r.status}</p><p><strong>Mô tả:</strong><br>${r.desc||''}</p><p class="small-muted"><strong>Ngày tạo:</strong> ${r.created||''}</p>`;
    document.getElementById('view-req-title').textContent = 'Chi tiết yêu cầu'; document.getElementById('view-req-body').innerHTML = html; modalViewReq.show();
  }

  /***************** Forms: handle submits *****************/
  document.getElementById('form-room').addEventListener('submit', async e=>{ e.preventDefault();
    const fd=new FormData(e.target);
    const obj={ id:fd.get('id').trim(), name:fd.get('name').trim(), capacity:parseInt(fd.get('capacity'))||null, notes:fd.get('notes').trim(), updatedAt: new Date().toISOString() };
    try{
      const isEdit = document.querySelector('#form-room [name=id]').hasAttribute('data-edit');
      if(isEdit){
        const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
        if(!(s.role==='admin' || s.role==='teacher')) return alert('Bạn không có quyền sửa phòng');
        await dataLayer.updateRoom(obj.id, obj);
      } else {
        if(!requireRole(['admin','teacher'])) return alert('Bạn không có quyền thêm phòng');
        await dataLayer.addRoom(obj);
      }
      modalRoom.hide();
      await renderAll();
      toast('Lưu phòng thành công');
    }catch(err){ alert(err.message); }
  });

  // asset image preview
  document.querySelector('#form-asset [name=image]').addEventListener('change', e=> {
    const f = e.target.files && e.target.files[0]; const pre = document.getElementById('asset-preview');
    if(!f){ pre.style.display='none'; pre.src=''; return; }
    const r = new FileReader(); r.onload = ()=> { pre.src = r.result; pre.style.display = 'block'; }; r.readAsDataURL(f);
  });

  document.getElementById('gen-asset-id').addEventListener('click', ()=> document.querySelector('#form-asset [name=id]').value = genId('AS-'));

  document.getElementById('form-asset').addEventListener('submit', async e=>{
    e.preventDefault();
    const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
    const fd=new FormData(e.target);
    const id = (fd.get('id')||'').trim() || genId('AS-');
    const obj = { id, name:fd.get('name').trim(), serial:fd.get('serial').trim(), category:fd.get('category').trim(), room:fd.get('room').trim(), status:fd.get('status'), purchased:fd.get('purchased')||'', price: Number(fd.get('price')||0), vendor:fd.get('vendor')||'', warranty: Number(fd.get('warranty')||0), desc:fd.get('desc')||'', updatedAt: new Date().toISOString() };
    const imgFile = e.target.querySelector('[name=image]').files[0];
    if(imgFile){ obj.image = await fileToDataURL(imgFile); } else { obj.image = obj.image || ''; }
    try{
      const existing = (await dataLayer.getAssets()).find(x=>x.id===id);
      if(existing){
        if(!requireRole(['admin','technician'])) return alert('Bạn không có quyền chỉnh sửa tài sản');
        await dataLayer.updateAsset(id,obj);
      } else {
        if(!requireRole(['admin','technician'])) return alert('Bạn không có quyền thêm tài sản');
        await dataLayer.addAsset(obj);
      }
      modalAsset.hide(); await renderAll(); toast('Lưu tài sản thành công');
    } catch(err){ alert(err.message); }
  });

  document.getElementById('form-inv').addEventListener('submit', async e=>{ e.preventDefault();
    const fd=new FormData(e.target);
    const obj={ id:fd.get('id').trim(), name:fd.get('name').trim(), qty:parseInt(fd.get('qty'))||0, unit:fd.get('unit').trim(), updatedAt: new Date().toISOString() };
    try{
      const existing = (await dataLayer.getInventory()).find(x=>x.id===obj.id);
      if(existing){
        if(!requireRole(['admin','technician'])) return alert('Bạn không có quyền sửa vật tư');
        await dataLayer.updateInv(obj.id,obj);
      } else {
        if(!requireRole(['admin','technician'])) return alert('Bạn không có quyền thêm vật tư');
        await dataLayer.addInv(obj);
      }
      modalInv.hide(); await renderAll(); toast('Lưu vật tư thành công');
    }catch(err){ alert(err.message);} 
  });

  document.getElementById('form-req').addEventListener('submit', async e=>{ e.preventDefault();
    const fd=new FormData(e.target); const obj={ id:fd.get('id').trim(), title:fd.get('title').trim(), room:fd.get('room').trim(), reporter:fd.get('reporter').trim(), desc:fd.get('desc').trim(), status:fd.get('status'), updatedAt: new Date().toISOString() };
    try{
      const existing = (await dataLayer.getRequests()).find(x=>x.id===obj.id);
      const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
      if(existing){
        if(!(s.role==='admin' || s.username === (existing.reporter && existing.reporter.toLowerCase()) || s.role==='technician')) return alert('Bạn không có quyền sửa yêu cầu này');
        await dataLayer.updateRequest(obj.id,obj);
      } else {
        obj.reporter = obj.reporter || s.username || '';
        obj.created = new Date().toISOString();
        await dataLayer.addRequest(obj);
      }
      modalReq.hide(); await renderAll(); toast('Lưu yêu cầu thành công');
    }catch(err){ alert(err.message);} 
  });

  // Open modals
  document.getElementById('btn-add-room').addEventListener('click', ()=> { document.getElementById('form-room').reset(); const idEl = document.querySelector('#form-room [name=id]'); idEl.removeAttribute('data-edit'); idEl.removeAttribute('readonly'); modalRoom.show(); });
  document.getElementById('btn-add-asset').addEventListener('click', ()=> { if(!requireRole(['admin','technician'])){ alert('Bạn cần đăng nhập với quyền admin hoặc technician.'); return; } document.getElementById('form-asset').reset(); document.querySelector('#form-asset [name=id]').removeAttribute('readonly'); document.getElementById('asset-preview').style.display='none'; modalAsset.show(); });
  document.getElementById('btn-add-request').addEventListener('click', ()=> { if(!requireRole(['admin','teacher','technician'])){ alert('Bạn cần đăng nhập.'); return; } document.getElementById('form-req').reset(); document.querySelector('#form-req [name=id]').removeAttribute('readonly'); modalReq.show(); });

  // search / filters
  document.getElementById('search-rooms').addEventListener('input', debounce(()=>renderRooms(),200));
  document.getElementById('search-assets').addEventListener('input', debounce(()=>renderAssets(),200));
  document.getElementById('filter-asset-status').addEventListener('change', ()=>renderAssets());
  document.getElementById('search-inv').addEventListener('input', debounce(()=>renderInv(),200));
  document.getElementById('search-req').addEventListener('input', debounce(()=>renderReqs(),200));
  document.getElementById('filter-req-status').addEventListener('change', ()=>renderReqs());

  document.getElementById('btn-export').addEventListener('click', async ()=> {
    const db = await dataLayer.getAll();
    const blob=new Blob([JSON.stringify(db,null,2)],{type:'application/json'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download='hanthuyen_export.json'; a.click();
  });
  document.getElementById('btn-import').addEventListener('click', ()=> document.getElementById('file-input').click());
  document.getElementById('file-input').addEventListener('change', async e=>{ if(!e.target.files.length) return; try{ const text = await e.target.files[0].text(); const data = JSON.parse(text); // basic merge into local or cloud
    if(typeof dataLayer.import === 'function'){ await dataLayer.import(e.target.files[0]); alert('Import thành công'); await renderAll(); } else {
      const db = loadDBLocal(); db.rooms = data.rooms || db.rooms; db.assets = data.assets || db.assets; db.inventory = data.inventory || db.inventory; db.requests = data.requests || db.requests; saveDBLocal(db); alert('Import (local) thành công'); await renderAll();
    }
  } catch(err){ alert('Import lỗi: '+err.message);} e.target.value=''; });

  document.getElementById('seed-data').addEventListener('click', async ()=> { if(confirm('Tạo dữ liệu mẫu (ghi đè)?')){ await dataLayer.seed(); await renderAll(); } });

  // helper: convert file to dataURL
  function fileToDataURL(file){ return new Promise((res,rej)=>{ const r=new FileReader(); r.onload=()=>res(r.result); r.onerror=()=>rej('File read error'); r.readAsDataURL(file); }); }

  // utils
  function genId(prefix='X'){ const n = Math.floor(Math.random()*900+100); return prefix + Date.now().toString().slice(-6) + n; }
  function debounce(fn, ms=200){ let t; return (...args)=>{ clearTimeout(t); t=setTimeout(()=>fn(...args), ms); }; }

  // Chart
  let chart = null;
  async function renderChart(){
    const ctx = document.getElementById('chartStatus').getContext('2d');
    const reqs = await dataLayer.getRequests();
    const statusCounts = { open:0, in_progress:0, done:0 }; (reqs||[]).forEach(r=> statusCounts[r.status] = (statusCounts[r.status]||0)+1 );
    const data = [statusCounts.open, statusCounts.in_progress, statusCounts.done];
    if(chart) chart.destroy();
    chart = new Chart(ctx, { type:'doughnut', data:{ labels:['Mới','Đang xử lý','Hoàn tất'], datasets:[{ data, backgroundColor:['#ffc107','#0d6efd','#198754'] }] }, options:{responsive:true, plugins:{legend:{position:'bottom'}}} });
  }

  // helper edit openers (room/asset/inv/req)
  async function openRoomEdit(id){
    const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
    if(!requireRole(['admin','teacher'])) return alert('Bạn không có quyền sửa phòng');
    const rows = await dataLayer.getRooms(); const r = rows.find(x=>x.id===id); if(!r) return alert('Không tìm thấy');
    document.getElementById('form-room').reset();
    const idEl = document.querySelector('#form-room [name=id]');
    idEl.value = r.id; idEl.setAttribute('data-edit','1'); idEl.setAttribute('readonly','readonly');
    document.querySelector('#form-room [name=name]').value = r.name || '';
    document.querySelector('#form-room [name=capacity]').value = r.capacity || '';
    document.querySelector('#form-room [name=notes]').value = r.notes || '';
    const isAdmin = (getSession() && getSession().role==='admin');
    document.querySelector('#form-room [name=name]').readOnly = false;
    document.querySelector('#form-room [name=capacity]').readOnly = false;
    document.querySelector('#form-room [name=notes]').readOnly = false;
    modalRoom.show();
  }

  async function editAsset(id){
    const arr = await dataLayer.getAssets(); const a = arr.find(x=>x.id===id); if(!a) return alert('Không tìm thấy');
    const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
    if(!(s.role==='admin' || s.role==='technician')) return alert('Bạn không có quyền chỉnh sửa');
    document.querySelector('#form-asset [name=id]').value = a.id; document.querySelector('#form-asset [name=id]').setAttribute('readonly','readonly');
    document.querySelector('#form-asset [name=name]').value = a.name; document.querySelector('#form-asset [name=serial]').value = a.serial||'';
    document.querySelector('#form-asset [name=category]').value = a.category||''; document.querySelector('#form-asset [name=room]').value = a.room||'';
    document.querySelector('#form-asset [name=status]').value = a.status||'normal'; document.querySelector('#form-asset [name=purchased]').value = a.purchased||'';
    document.querySelector('#form-asset [name=price]').value = a.price||''; document.querySelector('#form-asset [name=vendor]').value = a.vendor||'';
    document.querySelector('#form-asset [name=warranty]').value = a.warranty||''; document.querySelector('#form-asset [name=desc]').value = a.desc||'';
    if(a.image){ const pre = document.getElementById('asset-preview'); pre.src = a.image; pre.style.display='block'; } else { document.getElementById('asset-preview').style.display='none'; }
    modalAsset.show();
  }

  async function editInv(id){
    const rows = await dataLayer.getInventory(); const it = rows.find(x=>x.id===id); if(!it) return alert('Không tìm thấy');
    const s = getSession(); if(!s) return alert('Bạn cần đăng nhập'); if(!requireRole(['admin','technician'])) return alert('Bạn không có quyền sửa vật tư');
    document.getElementById('form-inv').reset();
    document.querySelector('#form-inv [name=id]').value = it.id; document.querySelector('#form-inv [name=id]').setAttribute('readonly','readonly');
    document.querySelector('#form-inv [name=name]').value = it.name; document.querySelector('#form-inv [name=qty]').value = it.qty; document.querySelector('#form-inv [name=unit]').value = it.unit;
    modalInv.show();
  }

  async function editReq(id){
    const rows = await dataLayer.getRequests(); const r = rows.find(x=>x.id===id); if(!r) return alert('Không tìm thấy');
    const s = getSession(); if(!s) return alert('Bạn cần đăng nhập');
    if(!(s.role==='admin' || s.role==='technician' || s.username === (r.reporter && r.reporter.toLowerCase()))) return alert('Bạn không có quyền sửa yêu cầu này');
    document.getElementById('form-req').reset();
    document.querySelector('#form-req [name=id]').value = r.id; document.querySelector('#form-req [name=id]').setAttribute('readonly','readonly');
    document.querySelector('#form-req [name=title]').value = r.title; document.querySelector('#form-req [name=room]').value = r.room; document.querySelector('#form-req [name=reporter]').value = r.reporter;
    document.querySelector('#form-req [name=desc]').value = r.desc; document.querySelector('#form-req [name=status]').value = r.status;
    modalReq.show();
  }

  // initial render
  (async ()=>{ try{ const db = await dataLayer.getAll(); if(!db || !(db.rooms && db.assets)) await dataLayer.seed(); renderCurrentUser(); await renderAll(); } catch(err){ console.error(err); alert('Lỗi khởi tạo: '+err.message); } })();

  </script>
  <!-- Full app JS END -->

  <!-- ========== FIREBASE SYNC: add below & the modal will call startRealtimeSync with pasted config ========== -->
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>

  <script>
  // startRealtimeSync function (same as provided earlier). It will be called with the config the user pastes.
  let cloudSyncHandle = null;

  function startRealtimeSync(firebaseConfig){
    if(!firebaseConfig || !firebaseConfig.apiKey){ console.warn('Firebase config missing or invalid, skip sync'); alert('Firebase config không hợp lệ'); return; }
    try{ if(!firebase.apps.length) firebase.initializeApp(firebaseConfig); } catch(e){ console.error('Firebase init', e); }
    const db = firebase.firestore();
    function loadLocal(){ try{ return JSON.parse(localStorage.getItem(DB_KEY)) || {rooms:[],assets:[],inventory:[],requests:[],users:{}} }catch(e){ return {rooms:[],assets:[],inventory:[],requests:[],users:{}} } }
    function saveLocal(obj){ localStorage.setItem(DB_KEY, JSON.stringify(obj)); }
    function mergeCollection(collName, docs){
      const local = loadLocal();
      const map = {};
      (local[collName] || []).forEach(it => map[it.id] = it);
      docs.forEach(d => {
        const t = d.updatedAt ? new Date(d.updatedAt).getTime() : 0;
        const ex = map[d.id];
        const exT = ex && ex.updatedAt ? new Date(ex.updatedAt).getTime() : 0;
        if(!ex || t >= exT) map[d.id] = d;
      });
      local[collName] = Object.values(map).sort((a,b)=> {
        const ta = a.updatedAt? new Date(a.updatedAt).getTime():0;
        const tb = b.updatedAt? new Date(b.updatedAt).getTime():0;
        return tb - ta;
      });
      saveLocal(local);
    }
    const collections = ['rooms','assets','inventory','requests','users'];
    const unsub = [];
    collections.forEach(coll => {
      const u = db.collection(coll).onSnapshot(snapshot => {
        const docs = [];
        snapshot.forEach(snap => {
          const data = snap.data(); data.id = data.id || snap.id;
          docs.push(data);
        });
        mergeCollection(coll === 'users' ? 'users' : coll, docs);
        if(typeof renderAll === 'function') renderAll();
      }, err => console.error('snapshot err', coll, err));
      unsub.push(u);
    });

    const cloudLayer = {
      async getAll(){ const raw = JSON.parse(localStorage.getItem(DB_KEY)) || {}; return raw; },
      async seed(){ return true; },
      async getRooms(){ return loadLocal().rooms || []; },
      async addRoom(r){ r.updatedAt = new Date().toISOString(); await db.collection('rooms').doc(r.id).set(r); return r; },
      async updateRoom(id,patch){ patch.updatedAt = new Date().toISOString(); await db.collection('rooms').doc(id).set(Object.assign({}, patch, {id}), {merge:true}); return (await db.collection('rooms').doc(id).get()).data(); },
      async deleteRoom(id){ await db.collection('rooms').doc(id).delete(); const local = loadLocal(); local.rooms = (local.rooms||[]).filter(x=>x.id!==id); saveLocal(local); return {ok:true}; },
      async getAssets(){ return loadLocal().assets || []; },
      async addAsset(a){ a.updatedAt = new Date().toISOString(); await db.collection('assets').doc(a.id).set(a); return a; },
      async updateAsset(id,patch){ patch.updatedAt = new Date().toISOString(); await db.collection('assets').doc(id).set(Object.assign({}, patch, {id}), {merge:true}); return (await db.collection('assets').doc(id).get()).data(); },
      async deleteAsset(id){ await db.collection('assets').doc(id).delete(); const local = loadLocal(); local.assets = (local.assets||[]).filter(x=>x.id!==id); saveLocal(local); return {ok:true}; },
      async getInventory(){ return loadLocal().inventory || []; },
      async addInv(it){ it.updatedAt = new Date().toISOString(); await db.collection('inventory').doc(it.id).set(it); return it; },
      async updateInv(id,patch){ patch.updatedAt = new Date().toISOString(); await db.collection('inventory').doc(id).set(Object.assign({}, patch, {id}), {merge:true}); return (await db.collection('inventory').doc(id).get()).data(); },
      async deleteInv(id){ await db.collection('inventory').doc(id).delete(); const local = loadLocal(); local.inventory = (local.inventory||[]).filter(x=>x.id!==id); saveLocal(local); return {ok:true}; },
      async getRequests(){ return loadLocal().requests || []; },
      async addRequest(r){ r.updatedAt = new Date().toISOString(); r.created = r.created || new Date().toISOString(); await db.collection('requests').doc(r.id).set(r); return r; },
      async updateRequest(id,patch){ patch.updatedAt = new Date().toISOString(); await db.collection('requests').doc(id).set(Object.assign({}, patch, {id}), {merge:true}); return (await db.collection('requests').doc(id).get()).data(); },
      async deleteRequest(id){ await db.collection('requests').doc(id).delete(); const local = loadLocal(); local.requests = (local.requests||[]).filter(x=>x.id!==id); saveLocal(local); return {ok:true}; },
      async getUsers(){ return loadLocal().users || {}; },
      async addUser(u){ await db.collection('users').doc(u.username).set(Object.assign({}, u, {updatedAt: new Date().toISOString()}), {merge:true}); return u; },
      async import(file){ const text = await file.text(); const data = JSON.parse(text); const promises = []; ['rooms','assets','inventory','requests'].forEach(coll => { (data[coll]||[]).forEach(it => { promises.push(db.collection(coll).doc(it.id).set(Object.assign({}, it, {updatedAt: it.updatedAt || new Date().toISOString()}), {merge:true})); }); }); await Promise.all(promises); return {ok:true}; }
    };

    dataLayer = cloudLayer;
    usersLayer = {
      register: async (username,password,role,name) => {
        const user = await registerUserLocal(username,password,role,name);
        await cloudLayer.addUser({ username: user.username, name: user.name, role: user.role, created: user.created });
        return user;
      },
      verify: verifyUserLocal,
      addCloud: async (u)=> cloudLayer.addUser(u),
      verifyCloud: null
    };

    cloudSyncHandle = { unsubscribeAll: ()=> unsub.forEach(fn=>fn()) };
    toast('Realtime sync đã kết nối (Firestore).');
    return cloudSyncHandle;
  }

  // Wire the "Bật đồng bộ" modal button
  document.getElementById('btn-open-sync').addEventListener('click', ()=> modalSync.show());
  document.getElementById('btn-start-sync').addEventListener('click', ()=> {
    const text = document.getElementById('firebase-config-input').value.trim();
    if(!text) return alert('Vui lòng dán firebaseConfig JSON');
    let cfg = null;
    try { cfg = JSON.parse(text); } catch(e){ alert('JSON không hợp lệ'); return; }
    try {
      startRealtimeSync(cfg);
      modalSync.hide();
    } catch(e){ alert('Không thể kết nối: '+e.message); console.error(e); }
  });

  </script>

</body>
</html>
