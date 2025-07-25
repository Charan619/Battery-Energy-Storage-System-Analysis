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
