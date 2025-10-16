# Rootly 
## Prototype 1

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
<img src="iconRootly.png" alt="Rootly Logo" width="200"/>

- **Description:**  
**ROOTLY** is an agricultural monitoring system designed under a service-oriented distributed architecture that integrates field devices and cloud services.

At the **physical layer**, sensors and microcontrollers capture environmental and soil data —such as humidity, temperature, and location— and transmit them to the central platform.

At the persistence layer, the architecture combines relational databases (PostgreSQL) for transactional information (users, configurations, profiles) with specialized NoSQL storage (InfluxDB) for time-series sensor data.

At the service layer, independent logical components process and validate the information, applying business rules and exposing REST/GraphQL interfaces for consumption by other modules.

Finally, at the presentation layer, a decoupled frontend (SOFEA) allows users to access reports with relevant metrics, visualize analytics, and manage their crop configurations in real time.

This architecture ensures scalability, resilience, and efficiency, enabling each service to grow independently, isolate failures, and optimize both structured data and sensor metrics.

---

## 🏛️ Architectural Structures

  ![Component-and-connector-view](C&C_View.png)

### 🌐 External Actor
- **Web Browser**  
  - The primary external actor.  
  - Executes the frontend Single Page Application (SPA).  
  - Communicates with the system through **HTTP** requests.  

### 🎨 Frontend
- **Frontend Service**  
  - Acts as the presentation layer.  
  - Provides dashboards, plant management, and real-time sensor data visualization.  
  - Communicates with backend services via **REST** and **GraphQL APIs**.  

### ⚙️ Backend Services
1. **be-autentication-and-roles**  
   - Provides user login, role-based access control (RBAC), and token validation.  
   - Exposes a **REST API** to the frontend.  
   - Stores user and role information in a **SQL Database**.  
   - Manages user assets such as profile photos in an **object bucket**.  

2. **be-user-plant-management**  
   - Manages plants, devices, and associations between users and IoT devices.  
   - Exposes a **REST API** to the frontend.  
   - Persists metadata in a **SQL Database**.  
   - Stores additional assets (e.g., plant images) in an **object bucket**.  

3. **be-analytics**  
   - Processes and analyzes sensor data, generating reports and metrics.  
   - Provides a **GraphQL API** for flexible queries.  
   - Depends on the Data Collection Service for historical sensor information.  

4. **be-data-management**  
   - Central ingestion point for IoT sensor data.  
   - Communicates directly with physical devices via **REST**.  
   - Stores validated data in a **NoSQL time-series database**.  
   - Saves raw files and backups in a **bucket**.  
   - Exposes controlled APIs to other services (e.g., Analytics).  

### 📡 IoT Devices
- **microcontroller-device**  
  - Capture field data such as soil humidity and temperature.  
  - Send measurements to the Data Collection Service via **REST API calls**.  

### 🗄️ Data Storage
- **Relational Databases (SQL):** Dedicated to authentication and plant/user management.  
- **NoSQL Database (Time-Series):** Optimized for sensor data (measurements, historical series).  
- **Buckets (Object Storage):** Handle unstructured data such as images, backups, and files.  

---

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
- **Authentication and Roles (be-autentication-and-roles):** Handles login, roles, and JWT tokens (port 8001, PostgreSQL).  
- **User and Plant Management (be-user-plant-management):** Manages users, plants, and microcontrollers (port 8003, PostgreSQL).  
- **Analytics (be-analytics):** Processes data, generates metrics, and analytical reports (port 8000, InfluxDB).  
- **Data Management (be-data-management):** Ingests and validates data from IoT devices, storing it in InfluxDB (port 8002, GraphQL).  

#### **microcontroller-device**
- ESP8266 microcontrollers with environmental sensors.  
- Capture humidity, temperature, etc., and send data to the backend via HTTP POST.  

#### **Databases and Storage**
- **PostgreSQL:** Users/Roles (port 5432) and Plants/Devices (port 5433).  
- **InfluxDB:** Time-series storage of agricultural data (port 8086).  
- **MinIO:** Distributed storage for profiles, plant images, and backups.  

