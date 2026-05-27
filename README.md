import { useState, useEffect, useRef } from "react";

// ══════════════════════════════════════════════════════════════════════
// 設定區
// ══════════════════════════════════════════════════════════════════════
const API_URL = "https://script.google.com/macros/s/AKfycby_1upKpH2eGGJL2oCDJm7RsxiYSins8Eb2he8BtPbH8gPI9LDN3f90UdAy8NxPJcyD/exec";
const ADMIN_PIN = "631015";
const POLL_MS = 3000; // 3秒同步一次

// 學生名單（姓名: 通行碼）
const STUDENT_CODES = {
  "呂嘉螢": "U3P25",
  "陳刀":   "HAW2H",
  "金小數": "PWTSS",
  "陳止觀": "B9QAU",
  "薛寶":   "4MNCG",
  "張靖":   "G4FDK",
  "倪耀":   "3V2FY",
  "恰吉":   "WW92K",
};

// 座位區域設定
const ZONES = [
  { id: "A", label: "A 區", color: "#4caf82", light: "#e8f5ee", border: "#4caf82" },
  { id: "B", label: "B 區", color: "#5b9bd5", light: "#e3eef8", border: "#5b9bd5" },
  { id: "C", label: "C 區", color: "#d4a843", light: "#fdf6e3", border: "#d4a843" },
  { id: "D", label: "D 區", color: "#d46b6b", light: "#fceaea", border: "#d46b6b" },
];
const SEATS_PER_ZONE = 30;
const COLS = 5;

// ══════════════════════════════════════════════════════════════════════
// 工具
// ══════════════════════════════════════════════════════════════════════
function buildEmptySeats() {
  const map = {};
  ZONES.forEach((z) => {
    for (let i = 1; i <= SEATS_PER_ZONE; i++)
      map[`${z.id}-${i}`] = { zone: z.id, num: i, student: null, order: null };
  });
  return map;
}

async function callApi(params) {
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
    return { seats, used: data.used || {} };
  } catch {
    return null;
  }
}

