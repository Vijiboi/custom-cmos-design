# Custom CMOS Logic, Memory Cell & Standard Cell Characterization (GPDK 90nm)

Transistor-level design, transient simulation, and parametric characterization executed in **Cadence Virtuoso (Spectre / ADE-L)** using the **Cadence GPDK090 (90nm)** bulk CMOS process node.

---

## 1. 6T SRAM & 1T-1C DRAM Cell Analysis

### 6T SRAM Bitcell Architecture
* **Technology:** Cadence GPDK090 (90nm bulk planar CMOS)
* **Supply Voltage ($V_{DD}$):** $1.2\text{ V}$ nominal
* **Transistor Sizing & Ratios:**
  * **Driver NMOS (PD):** $W/L = 240\text{ nm} / 100\text{ nm}$
  * **Access NMOS (PG):** $W/L = 120\text{ nm} / 100\text{ nm}$
  * **Load PMOS (PU):** $W/L = 120\text{ nm} / 100\text{ nm}$
  * **Cell Ratio ($\beta$):** $\beta = \frac{(W/L)_{PD}}{(W/L)_{PG}} = 2.0$ (ensures read stability and prevents read-upset)
  * **Pull-Up Ratio ($\gamma$):** $\gamma = \frac{(W/L)_{PU}}{(W/L)_{PG}} = 1.0$ (satisfies write-ability margin)
* **Static Noise Margin ($SNM$):** Measured Read $SNM \approx 195\text{ mV}$; Hold $SNM \approx 340\text{ mV}$.

### SRAM Transient Verification & Signal Mapping
![SRAM Transient Waveform](docs/sram_transient_analysis.png)

| Waveform Net | Circuit Signal | Role | Transient Dynamics & Performance |
| :--- | :--- | :--- | :--- |
| `/net1` | `WL` (Wordline) | Cell Select | $0\text{ V} \rightarrow 1.2\text{ V}$ pulse controlling pass-gate NMOS conduction. |
| `/net13` | `BL` (Bitline) | Data Line | Precharged to $V_{DD}$; differential discharge verified during Read '0'. |
| `/net15` | `BLB` (Bitline Bar) | Inverted Data | Precharged to $V_{DD}$; driven to $0\text{ V}$ during Write '1' sequence. |
| `/net3` | `Q` | Storage Node A | Internal node tracking logic state; immune to ground-bounce upset during Read. |
| `/net7` | `Q_BAR` | Storage Node B | Complementary internal node; fast pull-down via regenerative feedback loop. |

### 1T-1C DRAM Charge-Sharing Dynamics
* **Storage Cell:** Access NMOS ($W/L = 120\text{ nm} / 100\text{ nm}$) connected to an integrated storage capacitor ($C_S = 30\text{ fF}$).
* **Bitline Capacitance:** Lumped bitline parasitic modeled at $C_{BL} = 150\text{ fF}$.
* **Read Voltage Sensing Swing:**
  $$\Delta V_{BL} = \frac{C_S}{C_S + C_{BL}} \cdot \left(V_{DD} - \frac{V_{DD}}{2}\right) \approx 100\text{ mV}$$
* **Retention:** Evaluated subthreshold leakage discharge rate ($I_{\text{sub}}$) to define maximum refresh cycle period ($t_{\text{refresh}}$).

---

## 2. Transistor DC Characterization & Inverter VTC

### Device Extraction (NMOS & PMOS)
* **Test Conditions:** $V_{DS}$ swept from $0\text{ V}$ to $1.2\text{ V}$ across stepped $V_{GS}$ ($0.3\text{ V} \text{ to } 1.2\text{ V}$).
* **Extracted Parameters ($W/L = 120\text{ nm} / 100\text{ nm}$):**
  * **Threshold Voltage:** $V_{th,n} \approx +0.31\text{ V}$, $|V_{th,p}| \approx -0.34\text{ V}$
  * **Transconductance ($g_m$):** Peak $g_m \approx 105\ \mu\text{S}$ at saturation ($V_{GS} = V_{DS} = 1.2\text{ V}$)
  * **Drain Saturation Current ($I_{D,\text{sat}}$):** $\approx 78\ \mu\text{A}$ (NMOS), $\approx 32\ \mu\text{A}$ (PMOS)
  * **Mobility Ratio ($\mu_n / \mu_p$):** $\approx 2.45$

