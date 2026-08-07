---
title : "Install Docker, run the application, and validate it"
date : 2026-08-06
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Step 1: Install required tools

On the Ubuntu EC2 instance:

```bash
sudo apt-get update
sudo apt-get install -y docker.io git awscli curl
sudo apt-get install -y docker-compose-v2 || sudo apt-get install -y docker-compose-plugin
sudo systemctl enable --now docker
sudo docker version
sudo docker compose version
```

To run Docker without `sudo`, add the Ubuntu user to the `docker` group and reconnect. Never make the Docker socket world-accessible.

#### Step 2: Clone the source

```bash
sudo mkdir -p /opt/fcaj
sudo chown ubuntu:ubuntu /opt/fcaj
cd /opt/fcaj
git clone https://github.com/ngocchau04/fcaj-rag-chat.git app
cd app
git rev-parse --short HEAD
```

Record the commit hash so the deployment is reproducible. Review changes before any later `git pull` and do not overwrite local configuration accidentally.

#### Step 3: Create the environment file

Copy the example and edit it on the server:

```bash
cp .env.example .env
chmod 600 .env
nano .env
```

Add or update:

```dotenv
GOOGLE_API_KEY=your-key-value
APP_PORT=80
```

Do not print the key with `echo`; it may enter shell history or screenshots. Verify that Git ignores `.env`:

```bash
git status --short
git check-ignore .env
```

#### Step 4: Build the Docker image

The team Compose file uses the `lite` target. The first build can take time while dependencies and PDF.js are downloaded:

```bash
cd /opt/fcaj/app
sudo docker compose build fcaj-rag-chat
sudo docker image ls fcaj-rag-chat
```

If the build runs out of space, inspect `df -h` and `sudo docker system df`. Remove unused cache only after confirming the cause.

#### Step 5: Start the application

```bash
sudo docker compose up -d
sudo docker compose ps
sudo docker compose logs --tail=100 fcaj-rag-chat
```

Compose maps `${APP_PORT}:7860`; with `APP_PORT=80`, users connect to port 80. The health check calls `http://localhost:7860/` inside the container.

After startup, validate locally:

```bash
curl --fail --head http://localhost:80/
sudo docker inspect --format='{{json .State.Health}}' fcaj-rag-chat-fcaj-rag-chat-1
```

The container name can differ with the Compose project name. Use `sudo docker compose ps --format json` or `sudo docker ps` to find it.

#### Step 6: Validate from a browser

1. Open `http://PUBLIC_IP/` from an address allowed by the Security Group.
2. Confirm that the Kotaemon interface renders correctly.
3. Create a test chat session without uploading the official evaluation documents yet.
4. Review logs for repeated stack traces or accidental API-key output.

#### Common issues

| Symptom | Check | Resolution |
|---|---|---|
| Browser timeout | Security Group, public IPv4, published port | Allow TCP 80 only from the tester IP and verify `0.0.0.0:80` is published |
| Container is `unhealthy` | Compose logs and health-check start period | Wait for initialization and inspect dependency or Gemini configuration errors |
| `GOOGLE_API_KEY` is missing | Variable name and `.env` permissions | Correct the name and run `docker compose up -d --force-recreate` |
| Gemini returns HTTP 429 | Quota and concurrency | Reduce concurrency and use bounded exponential backoff with jitter |
| Build runs out of disk | Root volume and Docker cache | Expand storage or remove only confirmed unused cache/images |

{{% notice info %}}
After the interface works, continue to section 5.4 and place `ktem_app_data` on EBS before ingesting the official documents.
{{% /notice %}}
