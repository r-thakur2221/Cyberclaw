# CyberClaw — Environment Setup

## 1. Overview

This directory contains the Docker Compose configuration and supporting files for the CyberClaw security analysis environment.

The environment uses OpenClaw as the autonomous AI agent framework and Ollama as the local language-model inference service.

## 2. System Requirements

The tested environment uses:

| Component | Configuration |
|---|---|
| Host computer | Intel-based Mac |
| Virtualization | VirtualBox |
| Guest operating system | SEED Ubuntu 20.04.1 LTS |
| Architecture | x86_64 |
| CPU | 4 virtual CPUs |
| Memory | Approximately 10 GiB RAM |
| Docker Engine | 20.10.21 |
| Docker Compose | v2.30.3 |
| OpenClaw | 2026.9.3 |
| Ollama | 0.40.14 |
| Language model | qwen2.5:0.5b |

## 3. Architecture

Docker Compose manages two services:

**OpenClaw Gateway:** Runs the autonomous agent framework, coordinates model requests, and provides access to agent tools.

**Ollama:** Hosts the local Qwen 2.5 0.5B model and exposes its inference API to OpenClaw through Docker networking.

The containers communicate through the `cyberclaw-net` bridge network.

The host-side ports are restricted to the VM's loopback interface:

- OpenClaw Gateway: `127.0.0.1:18789`
- Ollama API: `127.0.0.1:11434`

## 4. Installation and Setup

### Step 1 — Verify Docker

```bash
docker --version
docker compose version
docker info
```

Ensure the Docker daemon is running.

### Step 2 — Enter the environment directory

From the repository root:

```bash
cd environment
```

### Step 3 — Create local directories

```bash
mkdir -p config workspace
sudo chown -R 1000:1000 config workspace
chmod 700 config workspace
```

### Step 4 — Generate a Gateway token

```bash
printf 'OPENCLAW_GATEWAY_TOKEN=%s\n' \
  "$(openssl rand -hex 32)" > .env

chmod 600 .env
```

The `.env` file contains a secret and must not be committed to Git.

### Step 5 — Create the OpenClaw configuration

The OpenClaw configuration is stored in `config/openclaw.json`.

It specifies the Gateway settings and local Ollama model provider.

The default model is:

```text
ollama/qwen2.5:0.5b
```

### Step 6 — Start the services

```bash
docker compose config --quiet
docker compose up -d
```

### Step 7 — Download the model

```bash
docker exec cyberclaw-ollama \
  ollama pull qwen2.5:0.5b
```

### Step 8 — Verify the model

```bash
docker exec cyberclaw-ollama \
  ollama list
```

### Step 9 — Check the containers

```bash
docker compose ps
```

Both containers should be running.

## 5. Functional Verification

Check Ollama:

```bash
curl http://127.0.0.1:11434/api/version
```

Check OpenClaw's model configuration:

```bash
docker exec cyberclaw-gateway \
  node openclaw.mjs models status
```

Test an agent response:

```bash
docker exec cyberclaw-gateway \
  node openclaw.mjs agent \
  --agent main \
  --session-id cyberclaw-verification \
  --message "Say hello from CyberClaw" \
  --json
```

A successful result should report `status: ok` and contain a generated response.

## 6. Troubleshooting

### Context overflow

The initial model test failed because Ollama allocated a context window of 4,096 tokens while OpenClaw generated a prompt exceeding 10,000 tokens.

The working environment uses:

```yaml
OLLAMA_CONTEXT_LENGTH: "16384"
```

After modifying this setting, recreate the Ollama container:

```bash
docker compose up -d --force-recreate ollama
```

### Gateway already running

Running an agent with `--local` while the Gateway is active can cause a state-directory conflict.

Use the existing Gateway instead by omitting `--local`.

### Inspect logs

```bash
docker compose logs --tail=50 openclaw
docker compose logs --tail=50 ollama
```

## 7. Security Considerations

The baseline deployment uses token authentication, a non-root OpenClaw container user, dropped Linux capabilities, and `no-new-privileges`.

However, the baseline OpenClaw security audit identified:

- Agent sandboxing disabled.
- Small model with potentially unsafe web tools available.
- Configuration file permissions set to `664`.
- Gateway authentication rate limiting not configured.

These findings are documented in `../analysis/README.md`.

The baseline environment is intended for controlled security analysis and should not be exposed to untrusted users or networks.

## 8. Stopping the Environment

To stop the containers without deleting persistent model data:

```bash
docker compose down
```

To restart:

```bash
docker compose up -d
```

Do not use `docker compose down -v` unless you intentionally want to delete persistent volumes, including downloaded Ollama model data.

## 9. Reproducibility Notes

Docker images use fixed version tags to reduce unexpected differences between deployments.

The model is downloaded separately through Ollama.

The `.env` file and local credentials are not included in the repository and must be created on each installation.

The configuration and security observations should be reviewed before using the environment outside an isolated lab.
