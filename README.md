# Smart Home System Simulation with Wokwi

!\[Smart Home Dashboard\](screenshots/dashboard-preview.png)

\## Overview

This project simulates a comprehensive Smart Home System using the Wokwi electronics simulator. The system integrates multiple sensors and actuators controlled by an ESP32 microcontroller, with cloud connectivity through MQTT protocol and a Node-RED dashboard for visualization and control. This simulation demonstrates core IoT principles including automated lighting control, environmental monitoring, and security systems.

\## Features

\- \*\*Environmental Monitoring\*\*: Real-time temperature and humidity tracking with DHT22 sensor

\- \*\*Automated Lighting\*\*: Smart LED control based on ambient light levels and motion detection

\- \*\*Security System\*\*: Motion-activated alarm with visual and audible alerts

\- \*\*Cloud Connectivity\*\*: MQTT protocol integration for remote monitoring and control

\- \*\*Interactive Dashboard\*\*: Multi-tab Node-RED interface with real-time data visualization

\- \*\*Remote Control\*\*: Ability to arm/disarm security system and control lights from dashboard

\## Simulated Hardware Components

| Component | Purpose | Connection Details |

|-----------|---------|-------------------|

| ESP32 Dev Module | Central controller | N/A |

| DHT22 Sensor | Temperature & humidity monitoring | VCC → 3.3V, GND → GND, DATA → GPIO 15 |

| PIR Motion Sensor | Motion detection for security | VCC → 3.3V, GND → GND, OUT → GPIO 27 |

| LDR Sensor (4-pin) | Ambient light measurement | VCC → 3.3V, GND → GND, SIG → GPIO 34 |

| Blue LED | Smart lighting simulation | Anode → 220Ω resistor → GPIO 16, Cathode → GND |

| Red LED | Status/Alarm indicator | Anode → 220Ω resistor → GPIO 17, Cathode → GND |

| Piezo Buzzer | Audible alarm | Positive → GPIO 26, Negative → 100Ω resistor → GND |

\## Software Dependencies

\### Development Environment

\- Arduino IDE 2.x

\- ESP32 board support package

\- Required libraries:

\- DHT sensor library (Adafruit)

\- PubSubClient library

\- WiFi library

\### Cloud Services

