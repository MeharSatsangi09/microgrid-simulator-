# Microgrid Simulator

**A clean, explainable 24-hour microgrid energy scheduler with rule-based control, weather uncertainty modeling, and a React dashboard.**

> Built for VLabs Hackathon 2026

---

## What Is This?

This project simulates a **grid-connected microgrid** — solar PV + battery storage + utility grid — over a 24-hour horizon. Every hour, the system decides how to optimally balance energy: use solar first, charge the battery with excess, discharge during expensive peak hours, and import from the grid only as a last resort.

Every decision is logged with a **plain-English explanation**, making the system fully transparent and suitable for decision-support analysis.

An optional **weather uncertainty module** introduces realistic forecast errors between planned and actual solar generation, showing how the controller adapts under imperfect information.

---

## Features

- 24-hour hourly simulation with deterministic time steps
- Rule-based scheduler with clear priority logic
- Battery model with realistic constraints (SoC limits, charge/discharge rates, round-trip efficiency)
- Strict energy balance enforcement at every timestep
- Time-of-Use (TOU) cost optimization
- Baseline vs. optimized cost comparison
- CO₂ emissions tracking and savings
- Explainable AI-style decision logs for every hour
- Optional weather uncertainty modeling (forecast vs. actual solar)
- FastAPI REST backend + React/Vite frontend dashboard

---

## Architecture

```
microgrid-simulator/
├── backend/
│   ├── main.py                    # FastAPI app, /simulate endpoint
│   ├── models/
│   │   ├── battery.py             # Battery model (SoC, charge/discharge, efficiency)
│   │   └── microgrid.py           # Microgrid system container
│   ├── simulator/
│   │   ├── time_engine.py         # 24-hour time-step manager
│   │   └── energy_balance.py      # Energy conservation validation
│   ├── scheduler/
│   │   ├── rule_engine.py         # Rule-based scheduling logic
│   │   └── optimizer.py           # Stub for Phase 2 optimization
│   ├── metrics/
│   │   ├── cost.py                # Cost calculation & savings
│   │   └── carbon.py              # CO₂ emissions tracking
│   ├── explainability/
│   │   └── decision_log.py        # Hourly decision explanations
│   └── data/
│       ├── load_profile.py        # 24-hour load demand profile
│       ├── solar_profile.py       # Solar forecast profile
│       └── price_profile.py       # Time-of-Use grid pricing
├── frontend/
│   └── src/
│       └── components/
│           ├── LandingPage.jsx
│           ├── Dashboard.jsx
│           ├── BatterySoCChart.jsx
│           ├── EnergyUsageChart.jsx
│           ├── SolarForecastChart.jsx
│           ├── DecisionTimeline.jsx
│           └── SummaryCards.jsx
└── docs/
    ├── QUICKSTART.md
    ├── DEMO_GUIDE.md
    └── PROJECT_SUMMARY.md
```

---

## Scheduling Logic

At every hourly timestep, the controller follows this priority order:

1. **Use solar to meet local load**
2. **Charge battery** with excess solar (if SoC allows)
3. **Discharge battery** during high-price periods (if SoC allows)
4. **Import from grid** as last resort
5. **Curtail solar** if battery is full and grid export is not possible

This guarantees physical feasibility, cost-aware operation, and fully explainable decisions.

---

## Weather Uncertainty

Real microgrids operate with imperfect solar forecasts. When enabled, this module separates:

- **Forecast solar** → used for scheduling decisions
- **Actual solar** → used for energy balance and cost calculations

Configurable error levels: ±10% (low), ±15% (medium), ±20% (high), ±30% (stress test).

When actual solar deviates significantly from forecast, the system logs corrective actions like emergency battery discharge or unexpected grid import.

---

## Setup

### Backend

```bash
cd backend
pip install -r requirements.txt
python main.py
# Server runs at http://localhost:8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
# App runs at http://localhost:3000
```

---

## API

### `POST /simulate`

Run a 24-hour simulation with custom configuration.

**Default configuration:**
- Solar: 6 kW
- Battery: 10 kWh, 5 kW charge/discharge, 95% efficiency, SoC 20–95%
- Grid carbon intensity: 0.42 kg CO₂/kWh

**Example request:**

```bash
curl -X POST http://localhost:8000/simulate \
  -H "Content-Type: application/json" \
  -d '{
    "solar_capacity": 6.0,
    "battery": {
      "capacity": 10.0,
      "min_soc": 0.2,
      "max_soc": 0.95,
      "initial_soc": 0.5,
      "max_charge_rate": 5.0,
      "max_discharge_rate": 5.0,
      "efficiency": 0.95
    },
    "grid_carbon_intensity": 0.42,
    "enable_weather_uncertainty": true,
    "forecast_error_range": 0.15
  }'
```

**Response includes:**
- `hourly_results` — 24 hours of energy flows, costs, battery SoC, and plain-English explanations
- `summary` — total cost, savings vs. baseline, CO₂ savings, renewable usage %

**Example hourly result:**

```json
{
  "hour": 12,
  "time": "12:00 PM",
  "load_kwh": 3.8,
  "solar_kwh": 4.6,
  "battery_soc_pct": 95.0,
  "grid_import_kwh": 0.0,
  "grid_export_kwh": 0.8,
  "battery_charge_kwh": 0.0,
  "battery_discharge_kwh": 0.0,
  "cost_usd": -0.06,
  "emissions_kg": -0.034,
  "decision_type": "SOLAR_ONLY",
  "explanation": "At 12:00 PM, solar fully met demand. Battery is at max SoC. Excess 0.8 kWh exported to grid."
}
```

**Other endpoints:**
- `GET /` — API info
- `GET /health` — Health check
- `GET /docs` — Interactive Swagger docs

---

## Key Assumptions

- Hourly time steps (1-hour energy quantities, not power flow)
- No grid export limits (net metering assumed)
- Linear battery efficiency (no degradation)
- Deterministic profiles for load, solar, and price
- Decision-support tool — not a full power flow solver

---

## Tech Stack

- **Backend:** Python 3.10+, FastAPI, Pydantic, Uvicorn
- **Frontend:** React, Vite, Tailwind CSS, Recharts
- **Deployment:** Vercel (frontend), local or cloud (backend)

---

## Future Enhancements

- Optimization-based scheduling (Linear Programming, MPC, Dynamic Programming)
- Multi-day simulation
- Demand response integration
- Real-time price and weather API feeds
- Multiple battery units and EV charging
- Grid export limits and islanding mode
