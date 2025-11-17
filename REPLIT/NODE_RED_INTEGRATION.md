# Node-RED Integration Guide for PlantGuard

## ⚡ Quick Fix: Re-import Updated Flow

**If your Node-RED dashboard shows zeros for all sensor values**, re-import the updated flow:

1. **Delete the old flow** in Node-RED (select the "PlantGuard Integration" tab, click the hamburger menu → Flows → Delete)
2. **Copy the contents** of `node-red-flows.json` from this project
3. **Import** in Node-RED: Hamburger menu → Import → Paste JSON → Import
4. **Update the API URL**: Double-click each HTTP Request node and replace `YOUR_REPLIT_APP_URL` with your actual Replit URL
5. **Deploy** the flow (click the red "Deploy" button)

**What was fixed:** The "Format Sensor Data" function now explicitly maps all sensor fields (temperature, pH, soilMoisture, nitrogen, phosphorus, potassium, etc.) instead of relying on object spreading which was dropping fields.

---

## Overview
Node-RED can connect to your existing MQTT broker and create visual flows for processing sensor data, triggering alerts, and automating plant care.

## MQTT Connection Settings

Your sensors are already publishing to MQTT. Node-RED can subscribe to the same topics:

### MQTT Broker Configuration
- **Broker**: `test.mosquitto.org`
- **Port**: `1883`
- **Protocol**: `mqtt://`
- **Topics**: 
  - Subscribe: `esp32/+/sensors` (all sensor data)
  - Subscribe: `esp32/SLAVE_001/sensors` (specific node)
  - Publish: `esp32/+/command` (send commands to ESP32)

### Authentication
Currently using public Mosquitto broker (no authentication required)

## Node-RED Flow Examples

### 1. Subscribe to Sensor Data

```json
[
  {
    "id": "mqtt_in",
    "type": "mqtt in",
    "topic": "esp32/+/sensors",
    "broker": "test_mosquitto",
    "name": "ESP32 Sensors"
  },
  {
    "id": "json_parse",
    "type": "json",
    "name": "Parse JSON"
  },
  {
    "id": "debug",
    "type": "debug",
    "name": "Show Sensor Data"
  }
]
```

### 2. Send Processed Data to PlantGuard API

Node-RED can POST data to your API endpoint:

**HTTP Request Node Settings:**
- Method: `POST`
- URL: `https://your-replit-app.replit.dev/api/node-red/sensor-data`
- Content-Type: `application/json`

**Payload Example:**
```json
{
  "nodeId": "node-red-processed",
  "temperature": 25.5,
  "soilMoisture": 65,
  "pH": 6.8,
  "nitrogen": 45,
  "phosphorus": 30,
  "potassium": 35,
  "source": "node-red"
}
```

### 3. Alert Flow Example

Create a flow that:
1. Subscribes to `esp32/+/sensors`
2. Checks if pH is outside safe range (6.0-7.5)
3. Sends HTTP request to create alert
4. Sends email/SMS notification

### 4. Automated Irrigation Flow

Create a flow that:
1. Monitors soil moisture levels
2. When moisture < 40%, publishes to `esp32/SLAVE_001/command`
3. Command: `{"action": "water", "duration": 30}`

## Pre-Built Node-RED Flows

### Complete Sensor Processing Flow

```json
[
  {
    "id": "sensor_input",
    "type": "mqtt in",
    "topic": "esp32/+/sensors",
    "broker": "test_mosquitto",
    "name": "ESP32 Sensor Data"
  },
  {
    "id": "parse_json",
    "type": "json",
    "name": "Parse JSON"
  },
  {
    "id": "extract_node_id",
    "type": "function",
    "func": "msg.nodeId = msg.topic.split('/')[1];\nreturn msg;",
    "name": "Extract Node ID"
  },
  {
    "id": "check_thresholds",
    "type": "function",
    "func": "const payload = msg.payload;\nconst alerts = [];\n\nif (payload.pH < 6.0 || payload.pH > 7.5) {\n  alerts.push({type: 'pH', value: payload.pH});\n}\n\nif (payload.soilMoisture < 40) {\n  alerts.push({type: 'moisture', value: payload.soilMoisture});\n}\n\nif (alerts.length > 0) {\n  msg.alerts = alerts;\n  return msg;\n}\nreturn null;",
    "name": "Check Thresholds"
  },
  {
    "id": "send_to_api",
    "type": "http request",
    "method": "POST",
    "url": "https://your-app.replit.dev/api/node-red/alert",
    "name": "Send Alert to API"
  }
]
```

## Integration with PlantGuard Database

Node-RED can:
1. **Read Data**: Query historical sensor data via GET `/api/sensor-readings/history/:nodeId`
2. **Write Data**: Send processed sensor readings via POST `/api/node-red/sensor-data`
3. **Create Alerts**: Send alerts via POST `/api/node-red/alert`
4. **Control Sensors**: Publish MQTT commands to `esp32/+/command`

## Common Node-RED Nodes to Install

In Node-RED, install these additional nodes for enhanced functionality:

1. **node-red-dashboard** - Create custom dashboards
2. **node-red-contrib-postgresql** - Direct database queries
3. **node-red-contrib-graphs** - Advanced charting
4. **node-red-contrib-moment** - Time/date manipulation
5. **node-red-node-email** - Email notifications
6. **node-red-contrib-twilio** - SMS alerts

## Example Use Cases

### 1. Data Enrichment
- Node-RED receives raw sensor data
- Calculates moving averages
- Adds weather data from external API
- Posts enriched data to PlantGuard

### 2. Multi-Sensor Correlation
- Monitors temperature AND humidity
- Calculates VPD (Vapor Pressure Deficit)
- Stores as custom metric in database

### 3. Scheduled Actions
- Every 6 hours, check soil moisture
- If low, trigger irrigation system
- Log action to database

### 4. External Integrations
- Send daily summary to Google Sheets
- Post alerts to Slack/Discord
- Trigger IFTTT webhooks

## Security Considerations

For production use:
1. Use a private MQTT broker with authentication
2. Enable TLS/SSL for MQTT connections
3. Add API key authentication to Node-RED endpoints
4. Use environment variables for sensitive data

## Getting Started

1. Install Node-RED locally or use Node-RED cloud service
2. Add MQTT broker connection (test.mosquitto.org:1883)
3. Import example flows from this guide
4. Update API URLs to match your Replit app URL
5. Start creating custom automation flows!

## API Endpoints Available

- `GET /api/sensor-nodes` - Get all sensor nodes
- `GET /api/sensor-readings/history/:nodeId` - Get historical data
- `POST /api/node-red/sensor-data` - Submit processed sensor data
- `POST /api/node-red/alert` - Create custom alerts
- `GET /api/sensor-readings/latest` - Get latest readings

## Need Help?

Your PlantGuard system is already MQTT-ready, so Node-RED integration is straightforward. The flows above will get you started with automated plant monitoring and control!
