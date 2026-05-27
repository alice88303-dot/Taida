import { useState, useEffect, useRef } from "react";

// ══════════════════════════════════════════════════════════════════════
// 設定區 (透過環境變數讀取，安全推上 GitHub 不外洩)
// ══════════════════════════════════════════════════════════════════════
const API_URL = import.meta.env.VITE_API_URL;
const ADMIN_PIN = import.meta.env.VITE_ADMIN_PIN;
const POLL_MS = 3000; 

const ZONES = [
  { id: "A", label: "A 區", color: "#4caf82", light: "#e8f5ee", border: "#4caf82" },
  { id: "B", label: "B 區", color: "#5b9bd5", light: "#e3eef8", border: "#5b9bd5" },
  { id: "C", label: "C 區", color: "#d4a843", light: "#fdf6e3", border: "#d4a843" },
  { id: "D", label: "D 區", color: "#d46b6b", light: "#fceaea", border: "#d46b6b" },
];
const SEATS_PER_ZONE = 30;
const COLS = 5;

function buildEmptySeats() {
  const map = {};
  ZONES.forEach((z) => {
    for (let i = 1; i <= SEATS_PER_ZONE; i++)
      map[`${z.id}-${i}`] = { zone: z.id, num: i, student: null, order: null };
  });
  return map;
}

async function callApi(params) {
  if (!API_URL) {
    console.error("錯誤：未設定 VITE_API_URL 環境變數");
    return { ok: false, error: "前端環境變數未設定" };
  }
  const url = API_URL + "?" + new URLSearchParams(params).toString();
  const res = await fetch(url);
  return res.json();
}

async function fetchState() {
  try {
    const data = await callApi({ action: "getState" });
    if (!data.ok) return null;
    const seats = buildEmptySeats();
    Object.entries(data.seats || {}).forEach(([k, v]) => {
      if (seats[k]) seats[k] = { ...seats[k], ...v };
    });
    return { seats, used: data.used || {}, studentList: data.studentList || [] };
  } catch {
    return null;
  }
}

