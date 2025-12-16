# Soul Engine Deployment Guide

This guide covers multiple deployment options for the OpenSouls Soul Engine.

## Architecture Overview

The Soul Engine consists of three main services:

1. **soul-engine-cloud** (Port 4000): Bun-based backend server with Hono framework
2. **soul-engine-ui** (Port 3000): Next.js debugger/chat interface
3. **beta-docs** (Port 3001): Documentation server (optional)

The backend uses **PGLite** (embedded PostgreSQL), so no external database is required!

---

## Quick Start (Local Development)

```bash
# 1. Install dependencies
bun install

# 2. Setup environment (will prompt for OpenAI API key)
bun run setup

# 3. Start all services
bun start

# 4. In a new terminal, run an example soul
cd souls/examples/simple-samantha
bunx soul-engine dev
```

---

## Option 1: Docker Compose (Recommended for Production)

### Prerequisites
- Docker & Docker Compose installed
- OpenAI API key

### Deployment Steps

```bash
# 1. Clone the repository
git clone https://github.com/opensouls/opensouls.git
cd opensouls

# 2. Create environment file
cp .env.example .env  # Or create manually (see below)

# 3. Edit .env with your API keys
# At minimum, set OPENAI_API_KEY

# 4. Build and start services
docker-compose up -d

# 5. (Optional) Include documentation server
docker-compose --profile docs up -d
```

### Environment Variables (.env)

Create a `.env` file in the project root:

```env
# Required
OPENAI_API_KEY=sk-your-openai-api-key-here

# Optional - Additional LLM providers
ANTHROPIC_API_KEY=
GOOGLE_API_KEY=

# UI Configuration
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### Access Points

- **UI**: http://localhost:3000
- **API/WebSocket**: http://localhost:4000 (ws://localhost:4000)
- **Docs**: http://localhost:3001 (if enabled)

---

## Option 2: Manual Deployment (VPS/Bare Metal)

### Prerequisites
- Bun 1.2+ installed
- Node.js 18+ (for Next.js)
- Git

### Backend (soul-engine-cloud)

```bash
# Navigate to backend
cd packages/soul-engine-cloud

# Install dependencies
bun install

# Generate Prisma client
bun run prisma:generate

# Create environment file
cat > .env << EOF
OPENAI_API_KEY=sk-your-key-here
DEBUG_SERVER_PORT=4000
PGLITE_DATA_DIR=./data/pglite
CODE_PATH=./data
EOF

# Start server
bun run dev

# For production with process manager (PM2):
bunx pm2 start "bun run dev" --name soul-engine-cloud
```

### Frontend (soul-engine-ui)

```bash
# Navigate to frontend
cd packages/soul-engine-ui

# Install dependencies
bun install

# Create environment file
cat > .env.local << EOF
NEXT_PUBLIC_HOCUS_POCUS_HOST=ws://your-backend-domain:4000
NEXT_PUBLIC_SITE_URL=https://your-frontend-domain.com
EOF

# Build for production
bun run build

# Start production server
bun run start

# For PM2:
bunx pm2 start "bun run start" --name soul-engine-ui
```

---

## Option 3: Vercel + Railway Deployment (Recommended for Production)

This is the recommended cloud deployment: **Vercel** for the UI, **Railway** for the backend.

### Part A: Deploy Backend to Railway

Railway supports persistent servers with WebSocket connections.

#### Step 1: Create Railway Account
1. Go to [railway.app](https://railway.app)
2. Sign up with GitHub

#### Step 2: Deploy via CLI
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Navigate to backend
cd packages/soul-engine-cloud

# Initialize and deploy
railway init
railway up
```

#### Step 3: Configure Environment Variables
In Railway dashboard → Your Project → Variables:
```
OPENAI_API_KEY=sk-your-key-here
DEBUG_SERVER_PORT=4000
NODE_ENV=production
```

#### Step 4: Get Your Backend URL
After deployment, Railway provides a URL like:
`https://soul-engine-cloud-production-xxxx.up.railway.app`

**Your WebSocket URL will be:**
`wss://soul-engine-cloud-production-xxxx.up.railway.app`

---

### Part B: Deploy UI to Vercel

#### Step 1: Push to GitHub
Make sure your code is pushed to GitHub.

