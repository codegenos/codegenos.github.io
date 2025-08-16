---
author: "CodeGenos"
title: "How to Self-Host n8n on Raspberry Pi 4 (ARM64) with Docker Compose"
date: 2025-08-16T00:00:00+03:00
lastmod: 2025-08-16T00:00:00+03:00
slug: "self-host-n8n-raspberry-pi-4-docker-compose"
description: "Step‑by‑step guide to self‑host n8n locally on Raspberry Pi 4 (ARM64) with Docker Compose, including environment, volumes, webhooks, and reverse proxy tips."
tags: ["n8n", "Docker", "Docker Compose", "Raspberry Pi"]
categories: ["n8n", "Docker"]
ShowToc: true
ShowBreadCrumbs: true
draft: false
---

This guide shows a clean, reliable way to run n8n on a Raspberry Pi 4 using Docker Compose.

Note: I want to use this setup primarily for AI workflows (transcription, summarization, RAG, content drafting).

## TL;DR / Quick start
1) Check architecture (arm64 preferred):
   ```bash
   uname -m
   # aarch64 -> 64-bit, armv7l -> 32-bit
   ```
2) Create `docker-compose.yml` using the snippet below.
3) Set `WEBHOOK_URL`, `GENERIC_TIMEZONE`, and `TZ` for your locale.
4) Start: `docker compose up -d`
5) Open: `http://raspi.local:5678` (or your Pi’s IP)
6) Logs: `docker compose logs -f n8n` — Stop: `docker compose stop`

## What is n8n?

n8n is an open‑source, self‑hostable workflow automation tool. You build automations by connecting nodes in a visual editor: triggers (webhooks, schedules/cron, IMAP, polling) start a workflow; you transform data, branch logic, handle errors, and call services/APIs or databases. It’s ideal for glue code and recurring tasks without writing and deploying full apps.

### What it’s used for
- Integrations between services and HTTP APIs
- ETL/data sync across files, databases, and SaaS tools
- Notifications/alerts and incident workflows
- Back‑office automations and internal tools
- AI automation and LLM workflows (transcription/summarization, data extraction, RAG, chatbots)

### Example scenarios on a Raspberry Pi
- Home status alerts: Ping devices on your LAN; if one is down, send a Telegram/email alert and log to SQLite in the `n8n_data` volume.
- Scheduled backups: Nightly tar selected directories, copy to NAS/S3 via SSH/rclone, and notify on success/failure.
- RSS to social: Poll an RSS feed, find new items, post to Mastodon/Telegram with throttling.
- Email to tasks: Watch a label via IMAP; create tasks in Notion/Trello/GitHub Issues; include attachments saved under `/files`.

### Example AI automation scenarios
- Email triage: For new messages, generate summaries and priority labels; draft reply suggestions with an LLM; route high‑priority items to a chat channel.
- RAG over local docs: When PDFs land in `/files/docs`, create embeddings (OpenAI or small local models); store/search with a lightweight vector DB (e.g., Qdrant container); expose a webhook to answer questions.
- Content drafting: For each RSS item, have an LLM generate a concise post with hashtags, then publish to Mastodon/Telegram; keep a dedupe key to avoid reposts.
- Safety/moderation: Run inputs through a moderation endpoint before posting or storing; auto‑redact PII using patterns plus an LLM pass.

## Prerequisites
- Raspberry Pi 4
- Docker and Docker Compose

Check architecture (arm64 is preferred):
```bash
uname -m
# aarch64 -> 64-bit, armv7l -> 32-bit
```

## Docker Compose file

Create a `docker-compose.yml`:

```yaml
version: "3.8"
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - N8N_RUNNERS_ENABLED=true
      - N8N_SECURE_COOKIE=false
      - NODE_ENV=production
      - WEBHOOK_URL=http://raspi.local:5678
      - GENERIC_TIMEZONE=Europe/Istanbul
      - TZ=Europe/Istanbul
    volumes:
      - n8n_data:/home/node/.n8n
      - ./local-files:/files

volumes:
  n8n_data:
```

