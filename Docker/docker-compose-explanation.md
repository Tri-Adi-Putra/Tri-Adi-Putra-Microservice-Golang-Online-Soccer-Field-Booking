# Understanding `docker compose up -d --build`

This document explains what happens when you run the following command inside a folder containing a `docker-compose.yml` file:

```bash
docker compose up -d --build
```

---

## Command Breakdown

The command consists of **3 parts**:

| Part | Description |
|---|---|
| `docker compose up` | Start all services defined in `docker-compose.yml` |
| `-d` | Run containers in the background (detached mode) |
| `--build` | Rebuild images before starting containers |

---

### `docker compose up`

This is the **main command** that reads your `docker-compose.yml` and starts all the defined services. In this project, it will spin up:

- 🐘 **Zookeeper** — Acts as the coordinator/manager for Kafka
- 📨 **Kafka** — The core message broker
- 🖥️ **Kafka UI** — A web-based dashboard to monitor Kafka (accessible on port `8070`)
- 🗄️ **PostgreSQL** — The relational database (accessible on port `5432`)
- 🛠️ **pgAdmin** — A web-based UI for managing PostgreSQL (accessible on port `5050`)

---

### `-d` — Detached Mode

By default, `docker compose up` **locks your terminal** and streams all container logs in real time. The `-d` flag tells Docker to run everything **in the background**, so your terminal remains free to use.

```bash
# Without -d → terminal is blocked, logs stream live
docker compose up

# With -d → containers run in background, terminal stays free
docker compose up -d
```

---

### `--build` — Rebuild Images

This flag forces Docker to **rebuild the images** before starting the containers. It is especially useful when:

- You have modified a `Dockerfile`
- You changed build arguments or dependencies

> **Note:** In this project, all services use pre-built public images (e.g., `confluentinc/cp-kafka:7.4.6`, `postgres:latest`), so `--build` won't have a significant effect here. However, it's a good habit to include it — especially if the project evolves to include custom-built services.

---

## What Happens Step by Step

```
1. Docker reads the docker-compose.yml file
2. Pulls any images not yet available locally
3. Creates the defined networks and volumes
4. Starts Zookeeper first (because Kafka has depends_on: zookeeper)
5. Starts Kafka after Zookeeper is ready
6. Starts Kafka UI, PostgreSQL, and pgAdmin
7. All containers run in the background
```

---

## Verifying Everything is Running

After the command completes, use these to confirm all services are up:

```bash
# Check the status of all containers
docker compose ps

# Stream live logs from all services
docker compose logs -f

# Stream logs from a specific service
docker compose logs -f kafka
```

---

## Accessing the Services

Once all containers are running, you can access the web interfaces at:

| Service | URL | Credentials |
|---|---|---|
| Kafka UI | http://localhost:8070 | — |
| pgAdmin | http://localhost:5050 | `admin@pgadmin.com` / `admin` |
| PostgreSQL | `localhost:5432` | `root` / `barca1899` |

---

## Stopping the Services

```bash
# Stop all running containers (keeps volumes and networks)
docker compose down

# Stop and remove volumes (clears all stored data)
docker compose down -v
```

---

> 💡 **Tip:** Always run `docker compose ps` after startup to make sure all containers have a `running` status and none have exited unexpectedly.
