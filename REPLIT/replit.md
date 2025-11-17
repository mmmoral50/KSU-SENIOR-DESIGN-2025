# Overview

PlantGuard is a comprehensive IoT plant monitoring system designed for greenhouse and agricultural applications. It provides real-time monitoring of environmental conditions such as pH, NPK, soil moisture, temperature, humidity, and light intensity. The system includes a React-based dashboard for visualization, an Express.js backend for data management, and integrates with ESP32 sensor nodes. Key capabilities include automated alerts for out-of-range sensor readings and a robust LoRa-based MASTER/SLAVE communication architecture for efficient data collection and remote configuration. The project aims to offer a scalable, real-time monitoring solution for optimizing plant health and agricultural yields.

# User Preferences

Preferred communication style: Simple, everyday language.

# System Architecture

## Frontend
- **Framework**: React 18 with TypeScript.
- **UI/UX**: Shadcn/ui (Radix UI-based) with Tailwind CSS for responsive design and custom theming (including dark mode).
- **State Management**: TanStack React Query for server state.
- **Routing**: Wouter for lightweight client-side navigation.
- **Real-time**: WebSocket integration for live sensor data and Chart.js for historical data visualization.
- **Mobile**: Progressive Web App (PWA) at `/mobile` route for touch-friendly access.

## Backend
- **Framework**: Express.js with TypeScript.
- **Database ORM**: Drizzle ORM with PostgreSQL (Neon serverless) for type-safe operations.
- **API**: RESTful endpoints with WebSocket support.
- **Validation**: Zod schemas for data integrity.
- **Session Management**: Express sessions with PostgreSQL store.

## Database Design
- **Type**: PostgreSQL with Neon hosting.
- **Schema**: Drizzle migrations.
- **Core Entities**: `sensor_nodes`, `sensor_readings` (includes RSSI/SNR for LoRa quality tracking), `alerts`, `esp32_configs`, `lora_configs`.
- **Relationships**: Foreign key constraints for data linkage.

## Real-time Communication
- **Protocol**: WebSockets for bi-directional communication.
- **Data Flow**: Initial sync, live sensor readings, alert notifications.
- **Broadcasting**: Real-time updates pushed to connected dashboard clients.
- **IoT Integration**: MQTT for ESP32 configuration updates and Node-RED integration.
- **Connection Monitoring**: Real-time LoRa signal quality tracking (RSSI/SNR), data rate monitoring, and connection health indicators displayed on dashboard.

## Development Workflow
- **Build**: Vite for frontend, esbuild for backend.
- **Environment**: Concurrent Express API and Vite development servers.
- **Type System**: Shared TypeScript types across frontend, backend, and database.
- **Structure**: Monorepo with shared schema definitions.

## LoRa MASTER ↔ SLAVE Communication
- **Architecture**: Dedicated firmware for MASTER (gateway) and SLAVE (sensor) nodes.
    - `ESP32_PlantGuard_MASTER.ino`: WiFi/MQTT gateway, receives LoRa from SLAVEs, forwards to MQTT.
    - `ESP32_PlantGuard_SLAVE.ino`: Battery-powered sensor node, reads RS485 sensors, transmits data via LoRa, receives config via MQTT.
- **Remote Configuration**: Dashboard-controlled MQTT updates for update intervals, deep sleep, OLED timeout, and LoRa radio settings (frequency, bandwidth, SF, TX power) on both MASTER and SLAVE nodes.
- **Power Management**: Deep sleep and OLED auto-sleep for extended battery life (2-4 weeks for SLAVEs).

# External Dependencies

## Database Services
- **Neon Database**: Serverless PostgreSQL hosting.
- **Drizzle Kit**: Database migration and schema management.

## UI and Styling
- **Radix UI**: Accessible component primitives.
- **Tailwind CSS**: Utility-first CSS framework.
- **Lucide React**: Icon library.
- **Chart.js**: Charting library for data visualization.

## Development Tools
- **TypeScript**: Static type checking.
- **Wouter**: Minimal routing library.

## IoT Integration
- **ESP32 Compatibility**: RESTful API endpoints for microcontroller data submission.
- **Node-RED**: Visual programming tool for sensor automation, data processing, and custom alerts via dedicated API endpoints and MQTT.
- **MQTT**: For real-time configuration updates to ESP32 devices and data forwarding from MASTER nodes.

## Authentication and Session Management
- **Express Sessions**: Server-side session storage.
- **CORS Configuration**: Cross-origin request handling.