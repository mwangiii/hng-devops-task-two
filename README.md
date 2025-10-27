# Blue/Green Deployment with Nginx – Stage 2 DevOps Task

## Overview

This project demonstrates a **Blue/Green Node.js service deployment behind Nginx** using pre-built container images.  

- Blue is the active service by default.  
- Green acts as a backup and receives traffic automatically if Blue fails.  
- Nginx handles failover and preserves headers (`X-App-Pool` and `X-Release-Id`).  

## Prerequisites

- Docker >= 20.10  
- Docker Compose >= 1.29  
- `.env` file configured (see `.env.example`)  

## Setup

1. **Create a working directory and download the repo contents** (or use your local folder):

```bash
mkdir bluegreen
cd bluegreen
```

2. **Copy `.env.example` to `.env`** and edit variables:

```bash
cp .env.example .env
```

Example `.env`:

```
BLUE_IMAGE=yimikaade/wonderful:devops-stage-two
GREEN_IMAGE=yimikaade/wonderful:devops-stage-two
ACTIVE_POOL=blue
RELEASE_ID_BLUE=blue-1
RELEASE_ID_GREEN=green-1
```

> The images are pulled from Docker Hub: [https://hub.docker.com/u/yimikaade](https://hub.docker.com/u/yimikaade)

3. **Start services**

```bash
docker-compose up -d
```

This will start:

- Blue on port `8081`  
- Green on port `8082`  
- Nginx on port `8080`  

---

## Testing the Deployment

### 1. Verify baseline (Blue active)

```bash
curl -i http://localhost:8080/version
```

Expected headers:

```
X-App-Pool: blue
X-Release-Id: blue-1
```

### 2. Trigger Chaos on Blue

```bash
curl -X POST http://localhost:8081/chaos/start?mode=error
```

- Blue simulates failure.  
- Nginx automatically routes traffic to Green.

```bash
curl -i http://localhost:8080/version
```

Expected headers after failover:

```
X-App-Pool: green
X-Release-Id: green-1
```

### 3. Stop Chaos

```bash
curl -X POST http://localhost:8081/chaos/stop
```

- Blue is healthy again.  
- Nginx may route traffic back to Blue automatically.

---

## Notes

- All traffic must go through Nginx (`http://localhost:8080`) — direct calls to Blue/Green ports are only for chaos testing.  
- Nginx configuration preserves headers and supports retries with automatic failover.  
- `.env` variables control image selection, release IDs, and active pool.  

---

## Files in the Repo

- `docker-compose.yml` – orchestrates Blue, Green, and Nginx  
- `.env.example` – template environment variables  
- `nginx.conf` – template Nginx configuration  
- `README.md` – instructions for setup and testing  
- (Optional) `DECISION.md` – notes on design decisions  
