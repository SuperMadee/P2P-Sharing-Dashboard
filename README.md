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

🔄 Excess solar energy follows the reverse path: charge battery first, then share with the network, then export to grid.

---

## 🔋 Battery Conditions

- 🛑 **20% SoC Floor**: Discharging stops when at State of Charge (SoC) of ≤ 20% of each battery device. This applies to both self-use discharge and P2P sharing.
- 📉 **Depth of Discharge**: Only 80% of battery capacity is usable (20% reserved).
- ⚡ **Charge/Discharge Efficiency**: 95% (5% loss each way).
- ⏱️ **C-Rate**: Max charge/discharge rate is 50% of capacity per hour.

---

## 🤝 P2P Sharing Rules

### 📤 Supplying Energy (Surplus Households)
- ☀️ **Excess PV sharing**: When a household generates more PV than it needs and its battery is full, it offers a percentage of the excess to the P2P pool based on its assigned sharing %.
- 🔋 **Battery sharing**: The algorithm monitors each household's total battery % to determine if the house can share excess to others. Eligibility requires:
  1. The household has no energy deficit itself
  2. Battery SoC % ≥ the household's assigned battery sharing % (e.g., 5 Rupee with 45% sharing needs SoC ≥ 45%)
  3. Battery SoC is above the 20% minimum floor
- ⚖️ **Equal distribution**: The algorithm manages equal distribution of excess energy from households while respecting each homeowner's % of shareable energy.

### 📅 Daily Sharing Cap
- The shareable energy % metric is **only applicable for 1 day**.
- Example: 5 Rupee can only share a total of 45% of their total battery's worth of energy per day, if this amount is available.
- 🔄 The cap resets each day.

### 📥 Receiving Energy (Deficit Households)
- The algorithm uses assigned sharing allocation % to set the amount of allowable % to get from each household per day.
- 🚫 P2P sharing does **not** fully charge other household's batteries — it only provides enough to support their electricity demands until their demand is less than their PV + battery generation.
- ✅ The house in need gets its required energy needs from the P2P pool proportionally.

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
- ⚡ The inverter efficiency (η) represents DC-to-AC conversion loss (~96%).
- 🏠 PV AC output meets house demand first (priority case).
- 🔋 Excess PV generation gets stored into the battery.
- 🔗 The P2P algorithm monitors total battery % to determine if a house can share excess to others.

---

## 🎲 Per-Household Demand Variation

Random demand increase and decrease is a **per-household function** (not a global function). Each household has its own:
- 📈 **High Demand Probability** — chance of a high-demand day (e.g., hot day, guests)
- 🔺 **High Demand Multiplier** — how much higher demand is on those days
- 📉 **Low Demand Probability** — chance of a low-demand day (e.g., vacation, mild weather)
- 🔻 **Low Demand Multiplier** — how much lower demand is on those days

This means one household can have a high-demand day while another has a low-demand day on the same date. 🏠↕️

---

## 🚀 How to Use

### 📂 Step 1: Open the File
Just open **`dashboard.html`** in any web browser (Chrome, Firefox, Edge, Safari). That's it — no installation, no server, no internet needed. ✅

### 📊 Step 2: Load & WPS Tab — Review Load Data
- View each household's daily load profiles and peak hours/months
- Review WPS (Willingness Participation Score) for each household

### 🔧 Step 3: Hardware Tab — Configure Equipment
- **Equipment Data**: Configure specs and costs for each equipment type:
  - ☀️ **Solar PV** — Capacity per panel (kW), capital (₱/kW), replacement (₱/kW), O&M costs (₱/kW/yr)
  - 🔋 **Battery** — Unit capacity (kWh), capital (₱/kWh), replacement (₱/kWh), O&M costs (₱/kWh/yr)
  - ⚡ **Inverter** — Efficiency (%), capital (₱/kW), replacement (₱/kW), O&M costs (₱/kW/yr)
- **Global Parameters**: Grid buy price, P2P trade price, export price, project lifetime, discount rate

