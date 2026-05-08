# ☀️ P2P Solar Energy Sharing Simulation Dashboard

A browser-based simulation tool for modeling **Peer-to-Peer (P2P) solar energy sharing** between households. Built for engineering students to explore how homes with solar panels and batteries can trade electricity with each other and reduce grid dependence. 🏠🔋⚡

![Dashboard Preview](https://img.shields.io/badge/Type-Simulation_Dashboard-blue)
![Standalone](https://img.shields.io/badge/Standalone-No_Install_Required-green)

---

## 🤔 What Does This Do?

Imagine 6 households, each with solar panels and batteries. During the day ☀️, some produce more electricity than they need. At night 🌙, they all need power. Instead of buying everything from the grid, **they can share energy with each other** — that's P2P energy sharing.

This dashboard simulates a **full year (365 days, 8,760 hours)** of energy generation, consumption, storage, and sharing between households, then calculates key metrics like cost of energy and self-sufficiency. 📊

---

## ⚡ P2P Logic (Priority Order)

For each hour of the simulation, every household follows this priority order:

```
1️⃣  PV GENERATED ELECTRICITY IS PRIORITY
    PV should cover almost all of household's power demands.

2️⃣  SAVED UP CHARGE AT BATTERY UNITS
    If PV isn't enough, discharge the home battery.
    Discharging stops when battery SoC reaches <= 20%.

3️⃣  P2P EXCESS ENERGY POOL
    If still short, request excess energy from P2P network.
    Households eligible to share must have SoC >= their assigned sharing %.

4️⃣  BUY FROM GRID (last/worst case scenario)
    Purchase remaining deficit from the utility grid.
```

🔄 **Surplus path** (when PV > demand): a portion of excess PV is sent to the **P2P pool first** (based on the household's Sharing % Excess PV), then any remainder charges the battery, and only what's left over is exported to the grid.

### 🎯 Priority-Based P2P Allocation

The P2P energy pool uses a **priority-ranked, two-stage** matching algorithm each hour:

1. **Identify** households with surplus and households with deficit
2. **Rank deficit households by priority** — highest energy need served first
3. **Stage 1 — Excess PV first**: Allocate available excess PV energy to demanders in priority order until either supply runs out or all demand is met
4. **Stage 2 — Battery shareable**: For any remaining unmet deficits, tap eligible households' battery shareable energy (subject to SoC threshold and daily cap) and allocate in priority order
5. **Any unmet deficit** falls through to the grid

---

## 🔋 Battery Conditions

- 🛑 **20% SoC Floor**: Discharging stops when at State of Charge (SoC) of ≤ 20% of each battery device. This applies to both self-use discharge and P2P sharing.
- 📉 **Depth of Discharge**: Only 80% of battery capacity is usable (20% reserved).
- ⚡ **Charge/Discharge Efficiency**: 92.2% each way (√0.85 round-trip).
- 🔋 **Default Battery Capacity**: 3.28 kWh per unit.
- ⏱️ **C-Rate**: Max charge/discharge rate is 50% of capacity per hour.

---

## 🤝 P2P Sharing Rules

### 📤 Supplying Energy (Surplus Households)
- ☀️ **Excess PV sharing**: When a household generates more PV than it needs and its battery is full, it offers a percentage of the excess to the P2P pool based on its assigned **Sharing % (Excess PV)** — independent from the battery sharing %.
- 🔋 **Battery sharing**: The algorithm monitors each household's total battery % to determine if the house can share stored energy with others. Eligibility requires:
  1. The household has no energy deficit itself
  2. Battery SoC % ≥ the household's assigned **Battery Share %**
  3. Battery SoC is above the 20% minimum floor
- 🎯 **Priority-based distribution**: Demand households are ranked by highest energy need first. Available P2P energy is allocated in that order — PV excess in Stage 1, then battery shareable energy in Stage 2 — instead of being split proportionally.

### 📅 Daily Sharing Cap
- The shareable energy % metric is **only applicable for 1 day**.
- Each household can share at most `battSharePct × usable battery capacity` (80% of total) per day.
- 🔄 The cap resets each day.

### 📥 Receiving Energy (Deficit Households)
- 🚫 P2P sharing does **not** fully charge other household's batteries — it only provides enough to cover their electricity demand.
- ✅ The house with the **largest deficit** is served first; remaining demanders are served in descending priority order until the P2P pool is exhausted.

---

## ☀️ PV Generation Flow

```
PV Generation (DC)
       │
       ▼
   Inverter
   P_AC = min(P_DC × η, P_capacity)
       │
       ├──────────────────┐
       ▼                  ▼
    Load              Battery
(House Demand)          ↕
                  P2P Algorithm
                  Monitors SoC %
                        ↓
                 Other Households
```

- ☀️ PV panels generate DC electricity, which the **inverter** converts to AC: **P_AC = min(P_DC × η, P_capacity)**.
- ⚡ The inverter efficiency (η) represents DC-to-AC conversion loss.
- 🏠 PV AC output meets house demand first (priority case).
- 🔋 Excess PV generation gets stored into the battery.
- 🔗 The P2P algorithm monitors total battery % to determine if a house can share excess to others.

---

## 🎲 Demand Modeling

The simulation uses a **deterministic load profile** for each household — demand is fully reproducible across runs (no stochastic variability). Load profiles are sourced from the embedded CSV data and can be viewed in the Household Summary tab.

---

## 🚀 How to Use

### 📂 Step 1: Open the File
Just open **`dashboard.html`** in any web browser (Chrome, Firefox, Edge, Safari). That's it — no installation, no server, no internet needed. ✅

### 📊 Step 2: Household Summary Tab — Review Load & WPS Data
- View each household's daily load profiles, peak hours/months, and WPS (Willingness Participation Score)
- Load data is sourced from the embedded CSV and displayed as summary cards

### 🔧 Step 3: Hardware Tab — Configure Global Parameters
These apply to all households unless overridden per-household in the Households tab.

| Parameter | Default | Notes |
|-----------|---------|-------|
| Grid Buy Price | ₱13.80/kWh | Retail electricity rate |
| P2P Trade Price | ₱0.00/kWh | Price per kWh exchanged between neighbours |
| Export Price | ₱0.00/kWh | Price per kWh sold back to the grid |
| Project Lifetime | 25 years | Used in LCOE calculation |
| Nominal Discount Rate | 5% | Market rate before inflation |
| Inflation Rate | 3% | Used with Fisher equation to get real rate |

> 💡 Equipment costs (Solar PV, Battery, Inverter) are configured **per household** in the Households tab.

### 🏠 Step 4: Households Tab — Configure Each Household
Each household card contains its own equipment specs and sharing settings. Default values from the dashboard:

**Shared equipment defaults (all households):**
| Component | Parameter | Default |
|-----------|-----------|---------|
| Solar PV | Capacity per panel | 0.3 kW |
| Solar PV | Capital cost | ₱9,500/panel |
| Solar PV | Replacement cost | ₱8,000/panel |
| Solar PV | O&M cost | ₱100/panel/yr |
| Solar PV | Lifetime | 25 years |
| Battery | Capacity per unit | 3.28 kWh |
| Battery | Capital cost | ₱135,500/unit |
| Battery | Replacement cost | ₱120,000/unit |
| Battery | O&M cost | ₱800/unit/yr |
| Inverter | O&M cost | ₱1,000/yr |
| Inverter | Lifetime | 10 years |

**Per-household configurable settings:**
- ☀️ **PV Capacity (kW)** — Size of the solar panel system
- 🔋 **Battery Units** — Number of battery units (3.28 kWh each)
- ⚡ **Inverter Capacity (kW)** — Maximum AC output and efficiency (varies by household)
- 💰 **Equipment Costs** — Capital, replacement, and O&M costs per component
- 📊 **WPS Score** — Willingness Participation Score
- 🔋 **Battery Share %** — Battery sharing limit (sets daily cap and SoC eligibility threshold)

### ▶️ Step 5: Simulate Tab — Run the Simulation
Click **"Run Simulation"** to simulate all 8,760 hours. The simulation uses deterministic load profiles for full reproducibility. Batteries start at 100% SoC; charge/discharge rate is fixed at 0.5 C. 📈

### 📊 Step 6: Reports Tab — Analyze Results
- **Summary Cards**: Total P2P energy shared, average LCOE, grid purchases, self-sufficiency
- **Detailed Results Table**: Per-household CAPEX, PW Salvage, 2 demand-based LCOE variants, energy flows, self-sufficiency
- **Charts** 📈 (6 views):
  - 📊 **Energy Breakdown** — Grouped bar with 4 side-by-side bars per household (PV, battery, P2P, grid)
  - 📅 **Monthly Trends** — Line chart of demand, generation, grid use, and P2P trading across the year
  - 🕐 **Daily Profile** — Average hourly energy profile for any household and any month
  - 🔋 **Battery SoC** — Per-household average battery state of charge across the year
  - 💰 **LCOE Comparison** — Demand-based LCOE with vs. without P2P for each household
  - 🔗 **P2P Network** — How much each household gave vs. received in the network
- **LCOE Computation Breakdown** 🧮: Expandable accordion showing the full step-by-step LCOE derivation (formula + substituted values) for each household
- **Detailed Computations** 📋: Expandable section showing a 24-hour hourly breakdown for any selected day, including Phase 1/2/3 dispatch steps, SoC tracking, P2P matching metadata, and a step-by-step numerical substitution for any selected hour. Exportable as PDF, DOCX, or XLSX.
- **Export** 📥: Reports tab results can be exported as PDF, DOCX, or XLSX.

### 🔬 Step 7: P2P Threshold Tab — Find the Value Sweet Spot
**P2P Value Threshold Analysis** 💡: Runs 50 full-year simulations sweeping PV size from 1–20 kW across all households to identify where P2P trading peaks and becomes financially significant. Two configurable thresholds:
- **P2P Threshold** (default: 3%) — minimum % of demand that must be traded via P2P
- **LCOE Gap Threshold** (default: ₱0.10/kWh) — minimum ₱/kWh savings from P2P participation

Both conditions must be met simultaneously to flag the meaningful sweet spot. Results include charts (P2P Traded vs PV Size, LCOE Gap vs PV Size, Self-Sufficiency vs PV Size, Self-Sufficiency vs P2P Traded) and a data table.

---

## 📖 Key Concepts

| Term | What It Means |
|------|---------------|
| ☀️ **PV (Photovoltaic)** | Solar panels that convert sunlight into electricity |
| ⚡ **kW (kilowatt)** | A measure of power — the "size" of a solar panel system |
| 🔌 **kWh (kilowatt-hour)** | A measure of energy — what you actually consume or produce over time |
| 🔋 **Battery SoC** | State of Charge — how full the battery is (0–100%). Minimum floor: 20% |
| 💰 **LCOE** | Levelized Cost of Energy — the total cost per kWh over the system's lifetime, including equipment, maintenance, salvage credit, and grid purchases |
| 💵 **CAPEX** | Capital Expenditure — total upfront cost of PV + Battery + Inverter |
| 💸 **PW Salvage** | Present Worth of leftover equipment value at the end of the project lifetime — credited back in the LCOE numerator |
| 📈 **Nominal Discount Rate (r)** | The market discount rate before adjusting for inflation (default: 5%) |
| 📊 **Inflation Rate (f)** | Annual price inflation (default: 3%); combined with nominal rate to derive real discount rate |
| 🧮 **Real Discount Rate (r')** | Inflation-adjusted discount rate via Fisher: r' = (1+r)/(1+f) − 1. Used in CRF, PW_replacement, and PW_salvage |
| 🏠 **Self-Sufficiency** | Percentage of demand met without buying from the grid |
| 🤝 **P2P Sharing** | Households selling/buying excess solar energy directly to/from each other |
| 🌤️ **GHI** | Global Horizontal Irradiance — how much solar energy hits a flat surface (kWh/m²/day) |
| 📉 **Derating Factor** | DC-side real-world efficiency loss (soiling, wiring, mismatch, aging) — set at 80% |
| ⚡ **Inverter** | Converts DC energy from solar panels to AC. Efficiency varies per household (90–96%). Output capped at inverter capacity |
| 🌡️ **Temp Coefficient** | How panel output drops as temperature rises (-0.38%/°C for this panel) |
| 📅 **Daily Sharing Cap** | Maximum battery energy a household can share per day (`battSharePct × usable capacity`) |
| 🔧 **Replacement Cost** | Cost to replace equipment during the project lifetime (discounted to present worth) |
| 🛠️ **O&M Cost** | Annual Operation and Maintenance cost per unit of equipment |
| ⏳ **Component Lifetime** | How many years each equipment type lasts before needing replacement |

---

## 🏘️ The 6 Households

Default configuration from the dashboard. Each household has a unique load profile, P2P sharing willingness, and equipment setup:

| Household | WPS | Batt Share % | PV (kW) | Battery Units | Total Batt (kWh) | Inverter (kW) | Inv. Efficiency |
|-----------|-----|--------------|---------|---------------|-------------------|---------------|-----------------|
| 🏠 House 1 | 3.8 | 66.67% | 2.5 | 1 | 3.28 | 3 | 90% |
| 🏠 House 2 | 5.0 | 100% | 2.5 | 2 | 6.56 | 3 | 90% |
| 🏠 House 3 | 3.6 | 66.67% | 3.5 | 2 | 6.56 | 3 | 90% |
| 🏠 House 4 | 4.6 | 100% | 9.5 | 9 | 29.52 | 8 | 96% |
| 🏠 House 5 | 2.8 | 66.67% | 5 | 5 | 16.40 | 4 | 96% |
| 🏠 House 6 | 2.4 | 33.33% | 7 | 4 | 13.12 | 5.5 | 95.8% |

> 💡 **Battery Share %** determines both the SoC eligibility threshold and the daily sharing cap. It applies to the **usable** (80% DoD) portion of the battery. Inverter capital costs are ₱75,000 for 3 kW houses, ₱145,000 for 4 kW, ₱220,000 for 5.5 kW, and ₱360,000 for 8 kW, with replacement costs approximately ₱15,000–30,000 less.

---

## 🔬 PV Generation Model

PV output is sourced from **HOMER Pro 8,760-hour time-series data** (one kWh value per hour per household), not a formula-based model. When the user changes a household's PV size in the Households tab, the output is scaled linearly from the HOMER baseline:

```
PV_gen (kWh/hr) = HOMER_8760_value[household][hour] × (UserPVkW / HOMER_BaselineKW)
```

HOMER baseline sizes per household: H1 = 2.5 kW, H2 = 2.5 kW, H3 = 3.5 kW, H4 = 9.5 kW, H5 = 5.0 kW, H6 = 7.0 kW. The HOMER CSV values already represent AC output at the inverter terminal, so no additional inverter efficiency multiplication is applied during the scaling step.

The inverter capacity cap is applied separately each hour:

```
PV_AC (kWh/hr) = min(PV_gen, InverterCapacity_kW)
```

Annual PV totals at HOMER baseline sizes: H1 ≈ 3,795 kWh, H2 ≈ 3,773 kWh, H3 ≈ 5,227 kWh, H4 ≈ 14,279 kWh, H5 ≈ 7,545 kWh, H6 ≈ 10,313 kWh.

---

## ⚙️ Simulation Formulas (Hour-by-Hour)

The simulation runs 8,760 hourly steps. Each hour has three phases executed in order.

### Pre-computed battery constants (computed once before the simulation)

```
battCap[i]   = battCount[i] × battCapPerUnit          (default 3.28 kWh/unit)
usableCap[i] = battCap[i] × DoD                       (DoD = 0.80)
minSoC[i]    = battCap[i] × minSoCPct                 (minSoCPct = 0.20)
maxRate[i]   = battCap[i] × cRate                     (cRate = 0.50 C/hr)
soc[i]       = battCap[i]                             (100% at simulation start)
```

---

### Phase 1 — Own Dispatch (PV + Battery, per household independently)

**Case A — PV surplus (pvGen ≥ demand):**

```
pvDirect    = demand
E_rem       = pvGen − demand

spaceLeft   = usableCap[i] − soc[i]
canCharge   = min(E_rem, maxRate[i], spaceLeft / chargeEff)   if spaceLeft > 0, else 0
battCharged = canCharge
soc[i]     += canCharge × chargeEff                           (chargeEff = 0.922)

excess      = E_rem − canCharge     ← post-battery surplus → grid export in Phase 3
deficit     = 0
```

**Case B — PV deficit (pvGen < demand):**

```
pvDirect     = pvGen
need         = demand − pvGen

availSoC     = max(0, soc[i] − minSoC[i])
canDischarge = min(need, maxRate[i], availSoC)
battUsed     = canDischarge
soc[i]      -= canDischarge / dischargeEff                    (dischargeEff = 0.922)
soc[i]       = max(minSoC[i], soc[i])                        (hard 20% floor clamp)

deficit      = need − canDischarge  ← goes to P2P pool or grid in Phase 2/3
excess       = 0
```

> 📝 **HOMER LF discharge logic**: `canDischarge` is bounded directly by `availSoC` (kWh above floor), not by `availSoC × dischargeEff`. The efficiency loss is only applied to the SoC depletion (`soc[i] -= canDischarge / dischargeEff`), matching HOMER's Load Following dispatch convention.

---

### Phase 2 — P2P Battery Sharing

Only battery energy is shared P2P. PV surplus goes directly to grid export in Phase 3.

**Supplier eligibility (per household, per hour):**

```
hhUsableCap    = battCap[i] × DoD                           (80% of total)
dailyCap       = hhUsableCap × battSharePct[i]              (daily sharing limit)
dailyRemaining = max(0, dailyCap − dailyBattShared[i])      (resets each day)

eligible to supply if:
    soc[i] > minSoC[i]   AND   dailyRemaining > 0

availableForShare = max(0, soc[i] − minSoC[i])
maxShareable      = availableForShare × dischargeEff        (max AC deliverable)
halfCRate         = maxRate[i] × 0.5
dailyLimit        = dailyRemaining × dischargeEff

battOffer[i] = min(maxShareable, halfCRate, dailyLimit)
```

**Pool and allocation:**

```
pool = Σ battOffer[i]   for all eligible suppliers

Demanders ranked by deficit size (largest first).
For each demander d in priority order:
    received[d] = min(deficit[d], remainingPool)
    remainingPool -= received[d]
    deficit[d]   -= received[d]

battUsedTotal = pool − remainingPool
shareRatio    = battUsedTotal / pool   (if pool > 0)

For each supplier s:
    shared[s]     = battOffer[s] × shareRatio
    socDeduction  = min(shared[s] / dischargeEff, max(0, soc[s] − minSoC[s]))
    soc[s]       -= socDeduction
    soc[s]        = max(minSoC[s], soc[s])
    dailyBattShared[s] += shared[s] / dischargeEff
```

---

### Phase 3 — Grid Interaction

```
gridBuy[i]    = deficit[i]             (any unmet demand after PV + battery + P2P)
gridExport[i] = excess[i]              (any post-battery PV surplus)

gridCost[i]   = gridBuy[i] × gridPrice
exportRev[i]  = gridExport[i] × exportPrice
```

---

### Self-Sufficiency

```
selfSufficiency[i]      = (1 − gridPurchased[i]      / totalDemand[i]) × 100   (with P2P)
selfSufficiencyNoP2P[i] = (1 − gridPurchasedNoP2P[i] / totalDemand[i]) × 100   (standalone)
```

---

## 💰 LCOE Calculation

Computed once per household after all 8,760 hours are simulated.

### Real discount rate (Fisher equation)

```
r' = (1 + r) / (1 + f) − 1

  r  = nominal discount rate  (default 5%)
  f  = inflation rate          (default 3%)
```

### Capital Recovery Factor

```
CRF = r'(1 + r')^N / ((1 + r')^N − 1)      if r' > 0
CRF = 1 / N                                  if r' = 0

  N = project lifetime  (default 25 years)
```

### Capital costs

```
numPanels   = pvKW / pvCapPerPanel                       (default 0.3 kW/panel)
C_PV        = numPanels   × pvCost                      (default ₱9,500/panel)
C_Batt      = battCount   × battCost                    (default ₱135,500/unit)
C_Inverter  = (invKW / invRatedKW) × inverterCost

CAPEX = C_PV + C_Batt + C_Inverter
```

### Present Worth of replacements

```
For each component j with replacement cost C_repl,j and lifetime L_j:

  m_j        = ceil(N / L_j) − 1          (number of replacements during project)
  C_repl_tot = numUnits_j × C_repl,j      (total replacement cost per event)

  PW_repl,j  = Σ(k=1 to m_j)  C_repl_tot / (1 + r')^(k × L_j)

PW_replacement = PW_repl,PV + PW_repl,Batt + PW_repl,Inverter
```

Default component lifetimes: PV = 25 yr, Battery = ~18–21 yr (per household), Inverter = 10 yr.

### Present Worth of salvage

```
For each component j:
  remainder_j      = N mod L_j
  remainingLife_j  = 0                     if remainder_j = 0
                   = L_j − remainder_j     otherwise

  salvageAtEnd_j   = C_repl_tot,j × (remainingLife_j / L_j)   (linear depreciation)
  PW_salvage,j     = salvageAtEnd_j / (1 + r')^N

PW_salvage = PW_salvage,PV + PW_salvage,Batt + PW_salvage,Inverter
```

### Annual O&M cost

```
annualOM = (numPanels × pvOM) + (battCount × battOM) + ((invKW / invRatedKW) × invOM)

  pvOM defaults:  ₱100/panel/yr
  battOM default: ₱800/unit/yr
  invOM default:  ₱1,000/yr (scaled by inverter ratio)
```

### Annualised base cost

```
annualBase = (CAPEX + PW_replacement − PW_salvage) × CRF + annualOM
```

### LCOE with P2P

```
netCost_P2P  = annualBase + C_grid_P2P − P2P_revenue − exportRevenue + P2P_cost

  C_grid_P2P   = gridPurchased × gridPrice
  P2P_revenue  = p2pShared × p2pPrice      (₱ earned from selling battery energy to neighbours)
  exportRevenue= gridExported × exportPrice (₱ earned from grid exports)
  P2P_cost     = p2pReceived × p2pPrice    (₱ paid for energy received from neighbours)

E_served_P2P  = max(totalDemand − p2pReceived + gridExported, 1)

LCOE_P2P = netCost_P2P / E_served_P2P
```

### LCOE without P2P (standalone)

```
netCost_noP2P = annualBase + C_grid_noP2P − exportRevenue_noP2P

  C_grid_noP2P        = gridPurchasedNoP2P × gridPrice
  exportRevenue_noP2P = gridExportedNoP2P  × exportPrice

E_served_noP2P = max(totalDemand + gridExportedNoP2P, 1)

LCOE_noP2P = netCost_noP2P / E_served_noP2P
```

> 📝 With default prices (P2P = ₱0, export = ₱0), the revenue terms all drop to zero. LCOE then simplifies to `(annualBase + gridCost) / E_served`. Non-zero P2P or export prices shift costs between households and reduce each household's net expenditure accordingly.

---

## 📁 Input Data Files

The data embedded in the dashboard comes from three CSV files:

| File | Contents |
|------|----------|
| 📄 `ME-Electric Load.csv` | 24-hour × 12-month energy consumption tables for all 6 households (kWh per hour) |
| 📄 `ME-P2P-Rules.csv` | Willingness Scores, sharing percentages, and battery sharing rules per household |
| 📄 `ME-PV Panel Formulas.csv` | Monthly solar GHI, ambient temperature, and panel specifications |

> 💡 These are included in the repository for reference. All data is embedded in the HTML file.

---

## 🗂️ File Structure

```
📦 P2P-Sharing-Dashboard
├── 🌐 dashboard.html              ← Open this file in your browser (standalone, works offline)
├── 📄 ME-Electric Load.csv        ← Source data: household load profiles
├── 📄 ME-P2P-Rules.csv            ← Source data: P2P sharing rules
├── 📄 ME-PV Panel Formulas.csv    ← Source data: solar and panel specifications
└── 📖 README.md                   ← This file
```

---

## ✅ Requirements

- 🌐 Any modern web browser (Chrome, Firefox, Edge, Safari)
- 📴 No internet connection needed
- 💻 No software installation needed
- 🖥️ Works on Windows, Mac, and Linux