### 📂 Frontend and Backend Services


| Component / Service                        | Port(s) | Responsibilities                                                                                                    | Boundaries                                                                                   | Interfaces                                                                 |
|--------------------------------------------|---------|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|
| **Frontend**             | 3000    | Provides main user interface, real-time visualization of sensor data, dashboards, and plant/device management.     | SPA executed in the browser, no critical business logic, depends entirely on backend APIs.    | React 19 + TypeScript, Vite, Tailwind; communicates via REST/GraphQL APIs. |
| **be-autentication-and-roles**         | 8001    | User authentication (login/logout), role-based access control (RBAC), JWT token management, user lifecycle.        | Independent microservice with dedicated database; does not access sensor/plant data directly. | FastAPI (Python 3.11), PostgreSQL (5432), MinIO Auth (9002–9003). REST APIs for auth and roles. |
| **be-user-plant-management**        | 8003    | Manages plants, user-device associations, microcontroller lifecycle, enabling/disabling devices.                    | Specialized in domain entities (plants, devices); relies on auth service for user validation. | FastAPI + SQLAlchemy, PostgreSQL (5433), MinIO User Plant (9004–9005). REST APIs for CRUD on devices/plants. |
| **be-analytics**                      | 8000    | Processes and analyzes sensor data, computes agricultural metrics (GDD, VPD, WDI, DLI, Dew Point), trend analysis. | Read-only access to sensor data; no modification of primary records; specialized in analytics.| FastAPI (Python 3.11), InfluxDB (8086). REST/GraphQL APIs for reports and trends. |
| **be-data-management**                | 8002    | Ingests sensor data from IoT devices, validates/normalizes, stores in InfluxDB and MinIO Data Lake.                 | Single entry point for IoT data; ensures data integrity; does not perform advanced analytics. | Go service, GraphQL API, InfluxDB (8086), MinIO Data Lake (9000–9001). HTTP endpoints for ingestion. |




### 📂 Databases and Storage

| Component / Service                   | Port(s)   | Description |
|---------------------------------------|-----------|-------------|
| **db-autentication-and-roles**        | 5432      | `auth_db` with users, roles, permissions, sessions, and tokens |
| **db-user-plant-management**          | 5433      | `rootly` with user–plant–device associations |
| **db-data-management**                | 8086      | `agricultural_data` bucket with measurements and historical metrics |
| **stg-autentication-and-roles**       | 9002–9003 | Storage for profile photos |
| **stg-data-management**               | 9000–9001 | Data files, backups, unstructured content |
| **stg-user-plant-management**         | 9004–9005 | Plant images and user resources |


---

---

## 🛠️ Prototype – Deployment Instructions

