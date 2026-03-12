# materiaali-hallinta
import { useState } from "react";

const INITIAL_MATERIALS = [
  { id: 1, name: "Mäntylankkupalkit", category: "Raaka-aineet", unit: "kpl", stock: 240, minStock: 50, price: 12.50 },
  { id: 2, name: "Teräsprofiili 40x40", category: "Raaka-aineet", unit: "m", stock: 85, minStock: 100, price: 8.90 },
  { id: 3, name: "Turvalasi 4mm", category: "Raaka-aineet", unit: "m²", stock: 62, minStock: 30, price: 45.00 },
  { id: 4, name: "Ulko-ovi perusrunko", category: "Valmiit osat", unit: "kpl", stock: 18, minStock: 10, price: 180.00 },
  { id: 5, name: "Saranat RST", category: "Valmiit osat", unit: "sarja", stock: 94, minStock: 40, price: 22.00 },
  { id: 6, name: "Tiivistenauha EPDM", category: "Valmiit osat", unit: "m", stock: 420, minStock: 200, price: 3.20 },
];

const INITIAL_ORDERS = [
  { id: 101, supplier: "PuuTukku Oy", material: "Mäntylankkupalkit", qty: 200, unit: "kpl", status: "Tilattu", date: "2026-03-14", total: 2500 },
  { id: 102, supplier: "MetalliExpert", material: "Teräsprofiili 40x40", qty: 300, unit: "m", status: "Toimitettu", date: "2026-03-10", total: 2670 },
  { id: 103, supplier: "LasiMaailma Oy", material: "Turvalasi 4mm", qty: 50, unit: "m²", status: "Käsittelyssä", date: "2026-03-15", total: 2250 },
];

const CATEGORIES = ["Kaikki", "Raaka-aineet", "Valmiit osat", "Varastosaldot"];

