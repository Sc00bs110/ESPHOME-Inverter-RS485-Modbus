ESPHome Solar Inverter Monitor (Sunsynk/Deye via Modbus RS485)

This ESPHome project provides a robust and efficient solution for monitoring Sunsynk, Deye, or other compatible solar inverters using a Lilygo ESP32 board with a built-in RS485 interface. It connects directly to the inverter's Modbus port and exposes a wide range of sensors and controls to Home Assistant.

A key feature of this configuration is its optimized, tiered polling strategy, which reduces the load on both the ESP32 and the inverter by querying critical, fast-changing data more frequently than static settings or daily totals.

Features

Direct RS485 Modbus RTU Communication: No need for extra cloud services or proprietary loggers.

Optimized Polling: Uses a single Modbus controller with a tiered update schedule (skip_updates) to ensure efficiency:

High Frequency (5s): For real-time power values (Solar, Grid, Load, Battery).

Medium Frequency (30s): For regularly changing data (Temperatures, SOC, Daily Totals).

Low Frequency (60s): For settings, configuration, and lifetime totals.

Comprehensive Data Monitoring: Exposes dozens of entities to Home Assistant, including:

Real-time Power (PV, Grid, Load, Battery, etc.)

Daily and Total Energy Figures (kWh)

Battery State of Charge (SOC), Voltage, Current, and Temperature

PV String Voltages and Currents

Inverter and Heatsink Temperatures

Grid Status and Frequency

Inverter Control: Provides Home Assistant entities to control inverter settings, such as:

Setting charge/discharge currents

Toggling grid charging and solar export

Changing work modes and energy patterns

Easy Configuration: Uses ESPHome substitutions to manage device names and polling intervals from one central place.

Hardware Specific: Tailored for the Lilygo T-CAN485 ESP32 board, using its integrated RS485 transceiver.

Hardware Requirements

ESP32 Board: Lilygo T-CAN485 (or another ESP32 with a MAX485/equivalent RS485 transceiver).

Solar Inverter: A Sunsynk, Deye, or other compatible inverter with an RS485 Modbus port.

Wiring: A suitable cable to connect the ESP32's RS485 A/B terminals to the inverter's Modbus A/B terminals.

Software Requirements

ESPHome: To compile and upload the firmware.

Home Assistant: For seamless integration and control.

Configuration

The inverter.yaml file is designed to be easily configurable through the substitutions block at the top.

substitutions:
  friendly_name: "inverter"
  # Define update intervals for easier modification
  fast_update_interval: "5s" # Base interval for the controller
  medium_skip_updates: "5"   # Effective interval: 30s (5s * (5 skips + 1))
  slow_skip_updates: "11"    # Effective interval: 60s (5s * (11 skips + 1))


friendly_name: Sets the base name for all entities created in Home Assistant.

fast_update_interval: The main polling rate of the Modbus controller. All real-time sensors update at this frequency.

medium_skip_updates / slow_skip_updates: These values determine the polling frequency for less critical sensors by skipping a number of fast update cycles. The formula is: Effective Interval = fast_update_interval * (skip_updates + 1).

WiFi & API Credentials

This project requires a secrets.yaml file in your ESPHome configuration directory to store your sensitive credentials. Create the file with the following content:

# secrets.yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"


The Home Assistant API key is hardcoded in the inverter.yaml file. You should generate a new one for your setup.

Installation & Setup

Clone the Repository: Download the inverter.yaml file to your ESPHome configuration directory.

Create secrets.yaml: Create the secrets.yaml file as described above and enter your WiFi credentials.

Verify Hardware Pins: The configuration uses the following pins for the Lilygo T-CAN485 board. If you are using different hardware, adjust these pins under the uart: section.

tx_pin: 17

rx_pin: 16

flow_control_pin: 4 (for RS485 direction control)

Compile & Upload: Compile the inverter.yaml file in ESPHome and upload it to your ESP32 board.

Home Assistant Integration

Once the device is connected to your network, Home Assistant should automatically discover it through the ESPHome integration.

Navigate to Settings > Devices & Services.

An "ESPHome" device should appear as a discovered integration. Click Configure.

Enter the encryption key found in the api: section of your inverter.yaml file.

The inverter device and all its entities will be added to Home Assistant.

Troubleshooting & Notes

Incorrect Sensor Values: Some inverter firmware versions may report values with a different scale (e.g., Watts vs. tenths of a Watt). If you notice a sensor is off by a factor of 10, add or adjust the multiply: filter for that specific sensor in the YAML file.

Modbus Errors: If you see frequent CRC check failed or no response received errors in the log, it may indicate electrical noise on the RS485 bus or incorrect A/B wiring. Ensure your wiring is correct and twisted if possible. You can also try slightly increasing the command_throttle value in the modbus_controller section.

License

This project is provided under the MIT License. See the LICENSE file for details.
