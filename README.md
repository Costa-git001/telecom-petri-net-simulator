# 📡 telecom-petri-net-simulator

> An interactive, browser-based Petri Net simulator modelling **Call Handling, Bandwidth Allocation, and Traffic Class Prioritization** in a Next-Generation Telecommunications Core Network.

Built as part of coursework for **ICS 2313 – Software Testing & Quality Assurance** at [Jomo Kenyatta University of Agriculture and Technology (JKUAT)](https://www.jkuat.ac.ke/).

---

## 🖼️ Preview

> Open `telecom_petri_net.html` in any modern browser — no installation required.

The simulator features:
- **Hierarchical Petri Net (HPN)** — top-level orchestration with substitution transitions
- **Colored Petri Net (CPN)** — tokens carry full color tuples `(class, session_id, BU, priority)`
- **Live simulation** — step-by-step or auto-run with adjustable speed
- **Token injection** — inject Voice, Data, Video, or Emergency sessions at runtime
- **Properties panel** — all Murata (1989) properties measured and displayed in real time

---

## 📋 Use Case

A Next-Generation Telecom Core Network receives incoming session requests across four traffic classes:

| Class | Priority | Bandwidth Units (BU) | Description |
|-------|----------|----------------------|-------------|
| 🔴 Emergency (E) | 4 (highest) | 20 BU | Pre-empts all; guaranteed QoS |
| 🔵 Voice (V) | 3 | 5 BU | Real-time, low latency |
| 🟣 Video (T) | 2 | 10 BU | Adaptive bitrate streaming |
| 🟠 Data (D) | 1 (lowest) | 3 BU | Best-effort, elastic |

The shared **bandwidth pool = 50 BU**. Sessions progress through:

```
Arrival → Classification → Priority Queue → Admission Control
       → Bandwidth Allocation → Active Session → QoS Monitoring
       → Release / Pre-emption / Degradation → BW Return
```

---

## 🔷 Hierarchical Petri Net (HPN)

### Top-Level Net — 10 Transitions, 12 Places

| ID | Place | Description |
|----|-------|-------------|
| P1 | Arrival Queue | Incoming call/session requests |
| P2 | Classifier | Traffic class detection |
| P3 | Emergency Queue | Priority-4 holding buffer |
| P4 | Voice Queue | Priority-3 holding buffer |
| P5 | Video Queue | Priority-2 holding buffer |
| P6 | Data Queue | Priority-1 holding buffer |
| P7 | Admit Control | Admission decision point |
| P8 | BW Pool (50 BU) | Shared bandwidth resource |
| P9 | Active Sessions | Currently live sessions |
| P10 | QoS Monitor | Under quality monitoring |
| P11 | Congestion Control | Overload management |
| P12 | Release | Session teardown |

| ID | Transition | Type | Description |
|----|-----------|------|-------------|
| T1 | Arrive | Normal | New session enters system |
| T2 | Classify-E | Normal | Route to Emergency queue |
| T3 | Classify-V | Normal | Route to Voice queue |
| T4 | Classify-T | Normal | Route to Video queue |
| T5 | Classify-D | Normal | Route to Data queue |
| T6 | Admit | **Substitution** | Admission control sub-net |
| T7 | Activate | Normal | Move to active state |
| T8 | Monitor | **Substitution** | QoS monitoring sub-net |
| T9 | Drop | Normal | Congestion-triggered drop |
| T10 | Release | Normal | Session ends, BW returned |

### Sub-Net A: Admission Control
Expands T6. Evaluates `bw_free ≥ bu(class)`, routes to admit or reject path.

### Sub-Net B: Bandwidth Manager
Expands T8. Handles allocation confirmation, deallocation, and pool return.

---

## 🎨 Colored Petri Net (CPN)

### Color Set Definitions

```sml
COLSET Class = {E, V, T, D};
COLSET BW    = INT;
COLSET Token = product Class * INT * BW * INT;
               (* (class, session_id, bu_demand, priority) *)

fun priorityOf E = 4 | priorityOf V = 3 | priorityOf T = 2 | priorityOf D = 1;
fun buOf       E = 20 | buOf V = 5 | buOf T = 10 | buOf D = 3;
```

### Guard Functions

| Transition | Guard |
|-----------|-------|
| CT3 Allocate BW | `[bw_free ≥ buOf(class)]` |
| CT10 Pre-empt | `[bw_free < buOf(incoming)] ∧ ∃ lower-priority active` |
| CT8 QoS Pass | `[class = E ∨ class = V]` |
| CT9 Degrade | `[class = T ∨ class = D]` |
| CT2 Dequeue | Priority-ordered: highest `priorityOf(class)` first |

### P-Invariant (Conservation Law)

```
BW_Pool + Σ(BU_per_active_session) = 50   ∀ reachable markings
```

This invariant is verified live in the Properties panel.

---

## 📊 Petri Net Properties (Murata 1989)

| Property | Status | Telecom Interpretation |
|----------|--------|----------------------|
| **Reachability** | ✅ Yes | Every valid session combination is reachable from idle |
| **Boundedness** | ✅ 10-bounded | BW pool capped; no place accumulates unboundedly |
| **Safeness** | ⚠️ Partial | Control places are safe; resource places intentionally multi-token |
| **Liveness** | ✅ L3 quasi-live | All transitions fire in every full simulation run |
| **Reversibility** | ✅ Yes | Network returns to idle after all sessions release |
| **Fairness** | ⚠️ Bounded-fair | Emergency/Voice always served; Data bounded-unfair by design |
| **Coverability** | ✅ Yes | Any valid BW allocation ≤ 50 BU is coverable |
| **Deadlock-freedom** | ⚠️ Conditional | Deadlock only under full congestion; resolved by pre-emption/timeout |
| **Persistence** | ❌ No | Voice and Data compete for BW pool — deliberate resource conflict |
| **Structural Boundedness** | ✅ Yes | P-invariant confirms BW conservation; no positive circuit in incidence matrix |

---

## 🚀 Running the Simulator

```bash
# Clone the repo
git clone https://github.com/Costa-git001/telecom-petri-net-simulator.git
cd telecom-petri-net-simulator

# Open in browser (no build step needed)
open telecom_petri_net.html
# or on Linux:
xdg-open telecom_petri_net.html
# or just double-click the file in your file manager
```

> **Requirements:** Any modern browser (Chrome, Firefox, Edge, Safari). No Node.js, Python, or internet connection needed.

---

## 🗂️ Repository Structure

```
telecom-petri-net-simulator/
│
├── telecom_petri_net.html   # Main simulator (self-contained, single file)
├── README.md                # This file
├── .gitignore               # Standard ignores
└── docs/
    └── petri_net_report.md  # Detailed academic writeup (optional)
```

---

## 🧠 Key Concepts Demonstrated

- **Hierarchical decomposition** of complex systems using substitution transitions
- **Colored tokens** carrying structured data (class, id, bu, priority)
- **Guard functions** enforcing QoS and admission policies
- **Conflict and choice** between competing resource consumers (non-persistence)
- **Conservation laws** expressed as P-invariants
- **Fairness analysis** using Jain's Fairness Index computed at runtime
- **Reachability** through interactive state exploration

---

## 📚 References

- Murata, T. (1989). *Petri nets: Properties, analysis and applications*. Proceedings of the IEEE, 77(4), 541–580.
- Jensen, K. (1997). *Coloured Petri Nets: Basic Concepts, Analysis Methods and Practical Use* (Vol. 1). Springer.
- ITU-T Recommendation G.1010 (2001). *End-user multimedia QoS categories*.

---

👤 Author

John Nyongesa
BCT Year 3 — Jomo Kenyatta University of Agriculture and Technology

- GitHub: [@Costa-git001](https://github.com/Costa-git001)
- LinkedIn: [linkedin.com/in/john-nyongesa](https://linkedin.com/in/john-nyongesa)

---

## 📄 License

This project is submitted as academic coursework. Feel free to study and reference it — please cite appropriately if you build on it.
