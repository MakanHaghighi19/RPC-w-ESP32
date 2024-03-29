Project Overview

This project aims to establish communication between a client program running on a PC and an ESP32 server using Remote Procedure Calls (RPCs) via socket programming. The interaction involves toggling LEDs and reading temperatures from a sensor connected to the ESP32. It showcases the implementation of threads to handle non-blocking temperature subscriptions.

Key Features
Toggle LEDs: Control LED states (on/off) through RPCs.
Read Temperature: Use the ESP32's ADC to read temperatures from a sensor.
Subscribe to Temperature Readings: Schedule temperature readings at specified intervals without blocking the main program.
Non-blocking Operation: Utilizes threads for the subscription feature to ensure the main loop remains responsive.

Hardware Setup
Safety First: Always disconnect the ESP32 from your PC when modifying the circuit to prevent short circuits.
Power Supply: Utilize the 3.3V supply (3V3) for all components. Avoid using the 5V supply (5V).
Verify Connections: Double-check all connections according to the schematic provided before powering the ESP32.
Temperature Sensor Orientation: Pay close attention to the temperature sensor's diagram to connect the leads correctly to the ESP32 (Vcc to 3V3 and GND to ground).

Software Implementation
Commands
toggle <color>: Toggles the specified LED's state. Implements an RPC without a return value.
read: Reads the temperature from the sensor via RPC. The command can block, waiting for data.
subscribe <number> <interval>: Initiates non-blocking temperature readings at specified intervals. Utilizes threads for asynchronous operation.
exit: Exits the main function gracefully, ensuring all threads are cleaned up properly.

Usage
The client program prompts the user for commands to interact with the ESP32. Available commands include toggling LEDs, reading the temperature sensor, and subscribing to periodic temperature readings.

