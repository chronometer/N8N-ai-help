# Installation Guide

There are multiple ways to install and run n8n. Choose the option that best fits your needs.

## 🌐 n8n Cloud (Recommended for Getting Started)

### Advantages
- ✅ No installation required
- ✅ Hosted and managed by n8n
- ✅ Start immediately
- ✅ Free tier available
- ✅ Automatic updates

### Getting Started

1. Go to [n8n Cloud](https://app.n8n.cloud/)
2. Sign up for a free account
3. Verify your email
4. Create your first workflow

### Free Tier Limitations
- Limited workflow executions per month
- Shared infrastructure
- Access to built-in nodes

### When to Use
- Learning n8n
- Quick prototyping
- Small-scale automation
- No local infrastructure needed

---

## 💻 Local Installation with npm

### Advantages
- ✅ Full control
- ✅ Local development
- ✅ No internet required for workflows
- ✅ Easy debugging
- ✅ Free

### Prerequisites
- Node.js v16 or higher
- npm or yarn
- ~500MB disk space

### Installation Steps

```bash
# 1. Install n8n globally
npm install -g n8n

# 2. Start n8n
n8n start

# 3. Access the UI
# Open browser to: http://localhost:5678
```

### Verify Installation

```bash
# Check Node.js version
node --version  # Should be v16 or higher

# Check npm version
npm --version

# Start n8n
n8n start
```

### Common npm Commands

```bash
# Start n8n with specific port
n8n start --port 3000

# Start with webhook URL
n8n start --webhookUrl https://yourdomain.com/

# View all options
n8n --help
```

### When to Use
- Local development
- Testing workflows
- Learning n8n
- Integration with local services

---

## 🐳 Docker Installation

### Advantages
- ✅ Consistent environment
- ✅ Easy deployment
- ✅ Portable across systems
- ✅ Production-ready
- ✅ Works on Mac, Linux, Windows

### Prerequisites
- Docker installed on your system
- ~1GB disk space
- Basic Docker knowledge

### Quick Start

```bash
# Pull the n8n image
docker pull n8nio/n8n

# Run n8n container
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e NODE_ENV=production \
  n8nio/n8n
```

### Access n8n
Open browser to: `http://localhost:5678`

### Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_SECURE_COOKIE=false
      - NODE_ENV=production
    volumes:
      - n8n_storage:/home/node/.n8n
    networks:
      - n8n_network

volumes:
  n8n_storage:

networks:
  n8n_network:
```

Start with:
```bash
docker-compose up -d
```

### Persistent Data

```bash
# With volume for data persistence
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e NODE_ENV=production \
  -v n8n_storage:/home/node/.n8n \
  n8nio/n8n
```

### When to Use
- Production deployments
- Consistent environments
- Microservices architecture
- Cloud deployments (Kubernetes, etc.)

---

## 🖥️ Self-Hosted Deployments

### Popular Platforms

#### AWS
- EC2 instances
- App Runner
- ECS
- Lambda integration

#### Google Cloud
- Cloud Run
- Compute Engine
- App Engine

#### Azure
- App Service
- Container Instances
- Virtual Machines

#### Heroku
- One-click deployment
- Limited free tier
- Auto-scaling available

#### DigitalOcean
- Droplets
- App Platform
- Marketplace (pre-configured)

### General Self-Hosting Setup

```bash
# 1. SSH into your server
ssh user@your-server

# 2. Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 3. Install n8n globally
sudo npm install -g n8n

# 4. Create systemd service file
sudo nano /etc/systemd/system/n8n.service
```

### Systemd Service File Example

```ini
[Unit]
Description=n8n
After=network.target

[Service]
Type=simple
User=n8n
WorkingDirectory=/home/n8n
Environment="NODE_ENV=production"
ExecStart=/usr/bin/n8n start
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Start the service:
```bash
sudo systemctl enable n8n
sudo systemctl start n8n
sudo systemctl status n8n
```

### Configuration with Environment Variables

```bash
# Set environment variables
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD=password

# Or in .env file
echo "N8N_BASIC_AUTH_ACTIVE=true" > .env

# Start n8n
n8n start
```

### SSL/HTTPS Setup

```bash
# Using Let's Encrypt with nginx reverse proxy
sudo apt install certbot python3-certbot-nginx
sudo certbot certonly --standalone -d your-domain.com

# Configure nginx to proxy to n8n on port 5678
```

---

## 🔐 Environment Variables

### Essential Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `N8N_PROTOCOL` | `http` | Protocol (http or https) |
| `N8N_HOST` | `localhost` | Hostname |
| `N8N_PORT` | `5678` | Port number |
| `N8N_BASIC_AUTH_ACTIVE` | `false` | Enable basic auth |
| `N8N_BASIC_AUTH_USER` | - | Basic auth username |
| `N8N_BASIC_AUTH_PASSWORD` | - | Basic auth password |
| `NODE_ENV` | `development` | Environment (development or production) |

### Database Configuration

```bash
# PostgreSQL
export DB_TYPE=postgresdb
export DB_POSTGRESDB_HOST=localhost
export DB_POSTGRESDB_PORT=5432
export DB_POSTGRESDB_DATABASE=n8n
export DB_POSTGRESDB_USER=n8n
export DB_POSTGRESDB_PASSWORD=password
```

See [Configuration Guide](../deployment/environment-variables.md) for complete reference.

---

## ✅ Verification Checklist

After installation, verify everything is working:

- [ ] n8n is running
- [ ] UI is accessible at correct URL
- [ ] Can create a new workflow
- [ ] Can add a node
- [ ] Can execute a workflow

---

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Change port
n8n start --port 3000

# Or find and kill process using port 5678
lsof -i :5678
kill -9 <PID>
```

### Node.js Not Found

```bash
# Install Node.js (macOS with Homebrew)
brew install node

# Or visit https://nodejs.org/
```

### Database Connection Issues

```bash
# Check database is running
# Verify connection string in environment variables
# Check credentials are correct
```

### Workflow Execution Fails

- Check workflow syntax
- Verify credentials are set
- Check API keys are valid
- Look at execution logs for errors

---

## 📚 Next Steps

1. **Access n8n** - Open the URL in your browser
2. **Create credentials** - Add API keys for services
3. **Build first workflow** - Follow [Your First AI Workflow](first-ai-workflow.md)
4. **Learn basics** - Read [Quick Start](quickstart.md)

---

## 🔗 Resources

- [Official Installation Docs](https://docs.n8n.io/hosting/installation/)
- [Docker Hub](https://hub.docker.com/r/n8nio/n8n)
- [Environment Variables Reference](https://docs.n8n.io/hosting/environment-variables/)
- [Self-Hosting Options](https://docs.n8n.io/hosting/installation/server-setups/)

---

**Installation Time:** 5-15 minutes depending on method  
**Difficulty:** Beginner  
**Next:** [Quick Start](quickstart.md)
