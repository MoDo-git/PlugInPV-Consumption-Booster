# PlugInPV-Consumption-Booster
The Consumption Booster is an intelligent power management solution for plug-in residential photovoltaic (PV) systems. By continuously monitoring total power usage and production with a Shelly 3EM device, the script detects excess solar energy and automatically activates lower-wattage devices to consume it efficiently.

## Overview
This script is designed to manage and optimize the energy usage of a set of devices connected to a balcony Photovoltaic (PV) plant. The main goal is to increase the self-consumption of the generated solar energy by intelligently controlling the devices based on the total power consumption and production.

## Features
Continuous Monitoring: The script continuously monitors the total power consumption and production using a Shelly 3EM device.
Excess Solar Energy Consumption: When excess solar energy is detected (negative total power), the script tries to turn on devices with lower wattage to consume as much of the excess energy as possible.
Prioritizing Higher Wattage Devices: When turning off devices to maintain a positive total power, the script checks if turning off a lower wattage device would enable a higher wattage device to be turned on, helping to maximize the consumption of available solar energy.
Minimum Runtime Consideration: The script ensures that devices are not turned off before their minimum runtime has been reached, to prevent frequent switching and ensure efficient device operation.
Simulation Options: The script provides options to simulate bad weather conditions and adjust the wattage offset, allowing users to test the system's behavior under different scenarios.
## Usage
Device Configurations: Ensure that the device configurations (IP addresses, wattage, and minimum runtime) are accurate for your setup.
Shelly 3EM IP Address: Verify the IP address of the Shelly 3EM device, which is used for total energy measurement.
Simulation Options: Adjust the simulation options (bad weather and wattage offset) as needed to test different scenarios.
Run the Script: Run the script and observe the output, which includes debugging information about the device states, power consumption, and actions taken by the script.
## Benefits
Automatic optimization of energy consumption based on available solar energy
Efficient utilization of excess solar energy by turning on lower wattage devices
Intelligent device switching to prioritize higher wattage devices
Simulation capabilities to test the system's behavior under different conditions
Increased self-consumption of the balcony PV plant's generated solar energy
## Limitations
The script assumes that the device configurations and Shelly 3EM IP address are correctly provided.
The script does not currently support dynamic addition or removal of devices.
The simulation options may not accurately represent real-world weather conditions or wattage variations.
## Future Improvements
Implement a more robust device management system, allowing for dynamic addition and removal of devices.
Enhance the simulation options to provide more accurate representation of real-world scenarios.
Integrate with external data sources (e.g., weather forecasts) to improve decision-making.
Provide a user-friendly interface for configuring and monitoring the system.
