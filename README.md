# Rootly

## 👥 Team
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
  ROOTLY es un sistema de monitoreo agrícola diseñado bajo una arquitectura distribuida orientada a servicios, que integra dispositivos de campo y servicios en la nube.  

  En el nivel físico, sensores y microcontroladores capturan datos ambientales y del suelo —como humedad, temperatura y ubicación— y los transmiten a la plataforma central.  

  En el nivel de persistencia, la arquitectura combina bases de datos relacionales (PostgreSQL) para información transaccional (usuarios, configuraciones, perfiles) con almacenamiento NoSQL especializado (InfluxDB) para datos de series temporales de los sensores.  

  En el nivel de servicios, componentes lógicos independientes procesan y validan la información, aplicando reglas de negocio y exponiendo interfaces REST/GraphQL para consumo por otros módulos.  

  Finalmente, en el nivel de presentación, un frontend desacoplado (SOFEA) permite a los usuarios acceder a reportes de métricas relevantes, visualizar analíticas y gestionar la configuración de sus cultivos en tiempo real.  

  Esta arquitectura asegura **escalabilidad, resiliencia y eficiencia**, al permitir que cada servicio crezca de manera independiente, aísle fallos y optimice tanto datos estructurados como métricas de sensores.  

---

## 🏛️ Architectural Structures

### 📌 Architectural Styles
1. **Cliente–Servidor**  
   El navegador (cliente) solicita recursos al frontend (servidor) vía HTTP. Esto asegura separación entre presentación y lógica de negocio.  

2. **SOFEA (Service-Oriented Front-End Architecture)**  
   El frontend es una capa desacoplada que consume únicamente servicios expuestos vía REST/GraphQL, lo que facilita escalabilidad y reutilización del backend por distintos clientes.  

3. **Corrección de anti-patrón**  
   Se eliminó el acceso directo del servicio de reportes analíticos a la base NoSQL, reemplazándolo por una interfaz controlada en el servicio de recolección de datos. Esto mejora seguridad y reduce acoplamiento:contentReference[oaicite:0]{index=0}.

---

### 📌 Architectural Elements & Relations

#### **Web Browser**
- Actor externo que ejecuta el frontend (SPA en React).  
- Consume APIs REST/GraphQL con autenticación JWT.  
- Es la interfaz de usuario entre cliente y sistema.  

#### **Frontend (rootly-frontend)**
- Interfaz de usuario en React + TypeScript.  
- Muestra métricas agrícolas y dashboards en tiempo real.  
- Se comunica solo con los servicios backend (puerto 3000).  

#### **Servicios Backend**
- **Autenticación y Roles (rootly-authentication-and-roles-backend):** Maneja login, roles y tokens JWT (puerto 8001, PostgreSQL).  
- **Gestión de Usuarios y Plantas (rootly-user-plant-management-backend):** Administra usuarios, plantas y microcontroladores (puerto 8003, PostgreSQL).  
- **Analytics (rootly-analytics-backend):** Procesa datos, genera métricas y reportes analíticos (puerto 8000, InfluxDB).  
- **Gestión de Datos (rootly-data-management-backend):** Ingresa y valida datos desde IoT, almacenándolos en InfluxDB (puerto 8002, GraphQL).  

#### **Dispositivos IoT**
- Microcontroladores ESP8266 con sensores ambientales.  
- Capturan humedad, temperatura, etc., y transmiten datos al backend vía HTTP POST.  

#### **Bases de Datos y Almacenamiento**
- **PostgreSQL:** Usuarios/Roles (puerto 5432) y Plantas/Dispositivos (puerto 5433).  
- **InfluxDB:** Series temporales de datos agrícolas (puerto 8086).  
- **MinIO:** Almacenamiento distribuido de perfiles, imágenes de plantas y backups:contentReference[oaicite:1]{index=1}.  

### 📂 Bases de Datos y Almacenamiento

| Componente / Servicio                 | Puerto(s) | Descripción |
|---------------------------------------|-----------|-------------|
| **PostgreSQL – Autenticación**        | 5432      | auth_db con usuarios, roles, permisos, sesiones y tokens |
| **PostgreSQL – Gestión de Plantas**   | 5433      | rootly con asociaciones usuario–planta–dispositivo |
| **InfluxDB – Series Temporales**      | 8086      | Bucket `agricultural_data` con mediciones y métricas históricas |
| **MinIO Auth**                        | 9002–9003 | Almacenamiento de fotografías de perfil |
| **MinIO Data Lake**                   | 9000–9001 | Archivos de datos, backups, no estructurados |
| **MinIO User Plant**                  | 9004–9005 | Imágenes de plantas y recursos de usuario |

---

---

## 🛠️ Prototype – Deployment Instructions

### 📋 Requisitos
- [Docker](https://docs.docker.com/get-docker/)  
- [Docker Compose](https://docs.docker.com/compose/install/)  
- [Git](https://git-scm.com/downloads)  
- Consola de comandos  

### 📂 Clonar Repositorios
```bash
git clone https://github.com/swarch-2f-rootly/rootly-frontend.git
git clone https://github.com/swarch-2f-rootly/rootly-analytics-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-deploy.git
git clone https://github.com/swarch-2f-rootly/rootly-user-plant-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-authentication-and-roles-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-data-management-backend.git
git clone https://github.com/swarch-2f-rootly/rootly-microcontroller.git
```

▶️ Desplegar con Docker

```bash
cd rootly-deploy/
./start.sh