### 📋 Requirements
- [Docker](https://docs.docker.com/get-docker/)  
- [Docker Compose](https://docs.docker.com/compose/install/)  
- [Git](https://git-scm.com/downloads)  
- Command-line console  

1. **📂Clone the repository** (if not already done)
```bash
git clone https://github.com/swarch-2f-rootly/rootly-frontend.git
git clone https://github.com/swarch-2f-rootly/rootly-analytics-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-deploy.git
git clone https://github.com/swarch-2f-rootly/rootly-user-plant-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-authentication-and-roles-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-data-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-microcontroller.git
git clone https://github.com/swarch-2f-rootly/rootly-apigateway.git
```


2. **Navigate to deployment directory**
   ```bash
   cd rootly-deploy
   ```

3. **Run setup script**
   ```bash
   ./setup.sh
   ```

4. **Start all services**
   ```bash
   docker-compose up -d
   ```

5. **Check service health**
   ```bash
   docker-compose ps
   ```

### Manual Setup

If you prefer to configure each service manually:

1. **Clone the repository** (if not already done)

2. **Navigate to deployment directory**
   ```bash
   cd rootly-deployment
   ```

3. **Configure environment variables for each service**

   **For Analytics Backend:**
   ```bash
   cd ../rootly-analytics-backend
   cp .env.example .env
   # Edit .env file if needed
   cd ../rootly-deployment
   ```

   **For Data Management Backend:**
   ```bash
   cd ../rootly-data-management-backend
   cp .env.example .env
   # Edit .env file if needed
   cd ../rootly-deployment
   ```

4. **Start all services**
   ```bash
   docker-compose up -d
   ```

5. **Check service health**
   ```bash
   docker-compose ps
   ```

## 🔍 Service Endpoints

Once started, the services will be available at:

- **be-autentication-and-roles**: http://localhost:8001
  - API Documentation: http://localhost:8001/docs
  - Health Check: http://localhost:8001/health

- **be-analytics**: http://localhost:8000
  - API Documentation: http://localhost:8000/docs
  - Health Check: http://localhost:8000/health

- **be-data-management**: http://localhost:8080
  - GraphQL Playground: http://localhost:8080/

- **db-data-management**: http://localhost:8086
  - Admin UI: Access via web browser
  - Credentials: admin / admin123

- **stg-data-management**: http://localhost:9000
  - Console: http://localhost:9001
  - Credentials: admin / admin123

- **stg-autentication-and-roles**: http://localhost:9002
  - Console: http://localhost:9003
  - Credentials: admin / admin123

- **stg-autentication-and-roles**: localhost:5432
  - Database: auth_db
  - User: auth_user
  - Password: admin123

## 📊 Service Dependencies

The services start in the correct order with health checks:

```
InfluxDB & MinIO (Infrastructure)
    ↳ Data Management Backend (depends on both)
    ↳ Analytics Backend (depends on InfluxDB only)
```

## 🛠️ Development Commands

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f analytics-backend
```

### Restart a service
```bash
docker-compose restart analytics-backend
```

### Rebuild and restart
```bash
docker-compose up --build -d analytics-backend
```

### Stop all services
```bash
docker-compose down
```

### Clean up (removes volumes)
```bash
docker-compose down -v
```

## 🔧 Configuration

### Environment Variables

All configuration is handled through the `.env` file. Key variables:

#### Infrastructure
- `INFLUXDB_*` - InfluxDB configuration
- `MINIO_*` - MinIO configuration

#### Applications
- `DATA_MANAGEMENT_PORT` - Data management backend port (default: 8080)
- `ANALYTICS_PORT` - Analytics backend port (default: 8000)

### Networking

All services communicate through the `rootly-network` Docker network:
- Services can reach each other by container name
- External access through published ports only

### Volumes

Persistent data is stored in named volumes:
- `influxdb_data` - InfluxDB database files
- `minio_data` - MinIO object storage files

## 🧪 Testing

For integration testing, use the test-specific configuration in the data-management-backend:

```bash
cd ../rootly-data-management-backend
docker-compose -f tests/integration/docker-compose.test.yml up -d
```

## 📈 Monitoring

### Health Checks

All services include health checks that verify:
- InfluxDB: Database connectivity
- MinIO: Object storage availability
- Backends: HTTP endpoint responsiveness

### Logs

Application logs are available via:
```bash
docker-compose logs -f [service-name]
```

## 🔒 Security Notes

- Change default passwords in production
- Use strong tokens for InfluxDB
- Consider using secrets management for sensitive data
- MinIO console should be protected in production

## 🐛 Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 8000, 8080, 8086, 9000, 9001 are available
2. **Memory issues**: Ensure at least 4GB RAM available
3. **Slow startup**: First run may take several minutes due to health checks

### Debug Commands

```bash
# Check service status
docker-compose ps

# Check resource usage
docker stats

# Inspect networks
docker network ls
docker network inspect rootly-network
```

## 📚 Additional Resources

- [InfluxDB Documentation](https://docs.influxdata.com/influxdb/)
- [MinIO Documentation](https://min.io/docs/minio/linux/index.html)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
