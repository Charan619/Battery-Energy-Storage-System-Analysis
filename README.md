# BESS Optimizer Model

### Utility-scale storage, commonly referred to as Battery Energy Storage Systems (BESS), plays a critical role in maintaining grid stability

The columns in the data are:

- **Day**: Day index (relative to start of dataset)
- **Clock Time**: Local time in HH:MM:SS
- **BESS SoC 1**: One measurement of State of Charge (percentage)
- **BESS SoC 2**: Another independent measurement of State of Charge (percentage)
- **Max Power Export [MW]**: Maximum power the BESS could export during the interval (MW)
- **Max Power Import [MW]**: Maximum power the BESS could import during the interval (MW)
- **Max Temp (degrees C)**: Highest measured cell temperature in the BESS
- **Min Temp (degrees C)**: Lowest measured cell temperature in the BESS
- **Power [MW]**: Average power imported (+ve) or exported (-ve) by the BESS in the interval (MW)

<img width="890" height="440" alt="image" src="https://github.com/user-attachments/assets/e0b26621-7c86-41ba-97a7-0fa022becad2" />
<img width="1009" height="554" alt="image" src="https://github.com/user-attachments/assets/625952f0-31db-4199-9739-0fd1eb2b3e72" />


This model simulates the operation of this particular Battery Energy Storage System (BESS) over a 24-hour period. The primary goal is to **maximize profit** by engaging in energy arbitrage—charging the battery when electricity prices are low and discharging (selling) when prices are high.

We simulate a realistic 24-hour electricity price by layering predictable daily patterns, such as morning/evening demand peaks and a midday solar dip, using Gaussian functions. A layer of random noise is then added to mimic real-world market volatility, providing a dynamic and challenging signal for the BESS to optimize against.

The model is formulated as a Mixed-Integer Linear Program (MILP), which allows it to make complex, conditional decisions based on a set of rules and physical limitations. Below are the key factors and constraints that govern its behavior.

#### 1. The Core Objective: Maximizing Profit

The model's single driving goal is to maximize its net profit. This profit is calculated as:

`Profit = Revenue from Discharging - Cost of Charging`

*   **Revenue:** Earned by discharging power (`p_discharge`) and selling it to the grid at the current `price_signal`.
*   **Cost of Charging:** Incurred by charging the battery (`p_charge`) with power from the grid at the current `price_signal`.


#### 2. Physical BESS Constraints

These constraints model the fundamental physical and hardware limitations of the battery system.

*   **State of Charge (SoC) Dynamics:** This is the core equation that tracks the battery's energy level. The energy in the next time step (`soc[t+1]`) is equal to the energy in the current step (`soc[t]`), plus the energy added during charging, minus the energy removed during discharging.
*   **Round-Trip Efficiency:** The model accounts for energy losses. Due to heat and chemical inefficiencies, not all the energy used to charge the battery is available for discharge. The `CHARGE_EFFICIENCY` and `DISCHARGE_EFFICIENCY` parameters (e.g., 90%) reduce the effective energy stored and delivered.
*   **SoC Limits (`MIN_SOC` and `MAX_SOC`):** To preserve battery health, a real-world BESS is never operated at 0% or 100% of its theoretical capacity. This model restricts the operational SoC to a safe window (e.g., between 10% and 80%).
*   **Maximum Power (`MAX_POWER_MW`):** The BESS hardware (specifically the power conversion system or "inverters") has a maximum rate at which it can charge or discharge, set here to 100 MW.

#### 3. Advanced Operational Constraints

These rules add a layer of realism to simulate how a sophisticated operator would manage the asset for safety, longevity, and to meet contractual obligations.

*   **SoC-Dependent Power Derating (Tapering):** This is a critical safety feature. The model's maximum charge and discharge power is reduced ("derated") at the extremes of its state of charge:
    *   **Discharge Tapering:** When the battery's SoC is low (e.g., below 25%), its maximum discharge power is reduced. This prevents excessive voltage drops that can damage the battery cells.
    *   **Charge Tapering:** When the battery's SoC is high (e.g., above 70%), its maximum charge power is reduced to prevent over-charging, which can also cause damage and safety risks.
*   **Ramp Rate Limits:** A physical system cannot change its power output instantaneously. This constraint limits how quickly the BESS can increase or decrease its charge/discharge rate from one 10-minute interval to the next, reflecting real-world mechanical and electrical inertia. (6.4 in our case)
*   **No Simultaneous Charge & Discharge:** Using binary (True/False) variables, the model is forced to choose one of three states for each interval: **charging**, **discharging**, or **idle**. This enforces the physical reality that a BESS cannot do both at the same time.
*  (Assumption) **Daily Cycle Limit:** To manage long-term health and adhere to warranty conditions, the model is constrained to a maximum daily energy throughput (e.g., 1.25 full cycles per day). This prevents the model from aggressively cycling the battery to capture every small price fluctuation, complementing the degradation cost.

  <img width="955" height="974" alt="image" src="https://github.com/user-attachments/assets/444d1c51-a6d2-440f-a582-1e3c25b9caab" />
