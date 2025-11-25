## Current version

``` python
import time
import paho.mqtt.client as mqtt
from bmp280 import BMP280
import RPi.GPIO as GPIO

broker = "broker.emqx.io"
port = 1883
topictemperature = f"/lxweather/temperature"
topicpressure = f"/lxweather/pressure"
client = mqtt.Client()
client.connect(broker, port)

GPIO.setmode(GPIO.BOARD)
GPIO.setup(16, GPIO.OUT, initial=GPIO.LOW)

try:
    from smbus2 import SMBus
except ImportError:
    from smbus import SMBus

bus = SMBus(1)
bmp280 = BMP280(i2c_dev=bus)

while True:
    # Read temperature and pressure
    temperature = bmp280.get_temperature()
    format_temp = "{:.2f}".format(temperature)
    pressure = bmp280.get_pressure()
    format_press = "{:.2f}".format(pressure)

    # Print formatted values
    print(f"Temperature: {format_temp} °C")
    print(f"Pressure: {format_press} hPa")

    # Publish sensor values
    client.publish(topictemperature, format_temp)
    client.publish(topicpressure, format_press)

    # Wait for 5 seconds
    time.sleep(5)
````
## Suggested version
```python
import paho.mqtt.client as mqtt
import time

BROKER = "mqtt.example.com"
TOPIC = "sensor/data"

def on_connect(client, userdata, flags, rc):
    print("Connected with result code", rc)
    client.subscribe(TOPIC)

def on_disconnect(client, userdata, rc):
    print("Disconnected. Trying to reconnect…")
    time.sleep(5)

client = mqtt.Client()
client.on_connect = on_connect
client.on_disconnect = on_disconnect

client.connect(BROKER, 1883, keepalive=60)
client.loop_start()

while True:
    # your sensor reading + publish logic
    publish_data = {"temp": 20, "pressure": 1000}
    client.publish(TOPIC, str(publish_data))
    time.sleep(5)
````
## Service file  
```bash
sudo nano /etc/systemd/system/mqtt-sensor.service
[Unit]
Description=MQTT Sensor Publisher
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/mqtt_sensor.py
WorkingDirectory=/home/pi
StandardOutput=inherit
StandardError=inherit
Restart=always
RestartSec=5

# Prevent rapid restart loops
StartLimitBurst=5
StartLimitIntervalSec=60

[Install]
WantedBy=multi-user.target
````
## Enable and start  
```bash
sudo systemctl daemon-reload
sudo systemctl enable mqtt-sensor.service
sudo systemctl start mqtt-sensor.service
```
## Check status, View logs  
```bash
systemctl status mqtt-sensor.service
journalctl -u mqtt-sensor.service -f
````
## Improved program  - emqdemo.py
```python
import time
import paho.mqtt.client as mqtt
from bmp280 import BMP280
import RPi.GPIO as GPIO

# MQTT settings
broker = "broker.emqx.io"
port = 1883
topictemperature = "/lxweather/temperature"
topicpressure = "/lxweather/pressure"

# GPIO setup
GPIO.setmode(GPIO.BOARD)
GPIO.setup(16, GPIO.OUT, initial=GPIO.LOW)

# Try to import SMBus
try:
    from smbus2 import SMBus
except ImportError:
    from smbus import SMBus

bus = SMBus(1)
bmp280 = BMP280(i2c_dev=bus)


# ---------------- MQTT CALLBACKS ---------------- #

def on_connect(client, userdata, flags, rc):
    print(f"[MQTT] Connected (rc={rc})")

def on_disconnect(client, userdata, rc):
    print("[MQTT] Disconnected → retrying in 5 seconds…")
    time.sleep(5)


# Create MQTT client
client = mqtt.Client()
client.on_connect = on_connect
client.on_disconnect = on_disconnect


# Keep trying initial connection until success
while True:
    try:
        print(f"[MQTT] Connecting to {broker}:{port}…")
        client.connect(broker, port, keepalive=60)
        break
    except Exception as e:
        print("[MQTT] Connect failed:", e)
        time.sleep(5)

# Start loop thread so reconnect works
client.loop_start()


# ---------------- MAIN LOOP ---------------- #

try:
    while True:
        try:
            # Read sensor
            temperature = bmp280.get_temperature()
            pressure = bmp280.get_pressure()

            format_temp = f"{temperature:.2f}"
            format_press = f"{pressure:.2f}"

            # Print to terminal
            print(f"Temperature: {format_temp} °C | Pressure: {format_press} hPa")

            # Publish MQTT
            client.publish(topictemperature, format_temp)
            client.publish(topicpressure, format_press)

        except Exception as err:
            print("[ERROR] Sensor or MQTT publish error:", err)

        time.sleep(5)

except KeyboardInterrupt:
    print("Stopping (Ctrl-C)…")

finally:
    client.loop_stop()
    GPIO.cleanup()
    print("Cleanup done. Exit.")
````