#### Step 2: Import to Vercel
1. Go to [vercel.com](https://vercel.com)
2. Click "Add New Project"
3. Import your GitHub repository

#### Step 3: Configure Project Settings
- **Framework Preset**: Next.js
- **Root Directory**: `packages/soul-engine-ui`
- **Build Command**: `bun run build`
- **Install Command**: `bun install`

#### Step 4: Set Environment Variables
Add these in Vercel's project settings → Environment Variables:

| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_HOCUS_POCUS_HOST` | `wss://your-railway-url.up.railway.app` |
| `NEXT_PUBLIC_SITE_URL` | `https://your-project.vercel.app` |

#### Step 5: Deploy
Click "Deploy" and wait for the build to complete.

---

### Alternative Backend Hosts

#### Render
1. Go to [render.com](https://render.com)
2. New → Web Service
3. Connect your GitHub repo
4. Set root directory to `packages/soul-engine-cloud`
5. Add environment variables
6. Deploy

#### Fly.io
```bash
# Install Fly CLI
curl -L https://fly.io/install.sh | sh

# Login and deploy
fly auth login
cd packages/soul-engine-cloud
fly launch
fly deploy
```

---

## Option 4: AWS / GCP / Azure

For cloud VMs:
1. Provision a VM (Ubuntu 22.04+ recommended)
2. Install Docker & Docker Compose
3. Follow the Docker Compose deployment steps above

---

## Environment Variables Reference

### soul-engine-cloud (.env)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | ✅ | - | OpenAI API key |
| `ANTHROPIC_API_KEY` | ❌ | - | Anthropic Claude API key |
| `GOOGLE_API_KEY` | ❌ | - | Google Gemini API key |
| `DEBUG_SERVER_PORT` | ❌ | 4000 | Server port |
| `CODE_PATH` | ❌ | ./data | Soul code storage path |
| `PGLITE_DATA_DIR` | ❌ | ./data/pglite | Database storage path |
| `ENABLE_INSTRUMENTATION` | ❌ | false | Enable OpenTelemetry |
| `ENABLE_PRISMA_QUERY_LOGS` | ❌ | false | Enable query logging |

### soul-engine-ui (.env.local)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NEXT_PUBLIC_HOCUS_POCUS_HOST` | ❌ | ws://localhost:4000 | Backend WebSocket URL |
| `NEXT_PUBLIC_SITE_URL` | ❌ | http://localhost:3000 | Public UI URL |

---

## Running Souls

After deploying the engine, run souls against it:

```bash
# From a soul directory
cd souls/examples/simple-samantha

# Development mode (connects to local engine)
bunx soul-engine dev

# Connect to custom engine URL
bunx soul-engine dev --engine ws://your-engine-url:4000
```

---

## Security Considerations

### Production Checklist

- [ ] Use HTTPS/WSS in production
- [ ] Set up a reverse proxy (nginx/Caddy) for SSL termination
- [ ] Restrict CORS origins
- [ ] Use environment-specific API keys
- [ ] Enable rate limiting
- [ ] Set up monitoring and logging
- [ ] Configure firewall rules (only expose ports 80/443)

### Reverse Proxy Example (nginx)

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # UI
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Backend API & WebSocket
    location /api {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

---

## Troubleshooting

### Common Issues

1. **"Model does not exist" error**
   - Ensure `OPENAI_API_KEY` is set correctly
   - Check if you have access to the required models

2. **WebSocket connection failed**
   - Verify `NEXT_PUBLIC_HOCUS_POCUS_HOST` points to the correct backend
   - Check firewall rules allow WebSocket connections
   - Ensure CORS is configured properly

3. **Database errors**
   - PGLite data might be corrupted; delete `data/pglite` and restart
   - Ensure the data directory is writable

4. **Docker build fails**
   - Ensure you're building from the repository root
   - Check that all workspace packages are available

### Logs

```bash
# Docker Compose logs
docker-compose logs -f soul-engine-cloud
docker-compose logs -f soul-engine-ui

# Direct execution logs
bun run dev 2>&1 | tee server.log
```

---

## Support

- **GitHub**: https://github.com/opensouls/opensouls
- **Website**: https://opensouls.org
- **Documentation**: Run `bun run dev:docs` or visit http://localhost:3001

