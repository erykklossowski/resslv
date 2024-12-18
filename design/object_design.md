# Object Design, by courtesy of ChatGPT :)

This document outlines the high-level object design for the Rich app.

## **1. Core Objects**

### **1.1 PriceCategorizer**
- **Purpose**: Categorize electricity prices into regimes (low, normal, peak) for further modeling.
- **Attributes**:
  - `raw_prices`: The historical or live price series.
  - `categories`: Categorized price regimes.
- **Methods**:
  - `categorize_prices()`: Categorize prices using quantiles or volatility measures or fuzzy logic.
  - `get_price_categories()`: Return categorized price regimes.

---

### **1.2 BatterySimulator**
- **Purpose**: Simulate battery states (charge, discharge, idle) using HMM based on categorized prices.
- **Attributes**:
  - `hmm_model`: Trained HMM model for battery states.
  - `viterbi_path`: Most probable sequence of battery states.
- **Methods**:
  - `train_hmm(price_categories)`: Train the HMM using categorized prices.
  - `generate_viterbi_path()`: Compute the Viterbi path for battery states.
  - `simulate_battery_states()`: Run the simulation based on HMM.

---

### **1.3 RLOptimizer**
- **Purpose**: Optimize the battery's operation schedule using reinforcement learning.
- **Attributes**:
  - `rl_agent`: Trained RL model.
  - `reward_function`: Defines rewards for battery operations.
- **Methods**:
  - `train_rl_agent(viterbi_path)`: Train RL agent using HMM results.
  - `optimize_schedule()`: Optimize charge/discharge schedules.

---

### **1.4 PriceForecaster**
- **Purpose**: Extend historical price series using HMM in price space based on battery simulation outcomes.
- **Attributes**:
  - `price_hmm`: Trained HMM model for price forecasting.
  - `forecast_horizon`: Number of steps to forecast.
- **Methods**:
  - `train_price_hmm(simulation_results)`: Train the HMM on simulated outcomes.
  - `forecast_prices()`: Generate short-term price forecasts.

---

### **1.5 CentralController**
- **Purpose**: Orchestrate the entire simulation workflow.
- **Methods**:
  - `run_simulation()`: Execute categorization, HMM, RL, and forecasting steps.
  - `get_results()`: Aggregate and present outputs.
