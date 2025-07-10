<b>Modbus Monitoring for Zabbix Agent 2</b>

This repository provides Zabbix templates specifically designed for monitoring Modbus TCP devices leveraging the native Modbus plugin of Zabbix Agent 2. This approach simplifies deployment by eliminating the need for custom external scripts, relying directly on Zabbix Agent 2's built-in capabilities for Modbus communication.

- Zabbix Agent 2 Modbus Plugin
- Zabbix Templates
- Requirements
- Installation
- Monitored Items
- Triggers
- Graphs
- Value Maps

Zabbix Agent 2 Modbus Plugin:

These templates utilize the modbus.get item key provided by the Zabbix Agent 2 Modbus plugin. This built-in plugin offers native support for Modbus TCP communication, handling connections, data type decoding, and error management directly within the agent.

The modbus.get item key has the following syntax:
modbus.get[<connection_string>,<slave_id>,<function_code>,<address>,<count>,<data_type>]

- <connection_string>: Typically tcp://<IP>:<Port>. In these templates, it's often derived from the host's IP address ({HOST.HOST}) and a macro for the port.
- <slave_id>: The Modbus slave ID (unit ID).
- <function_code>: Modbus function code (e.g., 3 for Holding Registers, 4 for Input Registers).
- <address>: The starting Modbus register address.
- <count>: The number of registers to read.
- <data_type>: The data type to interpret the registers as (e.g., float, uint16, int32, uint32, string).

Zabbix Templates:
This repository includes two distinct Zabbix templates:

- Modbus_an by zabbix agent2.xml (referred to as Modbus monitoring_an)
- Modbus_mp by zabbix agent2.xml (referred to as Modbus monitoring_mp)

Why Two Templates?

The presence of two separate templates is due to their specialized focus on different types of Modbus devices and their unique monitoring requirements. Each template is configured to collect a specific set of parameters, making them suitable for distinct industrial or energy monitoring applications.

Differences Between Templates

The templates vary significantly in the Modbus registers they query and the specific parameters they monitor, reflecting their intended use cases.

Modbus monitoring_an (for Analyzers)

This template is designed for monitoring energy analyzers or similar devices that provide general electrical grid parameters and digital I/O status.

Monitored Parameters:

- Digital Input/Output Status: Items like "Input 0.0 Status" and "Output 0.0 Status" use JavaScript preprocessing to extract bit states from Modbus words. The master item for these is modbus.get[{HOST.HOST},1,3209,1,uint32] for Input Status and modbus.get[{HOST.HOST},1,3,207,1,uint32] for Output Status.
- Electrical Parameters: Voltage ("Voltage L1-N" from address 1), Current ("Current L1" from address 13), Total Harmonic Distortion for Voltage and Current (e.g., "THD-R Voltage L1" from address 43, "THD-R CurrentL1" from address 49), and Frequency ("Frequency" from address 55).
- Power Metrics: Total Apparent Power, Total Active Power (from addresses like 63, 69).
- Modbus Addresses: Uses register addresses commonly associated with energy meters and analyzers.
- Triggers: Includes specific triggers for electrical anomalies such as low/high voltage, high current, high THD, and Modbus communication issues (no data).

Modbus monitoring_mp (for Motor Drives)

This template is tailored for monitoring motor drives or similar industrial automation equipment.

Monitored Parameters:

- Drive States: "Drive Fault State Active", "Drive Ready to Switch On Status", and other derived states from the raw drive state word (address 3240) using JavaScript preprocessing.
- Motor Data: "Motor Output Frequency" (address 3202, with a multiplier of 0.1), "Motor Output Current" (address 3204, with a multiplier of 0.001), "Motor Thermal State" (address 3208), "Motor Output Voltage"(address 3209), and "DC Bus Voltage" (address 7270).
- Operational Metrics: "Motor Run Time" (address 3244) and "Total Energy Consumption" (address 3250).
- Device Identification: "Product Code" (address 7200, read as text).
- Raw States: "Drive State Raw Word" (address 3240) for direct observation.
- Modbus Addresses: Uses addresses typically found in motor drive systems.
- Triggers: Focuses on drive-specific conditions such as drive fault, high motor current, high motor thermal state, and issues with increasing run time or energy consumption.

Requirements:

- Zabbix Server (version 7.0 or higher recommended)
- Zabbix Agent 2 (on the machine that will connect to Modbus devices)
- Zabbix Agent 2 must have the Modbus plugin enabled and configured.

Installation:

Zabbix Agent 2 Configuration

- Install Zabbix Agent 2: Ensure Zabbix Agent 2 is installed and running on the host that will communicate with your Modbus devices.
- Configure Modbus Plugin: Edit the Zabbix Agent 2 configuration file (e.g., /etc/zabbix/zabbix_agent2.conf or a file in zabbix_agent2.d/). You will need to add or uncomment lines to configure the Modbus plugin,typically using Plugins.Modbus.Sessions.
- Example configuration (adjust IP, port, and session name as needed):
- Ini, TOML

Plugins.Modbus.Sessions.modbus_device.Timeout=5s
Plugins.Modbus.Sessions.modbus_device.Mode=tcp
Plugins.Modbus.Sessions.modbus_device.Host=192.168.1.100
Plugins.Modbus.Sessions.modbus_device.Port=502

- modbus_device is a session name. You will refer to this name in Zabbix items.
- For hosts where the IP address and port might vary, you can configure a generic session and override the host/port via Zabbix host macros (as shown in the next step).

Restart Zabbix Agent 2:
Bash

    sudo systemctl restart zabbix-agent2

Template Installation
Download Templates: Download the Modbus_an by zabbix agent2.xml and Modbus_mp by zabbix agent2.xml files from this repository.
Import into Zabbix:

