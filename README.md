<<<<<<< HEAD
# Gateway

Nginx reverse proxy + Docker Compose orchestrator for all backend services on a single EC2 instance.

## Services

| Service | Source | Internal Port | Public Path |
|---------|--------|---------------|-------------|
| scheduler backend | `../scheduler` | 8080 | `/api/*` (prefix stripped) |
| smart-orchestrator frontend | `../smart-orchestrator` | 3000 | `/` |
| mood-detector | `../mood-detector` | 8080 | `/api/mood/*` |
| mood-drink-api | `../mood-drink-api` | 3001 | `/api/recommend` |
| nginx gateway | — | 80 | Entry point |

## EC2 Setup (one-time)

### 1. Launch instance

- AMI: Amazon Linux 2023
- Type: `t3.micro` (free tier)
- Storage: 20GB gp3
- Security group: ports 22, 80

### 2. Attach Elastic IP

Allocate an Elastic IP and associate it with the instance (free while attached).

### 3. Install Docker

```bash
sudo yum update -y
sudo yum install docker git -y
sudo systemctl enable docker && sudo systemctl start docker
sudo usermod -aG docker ec2-user

# Log out and back in for group to take effect

# Install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 4. Clone repos

```bash
mkdir -p ~/deploy && cd ~/deploy
git clone git@github.com:dagahimanshu/gateway.git
git clone git@github.com:dagahimanshu/scheduler.git
git clone git@github.com:dagahimanshu/smart-orchestrator.git
git clone git@github.com:dagahimanshu/mood-detector.git
git clone git@github.com:dagahimanshu/mood-drink-api.git
```

### 5. Configure environment

```bash
cd ~/deploy/gateway
cp .env.example .env
# Edit .env with your values (HuggingFace token, OAuth keys, etc.)
```

### 6. Start everything

```bash
cd ~/deploy/gateway
docker-compose up -d --build
```

### 7. Verify

```bash
curl http://localhost/health
curl http://localhost/api/mood/health
```

## Vercel Configuration (Distillate)

In Vercel project settings for Distillate, add these environment variables:

```
MOOD_DETECTOR_URL=http://<ELASTIC_IP>/api/mood
MOOD_DRINK_API_URL=http://<ELASTIC_IP>
```

Then update the Distillate API routes:
- `/api/analyze` calls `${MOOD_DETECTOR_URL}/analyze` (resolves to `http://<IP>/api/mood/analyze`)
- `/api/suggest-drink` calls `${MOOD_DRINK_API_URL}/api/recommend`

## Deploying Updates

Each service has a GitHub Actions workflow that SSHs into EC2 and rebuilds:

```bash
# From EC2, manually:
cd ~/deploy/gateway
git pull
docker-compose up -d --build mood-detector  # rebuild one service
docker-compose up -d --build                # rebuild all
docker-compose logs -f mood-detector        # check logs
```

## Resource Usage

| Service | Memory |
|---------|--------|
| mood-detector (JVM) | ~512MB |
| mood-drink-api (Node) | ~128MB |
| scheduler (JVM) | ~350MB |
| smart-orchestrator (Node) | ~128MB |
| nginx | ~5MB |
| **Total** | ~1.1GB |

Note: t3.micro has 1GB RAM. To fit all services, mood-detector and mood-drink-api have memory limits. If the scheduler is not needed simultaneously, this works. Otherwise consider t3.small (2GB, ~$8/month).
=======

>>>>>>> origin/main
