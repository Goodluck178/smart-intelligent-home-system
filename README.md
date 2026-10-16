# Academic Project

# Smart Intelligent Home System

https://ugochukwuofodum.wordpress.com/2025/05/09/develop-core-functionalitiesmovement-detection-and-occupancy-detection/

# Smart Intelligent Home System

A Raspberry Pi based home monitoring prototype that brings together environmental sensing, distance detection, local hardware feedback, email notifications, and a small HTTP interface.

Project areas: Python · Raspberry Pi · GPIO · I²C · Flask · sensor integration

## Project overview

This project demonstrates how a software application can coordinate physical devices and expose useful readings to other clients. It reads temperature and humidity from a DHT11, distance from an ultrasonic sensor, and an analogue light level through an ADC. Readings are shown on an LCD and in the console; the system also drives LEDs and a buzzer and sends email summaries.

The Flask service runs alongside the sensor cycle and provides simple endpoints for the latest readings. The project is a practical portfolio example of Python application structure, working with hardware interfaces, integrating third-party libraries, and building a lightweight web API.

## What it demonstrates

Python application development separates sensor routines, initialization, display behaviour, HTTP routes, and cleanup into named functions.
Hardware and peripheral integration work with Raspberry Pi GPIO, I²C devices, an LCD, LEDs, a buzzer, and multiple sensor types.
Web development fundamentals expose sensor readings through Flask routes while the monitoring loop runs in a separate thread.
External service integration sends periodic email summaries with readings and timestamps.
Systems thinking coordinates sensor sampling, visible feedback, notifications, and resource cleanup in one application.

## Features

Reads distance from an ultrasonic sensor and sounds a buzzer during sampling.
Reads an analogue photoresistor through a PCF8591 or ADS7830 ADC and adjusts an LED's brightness.
Reads DHT11 temperature and humidity and briefly lights a status LED.
Displays welcome, CPU-temperature, time, and sensor information on an I²C LCD.
Sends an email summary after each sensor routine completes its configured number of readings.
Serves the most recently stored readings over HTTP.

## How it works

At startup, the program initializes the LCD, shows a welcome screen and then displays the Raspberry Pi CPU temperature and current time. It starts the Flask server on port `5000` in a background thread, then runs the sensor routines in sequence:

1. Ultrasonic distance sampling (10 readings, followed by an email).
2. Photoresistor/ADC sampling (10 readings, followed by an email).
3. DHT11 sampling (5 readings, followed by an email).
4. A 60-second pause before the next cycle.

Each routine updates shared values used by the HTTP endpoints. 
This is a small single-process prototype; it is not designed as a production monitoring or security system.

## HTTP endpoints

The Flask service returns plain-text readings and a timestamp:

| Method | Endpoint | Data |
| --- | --- | --- |
| `GET` | `/ultrasonic` | Latest distance in centimetres |
| `GET` | `/photoresistor` | Latest ADC value and calculated voltage |
| `GET` | `/dht11` | Latest humidity and temperature |

For example, with the service running on the Raspberry Pi, request `http://<raspberry-pi-address>:5000/dht11`.

## Hardware configuration

The GPIO numbers below use the `gpiozero` default BCM numbering.

| Component | Connection / configuration |
| --- | --- |
| DHT11 | BCM GPIO 12 |
| Ultrasonic sensor | Trigger: BCM GPIO 23; echo: BCM GPIO 24 |
| Buzzer | BCM GPIO 26 |
| Status LED | BCM GPIO 19 |
| PWM LED | BCM GPIO 17 |
| LCD1602 I²C backpack | Automatically detects address `0x27` or `0x3F` |
| ADC | PCF8591 at `0x48` or ADS7830 at `0x4B`; reads analogue channel 0 |

Check the voltage requirements and wiring for each module before connecting it to a Raspberry Pi. Use appropriate level shifting or other protection where a module's output voltage is not Raspberry Pi GPIO-safe.

## Requirements

Raspberry Pi running a Linux distribution with I²C enabled.
Python 3 and the project's hardware/software dependencies: `gpiozero`, `pigpio`, `smbus`, Flask, and `yagmail`.
The Freenove DHT native library at `/usr/lib/libdht.so`, required by `Freenove_DHT.py`.
The listed sensors and peripherals, wired to match the configuration above.
A configured email account capable of sending messages.

The repository does not currently include a dependency lockfile or an automated setup script. Hardware-specific libraries and the DHT native library may need to be installed separately for the Raspberry Pi OS version in use.

## Running on Raspberry Pi

1. Clone or copy the project to a Raspberry Pi and connect the required hardware.
2. Enable I²C and install the system and Python dependencies listed above.
3. Install the Freenove DHT native library and confirm that the LCD and ADC appear on the I²C bus.
4. Configure email sending securely before starting the application (see  Security and deployment notes).
5. Run the program from the project directory:

   ```bash
   python3 smart_intelligent.py
   ```

The script starts the `pigpiod` daemon using `sudo`; make sure the environment is configured to allow that operation. 
Stop the application with `Ctrl+C` so its cleanup routine can close the hardware resources.

## Repository contents

| File | Purpose |
| --- | --- |
| `smart_intelligent.py` | Application entry point, sensor routines, notifications, and Flask routes |
| `ADCDevice.py` | I²C ADC support for PCF8591 and ADS7830 devices |
| `Freenove_DHT.py` | Python wrapper around the Freenove DHT native library |
| `LCD1602.py` | I²C LCD1602 driver |
| `DHT11.py` | Standalone DHT11 example |
| `UltrasonicRanging.py` | Standalone ultrasonic-ranging example |

## Security and deployment notes

Email credentials: The current application initializes email from credential values in its source file. Do not publish or reuse real credentials in source control. If valid credentials have ever been committed or shared, revoke or rotate them. Before use, update the application to read secrets from a secure configuration source; environment-variable configuration is not implemented yet.
HTTP access: Flask binds to `0.0.0.0` on port `5000`, and the endpoints have no authentication. Keep the service on a trusted network; do not expose it directly to the public internet.
Prototype scope: Sensor values are held in process memory and returned as plain text. There is no persistent storage, authentication, authorization, or production-grade error handling.





