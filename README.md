<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ระบบเช็คชื่อเข้าเรียนออนไลน์ (Smart Check-in)</title>
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600&display=swap" rel="stylesheet">
    <style>
        * { box-sizing: border-box; font-family: 'Sarabun', sans-serif; -webkit-tap-highlight-color: transparent; }
        body { background-color: #f4f6f9; margin: 0; padding: 12px; display: flex; justify-content: center; min-height: 100vh; }
        .container { width: 100%; max-width: 600px; }
        .card { background: white; padding: 20px; border-radius: 16px; box-shadow: 0 4px 20px rgba(0,0,0,0.06); margin-bottom: 16px; }
        h2, h3 { text-align: center; color: #1a73e8; margin-top: 0; margin-bottom: 8px; }
        
        /* เมนูแท็บสไตล์แอปมือถือ */
        .role-selector { display: flex; gap: 6px; margin-bottom: 16px; background: #e8ecef; padding: 4px; border-radius: 12px; }
        .role-btn { flex: 1; padding: 10px 4px; border: none; background: transparent; border-radius: 8px; font-weight: 600; cursor: pointer; font-size: 0.85rem; color: #555; transition: 0.2s; }
        .role-btn.active { background: #ffffff; color: #1a73e8; box-shadow: 0 2px 6px rgba(0,0,0,0.1); }

        .input-group { margin-bottom: 14px; text-align: left; }
        .input-group label { display: block; margin-bottom: 6px; font-weight: 600; color: #333; font-size: 0.95rem; }
        .input-group input, .input-group select { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 10px; font-size: 1rem; background: #fff; appearance: none; }
        
        /* กรอบกล้องปรับขนาดตามจอมือถือ/iPad */
        .video-box { width: 100%; max-width: 320px; aspect-ratio: 3/4; background: #000; border-radius: 12px; overflow: hidden; margin: 0 auto 15px auto; position: relative; }
        video { width: 100%; height: 100%; object-fit: cover; transform: scaleX(-1); } /* กลับภาพเหมือนกระจกเงา */
        
        .btn { width: 100%; padding: 14px; border: none; border-radius: 10px; font-size: 1.05rem; font-weight: 600; cursor: pointer; transition: 0.2s; }
        .btn-primary { background: #1a73e8; color: white; }
        .btn-success { background: #198754; color: white; }
        .btn-export { background: #0f9d58; color: white; width: 100%; padding: 10px; margin-bottom: 15px; }
        .btn-delete { background: #dc3545; color: white; border: none; padding: 6px 12px; border-radius: 6px; cursor: pointer; font-size: 0.85rem; }

        /* สถิติการ์ด */
        .stats-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-bottom: 15px; text-align: center; }
        .stat-card { background: #f8f9fa; padding: 12px; border-radius: 10px; border: 1px solid #e9ecef; }
        .stat-num { font-size: 1.4rem; font-weight: bold; margin-top: 2px; }

        /* ตารางสถิติ */
        .table-responsive { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 0.85rem; }
        th, td { padding: 10px 6px; text-align: center; border-bottom: 1px solid #eee; }
        th { background: #f8f9fa; color: #555; }
        
        .badge { display: inline-block; padding: 4px 8px; border-radius: 12px; font-size: 0.75rem; font-weight: bold; }
        .badge-normal { background: #e6f4ea; color: #137333; }
        .badge-late { background: #fce8e6; color: #c5221f; }
        .badge-absent { background: #f1f3f4; color: #5f6368; }
    </style>
</head>
<body>

<div class="container">
    <!-- แถบเมนูหลัก -->
    <div class="role-selector">
        <button class="role-btn active" onclick="switchTab('checkinSection', this)">📸 สแกนเช็คชื่อ</button>
        <button class="role-btn" onclick="switchTab('registerSection', this)">✍️ ลงทะเบียนใหม่</button>
        <button class="role-btn" onclick="switchTab('teacherAuth', this)">👩‍🏫 สถิติสำหรับครู</button>
    </div>

    <!-- 1. หน้าสแกนเช็คชื่อ -->
    <div id="checkinSection" class="card tab-content">
        <h2>📸 สแกนเช็คชื่อเข้าเรียน</h2>
        <p style="text-align: center; color: #666; font-size: 0.85rem; margin-top: -5px; margin-bottom: 15px;">เข้าเรียนหลัง 08:00 น. ถือว่า "มาสาย"</p>
        
        <div class="input-group">
            <label>เลือกชื่อ-นามสกุล:</label>
            <select id="studentSelect">
                <option value="">-- เลือกชื่อ-นามสกุลของคุณ --</option>
            </select>
        </div>
        
        <div class="video-box">
            <video id="webcamCheckin" autoplay playsinline muted></video>
        </div>
        
        <button class="btn btn-primary" onclick="submitCheckIn()">📸 กดเพื่อเช็คชื่อ</button>
        <div id="checkinResult" style="text-align:center; margin-top:15px;"></div>
    </div>

    <!-- 2. หน้าลงทะเบียนนักเรียนใหม่ -->
    <div id="registerSection" class="card tab-content" style="display: none;">
        <h2>✍️ ลงทะเบียนนักเรียนใหม่</h2>
        
        <div class="input-group">
            <label>ชื่อ-นามสกุล:</label>
            <input type="text" id="regName" placeholder="เช่น นายสมชาย ใจดี">
        </div>
        
        <div class="video-box">
            <video id="webcamRegister" autoplay playsinline muted></video>
        </div>
        
        <button class="btn btn-success" onclick="studentSelfRegister()">💾 บันทึกรายชื่อใหม่</button>
        <div id="registerResult" style="text-align:center; margin-top:15px;"></div>
    </div>

    <!-- 3. หน้ายืนยันตัวตนครู -->
    <div id="teacherAuth" class="card tab-content" style="display: none; max-width: 400px; margin: 0 auto;">
        <h3>🔐 ล็อกอินสำหรับคุณครู</h3>
        <div class="input-group">
            <label>รหัสผ่านครู:</label>
            <input type="password" id="teacherPass" placeholder="รหัสผ่าน: 123456789">
        </div>
        <button class="btn btn-primary" onclick="loginTeacher()">เข้าสู่แผงสถิติ</button>
        <div id="loginErr" style="color:red; text-align:center; margin-top:10px;"></div>
    </div>

    <!-- 4. แผงสถิติครู -->
    <div id="teacherDashboard" class="card tab-content" style="display: none;">
        <h3>📊 รายงานสถิติการเข้าเรียน</h3>
        <button class="btn btn-export" onclick="exportToCSV()">📥 ส่งออกสถิติ (CSV/Excel)</button>
        
        <div class="stats-grid">
            <div class="stat-card">นักเรียนทั้งหมด<div id="totalCnt" class="stat-num" style="color:#1a73e8;">0</div></div>
            <div class="stat-card">มาปกติ<div id="normalCnt" class="stat-num" style="color:#198754;">0</div></div>
            <div class="stat-card">มาสาย<div id="lateCnt" class="stat-num" style="color:#dc3545;">0</div></div>
            <div class="stat-card">ยังไม่มา<div id="absentCnt" class="stat-num" style="color:#6c757d;">0</div></div>
        </div>

        <div class="table-responsive">
            <table>
                <thead>
                    <tr>
                        <th>ลำดับ</th>
                        <th>ชื่อ-นามสกุล</th>
                        <th>เวลา</th>
                        <th>สถานะ</th>
                        <th>จัดการ</th>
                    </tr>
                </thead>
                <tbody id="studentTableBody"></tbody>
            </table>
        </div>
    </div>
</div>

<script>
    // ฐานข้อมูลในตัวเครื่อง (LocalStorage)
    let studentDB = JSON.parse(localStorage.getItem('studentDB')) || [
        { name: "นายสมชาย ใจดี", time: "-", status: "ยังไม่มา" },
        { name: "นางสาวมณี รักดี", time: "-", status: "ยังไม่มา" }
    ];

    let currentStream = null;

    // เปิดกล้องหน้า (รองรับ iOS & Android)
    function startCam(videoId) {
        if (currentStream) {
            currentStream.getTracks().forEach(track => track.stop());
        }
        
        // กำหนดการตั้งค่ากล้องหน้า
        const constraints = {
            video: {
                facingMode: "user", // บังคับใช้กล้องหน้า
                width: { ideal: 640 },
                height: { ideal: 480 }
            }
        };

        navigator.mediaDevices.getUserMedia(constraints)
            .then(stream => {
                currentStream = stream;
                const videoEl = document.getElementById(videoId);
                videoEl.srcObject = stream;
            })
            .catch(err => {
                alert("ไม่สามารถเปิดกล้องได้: กรุณากด 'อนุญาต' (Allow) ให้เว็บเข้าถึงกล้องถ่ายรูป");
            });
    }

    // อัปเดตรายชื่อใน Dropdown
    function updateDropdown() {
        const select = document.getElementById('studentSelect');
        select.innerHTML = '<option value="">-- เลือกชื่อ-นามสกุลของคุณ --</option>';
        studentDB.forEach(student => {
            const opt = document.createElement('option');
            opt.value = student.name;
            opt.innerText = student.name;
            select.appendChild(opt);
        });
    }

    // สลับหน้าแท็บ
    function switchTab(tabId, btnElement) {
        document.querySelectorAll('.tab-content').forEach(el => el.style.display = 'none');
        document.querySelectorAll('.role-btn').forEach(btn => btn.classList.remove('active'));
        
        document.getElementById(tabId).style.display = 'block';
        if (btnElement) btnElement.classList.add('active');

        if (tabId === 'checkinSection') {
            updateDropdown();
            startCam('webcamCheckin');
        }
        if (tabId === 'registerSection') {
            startCam('webcamRegister');
        }
        if (tabId === 'teacherAuth' && currentStream) {
            currentStream.getTracks().forEach(track => track.stop());
        }
    }

    // เริ่มต้นทำงาน
    updateDropdown();
    startCam('webcamCheckin');

    // 1. ระบบลงทะเบียน
    function studentSelfRegister() {
        const name = document.getElementById('regName').value.trim();
        if (!name) return alert("กรุณากรอกชื่อ-นามสกุล");

        if (studentDB.some(s => s.name === name)) {
            return alert("ชื่อนี้มีในระบบแล้ว สามารถไปสแกนเช็คชื่อได้เลยครับ");
        }

        studentDB.push({ name: name, time: "-", status: "ยังไม่มา" });
        localStorage.setItem('studentDB', JSON.stringify(studentDB));

        document.getElementById('registerResult').innerHTML = `
            <div style="color:#198754; font-weight:bold;">✅ ลงทะเบียนสำเร็จ!</div>
            <div style="font-size:0.9rem;">คุณ ${name} สแกนเช็คชื่อได้ทันที</div>
        `;

        document.getElementById('regName').value = '';
        updateDropdown();
    }

    // 2. ระบบเช็คชื่อ (ตัดสาย 08.00 น.)
    function submitCheckIn() {
        const selectedName = document.getElementById('studentSelect').value;
        if (!selectedName) return alert("กรุณาเลือกชื่อ-นามสกุลของคุณก่อนครับ");

        let student = studentDB.find(s => s.name === selectedName);
        const now = new Date();
        const timeStr = now.toTimeString().split(' ')[0];
        
        // เช็คเงื่อนไขเวลา 08:00 น.
        const isLate = (now.getHours() > 8) || (now.getHours() === 8 && (now.getMinutes() > 0 || now.getSeconds() > 0));
        const statusStr = isLate ? "มาสาย" : "มาปกติ";

        student.time = timeStr;
        student.status = statusStr;
        localStorage.setItem('studentDB', JSON.stringify(studentDB));

        const badgeClass = isLate ? 'badge-late' : 'badge-normal';
        document.getElementById('checkinResult').innerHTML = `
            <div style="font-weight:bold; color:#1a73e8;">✅ บันทึกเวลาสำเร็จ</div>
            <div style="font-size:0.9rem; margin-top:2px;">${student.name} (${timeStr} น.)</div>
            <div style="margin-top:6px;"><span class="badge ${badgeClass}">${statusStr}</span></div>
        `;
    }

    // 3. เข้าสู่ระบบครู
    function loginTeacher() {
        const pass = document.getElementById('teacherPass').value.trim();
        if (pass === "123456789") {
            document.getElementById('teacherAuth').style.display = 'none';
            document.getElementById('teacherDashboard').style.display = 'block';
            renderTeacherDashboard();
        } else {
            document.getElementById('loginErr').innerText = "รหัสผ่านไม่ถูกต้อง";
        }
    }

    // 4. แสดงรายงานสถิติ
    function renderTeacherDashboard() {
        const total = studentDB.length;
        const normal = studentDB.filter(s => s.status === "มาปกติ").length;
        const late = studentDB.filter(s => s.status === "มาสาย").length;
        const absent = studentDB.filter(s => s.status === "ยังไม่มา").length;

        document.getElementById('totalCnt').innerText = total;
        document.getElementById('normalCnt').innerText = normal;
        document.getElementById('lateCnt').innerText = late;
        document.getElementById('absentCnt').innerText = absent;

        const tbody = document.getElementById('studentTableBody');
        tbody.innerHTML = '';
        studentDB.forEach((s, index) => {
            let badgeClass = 'badge-absent';
            if (s.status === 'มาปกติ') badgeClass = 'badge-normal';
            if (s.status === 'มาสาย') badgeClass = 'badge-late';

            tbody.innerHTML += `
                <tr>
                    <td>${index + 1}</td>
                    <td style="text-align:left;">${s.name}</td>
                    <td>${s.time}</td>
                    <td><span class="badge ${badgeClass}">${s.status}</span></td>
                    <td><button class="btn-delete" onclick="deleteStudent(${index})">ลบ</button></td>
                </tr>
            `;
        });
    }

    // 5. ลบรายชื่อ
    function deleteStudent(index) {
        if (confirm(`ลบรายชื่อ "${studentDB[index].name}"?`)) {
            studentDB.splice(index, 1);
            localStorage.setItem('studentDB', JSON.stringify(studentDB));
            renderTeacherDashboard();
            updateDropdown();
        }
    }

    // 6. ส่งออก CSV
    function exportToCSV() {
        if (studentDB.length === 0) return alert("ไม่มีข้อมูล");

        let csvContent = "\uFEFF";
        csvContent += "ลำดับ,ชื่อ-นามสกุล,เวลาสแกน,สถานะ\n";

        studentDB.forEach((s, i) => {
            csvContent += `"${i + 1}","${s.name}","${s.time}","${s.status}"\n`;
        });

        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const url = URL.createObjectURL(blob);
        const link = document.createElement("a");
        link.setAttribute("href", url);
        link.setAttribute("download", `รายงานเช็คชื่อ_${new Date().toISOString().slice(0,10)}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }
</script>

</body>
</html>
