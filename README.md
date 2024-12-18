# resslv
# Rich app

In memoriam Marc Rich, the king of the commodities arbitrage

This project models and optimizes a single battery's and/or aggregated batterries operation using:
- Hidden Markov Models (HMM) for price categorization and forecasting.
- Reinforcement Learning (RL) for schedule optimization.

The workflow should be designed so that the outcome of the battery simulation (Step 1: Battery Space) feeds into the forecasting layer (Step 2: Price Space). This would result in a closed-loop process, where the battery’s historical states (charge, discharge, idle) and the corresponding observed prices train the HMM for price forecasting. 

## Workflow

Step 1: Historical Data – Battery Space

Simulate battery states (charge, discharge, idle) using the HMM-based simulator with historical prices (see the legacy WL code).
Collect two outputs:
Battery states: Hidden states from the HMM-based simulator.
Observed prices: Prices/price regimes corresponding to those battery states.

Step 2: Forecasting Layer – Price Space

Use the battery states and price regimes generated from Step 1 as input data.
Train a new HMM to model the transitions between price regimes (low, normal, peak) and the corresponding price emissions.
Forecast the price series for the next 24 hours based on the learned regime-switching behavior.

Step 3: Optimization – Battery Dispatch

Feed the forecasted prices into the battery optimization logic.
Use MILP (Gurobi solver), Differential Evolution (DE), Simulated Annealing (SA) or Reinforced Learning (RL) to optimize the charge/discharge schedule for the next 24 hours.

Step 4: Update and Repeat

Simulate the next day’s battery behavior using the optimized schedule.
Feed the updated battery states and prices back into the forecasting layer for further training.


## Features
1. Price categorization (low, normal, peak).
2. Battery state simulation using HMM.
3. Schedule optimization with RL.
4. Short-term price forecasting via HMM.

## Proposed structure of the app 

battery-simulation-app/
├── backend/
│   ├── app.py
│   ├── price_categorizer.py
│   ├── battery_simulator.py
│   ├── rl_optimizer.py
│   ├── price_forecaster.py
│   ├── central_controller.py
│   ├── utils/
│       ├── data_loader.py
│       ├── visualizer.py
├── frontend/
│   ├── public/
│   ├── src/
│       ├── components/
│           ├── PriceCategorizerComponent.js
│           ├── BatterySimulatorComponent.js
│           ├── RLOptimizerComponent.js
│           ├── PriceForecasterComponent.js
│       ├── App.js
│       ├── index.js
├── tests/
│   ├── test_price_categorizer.py
│   ├── test_battery_simulator.py
│   ├── test_rl_optimizer.py
│   ├── test_price_forecaster.py
├── README.md
├── requirements.txt
├── package.json
├── .gitignore

## Technology - working hypothesis regarding technology stack

1. Backend. hmmlearn and libraries this kit is based upon seem a good choice for backend. FastAPI plus fastapi_scheduler seem necessary for real time price data processing, serving and scheduling. 
2. Realtime data integration. WebSocket for API-based data feed seems a good option. AWS S3 and MongoDB Atlas seem an option for data storage and noSQL databeses. 
3. Frontend. React together with interactive chart libararies / data viz componenets like Nivo or Victory seem a good choice for frontend.
4. Cloud. AWS EC2. As an interim solution we can use Lambda.
