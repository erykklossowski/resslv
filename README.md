# resslv
# Rich app

This project models and optimizes a single battery's and/or aggregated batterries operation using:
- Hidden Markov Models (HMM) for price categorization and forecasting.
- Reinforcement Learning (RL) for schedule optimization.

## Features
1. Price categorization (low, normal, peak).
2. Battery state simulation using HMM.
3. Schedule optimization with RL.
4. Short-term price forecasting via HMM.

## Structure of the app 

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




