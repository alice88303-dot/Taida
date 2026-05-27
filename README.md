<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能選位系統 - 黑板排版版</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 1600px;
            margin: 0 auto;
        }

        .card {
            background: white;
            border-radius: 12px;
            box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
            padding: 30px;
            margin-bottom: 20px;
        }

        button {
            padding: 10px 20px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.3s ease;
            font-weight: 600;
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
        }

        .btn-primary {
            background: #2563eb;
            color: white;
        }

        .btn-primary:hover {
            background: #1d4ed8;
        }

        .btn-danger {
            background: #ef4444;
            color: white;
        }

        .btn-danger:hover {
            background: #dc2626;
        }

        .btn-secondary {
            background: #e5e7eb;
            color: #374151;
        }

        .btn-secondary:hover {
            background: #d1d5db;
        }

        /* ============ 登入頁面 ============ */
        .login-page {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .login-card {
            background: white;
            border-radius: 16px;
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.12);
            padding: 50px;
            width: 100%;
            max-width: 420px;
            animation: slideUp 0.6s ease-out;
        }

        @keyframes slideUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .login-title {
            text-align: center;
            margin-bottom: 10px;
        }

        .login-title h1 {
            font-size: 32px;
            color: #1f2937;
            margin-bottom: 5px;
        }

        .login-title p {
            color: #9ca3af;
            font-size: 14px;
        }

        .form-group {
            margin-bottom: 20px;
        }

        .form-group label {
            display: block;
            margin-bottom: 8px;
            color: #374151;
            font-weight: 600;
            font-size: 14px;
        }

        .form-group input {
            width: 100%;
            padding: 12px;
            border: 2px solid #e5e7eb;
            border-radius: 8px;
            font-size: 14px;
            transition: border-color 0.3s;
        }

        .form-group input:focus {
            outline: none;
            border-color: #2563eb;
            box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
        }

        .login-button {
            width: 100%;
            padding: 12px;
            background: #2563eb;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            margin-top: 10px;
        }

        .login-button:hover {
            background: #1d4ed8;
            transform: translateY(-2px);
        }

        .error-message {
            color: #ef4444;
            font-size: 13px;
            margin-top: 5px;
            display: none;
        }

        .mode-switch {
            text-align: center;
            margin-top: 20px;
            padding-top: 20px;
            border-top: 1px solid #e5e7eb;
        }

        .mode-switch button {
            font-size: 12px;
            margin: 5px;
        }

        /* ============ 選位頁面 ============ */
        .selection-page {
            display: none;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 30px;
        }

        .header h1 {
            font-size: 28px;
            color: #1f2937;
        }

        .user-info {
            text-align: right;
            font-size: 14px;
            color: #6b7280;
        }

        .user-info .username {
            font-weight: 600;
            color: #1f2937;
            display: block;
            margin-bottom: 5px;
        }

        /* ============ 黑板排版 ============ */
        .blackboard-title {
            background: #22c55e;
            color: white;
            padding: 15px;
            text-align: center;
            border-radius: 8px;
            font-size: 24px;
            font-weight: 700;
            margin-bottom: 20px;
        }

        .blackboard-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 20px;
        }

        .area-section {
            border: 3px solid #333;
            border-radius: 8px;
            padding: 20px;
            background: white;
        }

        .area-section.top {
            grid-column: auto;
        }

        .area-section.bottom {
            grid-column: auto;
        }

        .area-title {
            font-size: 20px;
            font-weight: 700;
            text-align: center;
            margin-bottom: 15px;
            padding: 10px;
            border-radius: 4px;
        }

        .area-a .area-title {
            background: #e0f2fe;
            color: #0369a1;
        }

        .area-b .area-title {
            background: #f0fdf4;
            color: #166534;
        }

        .area-c .area-title {
            background: #fef3c7;
            color: #b45309;
        }

        .area-d .area-title {
            background: #fce7f3;
            color: #9f1239;
        }

        .seat-table {
            width: 100%;
            border-collapse: collapse;
        }

        .seat-table td {
            width: 20%;
            aspect-ratio: 1;
            border: 2px solid #e5e7eb;
            text-align: center;
            vertical-align: middle;
            cursor: pointer;
            transition: all 0.2s ease;
            font-weight: 600;
            font-size: 14px;
        }

        .seat-table td:hover:not(.selected):not(.occupied) {
            background: #eff6ff;
            border-color: #2563eb;
            transform: scale(0.95);
        }

        .seat-table td.selected {
            background: #2563eb;
            color: white;
            border-color: #2563eb;
            box-shadow: inset 0 0 0 3px rgba(255, 255, 255, 0.2);
        }

        .seat-table td.occupied {
            background: #f3f4f6;
            color: #9ca3af;
            border-color: #d1d5db;
            cursor: not-allowed;
        }

        .seat-table td.available {
            background: white;
            color: #1f2937;
        }

        .legend {
            display: flex;
            gap: 20px;
            margin: 20px 0;
            padding: 15px;
            background: #f9fafb;
            border-radius: 8px;
            flex-wrap: wrap;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 14px;
        }

        .legend-color {
            width: 20px;
            height: 20px;
            border-radius: 4px;
            border: 2px solid #e5e7eb;
        }

        .action-buttons {
            display: flex;
            gap: 10px;
            margin-top: 30px;
        }

        .action-buttons button {
            flex: 1;
        }

        /* ============ 老師後台 ============ */
        .admin-page {
            display: none;
        }

        .admin-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 30px;
            flex-wrap: wrap;
            gap: 20px;
        }

        .admin-header h1 {
            font-size: 28px;
            color: #1f2937;
        }

        .admin-info {
            display: flex;
            gap: 15px;
            align-items: center;
            flex-wrap: wrap;
        }

        .sync-indicator {
            display: inline-block;
            width: 8px;
            height: 8px;
            background: #22c55e;
            border-radius: 50%;
            margin-right: 6px;
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }

        .tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            border-bottom: 2px solid #e5e7eb;
            flex-wrap: wrap;
        }

        .tab-button {
            padding: 12px 20px;
            background: none;
            border: none;
            border-bottom: 3px solid transparent;
            cursor: pointer;
            font-weight: 600;
            color: #9ca3af;
            transition: all 0.3s;
        }

        .tab-button.active {
            color: #2563eb;
            border-bottom-color: #2563eb;
        }

        .tab-content {
            display: none;
        }

        .tab-content.active {
            display: block;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 30px;
        }

        .stat-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 12px;
            text-align: center;
        }

        .stat-number {
            font-size: 32px;
            font-weight: 700;
            margin-bottom: 5px;
        }

        .stat-label {
            font-size: 14px;
            opacity: 0.9;
        }

        .students-table {
            width: 100%;
            border-collapse: collapse;
            background: white;
            border-radius: 8px;
            overflow: hidden;
        }

        .students-table thead {
            background: #f9fafb;
        }

        .students-table th,
        .students-table td {
            padding: 12px 16px;
            text-align: left;
            font-size: 14px;
            border-bottom: 1px solid #e5e7eb;
        }

        .students-table th {
            font-weight: 600;
            color: #374151;
        }

        .students-table tbody tr:hover {
            background: #f9fafb;
        }

        .seat-badge {
            background: #dbeafe;
            color: #1e40af;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            font-weight: 600;
        }

        .action-icons {
            display: flex;
            gap: 8px;
        }

        .icon-btn {
            padding: 6px 10px;
            font-size: 12px;
            border-radius: 4px;
            border: none;
            cursor: pointer;
        }

        .icon-btn.delete {
            background: #fee2e2;
            color: #991b1b;
            transition: all 0.3s;
        }

        .icon-btn.delete:hover {
            background: #fca5a5;
        }

        .upload-section {
            background: #f0fdf4;
            border: 2px dashed #86efac;
            border-radius: 8px;
            padding: 30px;
            text-align: center;
            margin-bottom: 20px;
        }

        textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #e5e7eb;
            border-radius: 6px;
            font-family: monospace;
            font-size: 13px;
            resize: vertical;
            min-height: 150px;
            margin: 15px 0;
        }

        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.5);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .modal.active {
            display: flex;
        }

        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 12px;
            max-width: 500px;
            text-align: center;
            animation: slideUp 0.3s ease-out;
        }

        .modal-content h2 {
            margin-bottom: 15px;
            color: #1f2937;
        }

        .modal-content p {
            color: #6b7280;
            margin-bottom: 20px;
        }

        .modal-buttons {
            display: flex;
            gap: 10px;
            justify-content: center;
        }

        .modal-buttons button {
            flex: 1;
        }

        .success-message {
            background: #dcfce7;
            border: 1px solid #86efac;
            color: #166534;
            padding: 12px 16px;
            border-radius: 6px;
            margin-bottom: 15px;
            display: none;
        }

        .success-message.active {
            display: block;
        }

        @media (max-width: 768px) {
            .card {
                padding: 20px;
            }

            .login-card {
                padding: 30px 20px;
            }

            .blackboard-container {
                grid-template-columns: 1fr;
            }

            .seat-table td {
                font-size: 12px;
                padding: 8px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- ============ 登入頁面 ============ -->
        <div id="loginPage" class="login-page">
            <div class="login-card">
                <div class="login-title">
                    <h1>🎓 選位</h1>
                    <p>請輸入姓名和通行碼開始選位</p>
                </div>
                <form onsubmit="handleLogin(event)">
                    <div class="form-group">
                        <label for="studentName">姓名</label>
                        <input 
                            type="text" 
                            id="studentName" 
                            placeholder="輸入你的名字"
                            required
                            autocomplete="off"
                        >
                    </div>
                    <div class="form-group">
                        <label for="passCode">通行碼</label>
                        <input 
                            type="text" 
                            id="passCode" 
                            placeholder="輸入通行碼"
                            required
                            autocomplete="off"
                        >
                        <div class="error-message" id="errorMessage"></div>
                    </div>
                    <button type="submit" class="login-button">開始選位</button>
                </form>
                <div class="mode-switch">
                    <button type="button" class="btn-secondary" onclick="switchToAdmin()">老師後台</button>
                </div>
            </div>
        </div>

        <!-- ============ 選位頁面 ============ -->
        <div id="selectionPage" class="selection-page">
            <div class="header">
                <h1>🎓 選位系統</h1>
                <div class="user-info">
                    <span class="username" id="displayName"></span>
                    <small id="seatStatus"></small>
                </div>
            </div>

            <div class="card">
                <div class="legend">
                    <div class="legend-item">
                        <div class="legend-color" style="background: white;"></div>
                        <span>可選擇</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #2563eb;"></div>
                        <span>你的座位</span>
                    </div>
                    <div class="legend-item">
                        <div class="legend-color" style="background: #f3f4f6;"></div>
                        <span>已被選擇</span>
                    </div>
                </div>

                <div class="blackboard-title">🎓 黑板</div>

                <div class="blackboard-container">
                    <!-- A區 -->
                    <div class="area-section area-a top">
                        <div class="area-title">A區</div>
                        <table class="seat-table" id="areaA"></table>
                    </div>

                    <!-- B區 -->
                    <div class="area-section area-b top">
                        <div class="area-title">B區</div>
                        <table class="seat-table" id="areaB"></table>
                    </div>

                    <!-- C區 -->
                    <div class="area-section area-c bottom">
                        <div class="area-title">C區</div>
                        <table class="seat-table" id="areaC"></table>
                    </div>

                    <!-- D區 -->
                    <div class="area-section area-d bottom">
                        <div class="area-title">D區</div>
                        <table class="seat-table" id="areaD"></table>
                    </div>
                </div>

                <div class="action-buttons">
                    <button type="button" class="btn-primary" onclick="confirmSelection()" id="confirmBtn" disabled>
                        確認選位
                    </button>
                    <button type="button" class="btn-secondary" onclick="logout()">登出</button>
                </div>
            </div>
        </div>

        <!-- ============ 老師後台 ============ -->
        <div id="adminPage" class="admin-page">
            <div class="admin-header">
                <h1>📊 老師後台</h1>
                <div class="admin-info">
                    <span class="sync-indicator"></span>
                    <span style="color: #6b7280; font-size: 14px;">實時同步中</span>
                    <button type="button" class="btn-secondary" onclick="logout()" style="margin-left: auto;">登出</button>
                </div>
            </div>

            <div class="tabs">
                <button class="tab-button active" onclick="switchAdminTab('students')">👥 學生名單</button>
                <button class="tab-button" onclick="switchAdminTab('overview')">📊 選位進度</button>
                <button class="tab-button" onclick="switchAdminTab('export')">💾 資料匯出</button>
                <button class="tab-button" onclick="switchAdminTab('settings')">⚙️ 設定</button>
            </div>

            <!-- 學生名單標籤 -->
            <div id="studentsTab" class="tab-content active">
                <div class="card">
                    <div class="success-message" id="successMessage"></div>
                    
                    <h2 style="margin-bottom: 20px; color: #1f2937;">📋 學生名單管理</h2>
                    
                    <div class="upload-section">
                        <h3 style="color: #166534; margin-bottom: 15px;">📝 添加學生名單</h3>
                        <p style="color: #6b7280; margin-bottom: 15px; font-size: 14px;">貼入新學生名單（每行一個），系統將自動追加到現有學生之後</p>
                        <textarea id="nameList" placeholder="新學生名字&#10;另一個新學生&#10;..."></textarea>
                        <button type="button" class="btn-primary" onclick="importStudents()">✅ 添加新學生</button>
                    </div>

                    <h3 style="margin-bottom: 15px; color: #1f2937;">📝 當前學生清單</h3>
                    <div style="overflow-x: auto;">
                        <table class="students-table">
                            <thead>
                                <tr>
                                    <th>序號</th>
                                    <th>姓名</th>
                                    <th>通行碼</th>
                                    <th>座位</th>
                                    <th>狀態</th>
                                    <th>操作</th>
                                </tr>
                            </thead>
                            <tbody id="studentsList">
                                <tr>
                                    <td colspan="6" style="text-align: center; color: #9ca3af;">尚無學生</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <div style="margin-top: 20px; text-align: right;">
                        <button type="button" class="btn-danger" onclick="clearAllStudents()">🗑️ 清除所有學生</button>
                    </div>
                </div>
            </div>

            <!-- 選位進度標籤 -->
            <div id="overviewTab" class="tab-content">
                <div class="card">
                    <div class="stats-grid" id="statsGrid"></div>
                </div>
            </div>

            <!-- 資料匯出標籤 -->
            <div id="exportTab" class="tab-content">
                <div class="card">
                    <h2 style="margin-bottom: 20px; color: #1f2937;">💾 資料匯出</h2>
                    
                    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 30px;">
                        <button type="button" class="btn-primary" onclick="exportStudentList()" style="padding: 15px;">📥 匯出學生名單</button>
                        <button type="button" class="btn-primary" onclick="exportResults()" style="padding: 15px;">📥 匯出選位結果</button>
                        <button type="button" class="btn-primary" onclick="exportJSON()" style="padding: 15px;">📥 完整備份</button>
                        <button type="button" class="btn-primary" onclick="printResults()" style="padding: 15px;">🖨️ 列印座位表</button>
                    </div>

                    <h3 style="margin: 20px 0; color: #1f2937;">📋 預覽</h3>
                    <div id="previewTable"></div>
                </div>
            </div>

            <!-- 設定標籤 -->
            <div id="settingsTab" class="tab-content">
                <div class="card">
                    <h2 style="margin-bottom: 30px; color: #1f2937;">⚙️ 系統設定</h2>
                    
                    <div style="background: #f0fdf4; border: 2px solid #86efac; border-radius: 8px; padding: 20px;">
                        <h3 style="color: #166534; margin-bottom: 15px;">🔐 修改老師密碼</h3>
                        <p style="color: #6b7280; margin-bottom: 15px; font-size: 14px;">設定一個只有你知道的新密碼</p>
                        
                        <div style="display: grid; gap: 15px;">
                            <div>
                                <label style="display: block; margin-bottom: 8px; color: #374151; font-weight: 600;">目前密碼</label>
                                <input type="password" id="currentPassword" placeholder="輸入目前的密碼" style="width: 100%; padding: 12px; border: 2px solid #e5e7eb; border-radius: 8px;">
                            </div>
                            
                            <div>
                                <label style="display: block; margin-bottom: 8px; color: #374151; font-weight: 600;">新密碼</label>
                                <input type="password" id="newPassword" placeholder="輸入新密碼（至少 4 個字符）" style="width: 100%; padding: 12px; border: 2px solid #e5e7eb; border-radius: 8px;">
                            </div>
                            
                            <div>
                                <label style="display: block; margin-bottom: 8px; color: #374151; font-weight: 600;">確認新密碼</label>
                                <input type="password" id="confirmPassword" placeholder="再輸入一次新密碼" style="width: 100%; padding: 12px; border: 2px solid #e5e7eb; border-radius: 8px;">
                            </div>
                            
                            <button type="button" class="btn-primary" onclick="changePassword()" style="margin-top: 10px;">✅ 修改密碼</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- 確認刪除 Modal -->
    <div id="deleteModal" class="modal">
        <div class="modal-content">
            <h2>⚠️ 確認刪除</h2>
            <p id="deleteMessage"></p>
            <div class="modal-buttons">
                <button type="button" class="btn-secondary" onclick="closeDeleteModal()">取消</button>
                <button type="button" class="btn-danger" onclick="confirmDelete()">確認刪除</button>
            </div>
        </div>
    </div>

    <script>
        // ============ 資料管理 ============
        const appState = {
            currentPage: 'login',
            currentUser: null,
            selectedSeat: null,
            students: {},
            selections: {},
            adminPassword: 'teacher2024',
            syncInterval: 2000
        };

        let deleteTargetCode = null;

        // 載入資料
        function loadData() {
            const saved = localStorage.getItem('seatSelectionData');
            if (saved) {
                try {
                    const loaded = JSON.parse(saved);
                    appState.students = loaded.students || {};
                    appState.selections = loaded.selections || {};
                    appState.adminPassword = loaded.adminPassword || 'teacher2024';
                } catch(e) {
                    console.error('載入資料失敗:', e);
                }
            }
        }

        // 儲存資料
        function saveData() {
            try {
                localStorage.setItem('seatSelectionData', JSON.stringify({
                    students: appState.students,
                    selections: appState.selections,
                    adminPassword: appState.adminPassword
                }));
            } catch(e) {
                console.error('儲存資料失敗:', e);
                alert('❌ 資料儲存失敗！');
            }
        }

        // ============ 認證功能 ============
        function handleLogin(event) {
            event.preventDefault();
            const name = document.getElementById('studentName').value.trim();
            const code = document.getElementById('passCode').value.trim();
            const errorMsg = document.getElementById('errorMessage');

            if (!appState.students[code] || appState.students[code] !== name) {
                errorMsg.textContent = '姓名或通行碼錯誤，請重新輸入';
                errorMsg.style.display = 'block';
                return;
            }

            errorMsg.style.display = 'none';
            appState.currentUser = { name, code };
            appState.currentPage = 'selection';
            renderSelectionPage();
            startAutoSync();
        }

        function switchToAdmin() {
            const password = prompt('🔐 請輸入老師密碼：');
            if (password === null) return;
            
            if (password !== appState.adminPassword) {
                alert('❌ 密碼錯誤');
                return;
            }
            
            appState.currentPage = 'admin';
            renderAdminPage();
            startAutoSync();
        }

        function logout() {
            appState.currentUser = null;
            appState.selectedSeat = null;
            appState.currentPage = 'login';
            document.getElementById('studentName').value = '';
            document.getElementById('passCode').value = '';
            document.getElementById('errorMessage').style.display = 'none';
            renderLoginPage();
        }

        // ============ 選位功能 ============
        function renderSelectionPage() {
            document.getElementById('loginPage').style.display = 'none';
            document.getElementById('selectionPage').style.display = 'block';
            document.getElementById('adminPage').style.display = 'none';

            const user = appState.currentUser;
            document.getElementById('displayName').textContent = `👋 ${user.name}`;

            if (appState.selections[user.code]) {
                const selected = appState.selections[user.code];
                appState.selectedSeat = selected.seat;
                document.getElementById('seatStatus').textContent = `已選擇：${selected.seat}`;
            } else {
                document.getElementById('seatStatus').textContent = '尚未選擇座位';
            }

            renderSeats();
        }

        function renderSeats() {
            const areas = [
                { id: 'areaA', name: 'A', color: '#e0f2fe' },
                { id: 'areaB', name: 'B', color: '#f0fdf4' },
                { id: 'areaC', name: 'C', color: '#fef3c7' },
                { id: 'areaD', name: 'D', color: '#fce7f3' }
            ];

            areas.forEach(area => {
                const table = document.getElementById(area.id);
                table.innerHTML = '';

                for (let i = 0; i < 6; i++) {
                    const row = document.createElement('tr');
                    for (let j = 0; j < 5; j++) {
                        const seatNum = i * 5 + j + 1;
                        const seatId = `${area.name}-${String(seatNum).padStart(2, '0')}`;
                        
                        const td = document.createElement('td');
                        td.textContent = seatNum;
                        td.className = 'available';

                        const isSelected = appState.selectedSeat === seatId;
                        const isOccupied = Object.values(appState.selections).some(s => 
                            s.seat === seatId && s.name !== appState.currentUser.name
                        );

                        if (isSelected) {
                            td.classList.add('selected');
                        } else if (isOccupied) {
                            td.classList.remove('available');
                            td.classList.add('occupied');
                        } else {
                            td.onclick = () => selectSeat(seatId);
                        }

                        row.appendChild(td);
                    }
                    table.appendChild(row);
                }
            });
        }

        function selectSeat(seatId) {
            appState.selectedSeat = seatId;
            document.getElementById('confirmBtn').disabled = false;
            renderSeats();
        }

        function confirmSelection() {
            if (!appState.selectedSeat) return;

            const user = appState.currentUser;
            appState.selections[user.code] = {
                name: user.name,
                seat: appState.selectedSeat
            };

            document.getElementById('seatStatus').textContent = `已選擇：${appState.selectedSeat}`;
            document.getElementById('confirmBtn').disabled = true;
            saveData();
            renderSeats();
        }

        // ============ 老師後台功能 ============
        function renderAdminPage() {
            document.getElementById('loginPage').style.display = 'none';
            document.getElementById('selectionPage').style.display = 'none';
            document.getElementById('adminPage').style.display = 'block';
            appState.currentPage = 'admin';
            updateAdminPage();
        }

        function updateAdminPage() {
            renderStudentsList();
            updateStats();
            renderPreviewTable();
        }

        function renderStudentsList() {
            const tbody = document.getElementById('studentsList');
            
            const students = Object.entries(appState.students).sort((a, b) => 
                a[0].localeCompare(b[0], undefined, { numeric: true })
            );

            if (students.length === 0) {
                tbody.innerHTML = '<tr><td colspan="6" style="text-align: center; color: #9ca3af;">尚無學生</td></tr>';
                return;
            }

            tbody.innerHTML = '';
            students.forEach(([code, name], idx) => {
                const selection = appState.selections[code];
                const row = document.createElement('tr');

                const statusText = selection ? `✓ 已選 (${selection.seat})` : '待選';
                const statusColor = selection ? '#22c55e' : '#f59e0b';

                row.innerHTML = `
                    <td>${idx + 1}</td>
                    <td>${name}</td>
                    <td><strong>${code}</strong></td>
                    <td>${selection ? selection.seat : '-'}</td>
                    <td><span class="seat-badge" style="background-color: ${statusColor}20; color: ${statusColor};">${statusText}</span></td>
                    <td>
                        <div class="action-icons">
                            ${selection ? `<button type="button" class="icon-btn delete" onclick="showDeleteConfirm('${code}', '${name}', '${selection.seat}')">🗑️ 刪</button>` : '<span style="color: #d1d5db;">-</span>'}
                        </div>
                    </td>
                `;
                tbody.appendChild(row);
            });
        }

        function updateStats() {
            const totalStudents = Object.keys(appState.students).length;
            const selectedCount = Object.keys(appState.selections).length;
            const remainingCount = totalStudents - selectedCount;

            const statsGrid = document.getElementById('statsGrid');
            if (!statsGrid) return;

            statsGrid.innerHTML = `
                <div class="stat-card" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);">
                    <div class="stat-number">${totalStudents}</div>
                    <div class="stat-label">總學生數</div>
                </div>
                <div class="stat-card" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);">
                    <div class="stat-number">${selectedCount}</div>
                    <div class="stat-label">已選位</div>
                </div>
                <div class="stat-card" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);">
                    <div class="stat-number">${remainingCount}</div>
                    <div class="stat-label">待選位</div>
                </div>
            `;
        }

        function importStudents() {
            const nameList = document.getElementById('nameList').value;
            if (!nameList.trim()) {
                alert('請輸入至少一個學生名字');
                return;
            }

            const names = nameList.split('\n').map(n => n.trim()).filter(n => n);
            if (names.length === 0) {
                alert('無效的輸入');
                return;
            }

            let maxIndex = 0;
            Object.keys(appState.students).forEach(code => {
                const match = code.match(/STU(\d+)/);
                if (match) {
                    const num = parseInt(match[1]);
                    if (num > maxIndex) maxIndex = num;
                }
            });

            const addedCount = names.length;
            names.forEach((name, index) => {
                const newIndex = maxIndex + index + 1;
                const code = `STU${String(newIndex).padStart(3, '0')}`;
                appState.students[code] = name;
            });

            saveData();
            document.getElementById('nameList').value = '';
            renderStudentsList();
            updateStats();

            const msg = document.getElementById('successMessage');
            msg.textContent = `✓ 成功添加 ${addedCount} 個新學生。現在共有 ${Object.keys(appState.students).length} 個學生`;
            msg.classList.add('active');
            setTimeout(() => msg.classList.remove('active'), 3000);
        }

        function showDeleteConfirm(code, name, seat) {
            deleteTargetCode = code;
            document.getElementById('deleteMessage').textContent = `確定要刪除 ${name} 的選位（${seat}）嗎？`;
            document.getElementById('deleteModal').classList.add('active');
        }

        function closeDeleteModal() {
            document.getElementById('deleteModal').classList.remove('active');
            deleteTargetCode = null;
        }

        function confirmDelete() {
            if (!deleteTargetCode || !appState.selections[deleteTargetCode]) {
                closeDeleteModal();
                return;
            }

            const selection = appState.selections[deleteTargetCode];
            delete appState.selections[deleteTargetCode];
            saveData();

            const msg = document.getElementById('successMessage');
            msg.textContent = `✓ 已刪除 ${selection.name} 的選位`;
            msg.classList.add('active');
            setTimeout(() => msg.classList.remove('active'), 3000);

            closeDeleteModal();
            renderStudentsList();
            updateStats();
            renderPreviewTable();
        }

        function clearAllStudents() {
            if (confirm('⚠️ 確定要清除所有學生嗎？此操作無法撤銷！')) {
                appState.students = {};
                appState.selections = {};
                saveData();
                renderStudentsList();
                updateStats();
                
                const msg = document.getElementById('successMessage');
                msg.textContent = `✓ 已清除所有學生資料`;
                msg.classList.add('active');
                setTimeout(() => msg.classList.remove('active'), 3000);
            }
        }

        function renderPreviewTable() {
            const container = document.getElementById('previewTable');
            if (Object.keys(appState.students).length === 0) {
                container.innerHTML = '<p style="color: #9ca3af; text-align: center;">尚無學生資料</p>';
                return;
            }

            let html = '<table class="students-table" style="margin-top: 15px;"><thead><tr><th>序號</th><th>姓名</th><th>通行碼</th><th>座位</th><th>狀態</th></tr></thead><tbody>';

            const sortedStudents = Object.entries(appState.students).sort((a, b) => 
                a[0].localeCompare(b[0], undefined, { numeric: true })
            );

            sortedStudents.forEach(([code, name], idx) => {
                const selection = appState.selections[code];
                const status = selection ? '✓ 已選' : '待選';
                const seat = selection ? selection.seat : '-';
                html += `<tr><td>${idx + 1}</td><td>${name}</td><td>${code}</td><td>${seat}</td><td>${status}</td></tr>`;
            });

            html += '</tbody></table>';
            container.innerHTML = html;
        }

        function exportStudentList() {
            const students = Object.entries(appState.students).sort((a, b) => 
                a[0].localeCompare(b[0], undefined, { numeric: true })
            );

            let csv = '\ufeff序號,姓名,通行碼\n';
            students.forEach(([code, name], idx) => {
                csv += `${idx + 1},"${name}","${code}"\n`;
            });

            downloadFile(csv, `student_list_${Date.now()}.csv`, 'text/csv;charset=utf-8');
            alert('✓ 學生名單已下載');
        }

        function exportResults() {
            const students = Object.entries(appState.students).sort((a, b) => 
                a[0].localeCompare(b[0], undefined, { numeric: true })
            );

            let csv = '\ufeff序號,姓名,通行碼,座位,狀態\n';
            students.forEach(([code, name], idx) => {
                const selection = appState.selections[code];
                const seat = selection ? selection.seat : '-';
                const status = selection ? '已選' : '待選';
                csv += `${idx + 1},"${name}","${code}","${seat}","${status}"\n`;
            });

            downloadFile(csv, `seat_results_${Date.now()}.csv`, 'text/csv;charset=utf-8');
            alert('✓ 選位結果已下載');
        }

        function exportJSON() {
            const data = {
                exportTime: new Date().toLocaleString('zh-TW'),
                students: appState.students,
                selections: appState.selections,
                totalStudents: Object.keys(appState.students).length,
                selectedCount: Object.keys(appState.selections).length
            };
            
            downloadFile(JSON.stringify(data, null, 2), `seat_data_${Date.now()}.json`, 'application/json;charset=utf-8');
            alert('✓ 完整備份已下載');
        }

        function printResults() {
            const date = new Date().toLocaleString('zh-TW');
            const students = Object.entries(appState.students).sort((a, b) => 
                a[0].localeCompare(b[0], undefined, { numeric: true })
            );

            let html = `
                <h2 style="text-align: center; margin: 20px 0;">選位系統 - 座位分配表</h2>
                <p style="text-align: center; color: #666; margin-bottom: 30px;">導出時間：${date}</p>
                
                <table style="width: 100%; border-collapse: collapse; margin: 30px 0;">
                    <thead>
                        <tr style="background: #f0f0f0;">
                            <th style="border: 1px solid #ddd; padding: 10px;">序號</th>
                            <th style="border: 1px solid #ddd; padding: 10px;">姓名</th>
                            <th style="border: 1px solid #ddd; padding: 10px;">通行碼</th>
                            <th style="border: 1px solid #ddd; padding: 10px;">座位</th>
                            <th style="border: 1px solid #ddd; padding: 10px;">狀態</th>
                        </tr>
                    </thead>
                    <tbody>
            `;

            students.forEach(([code, name], idx) => {
                const selection = appState.selections[code];
                const seat = selection ? selection.seat : '-';
                const status = selection ? '✓ 已選' : '待選';
                html += `
                    <tr>
                        <td style="border: 1px solid #ddd; padding: 10px; text-align: center;">${idx + 1}</td>
                        <td style="border: 1px solid #ddd; padding: 10px;">${name}</td>
                        <td style="border: 1px solid #ddd; padding: 10px; font-weight: bold;">${code}</td>
                        <td style="border: 1px solid #ddd; padding: 10px; text-align: center;">${seat}</td>
                        <td style="border: 1px solid #ddd; padding: 10px; text-align: center;">${status}</td>
                    </tr>
                `;
            });

            html += '</tbody></table>';

            const printWindow = window.open('', '', 'height=600,width=800');
            printWindow.document.write('<meta charset="UTF-8">');
            printWindow.document.write(html);
            printWindow.document.close();
            printWindow.print();
        }

        function downloadFile(content, filename, type) {
            const blob = new Blob([content], { type, charset: 'utf-8' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = filename;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        function switchAdminTab(tab) {
            document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));

            event.target.classList.add('active');
            document.getElementById(tab + 'Tab').classList.add('active');
        }

        function changePassword() {
            const currentPassword = document.getElementById('currentPassword').value;
            const newPassword = document.getElementById('newPassword').value;
            const confirmPassword = document.getElementById('confirmPassword').value;

            if (currentPassword !== appState.adminPassword) {
                alert('❌ 目前密碼錯誤！');
                return;
            }

            if (!newPassword || newPassword.length < 4) {
                alert('❌ 新密碼至少需要 4 個字符！');
                return;
            }

            if (newPassword !== confirmPassword) {
                alert('❌ 兩次輸入的新密碼不一致！');
                return;
            }

            appState.adminPassword = newPassword;
            saveData();

            document.getElementById('currentPassword').value = '';
            document.getElementById('newPassword').value = '';
            document.getElementById('confirmPassword').value = '';

            alert('✅ 密碼修改成功！\n\n下次進入老師後台請使用新密碼：' + newPassword);
        }

        function renderLoginPage() {
            document.getElementById('loginPage').style.display = 'flex';
            document.getElementById('selectionPage').style.display = 'none';
            document.getElementById('adminPage').style.display = 'none';
        }

        let syncTimer = null;

        function startAutoSync() {
            if (syncTimer) clearInterval(syncTimer);
            
            syncTimer = setInterval(() => {
                loadData();
                
                if (appState.currentPage === 'selection' && appState.currentUser) {
                    renderSeats();
                } else if (appState.currentPage === 'admin') {
                    updateAdminPage();
                }
            }, appState.syncInterval);
        }

        document.addEventListener('DOMContentLoaded', () => {
            loadData();
            renderLoginPage();
        });

        window.addEventListener('beforeunload', () => {
            saveData();
        });
    </script>
</body>
</html>
