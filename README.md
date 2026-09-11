# 🏠 IoT Smart Home Automation Hub

**Shivay00001/iot-smart-home-automation-hub** — a lightweight, high-performance Go service that acts as the core entry point for an IoT smart home automation hub. It exposes an HTTP health/status endpoint that reports real-time system operational state, forming the foundation for device telemetry, automation routing, and hub orchestration.

---

## 🚀 Overview

This repository contains a minimal, production-ready Go microservice built on the standard library only — **zero external dependencies**. It binds to port `8080` and responds to HTTP requests with the current system status and timestamp, making it ideal as the always-on heartbeat service of a smart home automation stack.

## ✨ Features

- **Zero-dependency Go service** — built entirely on `net/http`, `fmt`, `log`, and `time` from the standard library
- **Real-time status endpoint** — root handler returns `System Operational: <timestamp>` for health checks and uptime monitoring
- **Containerized deployment** — ships with a multi-stage-ready `Dockerfile` based on `golang:1.20-alpine` for a tiny footprint
- **Secrets hygiene** — `.gitignore` preconfigured to exclude `.env` files, credentials, certificates, and API keys — critical for IoT deployments handling device tokens
- **Structured startup logging** — logs service initialization and fatal errors via the standard `log` package

## 🏗️ Architecture — How It Works

The application is a single-binary HTTP service defined in `main.go`:

```
┌─────────────────────────────────────────────┐
│              Client / IoT Device            │
│              (HTTP GET /)                   │
└────────────────────┬────────────────────────┘
                     ▼
┌─────────────────────────────────────────────┐
│   Go HTTP Server  (net/http, port :8080)    │
│                                             │
│   http.HandleFunc("/", handler)             │
│   └── fmt.Fprintf(w, "System Operational:   │
│        %s", time.Now())                     │
│                                             │
│   log.Fatal(http.ListenAndServe(":8080"))   │
└─────────────────────────────────────────────┘
```

**Execution flow:**

1. **`main()`** registers a handler function on the root path `/` using the default `http.ServeMux`.
2. The handler writes a plain-text response containing the literal string `System Operational:` followed by the server's current timestamp (`time.Now()`), enabling consumers to verify both liveness and clock sync.
3. The server starts on **port 8080** via `http.ListenAndServe`. Startup is announced through `log.Println`, and any bind/runtime failure terminates the process via `log.Fatal` with a descriptive error.
4. The module (`go.mod`) is declared as `github.com/Shivay00001/iot-smart-home-automation-hub` targeting **Go 1.20**, with no third-party requirements.

This design makes the service a reliable **health-check / heartbeat backbone** onto which device registries, MQTT bridges, and automation rule engines can be layered.

## 📁 Repository Structure

```
.
├── main.go          # HTTP service entry point (port 8080)
├── go.mod           # Go module definition (Go 1.20)
├── Dockerfile       # Alpine-based container build
├── .gitignore       # Secrets, build artifacts, and OS file exclusions
├── LICENSE          # VisionQuantech Custom Commercial License
└── README.md        # This file
```

## 🐳 Docker Deployment (Recommended)

The service can run on any laptop or server with Docker installed — no Go toolchain required.

### Build and run with the Dockerfile

```bash
# 1. Build the image
docker build -t shivay00001/iot-smart-home-automation-hub .

# 2. Run the container, mapping host port 8080 → container port 8080
docker run -d --name iot-hub -p 8080:8080 shivay00001/iot-smart-home-automation-hub
```

### Using Docker Compose (optional)

If you prefer `docker-compose`, create a `docker-compose.yml`:

```yaml
version: "3.8"
services:
  iot-hub:
    build: .
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Then launch:

```bash
docker-compose up -d --build
```

### Verify it's running

```bash
curl http://localhost:8080/
# → System Operational: 2026-01-01 12:00:00.000000000 +0000 UTC ...
```

## 🛠️ Local Execution (Without Docker)

Requires **Go 1.20+**:

```bash
# Clone the repository
git clone https://github.com/Shivay00001/iot-smart-home-automation-hub.git
cd iot-smart-home-automation-hub

# Run directly
go run main.go

# Or compile a binary
go build -o iot-hub
./iot-hub
```

The service logs `Starting high-performance service on :8080` and begins accepting requests immediately.

## 🔐 Security Notes

- Never commit device credentials, API keys, or `.env` files — the `.gitignore` already excludes common secret patterns (`*.key`, `credentials.json`, `service-account.json`, etc.).
- When exposing port `8080` beyond localhost, place the service behind a reverse proxy (e.g., Nginx, Traefik) with TLS termination.

## 📄 License

This project is distributed under the **VisionQuantech Custom Commercial License**:

- **Personal / educational / non-earning use** — free.
- **Individual revenue-generating use** — subject to a 15–30% gross revenue share.
- **Business / enterprise use** — requires a separate commercial license: **visionquantech@proton.me**

See [LICENSE](LICENSE) for full terms. The software is provided **"AS IS"**, without warranty of any kind.

---

© 2026 Shivay00001 / VisionQuantech