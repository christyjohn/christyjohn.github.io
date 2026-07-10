---
title: "Clearing the persisted data in Keycloak Postgres storage"
date: 2026-04-14
draft: false
comments: true
categories: ["docker", "keycloak"]
tags: ["keycloak-reset", "docker", "keycloak"]
---

If you want to completely reset your local Keycloak setup and start from scratch, you’ll need to remove the persisted PostgreSQL volume that Docker is using to store the database data.

#### Step 1 — Stop and remove containers 

Run this in the folder containing your docker-compose.yml:

```bash
docker compose down
```

This stops and removes the running containers, but the database data will still remain because it is stored in a Docker volume.

#### Step 2 — Remove the persisted volume

Run the following command to remove the PostgreSQL volume:

```bash
docker volume rm postgres_data
```

In some cases, Docker automatically prefixes the volume name with the project or folder name. If the above command fails, first list all available volumes:

```bash
docker volume ls
```

You may see something similar to:

```yourfolder_postgres_data```

If so, remove that volume instead:

```bash
docker volume rm yourfolder_postgres_data
```

**Easier One-Line Option**
You can also remove the containers, networks, and volumes together using a single command:

```bash
docker compose down -v
```

This is usually the simplest and cleanest approach when resetting your local environment.

#### Step 3 — Start everything again

Now start the containers again:

```bash
docker compose up -d
```

Keycloak will now start as a completely fresh installation.

That means you’ll need to recreate everything manually, including:

- realms
- clients
- users
- identity providers (Facebook/Apple)
- roles
- any custom configuration

everything from scratch.

**Optional — Completely clean Docker images as well**

If you also want Docker to re-download the images from scratch, run:

```bash
docker compose down -v --rmi all
```

Then start the setup again with:

```bash
docker compose up -d
```

Docker will pull the images again before starting the containers.