### Environment variables
- `N8N_PORT`: The port n8n listens on inside the container (mapped to 5678 on the host).
- `N8N_PROTOCOL`: http for local/LAN, https when behind a reverse proxy (e.g., Traefik) terminating TLS.
- `WEBHOOK_URL`: The externally reachable base URL n8n uses to generate webhooks and OAuth callbacks. Include scheme and port, e.g., `http://raspi.local:5678` or `https://n8n.example.com`.
- `N8N_SECURE_COOKIE`: Set true when using HTTPS (prevents cookies over insecure transport); keep false for plain HTTP.
- `N8N_RUNNERS_ENABLED`: Enables n8n Runners to execute tasks in isolated processes/containers for performance and safety. If not using Runners, set to `false`.
- `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS`: Ensures secure permissions on settings files (helps protect secrets).
- `NODE_ENV`: Use `production` for better performance and fewer dev checks.
- `GENERIC_TIMEZONE` and `TZ`: Time zone for workflows and the container. Keep both aligned (e.g., `Europe/Istanbul`).

### Volumes
- `n8n_data`: A volume mapped to `/home/node/.n8n`. n8n saves its SQLite database file and encryption key here.
- `./local-files`: A local directory shared between the n8n instance and host. In n8n, use the `/files` path to read from and write to this directory.

## Networking
- This example exposes n8n over HTTP on your LAN and does not use a domain name.
- For HTTPS and a domain, place n8n behind a reverse proxy like Traefik with Let’s Encrypt. Set `N8N_PROTOCOL=https` and `WEBHOOK_URL` to your HTTPS URL (e.g., `https://n8n.example.com`); let the proxy terminate TLS and route requests to the n8n container.

## Start the stack

```bash
docker compose up -d
# Visit: http://raspi.local:5678 (or your Pi’s IP)
```

To see the n8n logs:

```bash
docker compose logs -f n8n
```

To stop the containers:

```bash
docker compose stop
```

## Tips
- Replace raspi.local with your Pi’s hostname or IP.
- Updates: `docker compose pull && docker compose up -d`
- Backups: stop the stack and copy the `n8n_data` volume (contains the SQLite DB and encryption key).
- Local files backup: copy the host folder `./local-files` (mapped to `/files` in n8n) to your backup location. You can copy it live, but stopping workflows avoids partial writes.
- HTTPS/domain: run n8n behind Traefik as a reverse proxy with Let’s Encrypt; set `N8N_PROTOCOL=https` and `WEBHOOK_URL` to your domain (e.g., `https://n8n.example.com`). The proxy handles TLS and forwards 443 traffic to n8n.

## Troubleshooting
- Wrong `WEBHOOK_URL` or port leads to failing webhooks/OAuth. Use the exact external URL clients will reach.
- Permissions on `./local-files`: if nodes can’t read/write, ensure the directory exists and is writable by the container user.
- Time zone drift: keep `GENERIC_TIMEZONE` and `TZ` consistent to avoid cron/schedule surprises.
- 32‑bit OS: the image may pull `arm/v7`. Prefer 64‑bit OS for better performance and compatibility.

## FAQ
- What port does n8n use by default?
  - 5678 inside the container. This guide maps host `5678` to container `5678`.
- Do I need HTTPS on my LAN?
  - Not strictly, but use HTTPS for remote access or when handling credentials. Put n8n behind a reverse proxy that terminates TLS and set `N8N_SECURE_COOKIE=true`.
- Where are my workflows and credentials stored?
  - In the `n8n_data` volume under `/home/node/.n8n` (SQLite DB and encryption key).
- How do I update n8n safely?
  - `docker compose pull && docker compose up -d`. Back up `n8n_data` first for rollback safety.

## Helpful resources

- Docs home: https://docs.n8n.io/
- Docker deployment guide: https://docs.n8n.io/hosting/installation/docker/
- Community forum: https://community.n8n.io/
- GitHub (releases, issues): https://github.com/n8n-io/n8n