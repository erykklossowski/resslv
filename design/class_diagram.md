+-------------------+         +-------------------+
| PriceCategorizer  |         | BatterySimulator  |
|-------------------|         |-------------------|
| - raw_prices      |         | - hmm_model       |
| - categories      |         | - viterbi_path    |
|-------------------|         |-------------------|
| + categorize()    |-------->| + train_hmm()     |
| + get_categories()|         | + generate_viterbi|
+-------------------+         | + simulate_states()|
                              +-------------------+
                                     |
                                     v
                          +-------------------+
                          | RLOptimizer       |
                          |-------------------|
                          | - rl_agent        |
                          | - reward_func     |
                          |-------------------|
                          | + train_agent()   |
                          | + optimize_sched()|
                          +-------------------+
                                     |
                                     v
                          +-------------------+
                          | PriceForecaster   |
                          |-------------------|
                          | - price_hmm       |
                          | - forecast_horizon|
                          |-------------------|
                          | + train_price_hmm()|
                          | + forecast_prices()|
                          +-------------------+
                                     |
                                     v
                          +-------------------+
                          | CentralController |
                          |-------------------|
                          | + run_simulation()|
                          | + get_results()   |
                          +-------------------+
                                     |
                                     v
                          +-------------------+
                          | UserInterface     |
                          |-------------------|
                          | - inputs          |
                          | - outputs         |
                          |-------------------|
                          | + collect_inputs()|
                          | + display_results()|
                          +-------------------+