\- HiveMQ Public MQTT Broker (\`broker.hivemq.com:1883\`)

\- Node-RED (local installation or cloud instance)

\- MQTT Explorer (for message monitoring)

\## Setup Instructions

\### 1. Wokwi Circuit Setup

1\. Create a new project at \[Wokwi\](https://wokwi.com)

2\. Import the circuit configuration:

\`\`\`json

https://github.com/allamimad/smart-home-wokwi/blob/main/wokwi/smart-home-circuit.json

\`\`\`

3\. Verify all component connections match the schematic

\### 2. ESP32 Firmware Installation

1\. Install Arduino IDE from \[arduino.cc\](https://www.arduino.cc/en/software)

2\. Add ESP32 board support:

\- File > Preferences

\- Add to "Additional Boards Manager URLs":

\`\`\`

https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package\_esp32\_index.json

\`\`\`

\- Tools > Board > Boards Manager > Install "ESP32 by Espressif Systems"

3\. Install required libraries via Library Manager:

\- "DHT sensor library" by Adafruit

\- "PubSubClient" by Nick O'Leary

4\. Upload firmware:

\`\`\`cpp

https://github.com/allamimad/smart-home-wokwi/blob/main/firmware/smart\_home\_system.ino

\`\`\`

\### 3. Node-RED Dashboard Setup

1\. Access Node-RED editor (local or cloud instance)

2\. Import dashboard configuration:

\`\`\`json

https://github.com/allamimad/smart-home-wokwi/blob/main/dashboard/smart-home-dashboard.json

\`\`\`

3\. Deploy the flow with the red "Deploy" button

4\. Access dashboard at \`/ui\` endpoint of your Node-RED instance

\### 4. MQTT Explorer Configuration

1\. Download and install \[MQTT Explorer\](http://mqtt-explorer.com)

2\. Create new connection with settings:

\- Name: HiveMQ Public Broker

\- Host: broker.hivemq.com

\- Port: 1883

\- No authentication required

3\. Subscribe to topics: \`home/#\` to monitor all messages

\## Usage Guide

\### Dashboard Navigation

The Node-RED dashboard features four tabs for different functionalities:

1\. \*\*Home Tab\*\*: System overview with critical status indicators

\- Temperature and humidity gauges

\- Ambient light level display

\- Security system status

\- Quick control buttons

2\. \*\*Climate Tab\*\*: Detailed environmental monitoring

\- Temperature history chart

\- Humidity history chart

\- Light level trend analysis

3\. \*\*Security Tab\*\*: Alarm system controls

\- Security system arming toggle

\- System status display

\- Alarm history

4\. \*\*Lighting Tab\*\*: Light control interface

\- Manual light controls

\- Light scheduling options

\### System Operation

1\. \*\*Start Simulation\*\*: Click "Run" in Wokwi to start the simulation

2\. \*\*Monitor Sensors\*\*:

\- Adjust DHT22 values using the sensor control panel

\- Change light levels with the LDR control slider

\- Trigger motion detection by clicking the PIR sensor

3\. \*\*Control System\*\*:

\- Use dashboard buttons to arm/disarm security system

\- Manually turn lights on/off regardless of sensor conditions

\- Monitor real-time data in charts and gauges

\## System Architecture

\`\`\`mermaid

graph TD

A\[Sensors\] -->|Data Collection| B(ESP32 Microcontroller)

B -->|Decision Making| C\[Actuators\]

B -->|MQTT Publish| D\[Cloud Broker\]

D -->|MQTT Subscribe| E\[Node-RED Dashboard\]

E -->|MQTT Publish| D

D -->|MQTT Subscribe| B

subgraph Perception Layer

A --> DHT22\[Temperature/Humidity\]

A --> PIR\[Motion Detection\]

A --> LDR\[Ambient Light\]

end

subgraph Processing Layer

B

end

subgraph Application Layer

E

D

end

subgraph Actuation Layer

C --> LED1\[Smart Lights\]

C --> LED2\[Status Indicator\]

C --> Buzzer\[Alarm System\]

end

\`\`\`

\## Troubleshooting

\### Common Issues and Solutions

| Issue | Symptoms | Solution |

|-------|----------|----------|

| MQTT Connection Failure | Dashboard shows no data, ESP32 fails to connect | Verify broker address (\`broker.hivemq.com\`) and port (\`1883\`), check internet connection |

| Sensor Readings Not Updating | Static values in dashboard, no changes when adjusting sensors | Check wiring connections, verify sensor initialization in code, ensure proper pull-up resistors |

| Lights Not Responding to Motion | Motion detected but lights don't turn on | Verify LDR threshold value (default: 2000), check GPIO pin assignments, confirm automation logic |

| Security Alarm False Triggers | Alarm activates randomly without motion | Implement software debouncing, add stabilization delay after power-up |

| Dashboard Values Reverting to Defaults | Temperature/humidity gauges show fixed values | Add data conversion nodes in Node-RED flow to convert string values to numbers |

\### Debugging Tools

\- \*\*Wokwi Serial Monitor\*\*: View ESP32 debug messages and sensor readings

\- \*\*MQTT Explorer\*\*: Monitor all MQTT messages and verify data formats

\- \*\*Node-RED Debug Panel\*\*: Inspect message payloads and flow execution

\## Project Structure

\`\`\`

smart-home-wokwi/

├── firmware/

│ └── smart_home_system.ino # ESP32 main firmware

├── wokwi/

│ └── smart-home-circuit.json # Wokwi circuit configuration

├── dashboard/

│ └── smart-home-dashboard.json # Node-RED dashboard configuration

├── screenshots/

│ ├── circuit-diagram.png # Circuit wiring diagram

│ ├── enhanced_dashboard.png # Dashboard interface screenshot

│ └── system_architecture.png # System architecture diagram

├── documentation/

│ └── Smart_Home_System_Report.pdf # Complete project report

└── README.md # This file

\`\`\`

\## Contributing

Contributions to enhance this Smart Home System simulation are welcome! Please follow these guidelines:

1\. Fork the repository

2\. Create a feature branch (\`git checkout -b feature/your-feature\`)

3\. Commit your changes (\`git commit -am 'Add some feature'\`)

4\. Push to the branch (\`git push origin feature/your-feature\`)

5\. Create a pull request

\## License

This project is licensed under the MIT License - see the \[LICENSE\](LICENSE) file for details.

\## Acknowledgements

\- \[Wokwi\](https://wokwi.com) for the excellent electronics simulation platform

\- \[HiveMQ\](https://www.hivemq.com) for providing a public MQTT broker

\- \[Node-RED\](https://nodered.org) community for the dashboard framework

\- Adafruit for the DHT sensor library

\- Nick O'Leary for the PubSubClient library

\---

\*\*Author\*\*: Allam Imad

\*\*Project Repository\*\*: \[https://github.com/allamimad/smart-home-wokwi\](https://github.com/allamimad/smart-home-wokwi)

\*\*Report\*\*: \[Smart Home System Simulation Report\](documentation/Smart_Home_System_Report.pdf)

\*\*Last Updated\*\*: November 13, 2025
