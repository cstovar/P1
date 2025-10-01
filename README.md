# Rootly

### Organization link:
https://github.com/swarch-2f-rootly


## 👥 Team 2F
- Carlos Santiago Sandoval Casallas  
- Cristian Santiago Tovar Bejarano  
- Danny Marcelo Yaluzan Acosta  
- Esteban Rodriguez Muñoz  
- Santiago Restrepo Rojas  
- Gabriela Guzmán Rivera  
- Gabriela Gallegos Rubio  
- Andrés Camilo Orduz Lunar  

---

## 💻 Software System
- **Name:** Rootly  
- **Logo:**  
  ![Rootly Logo](image.jpeg)  
- **Description:**  
**ROOTLY** is an agricultural monitoring system designed under a service-oriented distributed architecture that integrates field devices and cloud services.

At the **physical layer**, sensors and microcontrollers capture environmental and soil data —such as humidity, temperature, and location— and transmit them to the central platform.

At the persistence layer, the architecture combines relational databases (PostgreSQL) for transactional information (users, configurations, profiles) with specialized NoSQL storage (InfluxDB) for time-series sensor data.

At the service layer, independent logical components process and validate the information, applying business rules and exposing REST/GraphQL interfaces for consumption by other modules.

Finally, at the presentation layer, a decoupled frontend (SOFEA) allows users to access reports with relevant metrics, visualize analytics, and manage their crop configurations in real time.

This architecture ensures scalability, resilience, and efficiency, enabling each service to grow independently, isolate failures, and optimize both structured data and sensor metrics.

---

## 🏛️ Architectural Structures

### 📌 Architectural Styles
1. **Client–Server**  
   The browser (client) requests resources from the frontend (server) via HTTP. This ensures separation between presentation and business logic.  

2. **SOFEA (Service-Oriented Front-End Architecture)**  
   The frontend is a decoupled layer that consumes only services exposed via REST/GraphQL, which facilitates scalability and backend reuse by different clients.  

3. **Anti-Pattern Correction**  
   Direct access from the analytics reporting service to the NoSQL database was removed, replacing it with a controlled interface in the data collection service. This improves security and reduces coupling.


---

### 📌 Architectural Elements & Relations


#### **Web Browser**
- External actor that runs the frontend (SPA in React).  
- Consumes REST/GraphQL APIs with JWT authentication.  
- Serves as the user interface between client and system.  

#### **Frontend (rootly-frontend)**
- User interface built with React + TypeScript.  
- Displays agricultural metrics and real-time dashboards.  
- Communicates only with backend services (port 3000).  

#### **Backend Services**
- **Authentication and Roles (rootly-authentication-and-roles-backend):** Handles login, roles, and JWT tokens (port 8001, PostgreSQL).  
- **User and Plant Management (rootly-user-plant-management-backend):** Manages users, plants, and microcontrollers (port 8003, PostgreSQL).  
- **Analytics (rootly-analytics-backend):** Processes data, generates metrics, and analytical reports (port 8000, InfluxDB).  
- **Data Management (rootly-data-management-backend):** Ingests and validates data from IoT devices, storing it in InfluxDB (port 8002, GraphQL).  

#### **IoT Devices**
- ESP8266 microcontrollers with environmental sensors.  
- Capture humidity, temperature, etc., and send data to the backend via HTTP POST.  

#### **Databases and Storage**
- **PostgreSQL:** Users/Roles (port 5432) and Plants/Devices (port 5433).  
- **InfluxDB:** Time-series storage of agricultural data (port 8086).  
- **MinIO:** Distributed storage for profiles, plant images, and backups.  

### ⚙️ Frontend and Backend Services

| Component / Service                             | Port(s) | Description |
|-------------------------------------------------|---------|-------------|
| **Frontend (rootly-frontend)**                  | 3000    | SPA in React + TypeScript, displays real-time dashboards and metrics |
| **Authentication and Roles Service**            | 8001    | Handles login, roles, JWT, and user lifecycle management |
| **User and Plant Management Service**           | 8003    | Manages plants, devices, and user–plant–IoT relationships |
| **Analytics Service**                           | 8000    | Processes sensor data, generates reports, and provides advanced analytics |
| **Data Management Service**                     | 8002    | Ingests IoT data, validates it, and stores it in InfluxDB and MinIO |



### 📂 Databases and Storage

| Component / Service                   | Port(s)   | Description |
|---------------------------------------|-----------|-------------|
| **PostgreSQL – Authentication**       | 5432      | `auth_db` with users, roles, permissions, sessions, and tokens |
| **PostgreSQL – Plant Management**     | 5433      | `rootly` with user–plant–device associations |
| **InfluxDB – Time Series**            | 8086      | `agricultural_data` bucket with measurements and historical metrics |
| **MinIO Auth**                        | 9002–9003 | Storage for profile photos |
| **MinIO Data Lake**                   | 9000–9001 | Data files, backups, unstructured content |
| **MinIO User Plant**                  | 9004–9005 | Plant images and user resources |


---

---

## 🛠️ Prototype – Deployment Instructions

### 📋 Requirements
- [Docker](https://docs.docker.com/get-docker/)  
- [Docker Compose](https://docs.docker.com/compose/install/)  
- [Git](https://git-scm.com/downloads)  
- Command-line console  

### 📂 Clone Repositories
```bash
git clone https://github.com/swarch-2f-rootly/rootly-frontend.git
git clone https://github.com/swarch-2f-rootly/rootly-analytics-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-deploy.git
git clone https://github.com/swarch-2f-rootly/rootly-user-plant-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-authentication-and-roles-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-data-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-microcontroller.git
```

### ▶️ Deploy with Docker

```bash
cd rootly-deploy/
./start.sh