export default function App() {
  const [seats, setSeats] = useState(buildEmptySeats);
  const [used, setUsed] = useState({});          
  const [studentList, setStudentList] = useState([]); 
  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);
  const [page, setPage] = useState("enter");

  // 學生狀態
  const [nameInput, setNameInput] = useState("");
  const [codeInput, setCodeInput] = useState("");
  const [myName, setMyName] = useState("");
  const [viewOnly, setViewOnly] = useState(false);
  const [pendingSeat, setPendingSeat] = useState(null);

  // 管理員狀態
  const [pinInput, setPinInput] = useState("");
  const [pinError, setPinError] = useState("");
  const [showAdminLogin, setShowAdminLogin] = useState(false);
  const [confirmReset, setConfirmReset] = useState(null);
  const [showExport, setShowExport] = useState(false);
  const [exportText, setExportText] = useState("");
  const [newStudentName, setNewStudentName] = useState(""); 

  const [toast, setToast] = useState(null);
  const pollRef = useRef(null);

  useEffect(() => {
    fetchState().then((s) => {
      if (s) { 
        setSeats(s.seats); 
        setUsed(s.used); 
        setStudentList(s.studentList);
      }
      setLoading(false);
    });
  }, []);

  useEffect(() => {
    clearInterval(pollRef.current);
    if (page === "pick" || page === "admin") {
      pollRef.current = setInterval(() => {
        fetchState().then((s) => {
          if (s) { 
            setSeats(s.seats); 
            setUsed(s.used); 
            setStudentList(s.studentList);
          }
        });
      }, POLL_MS);
    }
    return () => clearInterval(pollRef.current);
  }, [page]);

  const toast$ = (msg, type = "ok") => {
    setToast({ msg, type });
    setTimeout(() => setToast(null), 2800);
  };

  const takenCount = Object.values(seats).filter((s) => s.student).length;
  const nextOrder = takenCount + 1;

  const handleVerify = async () => {
    const name = nameInput.trim();
    const code = codeInput.trim().toUpperCase();
    if (!name) { toast$("請輸入姓名", "warn"); return; }
    if (!code) { toast$("請輸入通行碼", "warn"); return; }
    
    setSaving(true);
    try {
      const res = await callApi({ action: "verifyStudent", name, code });
      if (res.ok) {
        const s = await fetchState();
        if (s) { setSeats(s.seats); setUsed(s.used); }
        setMyName(name);
        setViewOnly(res.isUsed);
        setPage("pick");
      } else {
        toast$(res.error || "驗證失敗", "warn");
      }
    } catch {
      toast$("連線伺服器失敗，請稍後再試", "warn");
    }
    setSaving(false);
  };

  const handleSeatClick = (zone, num) => {
    if (viewOnly) return;
    if (seats[`${zone}-${num}`].student) return;
    setPendingSeat({ zone, num });
    setPage("confirm");
  };

  const handleConfirm = async () => {
    const key = `${pendingSeat.zone}-${pendingSeat.num}`;
    setSaving(true);
    try {
      const r1 = await callApi({ action: "saveSeat", key, student: myName, order: nextOrder });
      if (!r1.ok) { 
        toast$(r1.error || "選位失敗", "warn"); 
        setSaving(false); 
        setPage("pick");
        return; 
      }
      await callApi({ action: "markUsed", name: myName });
      const s = await fetchState();
      if (s) { setSeats(s.seats); setUsed(s.used); }
      setViewOnly(true);
      setPage("done");
    } catch(err) {
      toast$("通訊錯誤：" + err.message, "warn");
    }
    setSaving(false);
  };

  const handleAddStudent = async () => {
    const name = newStudentName.trim();
    if (!name) { toast$("請輸入學生姓名", "warn"); return; }
    setSaving(true);
    try {
      const res = await callApi({ action: "addStudent", name });
      if (res.ok) {
        toast$(`成功生成！[${name}] 通行碼: ${res.code}`);
        setNewStudentName("");
        const s = await fetchState();
        if (s) { setStudentList(s.studentList); setUsed(s.used); }
      } else {
        toast$(res.error || "新增失敗", "warn");
      }
    } catch {
      toast$("連線失敗", "warn");
    }
    setSaving(false);
  };

  const handleClearSeat = async () => {
    const key = confirmReset;
    const student = seats[key]?.student;
    setConfirmReset(null);
    setSaving(true);
    try {
      const res = await callApi({ action: "clearSeat", key, student: student || "" });
      if(res.ok) {
        const s = await fetchState();
        if (s) { setSeats(s.seats); setUsed(s.used); }
        toast$("已清除，學生可重新登入選位");
      }
    } catch { toast$("清除失敗", "warn"); }
    setSaving(false);
  };

  const handleResetAll = async () => {
    setSaving(true);
    try {
      await callApi({ action: "resetAll" });
      const s = await fetchState();
      if (s) { setSeats(s.seats); setUsed(s.used); }
      toast$("已清空所有座位與選位紀錄");
    } catch { toast$("重置失敗", "warn"); }
    setSaving(false);
  };

  const handleExport = () => {
    const rows = [["報名順序", "姓名", "區域", "座號"]];
    Object.values(seats)
      .filter((s) => s.student)
      .sort((a, b) => a.order - b.order)
      .forEach((s) => {
        const z = ZONES.find((z) => z.id === s.zone);
        rows.push([s.order, s.student, z?.label, s.num]);
      });
    setExportText(rows.map((r) => r.join("\t")).join("\n"));
    setShowExport(true);
  };

  const Toaster = () => toast ? (
    <div style={{
      position: "fixed", top: 20, left: "50%", transform: "translateX(-50%)",
      background: toast.type === "warn" ? "#e74c3c" : "#27ae60",
      color: "#fff", padding: "10px 28px", borderRadius: 99,
      fontWeight: 700, fontSize: "0.9rem", zIndex: 9999,
      boxShadow: "0 4px 20px rgba(0,0,0,0.2)", whiteSpace: "nowrap",
    }}>{toast.msg}</div>
  ) : null;

  const Overlay = () => saving ? (
    <div style={{ position: "fixed", inset: 0, background: "rgba(255,255,255,0.65)", zIndex: 500, display: "flex", alignItems: "center", justifyContent: "center" }}>
      <div style={{ background: "#fff", borderRadius: 14, padding: "18px 36px", boxShadow: "0 4px 24px rgba(0,0,0,0.12)", color: "#555", fontSize: "0.95rem" }}>⏳ 處理中⋯</div>
    </div>
  ) : null;

  const SeatGrid = ({ isAdmin }) => (
    <>
      {ZONES.map((z) => (
        <div key={z.id} style={{ background: "#fff", borderRadius: 14, border: `2px solid ${z.border}`, overflow: "hidden", boxShadow: "0 2px 14px rgba(0,0,0,0.07)" }}>
          <div style={{ background: z.color, color: "#fff", textAlign: "center", padding: "10px 0", fontWeight: 800, fontSize: "1rem", letterSpacing: "0.1em" }}>{z.label}</div>
          <div style={{ display: "grid", gridTemplateColumns: `repeat(${COLS}, 1fr)`, gap: 6, padding: 12, background: z.light }}>
            {Array.from({ length: SEATS_PER_ZONE }, (_, i) => {
              const num = i + 1;
              const key = `${z.id}-${num}`;
              const seat = seats[key];
              const taken = !!seat.student;
              const mine = !isAdmin && taken && seat.student === myName;
              return (
                <button key={num}
                  onClick={() => {
                    if (isAdmin) { if (taken) setConfirmReset(key); }
                    else handleSeatClick(z.id, num);
                  }}
                  style={{
                    aspectRatio: "1", borderRadius: 8,
                    border: mine ? "2.5px solid #f0b800" : taken ? `2px solid ${z.color}` : "2px solid #e0e0e0",
                    background: mine ? "#ffd700" : taken ? z.color : "#fff",
                    color: mine ? "#333" : taken ? "#fff" : "#aaa",
                    fontSize: "0.78rem", fontWeight: taken ? 700 : 400,
                    cursor: isAdmin ? (taken ? "pointer" : "default") : (taken || viewOnly) ? "default" : "pointer",
                    display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center",
                    lineHeight: 1.2, padding: 2, transition: "transform 0.1s",
                    boxShadow: mine ? "0 0 8px #ffd70088" : taken ? `0 2px 6px ${z.color}44` : "none",
                  }}
                  onMouseEnter={(e) => {
                    if (!taken && !isAdmin && !viewOnly) e.currentTarget.style.transform = "scale(1.1)";
                    if (taken && isAdmin) e.currentTarget.style.background = "#e74c3c";
                  }}
                  onMouseLeave={(e) => {
                    e.currentTarget.style.transform = "scale(1)";
                    if (taken && isAdmin) e.currentTarget.style.background = mine ? "#ffd700" : z.color;
                  }}
                >
                  {taken
                    ? isAdmin
                      ? <><span style={{ fontSize: "0.68rem", opacity: 0.8 }}>{num}</span>
                          <span style={{ fontSize: "0.78rem", fontWeight: 900, maxWidth: "100%", overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap", padding: "0 2px" }}>
                            {seat.student.length > 3 ? seat.student.slice(0, 3) + "…" : seat.student}
                          </span></>
                      : mine ? <span style={{ fontSize: "0.8rem", fontWeight: 900 }}>我</span>
                             : <span style={{ fontSize: "1rem" }}>✓</span>
                    : <span>{num}</span>
                  }
                </button>
              );
            })}
          </div>
        </div>
      ))}
    </>
  );

  const Header = ({ subtitle }) => (
    <div style={{ background: "#1a1a2e", color: "#fff", padding: "16px 24px", display: "flex", justifyContent: "space-between", alignItems: "center", boxShadow: "0 2px 14px rgba(0,0,0,0.2)" }}>
      <div>
        <div style={{ fontSize: "1.2rem", fontWeight: 900, letterSpacing: "0.03em" }}>📚 台大補習班選位系統</div>
        {subtitle && <div style={{ fontSize: "0.78rem", opacity: 0.55, marginTop: 2 }}>{subtitle}</div>}
      </div>
      {page !== "enter" && page !== "confirm" && page !== "done" && (
        <div style={{ fontSize: "0.82rem", background: "rgba(255,255,255,0.1)", padding: "6px 14px", borderRadius: 8 }}>
          {page === "admin" ? "🔓 管理員" : `👤 ${myName}`}
        </div>
      )}
    </div>
  );

  const Blackboard = () => (
    <div style={{ background: "#2d8a50", color: "#fff", textAlign: "center", padding: "11px 0", borderRadius: 10, fontWeight: 900, letterSpacing: "0.3em", marginBottom: 18, boxShadow: "0 4px 14px rgba(45,138,80,0.25)" }}>黑　板</div>
  );

  if (loading) return (
    <div style={{ minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center", background: "#f5f5f0", fontFamily: "sans-serif" }}>
      <div style={{ textAlign: "center", color: "#999" }}>
        <div style={{ fontSize: "2.5rem", marginBottom: 12 }}>📚</div>
        <div>資料同步中⋯</div>
      </div>
    </div>
  );

  // ── 頁面：登入 ──
  if (page === "enter") return (
    <div style={S.page}><Toaster /><Overlay />
      <div style={S.card}>
        <div style={{ fontSize: "2.8rem", marginBottom: 10 }}>📚</div>
        <h2 style={{ margin: "0 0 6px", fontFamily: "'Georgia',serif", color: "#1a1a2e", fontSize: "1.5rem" }}>台大補習班選位系統</h2>
        <p style={{ color: "#999", fontSize: "0.88rem", marginBottom: 28 }}>請輸入姓名與老師給您的通行碼</p>

        <label style={S.label}>姓名</label>
        <input value={nameInput} onChange={(e) => setNameInput(e.target.value)}
          placeholder="請輸入完整姓名" style={S.input} />

        <label style={{ ...S.label, marginTop: 14 }}>通行碼</label>
        <input value={codeInput}
          onChange={(e) => setCodeInput(e.target.value.toUpperCase())}
          onKeyDown={(e) => e.key === "Enter" && handleVerify()}
          placeholder="輸入 5 碼通行碼" maxLength={5}
          style={{ ...S.input, textAlign: "center", letterSpacing: "0.3em", fontFamily: "monospace", fontSize: "1.15rem", fontWeight: 700 }} />

        <button onClick={handleVerify} style={{ ...S.btn, width: "100%", marginTop: 24, background: "#1a1a2e" }}>進入選位 →</button>

        <div style={{ marginTop: 28, borderTop: "1px solid #eee", paddingTop: 18 }}>
          {!showAdminLogin
            ? <button onClick={() => setShowAdminLogin(true)} style={S.ghost}>老師入口</button>
            : <div style={{ background: "#f8f8f8", borderRadius: 12, padding: 16 }}>
                <input type="password" value={pinInput} onChange={(e) => setPinInput(e.target.value)}
                  onKeyDown={(e) => { if (e.key === "Enter") { if (pinInput === ADMIN_PIN) { setPage("admin"); setPinInput(""); setPinError(""); setShowAdminLogin(false); } else setPinError("密碼錯誤"); }}}
                  placeholder="請輸入管理員密碼" style={{ ...S.input, marginBottom: 8 }} />
                {pinError && <div style={{ color: "#e74c3c", fontSize: "0.82rem", marginBottom: 8 }}>{pinError}</div>}
                <button onClick={() => { if (pinInput === ADMIN_PIN) { setPage("admin"); setPinInput(""); setPinError(""); setShowAdminLogin(false); } else setPinError("密碼錯誤"); }}
                  style={{ ...S.btn, width: "100%", background: "#1a1a2e" }}>進入管理</button>
              </div>
          }
        </div>
      </div>
    </div>
  );

  // ── 頁面：選位 ──
  if (page === "pick") {
    const mySeat = Object.values(seats).find((s) => s.student === myName);
    return (
      <div style={{ minHeight: "100vh", background: "#f5f5f0", fontFamily: "sans-serif" }}>
        <Toaster /><Overlay />
        <Header subtitle={`已選 ${takenCount} 位 · 每 ${POLL_MS/1000} 秒自動更新`} />

        {viewOnly && mySeat && (
          <div style={{ background: "#2d6a4f", color: "#fff", textAlign: "center", padding: "9px 16px", fontSize: "0.88rem", fontWeight: 600 }}>
            ✅ 您已選好座位：{ZONES.find(z => z.id === mySeat.zone)?.label} · {mySeat.num} 號（金色標示）
          </div>
        )}
        {viewOnly && !mySeat && (
          <div style={{ background: "#856404", color: "#fff", textAlign: "center", padding: "9px 16px", fontSize: "0.88rem", fontWeight: 600 }}>
            ⚠️ 您的座位已被班導清除，請聯絡班導重新選位
          </div>
        )}

        <div style={{ maxWidth: 900, margin: "0 auto", padding: "22px 16px 60px" }}>
          <Blackboard />
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16 }}>
            <SeatGrid isAdmin={false} />
          </div>
          <div style={{ display: "flex", gap: 16, justifyContent: "center", marginTop: 20, flexWrap: "wrap" }}>
            <Legend color="#fff" border="#ddd" label="可選" />
            <Legend color="#5b9bd5" label="已選（✓）" />
            <Legend color="#ffd700" border="#f0b800" label="您的座位" />
          </div>
        </div>
      </div>
    );
  }

  // ── 頁面：確認選位 ──
  if (page === "confirm" && pendingSeat) {
    const z = ZONES.find((z) => z.id === pendingSeat.zone);
    return (
      <div style={S.page}><Toaster /><Overlay />
        <div style={S.card}>
          <div style={{ fontSize: "2.5rem", marginBottom: 8 }}>🪑</div>
          <h2 style={{ margin: "0 0 4px", fontFamily: "'Georgia',serif", color: "#222" }}>確認選位</h2>
          <p style={{ color: "#999", fontSize: "0.88rem", marginBottom: 20 }}>請確認以下資訊是否正確</p>

          <div style={{ background: z.light, border: `2px solid ${z.color}`, borderRadius: 12, padding: 20, marginBottom: 14, textAlign: "left" }}>
            <InfoRow label="姓名" value={myName} />
            <InfoRow label="區域" value={z.label} color={z.color} />
            <InfoRow label="座號" value={`${pendingSeat.num} 號`} />
            <InfoRow label="報名順序" value={`第 ${nextOrder} 位`} last />
          </div>

          <div style={{ display: "flex", gap: 12 }}>
            <button onClick={() => setPage("pick")} style={S.btnOutline}>取消</button>
            <button onClick={handleConfirm} style={{ ...S.btn, flex: 1, background: z.color }}>✓ 確認選位</button>
          </div>
        </div>
      </div>
    );
  }

  // ── 頁面：選位完成 ──
  if (page === "done" && pendingSeat) {
    const z = ZONES.find((z) => z.id === pendingSeat.zone);
    return (
      <div style={S.page}>
        <div style={S.card}>
          <div style={{ fontSize: "3rem", marginBottom: 8 }}>🎉</div>
          <h2 style={{ margin: "0 0 4px", fontFamily: "'Georgia',serif", color: "#222" }}>選位成功！</h2>
          <div style={{ background: z.color, borderRadius: 14, padding: "22px 20px", color: "#fff", marginBottom: 14 }}>
            <div style={{ fontSize: "1.9rem", fontWeight: 900, marginBottom: 4 }}>{myName}</div>
            <div style={{ fontSize: "1.05rem", opacity: 0.9 }}>{z.label} · {pendingSeat.num} 號座位</div>
          </div>
          <button onClick={() => setPage("pick")} style={{ ...S.btnOutline, width: "100%" }}>查看座位圖 →</button>
        </div>
      </div>
    );
  }

  // ── 頁面：管理員 ──
  return (
    <div style={{ minHeight: "100vh", background: "#f5f5f0", fontFamily: "sans-serif" }}>
      <Toaster /><Overlay />

      {/* 清除確認 Modal */}
      {confirmReset && (() => {
        const seat = seats[confirmReset];
        const z = ZONES.find((z) => z.id === seat.zone);
        return (
          <div style={S.modalBg}>
            <div style={S.modal}>
              <div style={{ fontSize: "2rem", marginBottom: 10 }}>🗑</div>
              <div style={{ fontWeight: 800, fontSize: "1.05rem", marginBottom: 10 }}>確認清除座位？</div>
              <div style={{ background: z.light, border: `1.5px solid ${z.color}`, borderRadius: 10, padding: 14, marginBottom: 18 }}>
                <div style={{ fontWeight: 700, color: z.color, fontSize: "0.95rem" }}>{z.label} · {seat.num} 號</div>
                <div style={{ color: "#444", marginTop: 4, fontSize: "0.9rem" }}>{seat.student}</div>
              </div>
              <div style={{ display: "flex", gap: 10 }}>
                <button onClick={() => setConfirmReset(null)} style={{ ...S.btnOutline, flex: 1 }}>取消</button>
                <button onClick={handleClearSeat} style={{ ...S.btn, flex: 1, background: "#e74c3c" }}>確認清除</button>
              </div>
            </div>
          </div>
        );
      })()}

      {/* 匯出 Modal */}
      {showExport && (
        <div style={S.modalBg}>
          <div style={{ ...S.modal, maxWidth: 500 }}>
            <div style={{ fontWeight: 800, fontSize: "1.05rem", marginBottom: 6 }}>📋 匯出名單</div>
            <textarea readOnly value={exportText} onClick={(e) => e.target.select()}
              style={{ width: "100%", height: 220, padding: 12, border: "1.5px solid #ddd", borderRadius: 10, fontSize: "0.83rem", fontFamily: "monospace", resize: "none", boxSizing: "border-box", outline: "none", background: "#f8f8f8", color: "#333" }} />
            <div style={{ display: "flex", gap: 10, marginTop: 14 }}>
              <button onClick={() => { navigator.clipboard.writeText(exportText); toast$("已複製！貼到 Excel 即可"); }} style={{ ...S.btn, flex: 1, background: "#27ae60" }}>📋 一鍵複製</button>
              <button onClick={() => setShowExport(false)} style={{ ...S.btnOutline, flex: 1 }}>關閉</button>
            </div>
          </div>
        </div>
      )}

      <Header subtitle={`🔓 管理員模式 · 已選 ${takenCount} 位 · 每 ${POLL_MS/1000} 秒自動更新`} />

      <div style={{ maxWidth: 1120, margin: "0 auto", padding: "24px 16px 60px", display: "grid", gridTemplateColumns: "300px 1fr", gap: 24, alignItems: "start" }}>

        {/* 左欄：控制面板 */}
        <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
          
          {/* 新增功能：名單自動生成碼 */}
          <div style={S.panel}>
            <div style={S.panelTitle}>➕ 生成學生通行碼</div>
            <div style={{ display: "flex", gap: 8 }}>
              <input value={newStudentName} onChange={(e) => setNewStudentName(e.target.value)}
                placeholder="輸入學生姓名" style={{ ...S.input, padding: "8px 12px", fontSize: "0.9rem" }} />
              <button onClick={handleAddStudent} style={{ ...S.btn, background: "#1a1a2e", padding: "8px 14px", fontSize: "0.9rem", whiteSpace: "nowrap" }}>生成</button>
            </div>
          </div>

          {/* 學生狀態名單 */}
          <div style={S.panel}>
            <div style={S.panelTitle}>👥 學生名單總覽 ({studentList.length} 人)</div>
            <div style={{ maxHeight: "350px", overflowY: "auto" }}>
              {studentList.map(({ name, code }) => {
                const isUsed = !!used[name];
                const seatEntry = Object.values(seats).find((s) => s.student === name);
                return (
                  <div key={name} style={{ display: "flex", alignItems: "center", gap: 8, padding: "6px 8px", borderRadius: 8, marginBottom: 5, background: isUsed ? "#f0faf5" : "#f9f9f9", border: `1.5px solid ${isUsed ? "#4caf82" : "#e8e8e8"}` }}>
                    <span style={{ fontWeight: 700, fontSize: "0.85rem", flex: 1, color: "#333" }}>{name}</span>
                    <span style={{ fontFamily: "monospace", fontSize: "0.8rem", color: "#666", background: "#eee", padding: "2px 6px", borderRadius: 4 }}>{code}</span>
                    <span style={{ fontSize: "0.78rem", fontWeight: 700, color: isUsed ? "#4caf82" : "#ccc", whiteSpace: "nowrap" }}>
                      {isUsed ? (seatEntry ? `${seatEntry.zone}-${seatEntry.num}` : "✓") : "未選"}
                    </span>
                  </div>
                );
              })}
              {studentList.length === 0 && <div style={{ color: "#aaa", fontSize: "0.85rem", textAlign: "center", padding: 10 }}>目前尚無學生資料</div>}
            </div>
          </div>

          <div style={{ ...S.panel, gap: 10, display: "flex", flexDirection: "column" }}>
            <button onClick={handleExport} style={{ ...S.btn, background: "#27ae60" }}>📥 匯出選位結果</button>
            <button onClick={() => { if(confirm("確定要全面清空所有座位與選位狀態嗎？")) handleResetAll(); }} style={{ ...S.btn, background: "#e74c3c" }}>🗑 全部重置系統</button>
            <button onClick={() => setPage("enter")} style={S.btnOutline}>退出管理頁面</button>
          </div>
        </div>

        {/* 右欄：座位圖 */}
        <div>
          <div style={{ background: "#fff8e1", border: "2px solid #ffc107", borderRadius: 12, padding: "10px 18px", marginBottom: 16, color: "#856404", fontWeight: 600, fontSize: "0.87rem" }}>
            💡 提示：點擊已被選取的座位可強制清除該位，清除後學生刷新可重新選位。
          </div>
          <Blackboard />
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14, marginBottom: 24 }}>
            <SeatGrid isAdmin={true} />
          </div>
        </div>

      </div>
    </div>
  );
}

