# Kafka Broker — Docker Compose Setup

> **⚠️ Bitnami Docker Images No Longer Maintained**  
> The official Bitnami Kafka images have been deprecated and are no longer available on Docker Hub. This repository replaces the old Bitnami-based setup with the maintained [`bitnamilegacy/kafka`](https://hub.docker.com/r/bitnamilegacy/kafka) image, preserving the same configuration interface and workflow so you can keep moving without interruption.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2+)

---

## Getting Started

### 1. Clone and navigate

```bash
git clone https://github.com/paaradox-labs/distributed-kafka-docker-config.git
cd brokers
```

### 2. Start the broker

```bash
docker compose up -d
```

> The first run will pull the image automatically.

This launches a **single-node Kafka 4.0.0** in **KRaft mode** — no Zookeeper required. The node acts as both controller and broker, and is exposed on `localhost:9092`.

### 3. Verify it's running

```bash
docker compose ps
```

You should see the `kafka` service with status `Up`.

### 4. Create a topic

```bash
docker compose exec kafka /opt/bitnami/kafka/bin/kafka-topics.sh \
  --create \
  --topic product \
  --bootstrap-server localhost:9092
```

---

## Usage

### CLI Cheat Sheet

All common Kafka CLI commands are documented in [`kafka.md`](./kafka.md). Quick reference:

| Action | Command |
|---|---|
| Create topic | `kafka-topics.sh --create --topic <name> --bootstrap-server localhost:9092` |
| List topics | `kafka-topics.sh --list --bootstrap-server localhost:9092` |
| Describe topic | `kafka-topics.sh --describe --topic <name> --bootstrap-server localhost:9092` |
| Alter partitions | `kafka-topics.sh --alter --topic <name> --partitions <N> --bootstrap-server localhost:9092` |
| Publish message | `kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <name>` |
| Consume messages | `kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <name> --from-beginning` |

Run these inside the container:

```bash
docker compose exec kafka /opt/bitnami/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### Connecting from your application

```
bootstrap.servers=localhost:9092
```

---

## Configuration

The broker is configured entirely through environment variables in [`docker-compose.yaml`](./docker-compose.yaml). Key settings:

| Variable | Value | Description |
|---|---|---|
| `KAFKA_CFG_NODE_ID` | `0` | Unique node ID |
| `KAFKA_CFG_PROCESS_ROLES` | `controller,broker` | Combined KRaft role |
| `KAFKA_CFG_LISTENERS` | `PLAINTEXT://:9092,CONTROLLER://:9093` | Internal & controller listeners |
| `KAFKA_ADVERTISED_LISTENERS` | `PLAINTEXT://localhost:9092` | Advertised address for clients |
| `ALLOW_PLAINTEXT_LISTENER` | `yes` | Allow non-TLS connections |

> **For production**, configure `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP` with SSL and set `KAFKA_CFG_SASL_MECHANISM` appropriately. See the [Bitnami Kafka documentation](https://docs.bitnami.com/containers/apps/kafka/) for all available settings.

---

## Data Persistence

Kafka data is stored in `./kafka-data/` and mounted into the container at `/bitnami/kafka`. This directory is gitignored — each developer gets their own local data.

To reset all data:

```bash
docker compose down -v
rm -rf kafka-data
```

---

## Project Structure

```
brokers/
├── docker-compose.yaml      # Service definition & configuration
├── kafka.md                  # CLI command reference
├── kafka-data/               # Persisted Kafka data (gitignored)
└── README.md                 # This file
```

---

## Migration from Bitnami

If you were using the old `bitnami/kafka` image, the switch is straightforward:

1. Replace `image: bitnami/kafka:...` with `image: bitnamilegacy/kafka:4.0.0-debian-12-r10`
2. The environment variables (`KAFKA_CFG_*`) remain exactly the same
3. Run `docker compose up -d` — your existing data volume should carry over

No application-level changes are required.