// ══════════════════════════════════════════════════════════════════════
// 主元件
// ══════════════════════════════════════════════════════════════════════
export default function App() {
  const [seats, setSeats] = useState(buildEmptySeats);
  const [used, setUsed] = useState({});          // { 姓名: true }
  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);

  // 頁面：enter | pick | confirm | done | admin
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

  const [toast, setToast] = useState(null);
  const pollRef = useRef(null);

  // ── 初始載入 ────────────────────────────────────────────────────────
  useEffect(() => {
    fetchState().then((s) => {
      if (s) { setSeats(s.seats); setUsed(s.used); }
      setLoading(false);
    });
  }, []);

  // ── 輪詢（選位/查看/管理 頁面才啟動）──────────────────────────────
  useEffect(() => {
    clearInterval(pollRef.current);
    if (page === "pick" || page === "admin") {
      pollRef.current = setInterval(() => {
        fetchState().then((s) => {
          if (s) { setSeats(s.seats); setUsed(s.used); }
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
  const nextOrder = Object.values(seats).filter((s) => s.student).length + 1;

  // ── 學生：登入驗證 ──────────────────────────────────────────────────
  const handleVerify = async () => {
    const name = nameInput.trim();
    const code = codeInput.trim().toUpperCase();
    if (!name) { toast$("請輸入姓名", "warn"); return; }
    if (!code) { toast$("請輸入通行碼", "warn"); return; }
    if (!STUDENT_CODES[name]) { toast$("找不到此學生，請確認姓名是否正確", "warn"); return; }
    if (STUDENT_CODES[name] !== code) { toast$("通行碼錯誤，請再確認", "warn"); return; }
    setSaving(true);
    const s = await fetchState();
    if (s) { setSeats(s.seats); setUsed(s.used); }
    setSaving(false);
    setMyName(name);
    setViewOnly(!!(s?.used[name]));
    setPage("pick");
  };

  // ── 學生：點選座位 ──────────────────────────────────────────────────
  const handleSeatClick = (zone, num) => {
    if (viewOnly) return;
    if (seats[`${zone}-${num}`].student) return;
    setPendingSeat({ zone, num });
    setPage("confirm");
  };

  // ── 學生：確認選位 ──────────────────────────────────────────────────
  const handleConfirm = async () => {
    const key = `${pendingSeat.zone}-${pendingSeat.num}`;
    setSaving(true);
    try {
      const [r1, r2] = await Promise.all([
        callApi({ action: "saveSeat", key, student: myName, order: nextOrder }),
        callApi({ action: "markUsed", name: myName }),
      ]);
      console.log("saveSeat:", JSON.stringify(r1));
      console.log("markUsed:", JSON.stringify(r2));
      if (!r1.ok) { toast$("saveSeat 失敗：" + (r1.error || "unknown"), "warn"); setSaving(false); return; }
      const s = await fetchState();
      if (s) { setSeats(s.seats); setUsed(s.used); }
      setViewOnly(true);
      setPage("done");
    } catch(err) {
      toast$("錯誤：" + err.message, "warn");
    }
    setSaving(false);
  };

  // ── 管理員：清除座位 ────────────────────────────────────────────────
  const handleClearSeat = async () => {
    const key = confirmReset;
    const student = seats[key]?.student;
    setConfirmReset(null);
    setSaving(true);
    try {
      await callApi({ action: "clearSeat", key, student: student || "" });
      const s = await fetchState();
      if (s) { setSeats(s.seats); setUsed(s.used); }
      toast$("已清除，學生可重新選位");
    } catch { toast$("清除失敗", "warn"); }
    setSaving(false);
  };

  // ── 管理員：全部重置 ────────────────────────────────────────────────
  const handleResetAll = async () => {
    setSaving(true);
    try {
      await callApi({ action: "resetAll" });
      const s = await fetchState();
      if (s) { setSeats(s.seats); setUsed(s.used); }
      toast$("已重置所有座位");
    } catch { toast$("重置失敗", "warn"); }
    setSaving(false);
  };

  // ── 匯出 ────────────────────────────────────────────────────────────
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

  // ══════════════════════════════════════════════════════════════════
  // 共用元件
  // ══════════════════════════════════════════════════════════════════
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

  // 座位格子
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

  // ══════════════════════════════════════════════════════════════════
  // 載入中
  // ══════════════════════════════════════════════════════════════════
  if (loading) return (
    <div style={{ minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center", background: "#f5f5f0", fontFamily: "sans-serif" }}>
      <div style={{ textAlign: "center", color: "#999" }}>
        <div style={{ fontSize: "2.5rem", marginBottom: 12 }}>📚</div>
        <div>連線中⋯</div>
      </div>
    </div>
  );

  // ══════════════════════════════════════════════════════════════════
  // 頁面：登入
  // ══════════════════════════════════════════════════════════════════
  if (page === "enter") return (
    <div style={S.page}><Toaster /><Overlay />
      <div style={S.card}>
        <div style={{ fontSize: "2.8rem", marginBottom: 10 }}>📚</div>
        <h2 style={{ margin: "0 0 6px", fontFamily: "'Georgia',serif", color: "#1a1a2e", fontSize: "1.5rem" }}>台大補習班選位系統</h2>
        <p style={{ color: "#999", fontSize: "0.88rem", marginBottom: 28 }}>請輸入姓名與老師給您的通行碼</p>

        <label style={S.label}>姓名</label>
        <input value={nameInput} onChange={(e) => setNameInput(e.target.value)}
          placeholder="請輸入完整姓名"
          style={S.input}
          onFocus={(e) => e.target.style.borderColor = "#5b9bd5"}
          onBlur={(e) => e.target.style.borderColor = "#e0e0e0"} />

        <label style={{ ...S.label, marginTop: 14 }}>通行碼</label>
        <input value={codeInput}
          onChange={(e) => setCodeInput(e.target.value.toUpperCase())}
          onKeyDown={(e) => e.key === "Enter" && handleVerify()}
          placeholder="輸入通行碼（如 U3P25）" maxLength={8}
          style={{ ...S.input, textAlign: "center", letterSpacing: "0.3em", fontFamily: "monospace", fontSize: "1.15rem", fontWeight: 700 }}
          onFocus={(e) => e.target.style.borderColor = "#5b9bd5"}
          onBlur={(e) => e.target.style.borderColor = "#e0e0e0"} />

        <button onClick={handleVerify} style={{ ...S.btn, width: "100%", marginTop: 24, background: "#1a1a2e" }}>進入選位 →</button>

        <div style={{ marginTop: 28, borderTop: "1px solid #eee", paddingTop: 18 }}>
          {!showAdminLogin
            ? <button onClick={() => setShowAdminLogin(true)} style={S.ghost}>老師入口</button>
            : <div style={{ background: "#f8f8f8", borderRadius: 12, padding: 16 }}>
                <input type="password" value={pinInput} onChange={(e) => setPinInput(e.target.value)}
                  onKeyDown={(e) => { if (e.key === "Enter") { if (pinInput === ADMIN_PIN) { setPage("admin"); setPinInput(""); setPinError(""); setShowAdminLogin(false); } else setPinError("密碼錯誤"); }}}
                  placeholder="請輸入管理員密碼"
                  style={{ ...S.input, marginBottom: 8 }} />
                {pinError && <div style={{ color: "#e74c3c", fontSize: "0.82rem", marginBottom: 8 }}>{pinError}</div>}
                <button onClick={() => { if (pinInput === ADMIN_PIN) { setPage("admin"); setPinInput(""); setPinError(""); setShowAdminLogin(false); } else setPinError("密碼錯誤"); }}
                  style={{ ...S.btn, width: "100%", background: "#1a1a2e" }}>進入管理</button>
              </div>
          }
        </div>
      </div>
    </div>
  );

  // ══════════════════════════════════════════════════════════════════
  // 頁面：選位
  // ══════════════════════════════════════════════════════════════════
  if (page === "pick") {
    const mySeat = Object.values(seats).find((s) => s.student === myName);
    return (
      <div style={{ minHeight: "100vh", background: "#f5f5f0", fontFamily: "sans-serif" }}>
        <Toaster /><Overlay />
        <Header subtitle={`已選 ${takenCount} / ${ZONES.length * SEATS_PER_ZONE} 位 · 每 ${POLL_MS/1000} 秒自動更新`} />

        {/* 已選位提示條 */}
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
          <div style={{ textAlign: "center", marginTop: 14, color: "#bbb", fontSize: "0.82rem" }}>
            目前已有 {takenCount} 個座位被選走
          </div>
        </div>
      </div>
    );
  }

  // ══════════════════════════════════════════════════════════════════
  // 頁面：確認選位
  // ══════════════════════════════════════════════════════════════════
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

          <div style={{ background: "#fff8e1", border: "1.5px solid #ffc107", borderRadius: 10, padding: "12px 16px", marginBottom: 14, textAlign: "left" }}>
            <div style={{ fontWeight: 800, fontSize: "0.85rem", color: "#856404", marginBottom: 4 }}>⚠️ 注意事項</div>
            <div style={{ fontSize: "0.83rem", color: "#856404", lineHeight: 1.6 }}>
              確認後無法自行更改座位，如需異動請聯絡班導。
            </div>
          </div>

          <p style={{ fontSize: "0.8rem", color: "#bbb", marginBottom: 22 }}>🔒 座位圖上不會顯示您的姓名，僅老師可查看</p>

          <div style={{ display: "flex", gap: 12 }}>
            <button onClick={() => setPage("pick")} style={S.btnOutline}>取消</button>
            <button onClick={handleConfirm} style={{ ...S.btn, flex: 1, background: z.color }}>✓ 確認選位</button>
          </div>
        </div>
      </div>
    );
  }

  // ══════════════════════════════════════════════════════════════════
  // 頁面：選位完成
  // ══════════════════════════════════════════════════════════════════
  if (page === "done" && pendingSeat) {
    const z = ZONES.find((z) => z.id === pendingSeat.zone);
    const seat = seats[`${pendingSeat.zone}-${pendingSeat.num}`];
    return (
      <div style={S.page}>
        <div style={S.card}>
          <div style={{ fontSize: "3rem", marginBottom: 8 }}>🎉</div>
          <h2 style={{ margin: "0 0 4px", fontFamily: "'Georgia',serif", color: "#222" }}>選位成功！</h2>
          <p style={{ color: "#999", fontSize: "0.88rem", marginBottom: 22 }}>您的座位已登記完成</p>
          <div style={{ background: z.color, borderRadius: 14, padding: "22px 20px", color: "#fff", marginBottom: 14, boxShadow: `0 6px 24px ${z.color}55` }}>
            <div style={{ fontSize: "0.82rem", opacity: 0.85, marginBottom: 4 }}>報名第 {seat?.order} 位</div>
            <div style={{ fontSize: "1.9rem", fontWeight: 900, marginBottom: 4 }}>{myName}</div>
            <div style={{ fontSize: "1.05rem", opacity: 0.9 }}>{z.label} · {pendingSeat.num} 號座位</div>
          </div>
          <p style={{ fontSize: "0.8rem", color: "#bbb", marginBottom: 20 }}>
            🔒 座位圖上僅顯示「✓」，姓名不對外公開<br />
            下次可用同樣姓名＋通行碼查看座位
          </p>
          <button onClick={() => setPage("pick")} style={{ ...S.btnOutline, width: "100%" }}>查看座位圖 →</button>
        </div>
      </div>
    );
  }

  // ══════════════════════════════════════════════════════════════════
  // 頁面：管理員
  // ══════════════════════════════════════════════════════════════════
  const allStudents = Object.entries(STUDENT_CODES);
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
              <div style={{ background: z.light, border: `1.5px solid ${z.color}`, borderRadius: 10, padding: 14, marginBottom: 10 }}>
                <div style={{ fontWeight: 700, color: z.color, fontSize: "0.95rem" }}>{z.label} · {seat.num} 號</div>
                <div style={{ color: "#444", marginTop: 4, fontSize: "0.9rem" }}>{seat.student}</div>
              </div>
              <div style={{ background: "#f8f8f8", borderRadius: 8, padding: "10px 14px", fontSize: "0.82rem", color: "#555", marginBottom: 18, lineHeight: 1.6, textAlign: "left" }}>
                清除後該學生可用<b>原本的姓名＋通行碼</b>重新登入選位。
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
            <div style={{ fontSize: "0.82rem", color: "#999", marginBottom: 12 }}>點一鍵複製後貼到 Excel 或 Google 試算表（Tab 分隔自動對齊欄位）</div>
            <textarea readOnly value={exportText} onClick={(e) => e.target.select()}
              style={{ width: "100%", height: 220, padding: 12, border: "1.5px solid #ddd", borderRadius: 10, fontSize: "0.83rem", fontFamily: "monospace", resize: "none", boxSizing: "border-box", outline: "none", background: "#f8f8f8", color: "#333" }} />
            <div style={{ display: "flex", gap: 10, marginTop: 14 }}>
              <button onClick={() => { navigator.clipboard.writeText(exportText); toast$("已複製！貼到 Excel 即可"); }} style={{ ...S.btn, flex: 1, background: "#27ae60" }}>📋 一鍵複製</button>
              <button onClick={() => setShowExport(false)} style={{ ...S.btnOutline, flex: 1 }}>關閉</button>
            </div>
          </div>
        </div>
      )}

      <Header subtitle={`🔓 管理員模式 · 已選 ${takenCount} / ${ZONES.length * SEATS_PER_ZONE} 位 · 每 ${POLL_MS/1000} 秒自動更新`} />

      <div style={{ maxWidth: 1120, margin: "0 auto", padding: "24px 16px 60px", display: "grid", gridTemplateColumns: "280px 1fr", gap: 24, alignItems: "start" }}>

        {/* 左欄：控制面板 */}
        <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>

          {/* 學生狀態 */}
          <div style={S.panel}>
            <div style={S.panelTitle}>👥 學生選位狀態</div>
            {allStudents.map(([name, code]) => {
              const isUsed = !!used[name];
              const seatEntry = Object.values(seats).find((s) => s.student === name);
              const z = seatEntry ? ZONES.find((z) => z.id === seatEntry.zone) : null;
              return (
                <div key={name} style={{ display: "flex", alignItems: "center", gap: 8, padding: "8px 10px", borderRadius: 8, marginBottom: 5, background: isUsed ? "#f0faf5" : "#f9f9f9", border: `1.5px solid ${isUsed ? "#4caf82" : "#e8e8e8"}` }}>
                  <span style={{ fontWeight: 700, fontSize: "0.88rem", flex: 1, color: "#333" }}>{name}</span>
                  <span style={{ fontFamily: "monospace", fontSize: "0.75rem", color: "#bbb" }}>{code}</span>
                  <span style={{ fontSize: "0.78rem", fontWeight: 700, color: isUsed ? "#4caf82" : "#ccc", whiteSpace: "nowrap" }}>
                    {isUsed ? (seatEntry && z ? `${z.label}${seatEntry.num}號` : "✓") : "未選"}
                  </span>
                </div>
              );
            })}
          </div>

          {/* 操作 */}
          <div style={{ ...S.panel, gap: 10, display: "flex", flexDirection: "column" }}>
            <button onClick={handleExport} style={{ ...S.btn, background: "#27ae60" }}>📥 匯出名單</button>
            <button onClick={handleResetAll} style={{ ...S.btn, background: "#e74c3c" }}>🗑 全部重置座位</button>
            <button onClick={() => setPage("enter")} style={S.btnOutline}>退出管理</button>
          </div>
        </div>

        {/* 右欄：座位圖 */}
        <div>
          <div style={{ background: "#fff8e1", border: "2px solid #ffc107", borderRadius: 12, padding: "10px 18px", marginBottom: 16, color: "#856404", fontWeight: 600, fontSize: "0.87rem" }}>
            💡 點擊已選座位（顯示學生名字）可跳出確認視窗進行清除，清除後學生可重新選位
          </div>
          <Blackboard />
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 14, marginBottom: 24 }}>
            <SeatGrid isAdmin={true} />
          </div>

          {/* 報名順序 */}
          {takenCount > 0 && (
            <div style={S.panel}>
              <div style={S.panelTitle}>📋 報名順序（共 {takenCount} 人）</div>
              <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(155px, 1fr))", gap: 8 }}>
                {Object.values(seats).filter((s) => s.student).sort((a, b) => a.order - b.order).map((s) => {
                  const z = ZONES.find((z) => z.id === s.zone);
                  return (
                    <div key={`${s.zone}-${s.num}`} style={{ display: "flex", alignItems: "center", gap: 8, padding: "8px 12px", borderRadius: 8, background: z.light, border: `1.5px solid ${z.border}` }}>
                      <span style={{ fontWeight: 900, color: z.color, minWidth: 22, fontSize: "0.88rem" }}>{s.order}.</span>
                      <span style={{ fontWeight: 700, fontSize: "0.88rem", color: "#333" }}>{s.student}</span>
                      <span style={{ fontSize: "0.72rem", color: "#aaa", marginLeft: "auto" }}>{z.label} {s.num}</span>
                    </div>
                  );
                })}
              </div>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// ══════════════════════════════════════════════════════════════════════
// 小元件
// ══════════════════════════════════════════════════════════════════════
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

// ══════════════════════════════════════════════════════════════════════
// 樣式
// ══════════════════════════════════════════════════════════════════════
const S = {
  page: { minHeight: "100vh", background: "#f5f5f0", display: "flex", alignItems: "center", justifyContent: "center", fontFamily: "sans-serif", padding: 20 },
  card: { background: "#fff", borderRadius: 20, padding: "36px 32px", maxWidth: 400, width: "100%", textAlign: "center", boxShadow: "0 8px 40px rgba(0,0,0,0.1)" },
  panel: { background: "#fff", borderRadius: 14, padding: "16px 18px", boxShadow: "0 2px 12px rgba(0,0,0,0.06)" },
  panelTitle: { fontWeight: 800, fontSize: "0.95rem", color: "#222", marginBottom: 12 },
  label: { display: "block", textAlign: "left", fontSize: "0.82rem", color: "#666", fontWeight: 600, marginBottom: 6 },
  input: { width: "100%", padding: "12px 14px", border: "2px solid #e0e0e0", borderRadius: 10, fontSize: "1rem", outline: "none", boxSizing: "border-box", fontFamily: "inherit", transition: "border-color 0.2s" },
  btn: { padding: "12px 20px", borderRadius: 10, border: "none", color: "#fff", fontWeight: 800, fontSize: "1rem", cursor: "pointer", fontFamily: "inherit", letterSpacing: "0.02em" },
  btnOutline: { flex: 1, padding: "12px 20px", borderRadius: 10, border: "2px solid #ddd", color: "#666", fontWeight: 700, fontSize: "1rem", cursor: "pointer", background: "#fff", fontFamily: "inherit" },
  ghost: { background: "none", border: "none", color: "#ccc", cursor: "pointer", fontSize: "0.82rem", fontFamily: "inherit" },
  modalBg: { position: "fixed", inset: 0, background: "rgba(0,0,0,0.5)", zIndex: 300, display: "flex", alignItems: "center", justifyContent: "center", padding: 20 },
  modal: { background: "#fff", borderRadius: 18, padding: "28px 24px", maxWidth: 340, width: "100%", textAlign: "center", boxShadow: "0 8px 40px rgba(0,0,0,0.18)" },
};
