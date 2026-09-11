# 🌐 EventSphere — Local Staging & Development Environment

[![Java](https://img.shields.io/badge/Java-21_LTS-ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.4.2-6DB33F.svg?style=for-the-badge&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Docker](https://img.shields.io/badge/Docker-MySQL_Container-2496ED.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0%2B-4479A1.svg?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Frontend](https://img.shields.io/badge/Frontend-Vanilla_JS_SPA-F7DF1E.svg?style=for-the-badge&logo=javascript&logoColor=black)](eventsphere_frontend/)
[![License](https://img.shields.io/badge/License-Academic_Evaluation-blue.svg?style=for-the-badge)](#)

> **EventSphere** is an enterprise-grade, full-stack event management and ticket reservation platform. This repository is configured for an **isolated local development environment** utilizing a local **Docker MySQL** container for safe data testing, while maintaining live integration with **PayHere IPG (Sandbox)**, **Brevo SMTP (Transactional Email)**, and **Google Gemini 1.5 Flash (AI Recommendations)**.

---

## 📁 Monorepo Structure

```text
eventsphere-local-dev/
├── eventsphere_backend/               # Spring Boot 3 REST API Application
│   ├── src/main/java/                 # Controllers, Services, Entities, Repositories
│   ├── src/main/resources/            # Application configs, email templates, HTTP client
│   │   ├── http/                      # 01-auth.http to 08-testing.http IntelliJ suites
│   │   └── application.properties     # Spring properties (reads from .env)
│   ├── .env                           # Local secrets & DB coordinates (git-ignored)
│   ├── .env.example                   # Environment configuration template
│   ├── dev.bat / dev.sh               # One-click backend startup scripts
│   ├── Dockerfile                     # Multi-stage production container build
│   └── pom.xml                        # Maven dependencies & build definitions
│
├── eventsphere_frontend/              # Vanilla JavaScript & CSS Single Page App
│   ├── index.html                     # Landing page & event discovery
│   ├── pages/                         # Auth, Booking, Admin & Organizer portals
│   ├── js/                            # Modular API fetch wrappers, controllers, auth
│   │   ├── env.js                     # Auto-generated runtime environment config
│   │   └── api/config.js              # Centralized API fetch wrapper & JWT interceptor
│   ├── css/                           # Custom responsive UI styling & design tokens
│   ├── scripts/generate_env.py        # Python build script syncing .env to js/env.js
│   ├── dev.bat / dev.sh               # One-click frontend HTTP server scripts
│   ├── .env                           # Frontend public runtime config (git-ignored)
│   └── .env.example                   # Frontend environment template
│
├── PayHere Account Details.txt         # PayHere sandbox test cards & credentials
├── .gitignore                         # Root git ignore rules
└── README.md                          # Repository documentation & Cloud migration guide
```

---

## ⚡ Quick Start: Running Locally

### 1. Prerequisites
- **Java 21 LTS** (`java -version`)
- **Docker Desktop** (with MySQL container running)
- **Python 3** (or VS Code *Live Server* extension)

---

### 2. Set Up Local MySQL (Docker)

Ensure your Docker MySQL container is started:
```powershell
docker start mysql
```

*(If you are setting up MySQL in Docker for the first time)*:
```powershell
docker run --name mysql -e MYSQL_ROOT_PASSWORD=mysql -p 3306:3306 -d mysql:latest
```

Create the isolated database inside the container:
```powershell
docker exec -i mysql mysql -u root -pmysql -e "CREATE DATABASE IF NOT EXISTS eventsphere_viva_db;"
```

> [!TIP]
> Spring Boot Hibernate auto-generates all 16 database tables on initial launch via `spring.jpa.hibernate.ddl-auto=update`, and pre-seeds the administrator account automatically!

---

### 3. Start the Backend API

1. Navigate to the backend folder:
   ```powershell
   cd eventsphere_backend
   ```
2. Copy `.env.example` to `.env` (if not already present):
   ```powershell
   copy .env.example .env
   ```
   *Verify your `.env` contains*:
   ```properties
   PORT=7080
   DB_URL=jdbc:mysql://localhost:3306/eventsphere_viva_db?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true
   DB_USERNAME=root
   DB_PASSWORD=mysql
   ```
3. Run the backend dev server:
   ```cmd
   dev.bat
   ```
   *(Or on Linux/macOS: `./dev.sh`, or via Maven: `./mvnw spring-boot:run`)*
4. Backend will start on **`http://localhost:7080`**.

---

### 4. Start the Frontend Web App

1. Navigate to the frontend folder:
   ```powershell
   cd eventsphere_frontend
   ```
2. Copy `.env.example` to `.env` (if not already present):
   ```powershell
   copy .env.example .env
   ```
   *Verify `.env` contains*:
   ```properties
   API_BASE=http://localhost:7080/api/v1
   PORT=8000
   ```
3. Run the dev server script:
   ```cmd
   dev.bat
   ```
   *(Or right-click `index.html` in VS Code and click **Open with Live Server** on `http://localhost:5500`)*.

---

### 5. Pre-Seeded Default Accounts

| Role | Email | Password | Access / Dashboard |
| :--- | :--- | :--- | :--- |
| **Admin** | `eventsphere.tickets@gmail.com` | `ChangeMe123` | Full admin panel, approvals, user management |
| **Organizer** | `eventsphere.tickets@gmail.com` | `ChangeMe123` | Event creation, ticket tier setup, check-in scanner |

---

## 🔄 Cloud vs. Local Toggle Guide: Where to Make Changes

If you or examiners wish to switch this codebase between **Local Development** and **Cloud Production Deployment**, here are the exact files and lines to update:

```
                  ┌────────────────────────────────────────┐
                  │          DEPLOYMENT SELECTION          │
                  └──────────────────┬─────────────────────┘
                                     │
                 ┌───────────────────┴───────────────────┐
                 ▼                                       ▼
       [ LOCAL ENVIRONMENT ]                   [ CLOUD ENVIRONMENT ]
   • DB: Docker MySQL (localhost:3306)     • DB: Aiven Cloud Managed MySQL
   • API: localhost:7080                   • API: Render Cloud Service
   • Web: localhost:5500 / 8000            • Web: Vercel Cloud Hosting
```

### File 1: Backend Database & Server Config (`eventsphere_backend/.env`)

| Property | Local Environment (Docker) | Cloud Environment (Aiven / Render) |
| :--- | :--- | :--- |
| `DB_URL` | `jdbc:mysql://localhost:3306/eventsphere_viva_db?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true` | `jdbc:mysql://<aiven-host>:15664/defaultdb?createDatabaseIfNotExist=true&sslMode=REQUIRED&enabledTLSProtocols=TLSv1.2,TLSv1.3` |
| `DB_USERNAME` | `root` | `avnadmin` |
| `DB_PASSWORD` | `mysql` | `<your-aiven-password>` |
| `PAYHERE_RETURN_URL` | `http://localhost:5500/pages/my-bookings.html` | `https://eventsphere-webapp.vercel.app/pages/my-bookings.html` |
| `PAYHERE_CANCEL_URL` | `http://localhost:5500/pages/my-bookings.html` | `https://eventsphere-webapp.vercel.app/pages/my-bookings.html` |
| `PAYHERE_NOTIFY_URL` | `https://<your-ngrok-id>.ngrok-free.app/api/v1/payments/notify` | `https://its-1114-eventsphere-booking-platform.onrender.com/api/v1/payments/notify` |

---

### File 2: Frontend Backend Target (`eventsphere_frontend/.env`)

| Property | Local Environment | Cloud Environment (Render) |
| :--- | :--- | :--- |
| `API_BASE` | `http://localhost:7080/api/v1` | `https://its-1114-eventsphere-booking-platform.onrender.com/api/v1` |

> [!IMPORTANT]
> Whenever you change `eventsphere_frontend/.env`, re-run the generator to update runtime configuration:
> ```powershell
> python scripts/generate_env.py
> ```
> *(Or manually update line 7 of `eventsphere_frontend/js/env.js`)*.

---

### File 3: Fallback API URL (`eventsphere_frontend/js/api/config.js`)
If `js/env.js` is not loaded in specific static contexts, line 16 of `config.js` controls the fallback:
```javascript
// Local Development:
API_BASE: "http://localhost:7080/api/v1"

// Cloud Production:
API_BASE: "https://its-1114-eventsphere-booking-platform.onrender.com/api/v1"
```

---

### File 4: IntelliJ HTTP Client Environments (`eventsphere_backend/src/main/resources/http/http-client.env.json`)
Allows you to switch testing environments in IntelliJ HTTP requests with 1 click:
```json
{
  "aiven-cloud": {
    "host": "https://its-1114-eventsphere-booking-platform.onrender.com",
    "admin_email": "eventsphere.tickets@gmail.com",
    "admin_password": "ChangeMe123"
  },
  "local-dev": {
    "host": "http://127.0.0.1:7080",
    "admin_email": "eventsphere.tickets@gmail.com",
    "admin_password": "ChangeMe123"
  }
}
```

---

## 💳 PayHere Payment Gateway in Local Mode

Because PayHere's sandbox servers must invoke your webhook on localhost when an attendee pays:

### Option A: Tunnel Webhooks with ngrok or localtunnel
1. Run a tunnel:
   ```bash
   ngrok http 7080
   # or
   npx localtunnel --port 7080
   ```
2. Copy the generated HTTPS address (e.g. `https://xxxx.ngrok-free.app`) and paste into `eventsphere_backend/.env`:
   ```properties
   PAYHERE_NOTIFY_URL=https://xxxx.ngrok-free.app/api/v1/payments/notify
   ```

### Option B: The Instant Offline Simulation (Fail-Safe)
If internet access is restricted or PayHere sandbox is undergoing maintenance:
1. Open [`07-payment.http`](eventsphere_backend/src/main/resources/http/07-payment.http) in IntelliJ IDEA.
2. Select the `local-dev` environment.
3. Run request **#2**:
   ```http
   POST {{host}}/api/v1/payments/notify
   Content-Type: application/x-www-form-urlencoded

   merchant_id={{merchant_id}}&order_id={{order_id}}&payment_id=320001234&payhere_amount={{amount}}&payhere_currency={{currency}}&status_code=2&md5sig=PLACEHOLDER_MD5_HASH
   ```
4. Spring Boot accepts `PLACEHOLDER_MD5_HASH` in development mode, immediately confirms the order, issues the cryptographic QR ticket, and emails the PDF receipt.

---

## 🧪 Automated Verification & Testing

To run the complete test suite against your local Docker database:
```powershell
cd eventsphere_backend
.\mvnw.cmd clean test
```

Expected output:
```text
[INFO] Tests run: 6, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

---

## 👥 Author & Coursework Details
- **Module:** ITS 1114 – Advanced API Development
- **Developer:** Lasandi Salwathura
- **Repository:** [https://github.com/lasandisal/eventsphere-local-dev](https://github.com/lasandisal/eventsphere-local-dev)
