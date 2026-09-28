# Robot Telemetry Simulator

**Live page:** https://armelai.github.io/robot-telemetry-sim-web/

A fake fleet of warehouse robots (AMRs) that runs entirely in your browser. Each robot
publishes its state to an MQTT broker over WebSockets, one message at a time. It's meant as
a data source for an MQTT → Telegraf → QuestDB → Grafana pipeline.

## Using it

1. Open the page and enter a broker URL. It must be MQTT over WebSockets:
   - `ws://localhost:9001` for a broker on your own computer, for example Mosquitto with a
     `protocol websockets` listener.
   - A public test broker such as `wss://broker.hivemq.com:8884/mqtt`. Anyone can read those
     topics, so choose a unique topic name.
2. Pick the number of robots (1–10) and the publish rate per robot (1–10 Hz), then press
   **Start**.

**Localhost from this page:** browsers block a public website from reaching `localhost`
until you allow it. When Chrome or Edge asks for **Local network access**, choose **Allow**,
or allow it later from the site-settings icon next to the address bar. Brave calls it
**Localhost access**. Safari may block it entirely.

## Message format

- **Topic:** `robots/telemetry`, QoS 0, not retained.
- **Payload:** one line of Influx Line Protocol per message, with a nanosecond timestamp:

```
robot,robot_id=amr-01,status=moving x=12.40,y=3.80,heading=1.571,speed=0.82,battery=87.3,motor_temp=41.2,obstacle_dist=2.40,odometer=153.7 1790000000123000000
```

| Field | Unit / values |
|---|---|
| `robot_id` (tag) | `amr-01` … `amr-10` |
| `status` (tag) | `idle`, `moving`, `picking`, `charging`, `error` |
| `x`, `y` | metres on a 40 m × 20 m floor, origin bottom-left |
| `heading` | radians, 0 = +x |
| `speed` | m/s |
| `battery` | % |
| `motor_temp` | °C |
| `obstacle_dist` | metres (5 = nothing in range) |
| `odometer` | metres since the page was opened |

Telegraf can read this directly with `inputs.mqtt_consumer` and `data_format = "influx"`.

## Privacy

The page has no backend and stores nothing on a server. It remembers your broker settings in
your browser's local storage, but never the password. Its only external dependency is
[MQTT.js](https://github.com/mqttjs/MQTT.js), loaded from jsDelivr.