- Navigate to Configuration -> Templates in your Zabbix frontend.
- Click Import in the top right corner.
- Click Browse and select one of the XML template files.
- Click Import.
- Repeat for the second template.

Host Configuration:

- Create/Select Host: Go to Configuration -> Hosts. Create a new host or select an existing one representing your Modbus device.
- Add Zabbix Agent Interface: Ensure the host has a Zabbix Agent interface configured, pointing to the IP address of the machine where Zabbix Agent 2 is running.
- Link Template:
    Go to the Templates tab of the host configuration.
    In the Link new templates section, search for either Modbus monitoring_an or Modbus monitoring_mp (choose the one relevant to your device type) and select it.
    Click Add, then click Update on the host configuration page.
- Configure Macros: The templates use host macros to define Modbus communication parameters.
    On the host configuration page, go to the Macros tab.
    You might need to configure the {$MODBUS_ENDPOINT} macro if the modbus.get items use it directly in the key instead of the session name from zabbix_agent2.conf. Looking at the templates, the modbus.get key uses [{HOST.HOST},... which means the agent will use its own configured sessions or default Modbus TCP parameters based on the host IP. If you've configured a named session in zabbix_agent2.conf (e.g., modbus_device), you might need to manually modify the item keys in the template to modbus.get[modbus_device,...] or rely on Zabbix Agent 2's default behavior.
    The templates themselves do not define a {$MODBUS_TIMEOUT} macro for the item keys directly as this is handled in the Agent 2 configuration.

Monitored Items:
The templates define various items based on their specific Modbus register mappings and utilize the modbus.get key.

Modbus monitoring_an Items:

- digital.input.0_0.status (Input 0.0 Status)
- digital.output.0_0.status (Output 0.0 Status)
- modbus.get[{HOST.HOST},1,3,1,1,float] (Voltage L1-N)
- modbus.get[{HOST.HOST},1,3,13,1,float] (Current L1)
- modbus.get[{HOST.HOST},1,3,43,1,float] (THD-R Voltage L1)
- modbus.get[{HOST.HOST},1,3,49,1,float] (THD-R Current L1)
- modbus.get[{HOST.HOST},1,3,55,1,float] (Frequency)

Modbus monitoring_mp Items:

- drive.fault.state (Drive Fault State Active)
- drive.ready.status (Drive Ready to Switch On Status)
- modbus.get[{HOST.HOST},0,3,3202,1,int16] (Motor Output Frequency)
- modbus.get[{HOST.HOST},0,3,3204,1,uint16] (Motor Output Current)
- modbus.get[{HOST.HOST},0,3,3208,1,uint16] (Motor Thermal State)
- modbus.get[{HOST.HOST},0,3,3240,1,uint16] (Drive State Raw Word)
- modbus.get[{HOST.HOST},0,3,3244,2,uint32] (Motor Run Time)
- modbus.get[{HOST.HOST},0,3,3250,2,uint32] (Total Energy Consumption)
- modbus.get[{HOST.HOST},0,3,7200,8,uint16] (Product Code, with JavaScript preprocessing for string conversion)

Triggers:
Each template includes a set of pre-defined triggers relevant to the device type it monitors.

Modbus monitoring_an Triggers:

- {HOST.NAME} - Low Voltage L1-N ({ITEM.LASTVALUE} V) (Priority: DISASTER)
- {HOST.NAME} - Wysoki Prąd L1 ({ITEM.LASTVALUE} A) (High Current L1, Priority: AVERAGE)
- {HOST.NAME} - Częstotliwość Zbyt Niska ({ITEM.LASTVALUE} Hz) (Frequency Too Low, Priority: HIGH)
- {HOST.NAME} - Sterownik Modbus Niedostępny (Modbus Controller Unavailable, Priority: HIGH, based on nodata() for Voltage L1-N item)
- {HOST.NAME} - Digital Output 0.0 Activated (Priority: INFO)

Modbus monitoring_mp Triggers:

- {HOST.NAME} - DRIVE IS IN FAULT STATE! (Priority: DISASTER)
- {HOST.NAME} - High Motor Output Current ({ITEM.LASTVALUE} A) (Priority: AVERAGE)
- {HOST.NAME} - High Motor Thermal State ({ITEM.LASTVALUE}%) (Priority: WARNING)
- {HOST.NAME} - Motor Run Time Not Increasing (Priority: WARNING)
- {HOST.NAME} - Nieoczekiwana zmiana surowego słowa statusu napędu (Unexpected change in raw drive status word, Priority: HIGH)

Graphs:
The templates provide useful graphs for visualizing the collected data.

Modbus monitoring_an Graphs:

- "Napięcia L1-N i L2-N" (Voltage L1-N and L2-N)
- "Prądy L1, L2, L3" (Currents L1, L2, L3)
- "THD-R Napięcie & Prąd" (THD-R Voltage & Current)
- "Modbus Availability"

Modbus monitoring_mp Graphs:

- "Częstotliwość i Prąd Wyjściowy Silnika" (Motor Output Frequency and Current)
- "Temperatura i Stan Termiczny Napędu" (Drive Temperature and Thermal State)
- "Napięcia Magistrali DC" (DC Bus Voltages)
- "Modbus Availability"

Value Maps:
Both templates extensively use Zabbix Value Maps to translate raw numerical Modbus states into human-readable text.

- Modbus Digital Status: Maps 0 to "OFF" and 1 to "ON" for digital inputs/outputs.
- Modbus Drive State: Maps various bit combinations from the drive state word to meaningful descriptions for the Modbus monitoring_mp template.
- Common Zabbix Status: Maps standard Zabbix internal values (e.g., for availability) to "Available" and "Not available".
