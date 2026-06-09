<!-- Part of the SkyDive·Live showcase — see README.md -->

# 📐 SkyDive·Live — The numbers

*This is why the project is not a hobby gamble: every critical path is **calculated, with margin** — and honestly split into what is formula/manufacturer spec and what still has to be **measured** (gates G1–G3). When a calculation changes, the change is documented here, not hidden.*

---

## 1 · Radio link — the budget

**Base chain (identical for every scenario):** 5 800 MHz · TX **+30 dBm (1 W)** · cable + LPF −3 dB · **FSPL @ 4 000 m = 119.8 dB** (Friis, derived) · ground patch +13 dBic (tracked on the jumper) · threshold for a *stable picture* **−90 dBm** (deliberately conservative — the VRX spec is −105 dBm, so there are ~15 dB of hidden reserve before total loss: the amber rows below mean *artefacts*, not black).

```
margin = 10.2 dB + G_TX(angle) − polarisation − body shadow
```

### ⚠️ Revision 2026-06: the geometry moved, so the numbers moved

Earlier revisions of this file quoted **+13.2 dB belly margin**. That figure was computed for a patch antenna in the **bottom** of the housing, boresight at the ground. The current **v5** design carries the patch **flush in the side end-cap** (snag-free, aluminium body as its ground plane) plus a λ/2 dipole in the top module. Re-running the budget for the real geometry gives smaller, honest numbers — and one new weak spot. Every assumption is marked.

| attitude | side patch | top dipole | **diversity (best of both)** | | old (bottom patch) |
|---|:--:|:--:|:--:|:--:|:--:|
| Belly, 4 km slant @ 45°, favourable heading | **+7.7 dB** | +1.7 | **≈ +8 dB** | 🟢 | +13.2 |
| Belly, 4 km slant @ 45°, worst heading | −5.3 | +1.7 | **≈ +2 dB** | 🟡 | +13.2 |
| Belly, directly above the DZ | −1.8 | −0.6 | **≈ −1 dB** | 🟠 | +13.2 |
| Head-down, station @ 45° elevation | −5.8 | −1.7 | **≈ −2 dB** | 🟠 | +2.0 |
| Head-down, near-nadir | ~−10 | ~−15 | **≈ −10 dB** | 🔴 | — |
| Sit / back | −2…+2 | 0…+2 | **0…+4 dB** | 🟡 | — |

**What the re-run actually says:**

1. **Belly becomes a heading lottery.** The side patch points one way; the jumper's heading decides whether the ground station sits in its beam (~13 dB spread). The dipole + diversity floor it at ≈ +2 dB.
2. **Head-down is carried by the dipole alone** — fluctuating *around* the threshold (≈ −2 dB central estimate), not a dead link: spin opens line-of-sight windows and HDZero re-syncs in ~100 ms. Near-nadir, both antennas sit in pattern nulls — there the link is honestly gone until geometry changes.
3. **Dominant assumptions** (each worth ±2.5…±5 dB): the front-to-side ratio of the patch *as installed* (−10…−15 dB assumed; aluminium plane + ASA window unmeasured) and head-down body shadow (−5…−15 dB literature, −8 dB used — the helmet is the lowest point of a head-down flyer, which argues for the mild end).

**The one measurement that kills the biggest unknowns:** a turntable pattern sweep of the *fully assembled* sender at 25 mW (licence-free SRD), RSSI logged at the HDZero VRX, both antennas, two cut planes. S11 on the VNA (gate G1-R) is the prerequisite; **S11 ≠ pattern** — the turntable is the real answer.

**If the margins stay thin, four levers (in cost order):** tilt the patch 45° down-and-out inside the end-cap (+5…+8 dB on the red belly rows, pure CAD change) · crossed RHCP turnstile instead of the linear dipole (+3 dB polarisation, fills the axial null) · a tracked high-gain ground dish (+5…+11 dB on *every* row) · a second receive site 200–500 m offset, Corliss/Vislink-style (+5…+10 dB against fades).

**Why head-down breaks a single antenna at all:** patch back-lobe −12 dB + body shadow −15 dB + polarisation −2 dB ≈ **−29 dB**. The answer is architectural, not incremental: two antennas at the sender **plus** diversity at the ground — the same recipe the only proven precedent (Corliss/Vislink 2016: 1 W COFDM + 4-head MRC diversity) used.