export default function App() {
  const [view, setView] = useState("dashboard");
  const [materials, setMaterials] = useState(INITIAL_MATERIALS);
  const [orders, setOrders] = useState(INITIAL_ORDERS);
  const [filterCat, setFilterCat] = useState("Kaikki");
  const [showAddModal, setShowAddModal] = useState(false);
  const [showOrderModal, setShowOrderModal] = useState(false);
  const [adjustItem, setAdjustItem] = useState(null);
  const [adjustQty, setAdjustQty] = useState("");
  const [newMat, setNewMat] = useState({ name: "", category: "Raaka-aineet", unit: "kpl", stock: "", minStock: "", price: "" });
  const [newOrder, setNewOrder] = useState({ supplier: "", material: "", qty: "", unit: "kpl", date: "" });
  const [notification, setNotification] = useState(null);

  const lowStock = materials.filter(m => m.stock <= m.minStock);
  const totalValue = materials.reduce((s, m) => s + m.stock * m.price, 0);

  const notify = (msg, type = "success") => {
    setNotification({ msg, type });
    setTimeout(() => setNotification(null), 2500);
  };

  const handleAdjust = (item, delta) => {
    setMaterials(ms => ms.map(m => m.id === item.id ? { ...m, stock: Math.max(0, m.stock + delta) } : m));
    notify(`${item.name}: varasto päivitetty`);
    setAdjustItem(null);
    setAdjustQty("");
  };

  const handleAddMaterial = () => {
    if (!newMat.name || !newMat.stock) return;
    setMaterials(ms => [...ms, { ...newMat, id: Date.now(), stock: +newMat.stock, minStock: +newMat.minStock || 0, price: +newMat.price || 0 }]);
    setNewMat({ name: "", category: "Raaka-aineet", unit: "kpl", stock: "", minStock: "", price: "" });
    setShowAddModal(false);
    notify("Materiaali lisätty!");
  };

  const handleAddOrder = () => {
    if (!newOrder.supplier || !newOrder.material || !newOrder.qty) return;
    setOrders(os => [...os, { ...newOrder, id: Date.now(), qty: +newOrder.qty, status: "Tilattu", total: 0 }]);
    setNewOrder({ supplier: "", material: "", qty: "", unit: "kpl", date: "" });
    setShowOrderModal(false);
    notify("Tilaus lisätty!");
  };

  const filtered = filterCat === "Kaikki" ? materials : materials.filter(m => m.category === filterCat);

  const statusColor = (s) => ({ "Tilattu": "#f59e0b", "Toimitettu": "#22c55e", "Käsittelyssä": "#3b82f6" }[s] || "#888");

  return (
    <div style={{
      fontFamily: "'DM Sans', 'Segoe UI', sans-serif",
      background: "#0f1117",
      minHeight: "100vh",
      maxWidth: 430,
      margin: "0 auto",
      color: "#e8eaf0",
      position: "relative",
      overflow: "hidden"
    }}>
      {/* BG accent */}
      <div style={{ position: "fixed", top: -80, right: -80, width: 260, height: 260, borderRadius: "50%", background: "radial-gradient(circle, rgba(234,88,12,0.18) 0%, transparent 70%)", pointerEvents: "none" }} />

      {/* Notification */}
      {notification && (
        <div style={{
          position: "fixed", top: 16, left: "50%", transform: "translateX(-50%)",
          background: notification.type === "success" ? "#22c55e" : "#ef4444",
          color: "#fff", borderRadius: 12, padding: "10px 22px", fontWeight: 700,
          fontSize: 14, zIndex: 1000, boxShadow: "0 4px 20px rgba(0,0,0,0.4)",
          whiteSpace: "nowrap"
        }}>
          {notification.msg}
        </div>
      )}

      {/* Header */}
      <div style={{ padding: "20px 20px 0", display: "flex", alignItems: "center", justifyContent: "space-between" }}>
        <div>
          <div style={{ fontSize: 11, color: "#ea580c", fontWeight: 700, letterSpacing: 2, textTransform: "uppercase" }}>OviTehdas Pro</div>
          <div style={{ fontSize: 22, fontWeight: 800, marginTop: 2 }}>Materiaalihallinta</div>
        </div>
        <div style={{ width: 40, height: 40, borderRadius: 12, background: "#ea580c", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 20 }}>🚪</div>
      </div>

      {/* Content */}
      <div style={{ padding: "20px 20px 90px" }}>

        {/* DASHBOARD */}
        {view === "dashboard" && (
          <div>
            {/* Alert */}
            {lowStock.length > 0 && (
              <div style={{ background: "rgba(239,68,68,0.12)", border: "1px solid rgba(239,68,68,0.3)", borderRadius: 14, padding: "12px 16px", marginBottom: 16, display: "flex", alignItems: "center", gap: 10 }}>
                <span style={{ fontSize: 20 }}>⚠️</span>
                <div>
                  <div style={{ fontWeight: 700, color: "#ef4444", fontSize: 13 }}>Alhainen varasto!</div>
                  <div style={{ fontSize: 12, color: "#fca5a5" }}>{lowStock.length} materiaalia alle minimivaraston</div>
                </div>
              </div>
            )}

            {/* Stats */}
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12, marginBottom: 16 }}>
              {[
                { label: "Materiaaleja", value: materials.length, icon: "📦", color: "#3b82f6" },
                { label: "Hälytyksiä", value: lowStock.length, icon: "🔔", color: "#ef4444" },
                { label: "Tilauksia", value: orders.length, icon: "🚚", color: "#f59e0b" },
                { label: "Var. arvo", value: `${(totalValue / 1000).toFixed(1)}k€`, icon: "💶", color: "#22c55e" },
              ].map(s => (
                <div key={s.label} style={{ background: "#1a1d27", borderRadius: 16, padding: "16px 14px", border: "1px solid #2a2d3a" }}>
                  <div style={{ fontSize: 22 }}>{s.icon}</div>
                  <div style={{ fontSize: 22, fontWeight: 800, color: s.color, marginTop: 4 }}>{s.value}</div>
                  <div style={{ fontSize: 12, color: "#888", marginTop: 2 }}>{s.label}</div>
                </div>
              ))}
            </div>

            {/* Low stock list */}
            {lowStock.length > 0 && (
              <div style={{ background: "#1a1d27", borderRadius: 16, padding: 16, border: "1px solid #2a2d3a", marginBottom: 16 }}>
                <div style={{ fontWeight: 700, fontSize: 14, marginBottom: 12, color: "#ef4444" }}>🔔 Hälytysmateriaalit</div>
                {lowStock.map(m => (
                  <div key={m.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
                    <div>
                      <div style={{ fontSize: 13, fontWeight: 600 }}>{m.name}</div>
                      <div style={{ fontSize: 11, color: "#888" }}>{m.category}</div>
                    </div>
                    <div style={{ textAlign: "right" }}>
                      <div style={{ fontSize: 14, fontWeight: 800, color: "#ef4444" }}>{m.stock} {m.unit}</div>
                      <div style={{ fontSize: 11, color: "#666" }}>min {m.minStock}</div>
                    </div>
                  </div>
                ))}
              </div>
            )}

            {/* Recent orders */}
            <div style={{ background: "#1a1d27", borderRadius: 16, padding: 16, border: "1px solid #2a2d3a" }}>
              <div style={{ fontWeight: 700, fontSize: 14, marginBottom: 12 }}>🚚 Viimeisimmät tilaukset</div>
              {orders.slice(-3).reverse().map(o => (
                <div key={o.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
                  <div>
                    <div style={{ fontSize: 13, fontWeight: 600 }}>{o.material}</div>
                    <div style={{ fontSize: 11, color: "#888" }}>{o.supplier}</div>
                  </div>
                  <div style={{ background: statusColor(o.status) + "22", color: statusColor(o.status), borderRadius: 8, padding: "3px 10px", fontSize: 11, fontWeight: 700 }}>{o.status}</div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* VARASTO */}
        {view === "varasto" && (
          <div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 14 }}>
              <div style={{ fontWeight: 700, fontSize: 17 }}>Varasto</div>
              <button onClick={() => setShowAddModal(true)} style={{ background: "#ea580c", color: "#fff", border: "none", borderRadius: 10, padding: "8px 14px", fontWeight: 700, fontSize: 13, cursor: "pointer" }}>+ Lisää</button>
            </div>
            {/* Category filter */}
            <div style={{ display: "flex", gap: 8, marginBottom: 16, overflowX: "auto", paddingBottom: 4 }}>
              {CATEGORIES.map(c => (
                <button key={c} onClick={() => setFilterCat(c)} style={{
                  background: filterCat === c ? "#ea580c" : "#1a1d27",
                  color: filterCat === c ? "#fff" : "#888",
                  border: "1px solid " + (filterCat === c ? "#ea580c" : "#2a2d3a"),
                  borderRadius: 10, padding: "6px 14px", fontSize: 12, fontWeight: 700, cursor: "pointer", whiteSpace: "nowrap"
                }}>{c}</button>
              ))}
            </div>

            {filtered.map(m => (
              <div key={m.id} style={{ background: "#1a1d27", borderRadius: 16, padding: 14, marginBottom: 10, border: `1px solid ${m.stock <= m.minStock ? "rgba(239,68,68,0.4)" : "#2a2d3a"}` }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 8 }}>
                  <div>
                    <div style={{ fontWeight: 700, fontSize: 14 }}>{m.name}</div>
                    <div style={{ fontSize: 11, color: "#888" }}>{m.category} • {m.price}€/{m.unit}</div>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontSize: 20, fontWeight: 800, color: m.stock <= m.minStock ? "#ef4444" : "#22c55e" }}>{m.stock}</div>
                    <div style={{ fontSize: 11, color: "#666" }}>{m.unit}</div>
                  </div>
                </div>
                {/* Stock bar */}
                <div style={{ background: "#0f1117", borderRadius: 6, height: 5, marginBottom: 10 }}>
                  <div style={{ background: m.stock <= m.minStock ? "#ef4444" : "#22c55e", borderRadius: 6, height: 5, width: `${Math.min(100, (m.stock / (m.minStock * 3)) * 100)}%` }} />
                </div>
                {adjustItem?.id === m.id ? (
                  <div style={{ display: "flex", gap: 8 }}>
                    <input type="number" value={adjustQty} onChange={e => setAdjustQty(e.target.value)} placeholder="Määrä" style={{ flex: 1, background: "#0f1117", border: "1px solid #2a2d3a", borderRadius: 8, padding: "6px 10px", color: "#fff", fontSize: 13 }} />
                    <button onClick={() => handleAdjust(m, +adjustQty)} style={{ background: "#22c55e", color: "#fff", border: "none", borderRadius: 8, padding: "6px 12px", fontWeight: 700, cursor: "pointer" }}>+</button>
                    <button onClick={() => handleAdjust(m, -(+adjustQty))} style={{ background: "#ef4444", color: "#fff", border: "none", borderRadius: 8, padding: "6px 12px", fontWeight: 700, cursor: "pointer" }}>−</button>
                    <button onClick={() => setAdjustItem(null)} style={{ background: "#2a2d3a", color: "#fff", border: "none", borderRadius: 8, padding: "6px 10px", cursor: "pointer" }}>✕</button>
                  </div>
                ) : (
                  <button onClick={() => setAdjustItem(m)} style={{ background: "#2a2d3a", color: "#e8eaf0", border: "none", borderRadius: 8, padding: "6px 14px", fontSize: 12, fontWeight: 600, cursor: "pointer", width: "100%" }}>Muokkaa varastoa</button>
                )}
              </div>
            ))}

            {/* Add modal */}
            {showAddModal && (
              <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.7)", zIndex: 100, display: "flex", alignItems: "flex-end" }}>
                <div style={{ background: "#1a1d27", borderRadius: "20px 20px 0 0", padding: 24, width: "100%", maxWidth: 430, margin: "0 auto" }}>
                  <div style={{ fontWeight: 800, fontSize: 18, marginBottom: 18 }}>Lisää materiaali</div>
                  {[["Nimi", "name", "text"], ["Varasto", "stock", "number"], ["Minimi", "minStock", "number"], ["Hinta €", "price", "number"]].map(([label, key, type]) => (
                    <div key={key} style={{ marginBottom: 12 }}>
                      <div style={{ fontSize: 12, color: "#888", marginBottom: 4 }}>{label}</div>
                      <input type={type} value={newMat[key]} onChange={e => setNewMat(n => ({ ...n, [key]: e.target.value }))}
                        style={{ width: "100%", background: "#0f1117", border: "1px solid #2a2d3a", borderRadius: 10, padding: "10px 12px", color: "#fff", fontSize: 14, boxSizing: "border-box" }} />
                    </div>
                  ))}
                  <div style={{ marginBottom: 16 }}>
                    <div style={{ fontSize: 12, color: "#888", marginBottom: 4 }}>Kategoria</div>
                    <select value={newMat.category} onChange={e => setNewMat(n => ({ ...n, category: e.target.value }))}
                      style={{ width: "100%", background: "#0f1117", border: "1px solid #2a2d3a", borderRadius: 10, padding: "10px 12px", color: "#fff", fontSize: 14 }}>
                      <option>Raaka-aineet</option><option>Valmiit osat</option>
                    </select>
                  </div>
                  <div style={{ display: "flex", gap: 10 }}>
                    <button onClick={handleAddMaterial} style={{ flex: 1, background: "#ea580c", color: "#fff", border: "none", borderRadius: 12, padding: 14, fontWeight: 800, fontSize: 15, cursor: "pointer" }}>Lisää</button>
                    <button onClick={() => setShowAddModal(false)} style={{ flex: 1, background: "#2a2d3a", color: "#fff", border: "none", borderRadius: 12, padding: 14, fontWeight: 700, cursor: "pointer" }}>Peruuta</button>
                  </div>
                </div>
              </div>
            )}
          </div>
        )}

        {/* TILAUKSET */}
        {view === "tilaukset" && (
          <div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16 }}>
              <div style={{ fontWeight: 700, fontSize: 17 }}>Tilaukset</div>
              <button onClick={() => setShowOrderModal(true)} style={{ background: "#ea580c", color: "#fff", border: "none", borderRadius: 10, padding: "8px 14px", fontWeight: 700, fontSize: 13, cursor: "pointer" }}>+ Uusi</button>
            </div>
            {orders.map(o => (
              <div key={o.id} style={{ background: "#1a1d27", borderRadius: 16, padding: 16, marginBottom: 10, border: "1px solid #2a2d3a" }}>
                <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
                  <div style={{ fontWeight: 700, fontSize: 14 }}>{o.material}</div>
                  <div style={{ background: statusColor(o.status) + "22", color: statusColor(o.status), borderRadius: 8, padding: "3px 10px", fontSize: 11, fontWeight: 700 }}>{o.status}</div>
                </div>
                <div style={{ fontSize: 12, color: "#888", marginBottom: 4 }}>🏭 {o.supplier}</div>
                <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, color: "#aaa" }}>
                  <span>📦 {o.qty} {o.unit}</span>
                  <span>📅 {o.date}</span>
                  {o.total > 0 && <span style={{ color: "#22c55e", fontWeight: 700 }}>{o.total}€</span>}
                </div>
              </div>
            ))}

            {showOrderModal && (
              <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.7)", zIndex: 100, display: "flex", alignItems: "flex-end" }}>
                <div style={{ background: "#1a1d27", borderRadius: "20px 20px 0 0", padding: 24, width: "100%", maxWidth: 430, margin: "0 auto" }}>
                  <div style={{ fontWeight: 800, fontSize: 18, marginBottom: 18 }}>Uusi tilaus</div>
                  {[["Toimittaja", "supplier", "text"], ["Materiaali", "material", "text"], ["Määrä", "qty", "number"], ["Päivämäärä", "date", "date"]].map(([label, key, type]) => (
                    <div key={key} style={{ marginBottom: 12 }}>
                      <div style={{ fontSize: 12, color: "#888", marginBottom: 4 }}>{label}</div>
                      <input type={type} value={newOrder[key]} onChange={e => setNewOrder(n => ({ ...n, [key]: e.target.value }))}
                        style={{ width: "100%", background: "#0f1117", border: "1px solid #2a2d3a", borderRadius: 10, padding: "10px 12px", color: "#fff", fontSize: 14, boxSizing: "border-box" }} />
                    </div>
                  ))}
                  <div style={{ display: "flex", gap: 10 }}>
                    <button onClick={handleAddOrder} style={{ flex: 1, background: "#ea580c", color: "#fff", border: "none", borderRadius: 12, padding: 14, fontWeight: 800, fontSize: 15, cursor: "pointer" }}>Tallenna</button>
                    <button onClick={() => setShowOrderModal(false)} style={{ flex: 1, background: "#2a2d3a", color: "#fff", border: "none", borderRadius: 12, padding: 14, fontWeight: 700, cursor: "pointer" }}>Peruuta</button>
                  </div>
                </div>
              </div>
            )}
          </div>
        )}

        {/* RAPORTTI */}
        {view === "raportti" && (
          <div>
            <div style={{ fontWeight: 700, fontSize: 17, marginBottom: 16 }}>Raportointi</div>
            <div style={{ background: "#1a1d27", borderRadius: 16, padding: 16, marginBottom: 12, border: "1px solid #2a2d3a" }}>
              <div style={{ fontWeight: 700, marginBottom: 12, color: "#ea580c" }}>📊 Varaston arvo kategorioittain</div>
              {["Raaka-aineet", "Valmiit osat"].map(cat => {
                const catMats = materials.filter(m => m.category === cat);
                const val = catMats.reduce((s, m) => s + m.stock * m.price, 0);
                const pct = totalValue ? (val / totalValue) * 100 : 0;
                return (
                  <div key={cat} style={{ marginBottom: 14 }}>
                    <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 4 }}>
                      <span style={{ fontSize: 13, fontWeight: 600 }}>{cat}</span>
                      <span style={{ fontSize: 13, color: "#22c55e", fontWeight: 700 }}>{val.toFixed(0)}€</span>
                    </div>
                    <div style={{ background: "#0f1117", borderRadius: 6, height: 8 }}>
                      <div style={{ background: "#ea580c", borderRadius: 6, height: 8, width: `${pct}%`, transition: "width 0.5s" }} />
                    </div>
                    <div style={{ fontSize: 11, color: "#666", marginTop: 2 }}>{pct.toFixed(1)}% kokonaisarvosta</div>
                  </div>
                );
              })}
              <div style={{ borderTop: "1px solid #2a2d3a", paddingTop: 12, display: "flex", justifyContent: "space-between" }}>
                <span style={{ fontWeight: 700 }}>Yhteensä</span>
                <span style={{ fontWeight: 800, color: "#22c55e", fontSize: 16 }}>{totalValue.toFixed(0)}€</span>
              </div>
            </div>

            <div style={{ background: "#1a1d27", borderRadius: 16, padding: 16, border: "1px solid #2a2d3a" }}>
              <div style={{ fontWeight: 700, marginBottom: 12, color: "#ea580c" }}>📋 Tilausten tila</div>
              {["Tilattu", "Käsittelyssä", "Toimitettu"].map(status => {
                const count = orders.filter(o => o.status === status).length;
                return (
                  <div key={status} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
                    <span style={{ fontSize: 13 }}>{status}</span>
                    <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                      <div style={{ background: statusColor(status) + "33", borderRadius: 20, height: 8, width: 80 }}>
                        <div style={{ background: statusColor(status), borderRadius: 20, height: 8, width: `${orders.length ? (count / orders.length) * 100 : 0}%` }} />
                      </div>
                      <span style={{ fontWeight: 700, color: statusColor(status), fontSize: 14, minWidth: 20 }}>{count}</span>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}
      </div>

      {/* Bottom Nav */}
      <div style={{
        position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)",
        width: "100%", maxWidth: 430, background: "#1a1d27",
        borderTop: "1px solid #2a2d3a", display: "flex", justifyContent: "space-around",
        padding: "10px 0 20px"
      }}>
        {[
          { id: "dashboard", icon: "🏠", label: "Etusivu" },
          { id: "varasto", icon: "📦", label: "Varasto" },
          { id: "tilaukset", icon: "🚚", label: "Tilaukset" },
          { id: "raportti", icon: "📊", label: "Raportit" },
        ].map(tab => (
          <button key={tab.id} onClick={() => setView(tab.id)} style={{
            background: "none", border: "none", cursor: "pointer",
            display: "flex", flexDirection: "column", alignItems: "center", gap: 2
          }}>
            <div style={{ fontSize: 22 }}>{tab.icon}</div>
            <div style={{ fontSize: 11, fontWeight: 700, color: view === tab.id ? "#ea580c" : "#555" }}>{tab.label}</div>
            {view === tab.id && <div style={{ width: 4, height: 4, borderRadius: "50%", background: "#ea580c" }} />}
          </button>
        ))}
      </div>
    </div>
  );
}