### 🏠 Step 4: Households Tab — Configure Households
- **Household Cards** 🏠: Each household can be configured with:
  - ☀️ **PV Panels** — Number of solar panels (total kW = panels × capacity/panel)
  - 🔋 **Battery Units** — Number of battery units (default 5 kWh each)
  - ⚡ **Inverter Capacity (kW)** — Maximum AC output capacity of the inverter
  - 📊 **WPS Score** — Willingness Participation Score (determines sharer type)
  - 🤝 **Battery Sharing %** — Unified sharing limit for excess PV and battery energy (also sets daily cap and SoC eligibility threshold)
  - 🎲 **Demand Variation** — Per-household high/low demand probability and multiplier

### ▶️ Step 5: Simulate Tab — Run the Simulation
Click **"Run Simulation"** to simulate all 8,760 hours. Each household independently gets random demand variation per day based on its own settings. 📈📉

### 📊 Step 6: Reports Tab — Analyze Results
- **Summary Cards**: Total P2P energy shared, average LCOE, grid purchases, self-sufficiency
- **Charts** 📈 (5 views):
  - 📊 **Energy Breakdown** — Stacked bar showing where each household's energy came from (PV, battery, P2P, grid)
  - 📅 **Monthly Trends** — Line chart of demand, generation, grid use, and P2P trading across the year
  - 🕐 **Daily Profile** — Average hourly energy profile for any household and any month
  - 💰 **LCOE Comparison** — Bar chart comparing cost of energy across households
  - 🔗 **P2P Network** — How much each household gave vs. received in the network

### 🎯 Step 7: Optimizer Tab — Find the Best Setup
**Homeowner PV Setup Optimizer** 💡: Based on HOMER Pro PV + Batt setup P2P simulation report, the optimizer tool finds the optimal PV + Battery combination that fits well with self-sustaining and P2P sharing. It tests every combination of PV size (1–20 kW) and battery count (0–10 units) to find the configuration with the **lowest LCOE** while respecting all P2P rules (20% SoC floor, daily sharing caps, SoC eligibility thresholds). You can apply the result with one click.

---

## 📖 Key Concepts

| Term | What It Means |
|------|---------------|
| ☀️ **PV (Photovoltaic)** | Solar panels that convert sunlight into electricity |
| ⚡ **kW (kilowatt)** | A measure of power — the "size" of a solar panel system |
| 🔌 **kWh (kilowatt-hour)** | A measure of energy — what you actually consume or produce over time |
| 🔋 **Battery SoC** | State of Charge — how full the battery is (0–100%). Minimum floor: 20% |
| 💰 **LCOE** | Levelized Cost of Energy — the total cost per kWh over the system's lifetime, including equipment, maintenance, and grid purchases |
| 🏠 **Self-Sufficiency** | Percentage of demand met without buying from the grid |
| 🤝 **P2P Sharing** | Households selling/buying excess solar energy directly to/from each other |
| 🌤️ **GHI** | Global Horizontal Irradiance — how much solar energy hits a flat surface (kWh/m²/day) |
| 📉 **Derating Factor** | DC-side real-world efficiency loss (soiling, wiring, mismatch, aging) — set at 80% |
| ⚡ **Inverter** | Converts DC energy from solar panels to AC. Efficiency ~96%. Output capped at inverter capacity |
| 🌡️ **Temp Coefficient** | How panel output drops as temperature rises (-0.38%/°C for this panel) |
| 📅 **Daily Sharing Cap** | Maximum battery energy a household can share per day (battSharePct × battery capacity) |
| 🔧 **Replacement Cost** | Cost to replace equipment during the project lifetime (annualized in LCOE) |
| 🛠️ **O&M Cost** | Annual Operation and Maintenance cost per unit of equipment capacity |

---

## 🏘️ The 6 Households

Each household has a unique load profile (24-hour energy consumption pattern for each month), P2P sharing willingness, and independent demand variation:

| Household | WPS | Sharing Level | Sharing % | PV Panels | Inverter (kW) |
|-----------|-----|---------------|-----------|-----------|---------------|
| 🏠 5 Rupee | 3.127 | 🟢 Active Sharer | 45% | 6 | 3 |
| 🏠 7 Rial | 3.167 | 🟢 Active Sharer | 45% | 6 | 3 |
| 🏠 19 Baht | 2.683 | 🟡 Moderate Sharer | 60% | 8 | 4 |
| 🏠 33 Guilder | 2.500 | 🟡 Moderate Sharer | 35% | 20 | 10 |
| 🏠 38 Rand | 2.927 | 🟡 Moderate Sharer | 35% | 10 | 5 |
| 🏠 40 Guilder | 2.077 | 🔴 Conservative Sharer | 10% | 10 | 5 |

---

## 🔬 PV Generation Model

Solar DC output is calculated hourly, then passed through the inverter:

```
PV_DC (kWh)  = PV_capacity × Irradiance × Derating_Factor × Temperature_Correction
PV_AC (kWh)  = min(PV_DC × Inverter_Efficiency, Inverter_Capacity)
```

Where:
- ☀️ **Irradiance** is distributed across hours 6:00–17:00 using a sinusoidal (bell-curve) pattern from the daily GHI value
- 🌡️ **Cell Temperature** = Ambient Temp + (NOCT − 20) / 0.8 × Irradiance
- 📐 **Temperature Correction** = 1 + (−0.38 / 100) × (T_cell − 25°C)
- 🔧 **Panel**: LONGi Solar LR6-60PB, NOCT = 45°C
- ⚡ **Inverter Efficiency** (η): DC-to-AC conversion efficiency (default 96%)
- 🔌 **Inverter Capacity**: Maximum AC output the inverter can deliver (kW). If PV_DC × η exceeds this, output is clipped.

---

## 💰 LCOE Calculation

```
LCOE = Annual Total Cost / Annual Energy Consumed (₱/kWh)

Annual Total Cost = Annualized Capital + Annual O&M + Grid Purchases + P2P Purchases − P2P Revenue − Export Revenue

Capital Costs:
  PV Capital       = PV_kW × PV_Cost_per_kW
  Battery Capital   = Battery_kWh × Battery_Cost_per_kWh
  Inverter Capital  = Inverter_kW × Inverter_Cost_per_kW
  Total Replacement = PV_Replacement + Battery_Replacement + Inverter_Replacement

Annualized Capital = (Total_Capital + Total_Replacement) × CRF
CRF = r(1+r)^n / ((1+r)^n − 1)    where r = discount rate, n = project lifetime in years

Annual O&M = (PV_kW × PV_OM) + (Battery_kWh × Battery_OM) + (Inverter_kW × Inverter_OM)
```

---

## 📁 Input Data Files

The data embedded in the dashboard comes from three CSV files:

| File | Contents |
|------|----------|
| 📄 `ME-Electric Load.csv` | 24-hour × 12-month energy consumption tables for all 6 households (kWh per hour) |
| 📄 `ME-P2P Rules.csv` | Willingness Scores, sharing percentages, and battery sharing rules per household |
| 📄 `ME-PV Panel Formulas.csv` | Monthly solar GHI, ambient temperature, and panel specifications |

> 💡 These are included in the repository for reference but are **not needed to run the dashboard** — all data is already embedded in the HTML file.

---

## 🗂️ File Structure

```
📦 P2P-Sharing-Dashboard
├── 🌐 dashboard.html              ← Open this file in your browser (standalone, works offline)
├── 📄 ME-Electric Load.csv        ← Source data: household load profiles
├── 📄 ME-P2P Rules.csv            ← Source data: P2P sharing rules
├── 📄 ME-PV Panel Formulas.csv    ← Source data: solar and panel specifications
└── 📖 README.md                   ← This file
```

---

## ✅ Requirements

- 🌐 Any modern web browser (Chrome, Firefox, Edge, Safari)
- 📴 No internet connection needed
- 💻 No software installation needed
- 🖥️ Works on Windows, Mac, and Linux