**Harmonics compliance:** ≥ 50 dBc required; the HDZero PA delivers 30–45 dBc → **a low-pass filter is mandatory** (Mini-Circuits LFCW-6000+, −1.6 dB, already in the chain above). Proof needs a ≥ 18 GHz analyzer (gate G1-R, open).

---

## 2 · Thermals — solved by the fall itself

**Dissipation:** at 1 W RF the VTX draws **~14 W** (HDZero spec 6–15 W) → **Q ≈ 13–14 W of heat** (1 W leaves as RF).

**Required thermal resistance:** `R_th = ΔT_limit / Q = 25 K / 14 W = 1.79 K/W`

| regime | condition | ΔT | case temp | target ΔT < 25 K? |
|---|---|---|---|---|
| ground, fan on | Q = 13 W | ~55–80 K | 80–105 °C | marginal |
| ground, passive aluminium | Q ≤ 10 W | ~25 K | ~50 °C | ✅ |
| **freefall, v = 50 m/s, aluminium** | Q = 14 W | **~6 K** | ~−9 °C (4 000 m) | **✅ comfortably** |

**The decisive lever:** in freefall the aluminium shell sits in ≥ 55 m/s ram-air → forced convection `h ≈ 90 W/m²K`, `R_th = 1/(90 · 0.026 m²) = 0.43 K/W` → `ΔT = 14 · 0.43 ≈ 6 K`. Thermal mass (C ≈ 170 J/K) buffers the 60 s of freefall easily — **doctrine: power the VTX up ≤ 10 min before exit; it starts cold.**

**Hardware protection** (ATtiny + NTC, circuit designed): fan at 45 °C · OSD warning 65 °C · **VTX cutoff at 75 °C** (latch-off, 5 K under the ~80 °C limit) · re-enable at 55 °C. The cutoff is a safety requirement, not a backlog item.

---

## 3 · Regulatory — the honest path

| criterion | value |
|---|---|
| band | 5 650–5 850 MHz |
| **event operation (championship demo)** | **PMSE short-term frequency assignment** (BNetzA) — the same mechanism broadcast productions use; application lead time ~15 working days |
| **development & testing** | **SRD licence-free**, 5 725–5 875 MHz @ 25 mW EIRP — every bench, turntable and field test runs here |
| amateur radio (Class E, 5 W PEP) | usable as an *experimentation* frame — but a public-display demo under amateur rules is a legal grey zone (§5 AFuG / §16 AFuV), so it is **not** the plan of record |
| harmonics | ≥ 50 dBc / ≤ −20 dBm → LPF mandatory |

*This project found its own legal problem before anyone else did: "just use the ham licence" does not survive contact with the public-display rules. PMSE for the event, 25 mW for everything before it.*

---

## 4 · Energy & mass

| parameter | value | basis |
|---|---|---|
| battery | 3S LiPo 850 mAh (low-temp) | BOM |
| current draw | **1.26 A @ 11.1 V** | 14 W / 11.1 V |
| runtime (theoretical) | **~40 min** | 850 mAh / 1 260 mA |
| runtime (80 % DoD) | **~32 min** | dimensioned |
| operating doctrine | ≤ 10 min active before exit | thermals |
| **sender mass** | **~200–250 g** | dimensioned |

---

## 5 · Verification status — honestly separated

| value | type | status |
|---|---|---|
| FSPL 119.8 dB | formula | ✅ derived |
| link margins (table above, v5 side-patch geometry) | calculated from specs + literature shadowing | ⏳ turntable pattern (25 mW) → field test (G2) |
| Q = 13–14 W | external measurement + spec | ⏳ measurement on the VTX (G1-T) |
| freefall ΔT ≈ 6 K | flat-plate convection | ⏳ flight test (G3) |
| runtime ~32 min | capacity calculation | ⏳ pack measurement |
| S11 < −10 dB, patch & dipole installed | theory/spec | ⏳ VNA (G1-R) |
| antenna decoupling S21 < −20 dB | assumption (1λ spacing) | ⏳ VNA (G1-R) |
| cutoff @ 75 °C | dimensioned + code | ⏳ hardware cycle test |

> **The stance:** calculate first, design with margin, then build and measure — and when a calculation changes, publish the change. Nothing here carries a "measured" badge yet; that is exactly what gates G1–G3 exist for. The next step is the built prototype, not another document.
