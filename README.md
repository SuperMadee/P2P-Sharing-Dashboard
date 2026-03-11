# P2P Solar Energy Sharing Simulation Dashboard

A browser-based simulation tool for modeling **Peer-to-Peer (P2P) solar energy sharing** between households. Built for engineering students to explore how homes with solar panels and batteries can trade electricity with each other and reduce grid dependence.

![Dashboard Preview](https://img.shields.io/badge/Type-Simulation_Dashboard-blue)
![Standalone](https://img.shields.io/badge/Standalone-No_Install_Required-green)

---

## What Does This Do?

Imagine 6 households, each with solar panels and batteries. During the day, some produce more electricity than they need. At night, they all need power. Instead of buying everything from the grid, **they can share energy with each other** — that's P2P energy sharing.

This dashboard simulates a **full year (365 days, 8,760 hours)** of energy generation, consumption, storage, and sharing between households, then calculates key metrics like cost of energy and self-sufficiency.

---

## How Energy Flows (Priority Logic)

For each hour of the simulation, every household follows this priority order:

```
1. USE SOLAR PV  →  Generated energy goes directly to meet the house's demand
2. USE BATTERY   →  If PV isn't enough, discharge the home battery
3. ASK NETWORK   →  If still short, request excess energy from other P2P households
4. BUY FROM GRID →  Last resort — purchase from the utility grid
```

Excess solar energy follows the reverse path: charge battery first, then share with the network, then export to grid.

---

## How to Use

### Step 1: Open the File
Just open **`dashboard.html`** in any web browser (Chrome, Firefox, Edge, Safari). That's it — no installation, no server, no internet needed.

### Step 2: Setup Tab — Configure Parameters
- **Global Settings**: Grid electricity price, P2P trading price, PV/battery costs, system lifetime, discount rate
- **Household Cards**: Each household can be configured with:
  - **PV Capacity (kW)** — Size of the solar panel system
  - **Battery Units** — Number of battery units (default 5 kWh each)
  - **Sharing %** — How much of their excess PV energy they're willing to share
  - **Battery Share %** — How much of their stored battery energy they'll offer to the network

### Step 3: Simulate Tab — Run the Simulation
Click **"Run Simulation"** to simulate all 8,760 hours. Each day is randomly assigned as a normal, high-demand, or low-demand day to add realistic variation.

### Step 4: Reports Tab — Analyze Results
- **Summary Cards**: Total P2P energy shared, average LCOE, grid purchases, self-sufficiency
- **Charts** (5 views):
  - **Energy Breakdown** — Stacked bar showing where each household's energy came from (PV, battery, P2P, grid)
  - **Monthly Trends** — Line chart of demand, generation, grid use, and P2P trading across the year
  - **Daily Profile** — Average hourly energy profile for any household and any month
  - **LCOE Comparison** — Bar chart comparing cost of energy across households
  - **P2P Network** — How much each household gave vs. received in the network

### Step 5: Optimizer Tab — Find the Best Setup
Select a household (or all) and click **"Run Optimizer"**. It tests every combination of PV size (1–20 kW) and battery count (0–10 units) to find the configuration with the **lowest LCOE**. You can then apply the result with one click.

---

## Key Concepts

| Term | What It Means |
|------|---------------|
| **PV (Photovoltaic)** | Solar panels that convert sunlight into electricity |
| **kW (kilowatt)** | A measure of power — the "size" of a solar panel system |
| **kWh (kilowatt-hour)** | A measure of energy — what you actually consume or produce over time |
| **Battery SoC** | State of Charge — how full the battery is (0–100%) |
| **LCOE** | Levelized Cost of Energy — the total cost per kWh over the system's lifetime, including equipment, maintenance, and grid purchases |
| **Self-Sufficiency** | Percentage of demand met without buying from the grid |
| **P2P Sharing** | Households selling/buying excess solar energy directly to/from each other |
| **GHI** | Global Horizontal Irradiance — how much solar energy hits a flat surface (kWh/m²/day) |
| **Derating Factor** | Real-world efficiency loss (dust, wiring, inverter) — set at 80% |
| **Temp Coefficient** | How panel output drops as temperature rises (-0.38%/°C for this panel) |

---

## The 6 Households

Each household has a unique load profile (24-hour energy consumption pattern for each month) and P2P sharing willingness:

| Household | Willingness Score | Sharing Level | Notes |
|-----------|------------------|---------------|-------|
| 5 Rupee | 3.127 | Active Sharer | Shares 45% of excess, 45% of battery |
| 7 Rial | 3.167 | Active Sharer | Shares 45% of excess, 45% of battery |
| 19 Baht | 2.683 | Moderate Sharer | Shares 60% of excess, 60% of battery |
| 33 Guilder | 2.500 | Moderate Sharer | Largest consumer in the network |
| 38 Rand | 2.927 | Moderate Sharer | Shares 45% of excess, 35% of battery |
| 40 Guilder | 2.077 | Conservative Sharer | Shares only 10% — least cooperative |

---

## PV Generation Model

Solar output is calculated hourly using:

```
PV Output (kWh) = PV_capacity × Irradiance × Derating_Factor × Temperature_Correction
```

Where:
- **Irradiance** is distributed across hours 6:00–17:00 using a sinusoidal (bell-curve) pattern from the daily GHI value
- **Cell Temperature** = Ambient Temp + (NOCT − 20) / 0.8 × Irradiance
- **Temperature Correction** = 1 + (−0.38 / 100) × (T_cell − 25°C)
- **Panel**: LONGi Solar LR6-60PB, NOCT = 45°C

---

## LCOE Calculation

```
LCOE = Annual Total Cost / Annual Energy Consumed ($/kWh)

Annual Total Cost = Annualized Capital + O&M + Grid Purchases + P2P Purchases − P2P Revenue − Export Revenue

Annualized Capital = (PV Cost + Battery Cost) × CRF
CRF = r(1+r)^n / ((1+r)^n − 1)    where r = discount rate, n = system lifetime in years
O&M = 1% of total capital cost per year
```

---

## Input Data Files

The data embedded in the dashboard comes from three CSV files:

| File | Contents |
|------|----------|
| `ME-Electric Load.csv` | 24-hour × 12-month energy consumption tables for all 6 households (kWh per hour) |
| `ME-P2P Rules.csv` | Willingness Scores, sharing percentages, and battery sharing rules per household |
| `ME-PV Panel Formulas.csv` | Monthly solar GHI, ambient temperature, and panel specifications |

These are included in the repository for reference but are **not needed to run the dashboard** — all data is already embedded in the HTML file.

---

## File Structure

```
├── dashboard.html              ← Open this file in your browser (standalone, works offline)
├── ME-Electric Load.csv        ← Source data: household load profiles
├── ME-P2P Rules.csv            ← Source data: P2P sharing rules
├── ME-PV Panel Formulas.csv    ← Source data: solar and panel specifications
└── README.md                   ← This file
```

---

## Requirements

- Any modern web browser (Chrome, Firefox, Edge, Safari)
- No internet connection needed
- No software installation needed
- Works on Windows, Mac, and Linux
