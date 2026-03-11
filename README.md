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
PV Panel ──→ Inverter ──→ Battery
   │            (* excess generated gets stored into battery)
   │ Priority
   │ case
   ↓
House Appliance              Battery
Electricity Demand           ↕
                     P2P Algo Monitoring
                     Total Batt % - if house
                     can share excess to others
                             ↓
                      Other Households
```

- ☀️ PV output meets house demand first (priority case).
- 🔋 Excess PV generation flows through the inverter and gets stored into the battery.
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

### ⚙️ Step 2: Setup Tab — Configure Parameters
- **Global Settings**: Grid electricity price, P2P trading price, PV/battery costs, system lifetime, discount rate
- **Household Cards** 🏠: Each household can be configured with:
  - ☀️ **PV Capacity (kW)** — Size of the solar panel system
  - 🔋 **Battery Units** — Number of battery units (default 5 kWh each)
  - 🤝 **Sharing % (Excess PV)** — How much of their excess PV energy they're willing to share
  - 🔋🤝 **Battery Share %** — How much of their stored battery energy they'll offer to the network (also sets the daily cap and SoC eligibility threshold)
  - 🎲 **Demand Variation** — Per-household high/low demand probability and multiplier

### ▶️ Step 3: Simulate Tab — Run the Simulation
Click **"Run Simulation"** to simulate all 8,760 hours. Each household independently gets random demand variation per day based on its own settings. 📈📉

### 📊 Step 4: Reports Tab — Analyze Results
- **Summary Cards**: Total P2P energy shared, average LCOE, grid purchases, self-sufficiency
- **Charts** 📈 (5 views):
  - 📊 **Energy Breakdown** — Stacked bar showing where each household's energy came from (PV, battery, P2P, grid)
  - 📅 **Monthly Trends** — Line chart of demand, generation, grid use, and P2P trading across the year
  - 🕐 **Daily Profile** — Average hourly energy profile for any household and any month
  - 💰 **LCOE Comparison** — Bar chart comparing cost of energy across households
  - 🔗 **P2P Network** — How much each household gave vs. received in the network

### 🎯 Step 5: Optimizer Tab — Find the Best Setup
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
| 📉 **Derating Factor** | Real-world efficiency loss (dust, wiring, inverter) — set at 80% |
| 🌡️ **Temp Coefficient** | How panel output drops as temperature rises (-0.38%/°C for this panel) |
| 📅 **Daily Sharing Cap** | Maximum battery energy a household can share per day (battSharePct × battery capacity) |

---

## 🏘️ The 6 Households

Each household has a unique load profile (24-hour energy consumption pattern for each month), P2P sharing willingness, and independent demand variation:

| Household | Willingness Score | Sharing Level | PV Share % | Batt Share % (Daily Cap) |
|-----------|------------------|---------------|------------|--------------------------|
| 🏠 5 Rupee | 3.127 | 🟢 Active Sharer | 45% | 45% |
| 🏠 7 Rial | 3.167 | 🟢 Active Sharer | 45% | 45% |
| 🏠 19 Baht | 2.683 | 🟡 Moderate Sharer | 60% | 60% |
| 🏠 33 Guilder | 2.500 | 🟡 Moderate Sharer | 35% | 35% |
| 🏠 38 Rand | 2.927 | 🟡 Moderate Sharer | 45% | 35% |
| 🏠 40 Guilder | 2.077 | 🔴 Conservative Sharer | 10% | 10% |

---

## 🔬 PV Generation Model

Solar output is calculated hourly using:

```
PV Output (kWh) = PV_capacity × Irradiance × Derating_Factor × Temperature_Correction
```

Where:
- ☀️ **Irradiance** is distributed across hours 6:00–17:00 using a sinusoidal (bell-curve) pattern from the daily GHI value
- 🌡️ **Cell Temperature** = Ambient Temp + (NOCT − 20) / 0.8 × Irradiance
- 📐 **Temperature Correction** = 1 + (−0.38 / 100) × (T_cell − 25°C)
- 🔧 **Panel**: LONGi Solar LR6-60PB, NOCT = 45°C

---

## 💰 LCOE Calculation

```
LCOE = Annual Total Cost / Annual Energy Consumed (₱/kWh)

Annual Total Cost = Annualized Capital + O&M + Grid Purchases + P2P Purchases − P2P Revenue − Export Revenue

Annualized Capital = (PV Cost + Battery Cost) × CRF
CRF = r(1+r)^n / ((1+r)^n − 1)    where r = discount rate, n = system lifetime in years
O&M = 1% of total capital cost per year
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
