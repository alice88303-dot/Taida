<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能選位系統</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        html, body { width: 100%; height: 100%; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background: #f0f4f8; }
        .container { width: 100%; max-width: 1200px; margin: 0 auto; padding: 0; }
        #loginPage { display: flex; justify-content: center; align-items: center; min-height: 100vh; width: 100%; padding: 20px; }
        .login-box { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); width: 100%; max-width: 400px; }
        .login-box h2 { text-align: center; margin-bottom: 30px; font-size: 24px; color: #333; }
        .form-group { margin-bottom: 20px; }
        .form-group label { display: block; margin-bottom: 8px; color: #555; font-weight: 600; }
        .form-group input, .form-group select { width: 100%; padding: 14px; border: 2px solid #ddd; border-radius: 6px; font-size: 16px; -webkit-appearance: none; appearance: none; }
        .form-group input:focus, .form-group select:focus { outline: none; border-color: #2563eb; box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1); }
        .btn { width: 100%; padding: 14px; margin: 10px 0; border: none; border-radius: 6px; font-size: 16px; font-weight: 600; cursor: pointer; -webkit-appearance: none; appearance: none; min-height: 48px; transition: all 0.3s; }
        .btn-primary { background: #2563eb; color: white; }
        .btn-primary:active { background: #1d4ed8; transform: scale(0.98); }
        .btn-secondary { background: #e5e7eb; color: #374151; }
        .btn-secondary:active { background: #d1d5db; }
        .error { color: #ef4444; font-size: 14px; margin-top: 5px; display: none; }
        #selectionPage, #adminPage { display: none; width: 100%; min-height: 100vh; padding: 20px; }
        #selectionPage.active, #adminPage.active { display: block; }
        .header { background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .header h1 { font-size: 24px; color: #333; margin-bottom: 10px; }
        .blackboard { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px; }
        .area { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .area-title { text-align: center; font-size: 20px; font-weight: 700; margin-bottom: 15px; padding: 10px; border-radius: 6px; color: white; }
        .area-a .area-title { background: #22c55e; }
        .area-b .area-title { background: #3b82f6; }
        .area-c .area-title { background: #f59e0b; }
        .area-d .area-title { background: #ec4899; }
        .seats { display: grid; grid-template-columns: repeat(5, 1fr); gap: 10px; }
        .seat { aspect-ratio: 1; border: 2px solid #ddd; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-weight: 700; cursor: pointer; font-size: 18px; background: white; transition: all 0.2s; -webkit-user-select: none; user-select: none; }
        .seat:active { transform: scale(0.95); }
        .seat.mine { background: #22c55e; color: white; border-color: #16a34a; }
        .seat.others { background: #f3f4f6; color: #9ca3af; cursor: not-allowed; }
        .area-b .seat.mine { background: #3b82f6; border-color: #1e40af; }
        .area-c .seat.mine { background: #f59e0b; border-color: #d97706; }
        .area-d .seat.mine { background: #ec4899; border-color: #be185d; }
        .modal { display: none; position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); z-index: 1000; justify-content: center; align-items: center; }
        .modal.active { display: flex; }
        .modal-content { background: white; padding: 30px; border-radius: 12px; max-width: 400px; width: 90%; text-align: center; }
        .modal-content h2 { margin-bottom: 20px; color: #333; }
        .modal-content p { margin-bottom: 15px; color: #666; font-size: 15px; }
        .modal-buttons { display: flex; gap: 10px; justify-content: center; margin-top: 20px; }
        .modal-buttons button { padding: 10px 20px; border: none; border-radius: 6px; cursor: pointer; font-weight: 600; flex: 1; -webkit-appearance: none; appearance: none; min-height: 44px; }
        @media (max-width: 640px) { .blackboard { grid-template-columns: 1fr; } .login-box { padding: 25px 20px; } }
    </style>
</head>
<body>
    <div id="loginPage" class="container">
        <div class="login-box">
            <h2>🎓 選位系統</h2>
            <div class="form-group">
                <label>角色選擇</label>
                <select id="roleSelect" style="width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; font-size: 16px;">
                    <option value="student">學生</option>
                    <option value="teacher">老師</option>
                </select>
            </div>
            <div id="studentForm">
                <div class="form-group">
                    <label>姓名</label>
                    <input type="text" id="studentName" placeholder="輸入你的名字">
                </div>
                <div class="form-group">
                    <label>通行碼</label>
                    <input type="text" id="passCode" placeholder="輸入通行碼">
                </div>
                <div class="error" id="studentError"></div>
                <button class="btn btn-primary" onclick="loginStudent()">開始選位</button>
            </div>
            <div id="teacherForm" style="display: none;">
                <div class="form-group">
                    <label>老師密碼</label>
                    <input type="password" id="teacherPassword" placeholder="輸入密碼">
                </div>
                <div class="error" id="teacherError"></div>
                <button class="btn btn-primary" onclick="loginTeacher()">進入後台</button>
            </div>
        </div>
    </div>

    <div id="selectionPage">
        <div class="header">
            <h1>👋 歡迎，<span id="userName"></span></h1>
            <p id="seatStatus" style="color: #666; margin: 10px 0;">尚未選擇座位</p>
            <button class="btn btn-secondary" onclick="logout()" style="max-width: 150px; margin-top: 10px;">登出</button>
        </div>
        <div class="blackboard">
            <div class="area area-a"><div class="area-title">A區</div><div class="seats" id="seatsA"></div></div>
            <div class="area area-b"><div class="area-title">B區</div><div class="seats" id="seatsB"></div></div>
            <div class="area area-c"><div class="area-title">C區</div><div class="seats" id="seatsC"></div></div>
            <div class="area area-d"><div class="area-title">D區</div><div class="seats" id="seatsD"></div></div>
        </div>
    </div>

    <div id="adminPage">
        <div class="header">
            <h1>📊 老師後台</h1>
            <button class="btn btn-secondary" onclick="logout()" style="max-width: 150px; margin-top: 10px;">登出</button>
        </div>
        <div style="background: white; padding: 20px; border-radius: 8px; margin-bottom: 20px;">
            <h3 style="margin-bottom: 15px;">📋 學生清單</h3>
            <table style="width: 100%; border-collapse: collapse; font-size: 14px;">
                <thead>
                    <tr style="border-bottom: 2px solid #eee;">
                        <th style="padding: 10px; text-align: left;">姓名</th>
                        <th style="padding: 10px; text-align: left;">座位</th>
                        <th style="padding: 10px; text-align: center;">操作</th>
                    </tr>
                </thead>
                <tbody id="studentList"></tbody>
            </table>
        </div>
        <div class="blackboard">
            <div class="area area-a"><div class="area-title">A區</div><div class="seats" id="adminSeatsA"></div></div>
            <div class="area area-b"><div class="area-title">B區</div><div class="seats" id="adminSeatsB"></div></div>
            <div class="area area-c"><div class="area-title">C區</div><div class="seats" id="adminSeatsC"></div></div>
            <div class="area area-d"><div class="area-title">D區</div><div class="seats" id="adminSeatsD"></div></div>
        </div>
    </div>

    <div id="confirmModal" class="modal">
        <div class="modal-content">
            <h2>✓ 確認選位</h2>
            <p id="confirmText"></p>
            <div style="background: #f0fdf4; padding: 15px; border-radius: 6px; margin: 15px 0; text-align: left;">
                <p><strong>姓名：</strong><span id="confirmName"></span></p>
                <p><strong>區域：</strong><span id="confirmArea"></span></p>
                <p><strong>座位：</strong><span id="confirmSeat"></span></p>
            </div>
            <p style="background: #fef3c7; padding: 10px; border-radius: 6px; font-size: 13px; color: #92400e;">⚠️ 確認後將無法自行更改</p>
            <div class="modal-buttons">
                <button class="btn btn-secondary" onclick="closeModal()">取消</button>
                <button class="btn btn-primary" onclick="confirmSelect()">確認選位</button>
            </div>
        </div>
    </div>

    <div id="deleteModal" class="modal">
        <div class="modal-content">
            <h2>清除選位記錄</h2>
            <p id="deleteText"></p>
            <div class="modal-buttons">
                <button class="btn btn-secondary" onclick="closeDeleteModal()">取消</button>
                <button class="btn btn-primary" onclick="deleteConfirm()">確認清除</button>
            </div>
        </div>
    </div>

    <script>
        // 全局數據（不使用 localStorage）
        const data = {
            students: {
                'STU001': '陳止觀',
                'STU002': '李明',
                'STU003': '王美麗'
            },
            selections: {},
            adminPassword: '631015'
        };

        let currentUser = null;
        let selectedSeat = null;
        let deleteTarget = null;

        // 角色選擇
        document.getElementById('roleSelect').addEventListener('change', (e) => {
            if (e.target.value === 'student') {
                document.getElementById('studentForm').style.display = 'block';
                document.getElementById('teacherForm').style.display = 'none';
            } else {
                document.getElementById('studentForm').style.display = 'none';
                document.getElementById('teacherForm').style.display = 'block';
            }
        });

        // 學生登入
        function loginStudent() {
            const name = document.getElementById('studentName').value.trim();
            const code = document.getElementById('passCode').value.trim();
            const errorEl = document.getElementById('studentError');

            if (!name || !code) {
                errorEl.textContent = '請輸入姓名和通行碼';
                errorEl.style.display = 'block';
                return;
            }

            if (!data.students[code] || data.students[code] !== name) {
                errorEl.textContent = '姓名或通行碼錯誤（試試: 陳止觀 / STU001）';
                errorEl.style.display = 'block';
                return;
            }

            currentUser = { name, code };
            showSelection();
        }

        // 老師登入
        function loginTeacher() {
            const password = document.getElementById('teacherPassword').value.trim();
            const errorEl = document.getElementById('teacherError');

            if (!password) {
                errorEl.textContent = '請輸入密碼';
                errorEl.style.display = 'block';
                return;
            }

            if (password !== data.adminPassword) {
                errorEl.textContent = '密碼錯誤（631015）';
                errorEl.style.display = 'block';
                return;
            }

            showAdmin();
        }

        // 顯示選位頁面
        function showSelection() {
            document.getElementById('loginPage').style.display = 'none';
            document.getElementById('selectionPage').classList.add('active');
            document.getElementById('adminPage').classList.remove('active');
            document.getElementById('userName').textContent = currentUser.name;
            renderSeats();
        }

        // 顯示後台
        function showAdmin() {
            document.getElementById('loginPage').style.display = 'none';
            document.getElementById('selectionPage').classList.remove('active');
            document.getElementById('adminPage').classList.add('active');
            renderAdminSeats();
            renderStudentList();
        }

        // 渲染座位
        function renderSeats() {
            const areas = ['A', 'B', 'C', 'D'];
            areas.forEach(area => {
                const container = document.getElementById('seats' + area);
                container.innerHTML = '';
                for (let i = 1; i <= 30; i++) {
                    const seatId = area + '-' + String(i).padStart(2, '0');
                    const div = document.createElement('div');
                    div.className = 'seat';

                    const mySeat = data.selections[currentUser.code]?.seat === seatId;
                    const otherSeat = Object.values(data.selections).some(s => s.seat === seatId && s.name !== currentUser.name);

                    if (mySeat) {
                        div.classList.add('mine');
                        div.textContent = '我';
                    } else if (otherSeat) {
                        div.classList.add('others');
                        div.textContent = '✓';
                    } else if (!data.selections[currentUser.code]?.locked) {
                        div.textContent = i;
                        div.onclick = () => selectSeat(seatId, area, i);
                    } else {
                        div.textContent = i;
                    }

                    container.appendChild(div);
                }
            });

            const sel = data.selections[currentUser.code];
            if (sel?.locked) {
                document.getElementById('seatStatus').textContent = '🔒 已鎖定：' + sel.seat;
            } else if (sel) {
                document.getElementById('seatStatus').textContent = '已選擇：' + sel.seat;
            } else {
                document.getElementById('seatStatus').textContent = '尚未選擇座位';
            }
        }

        // 選座位
        function selectSeat(seatId, area, num) {
            if (data.selections[currentUser.code]?.locked) {
                alert('❌ 您的選位已確認鎖定，無法自行更改。\n\n如需修改，請聯繫老師。');
                return;
            }

            selectedSeat = seatId;
            document.getElementById('confirmName').textContent = currentUser.name;
            document.getElementById('confirmArea').textContent = area + '區';
            document.getElementById('confirmSeat').textContent = num + '號';
            document.getElementById('confirmModal').classList.add('active');
        }

        // 確認選位
        function confirmSelect() {
            data.selections[currentUser.code] = {
                name: currentUser.name,
                seat: selectedSeat,
                locked: true,
                time: new Date().toLocaleString('zh-TW')
            };
            closeModal();
            renderSeats();
            alert('✅ 選位已確認！無法自行更改。\n\n如需修改，請聯繫老師。');
        }

        // 渲染學生清單
        function renderStudentList() {
            const tbody = document.getElementById('studentList');
            tbody.innerHTML = '';
            Object.entries(data.students).forEach(([code, name]) => {
                const sel = data.selections[code];
                const tr = document.createElement('tr');
                tr.style.borderBottom = '1px solid #eee';
                tr.innerHTML = `
                    <td style="padding: 10px;">${name}</td>
                    <td style="padding: 10px;">${sel ? sel.seat : '（未選）'}</td>
                    <td style="padding: 10px; text-align: center;">
                        ${sel ? `<button class="btn btn-secondary" onclick="showDeleteModal('${code}', '${name}')" style="width: 80px; padding: 6px; margin: 0; font-size: 12px;">清除</button>` : '-'}
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // 渲染老師座位視圖
        function renderAdminSeats() {
            const areas = ['A', 'B', 'C', 'D'];
            areas.forEach(area => {
                const container = document.getElementById('adminSeats' + area);
                container.innerHTML = '';
                for (let i = 1; i <= 30; i++) {
                    const seatId = area + '-' + String(i).padStart(2, '0');
                    const div = document.createElement('div');
                    div.className = 'seat';

                    const sel = Object.values(data.selections).find(s => s.seat === seatId);
                    if (sel) {
                        div.classList.add('mine');
                        div.innerHTML = '<div style="font-size: 11px; line-height: 1;">' + i + '<br>' + sel.name + '</div>';
                        div.style.cursor = 'pointer';
                        div.onclick = () => {
                            deleteTarget = Object.entries(data.students).find(([k, v]) => v === sel.name)?.[0];
                            showDeleteModal(deleteTarget, sel.name);
                        };
                    } else {
                        div.textContent = i;
                    }

                    container.appendChild(div);
                }
            });
        }

        // 顯示刪除對話框
        function showDeleteModal(code, name) {
            deleteTarget = code;
            document.getElementById('deleteText').textContent = '清除 ' + name + ' 的選位記錄？';
            document.getElementById('deleteModal').classList.add('active');
        }

        // 刪除確認
        function deleteConfirm() {
            if (deleteTarget && data.selections[deleteTarget]) {
                delete data.selections[deleteTarget];
                closeDeleteModal();
                if (currentUser) renderSeats();
                renderAdminSeats();
                renderStudentList();
            }
        }

        // 關閉
        function closeModal() {
            document.getElementById('confirmModal').classList.remove('active');
        }

        function closeDeleteModal() {
            document.getElementById('deleteModal').classList.remove('active');
        }

        function logout() {
            currentUser = null;
            selectedSeat = null;
            document.getElementById('loginPage').style.display = 'flex';
            document.getElementById('selectionPage').classList.remove('active');
            document.getElementById('adminPage').classList.remove('active');
            document.getElementById('studentName').value = '';
            document.getElementById('passCode').value = '';
            document.getElementById('teacherPassword').value = '';
            document.getElementById('studentError').style.display = 'none';
            document.getElementById('teacherError').style.display = 'none';
            document.getElementById('roleSelect').value = 'student';
            document.getElementById('studentForm').style.display = 'block';
            document.getElementById('teacherForm').style.display = 'none';
        }
    </script>
</body>
</html>
