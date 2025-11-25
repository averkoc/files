<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather Station</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
        .value { font-size: 2em; }
        #status { 
            position: fixed; 
            bottom: 0; 
            width: 100%; 
            padding: 10px; 
            font-weight: bold; 
            border-top: 1px solid #ccc; 
        }
        .connected { color: white; background-color: green; }
        .disconnected { color: white; background-color: red; }
        .reconnecting { color: black; background-color: yellow; animation: flash 1s infinite; }
        @keyframes flash {
            0%, 50%, 100% { opacity: 1; }
            25%, 75% { opacity: 0.5; }
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js"></script>
    <script>
        const client = new Paho.MQTT.Client("broker.emqx.io", Number(8084), "clientId");

        const statusEl = document.getElementById("status");

        function updateStatus(state) {
            if (state === "connected") {
                statusEl.innerText = "Broker Status: Connected";
                statusEl.className = "connected";
            } else if (state === "disconnected") {
                statusEl.innerText = "Broker Status: Disconnected";
                statusEl.className = "disconnected";
            } else if (state === "reconnecting") {
                statusEl.innerText = "Broker Status: Reconnecting...";
                statusEl.className = "reconnecting";
            }
        }

        client.onConnectionLost = function (responseObject) {
            if (responseObject.errorCode !== 0) {
                console.log("onConnectionLost:" + responseObject.errorMessage);
                updateStatus("reconnecting");
                reconnect();
            }
        };

        client.onMessageArrived = function (message) {
            if (message.destinationName === "/lxweather/temperature") {
                document.getElementById("temperature").innerText = message.payloadString;
            } else if (message.destinationName === "/lxweather/pressure") {
                document.getElementById("pressure").innerText = message.payloadString;
            }
        };

        function onConnect() {
            console.log("Connected");
            updateStatus("connected");
            client.subscribe("/lxweather/temperature");
            client.subscribe("/lxweather/pressure");
            startKeepAlive();
        }

        function reconnect() {
            setTimeout(() => {
                client.connect({ onSuccess: onConnect, useSSL: true });
            }, 5000); // Reconnect after 5 seconds
        }

        function startKeepAlive() {
            setInterval(() => {
                if (client.isConnected()) {
                    client.send("/keepalive", "ping", 0);
                }
            }, 60000);
        }

        window.addEventListener("load", () => {
            updateStatus("disconnected");
            client.connect({ onSuccess: onConnect, useSSL: true });
        });
    </script>
</head>
<body>
    <h1>Current Indoor Weather (SM)</h1>
    <div>
        <p>Temperature: <span class="value" id="temperature">--</span> °C</p>
        <p>Pressure: <span class="value" id="pressure">--</span> hPa</p>
    </div>
    <div id="status">Broker Status: Disconnected</div>
</body>
</html>