function InfoRow({ label, value, color, last }) {
  return (
    <div style={{ display: "flex", justifyContent: "space-between", paddingBottom: last ? 0 : 10, marginBottom: last ? 0 : 10, borderBottom: last ? "none" : "1px solid rgba(0,0,0,0.06)" }}>
      <span style={{ color: "#999", fontSize: "0.87rem" }}>{label}</span>
      <span style={{ fontWeight: 700, color: color || "#222", fontSize: "0.93rem" }}>{value}</span>
    </div>
  );
}

function Legend({ color, border, label }) {
  return (
    <div style={{ display: "flex", alignItems: "center", gap: 7, fontSize: "0.83rem", color: "#777" }}>
      <div style={{ width: 26, height: 20, borderRadius: 5, background: color, border: `1.5px solid ${border || color}` }} />
      {label}
    </div>
  );
}

const S = {
  page: { minHeight: "100vh", background: "#f5f5f0", display: "flex", alignItems: "center", justifyContent: "center", fontFamily: "sans-serif", padding: 20 },
  card: { background: "#fff", borderRadius: 20, padding: "36px 32px", maxWidth: 400, width: "100%", textAlign: "center", boxShadow: "0 8px 40px rgba(0,0,0,0.1)" },
  panel: { background: "#fff", borderRadius: 14, padding: "16px 18px", boxShadow: "0 2px 12px rgba(0,0,0,0.06)" },
  panelTitle: { fontWeight: 800, fontSize: "0.95rem", color: "#222", marginBottom: 12 },
  label: { display: "block", textAlign: "left", fontSize: "0.82rem", color: "#666", fontWeight: 600, marginBottom: 6 },
  input: { width: "100%", padding: "12px 14px", border: "2px solid #e0e0e0", borderRadius: 10, fontSize: "1rem", outline: "none", boxSizing: "border-box", fontFamily: "inherit" },
  btn: { padding: "12px 20px", borderRadius: 10, border: "none", color: "#fff", fontWeight: 800, fontSize: "1rem", cursor: "pointer", fontFamily: "inherit" },
  btnOutline: { flex: 1, padding: "12px 20px", borderRadius: 10, border: "2px solid #ddd", color: "#666", fontWeight: 700, fontSize: "1rem", cursor: "pointer", background: "#fff", fontFamily: "inherit" },
  ghost: { background: "none", border: "none", color: "#ccc", cursor: "pointer", fontSize: "0.82rem", fontFamily: "inherit" },
  modalBg: { position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)", zIndex: 300, display: "flex", alignItems: "center", justifyContent: "center", padding: 20 },
  modal: { background: "#fff", borderRadius: 18, padding: "28px 24px", maxWidth: 340, width: "100%", textAlign: "center", boxShadow: "0 8px 40px rgba(0,0,0,0.18)" },
};