![MOSFET DC Characteristics](docs/mosfet_dc_characteristics.png)

### Symmetrical CMOS Inverter VTC
* **Sizing:** Sized $(W/L)_p = 300\text{ nm} / 100\text{ nm}$ and $(W/L)_n = 120\text{ nm} / 100\text{ nm}$ ($W_p/W_n = 2.5$) to compensate for carrier mobility disparity.
* **Switching Threshold ($V_M$):** $V_M \approx 0.59\text{ V} \approx V_{DD}/2$.
* **Static Noise Margins:**
  * $V_{IL} = 0.48\text{ V}$, $V_{IH} = 0.72\text{ V}$
  * $NML = V_{IL} - V_{OL} = 0.48\text{ V} - 0.02\text{ V} = 0.46\text{ V}$
  * $NMH = V_{OH} - V_{IH} = 1.18\text{ V} - 0.72\text{ V} = 0.46\text{ V}$

---

## 3. High-Speed Clock Generation: 3-Stage Ring Oscillator

### Topology & Transient Analysis
* **Architecture:** Closed-loop odd-stage inverter chain ($N = 3$) satisfying Barkhausen phase-shift criteria ($180^\circ$ DC inversion $+ 180^\circ$ loop phase lag).
* **Oscillation Frequency ($f_{osc}$):** $5.12\text{ GHz}$ under nominal self-loaded condition ($V_{DD} = 1.2\text{ V}$, $T = 27^\circ\text{C}$).
* **Per-Stage Propagation Delay ($t_{pd}$):**
  $$t_{pd} = \frac{1}{2 \cdot N \cdot f_{osc}} = \frac{1}{2 \cdot 3 \cdot 5.12 \times 10^9} \approx 32.6\text{ ps}$$
* **Sensitivity:** Measured $f_{osc}$ scaling across supply variations ($V_{DD} = 0.9\text{ V} \text{ to } 1.3\text{ V}$) to determine frequency-voltage push factor.

---

## 4. Digital Combinational Logic Primitives

Dynamic timing analysis evaluated across complementary CMOS gate topologies driving a nominal capacitive load ($C_L = 10\text{ fF}$):

| Logic Primitive | Sizing Scheme ($W_p / W_n$) | Rise Time $t_r$ ($10\%\text{--}90\%$) | Fall Time $t_f$ ($90\%\text{--}10\%$) | Avg Propagation Delay $t_{pd}$ |
| :--- | :--- | :--- | :--- | :--- |
| **CMOS Inverter** | $300\text{ nm} / 120\text{ nm}$ | $36\text{ ps}$ | $31\text{ ps}$ | $28.5\text{ ps}$ |
| **2-Input NAND** | $300\text{ nm} / 240\text{ nm}$ (stacked NMOS) | $39\text{ ps}$ | $52\text{ ps}$ | $48.2\text{ ps}$ |
| **2-Input NOR** | $600\text{ nm} / 120\text{ nm}$ (stacked PMOS) | $82\text{ ps}$ | $35\text{ ps}$ | $74.6\text{ ps}$ |
| **2-Input XOR** | Complementary Pass/Static | $71\text{ ps}$ | $68\text{ ps}$ | $66.1\text{ ps}$ |
| **2-Input XNOR** | Transmission Gate Based | $68\text{ ps}$ | $65\text{ ps}$ | $63.4\text{ ps}$ |

---

## 5. Toolchain & Environment
* **Design & Schematics:** Cadence Virtuoso Schematic Editor (IC6.1.8)
* **Simulation Engine:** Spectre Circuit Simulator
* **Analysis Suite:** Analog Design Environment (ADE-L)
* **Technology PDK:** Cadence GPDK090 (Version 4.6, 90nm Generic Process Design Kit)