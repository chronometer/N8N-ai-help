# Deployment Guide

Deploy n8n to production environments.

## 📖 Contents

### [Self-Hosting Options](self-hosting.md)
Deploy n8n on your own infrastructure.

### [n8n Cloud](cloud.md)
Use n8n's managed hosting service.

### [Docker Deployment](docker.md)
Deploy using Docker containers.

### [Environment Variables](environment-variables.md)
Configure n8n with environment variables.

### [Scaling Considerations](scaling.md)
Handle growth and performance requirements.

---

## 🎯 Quick Deployment Decision

### I want the easiest setup
→ **n8n Cloud** - Just sign up and go

### I want to self-host
→ **Docker** - Use Docker Compose for all-in-one

### I want to self-host with control
→ **Self-Hosting** - Choose your platform

### I'm already using cloud
→ **AWS** / **Google Cloud** / **Azure** - Deploy there

---

## 🚀 Quick Start Deployment

### Option 1: n8n Cloud (5 minutes)
1. Go to https://app.n8n.cloud
2. Sign up
3. Start building

### Option 2: Docker (15 minutes)
```bash
docker run -it --rm \
  -p 5678:5678 \
  -e NODE_ENV=production \
  n8nio/n8n
```

### Option 3: npm (10 minutes)
```bash
npm install -g n8n
n8n start
```

---

## 🏗️ Deployment Options Comparison

| Option | Setup | Control | Cost | Scaling |
|--------|-------|---------|------|---------|
| n8n Cloud | Easiest | Limited | Pay/execute | Auto |
| Docker | Easy | Full | Infrastructure | Manual |
| npm | Medium | Full | Infrastructure | Manual |
| AWS | Medium | Full | Infrastructure | Manual |
| GCP | Medium | Full | Infrastructure | Manual |
| Azure | Medium | Full | Infrastructure | Manual |

---

## 🔑 Key Configuration

### Essential Variables

```bash
# Server
N8N_HOST=0.0.0.0
N8N_PORT=5678
N8N_PROTOCOL=https

# Database
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_DATABASE=n8n

# Security
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=password

# Execution
N8N_EXECUTION_MODE=regular
```

See [Environment Variables](environment-variables.md) for complete list.

---

## 📊 Deployment Checklist

- [ ] Choose deployment method
- [ ] Set up infrastructure
- [ ] Configure database
- [ ] Configure environment variables
- [ ] Set up SSL/HTTPS
- [ ] Test workflows
- [ ] Set up monitoring
- [ ] Configure backups
- [ ] Document setup

---

## 🔒 Security Setup

### 1. Enable HTTPS
- Use Let's Encrypt for SSL
- Configure reverse proxy (nginx, etc.)

### 2. Enable Authentication
- Set basic auth credentials
- Or use SSO (OIDC, SAML)

### 3. Secure Credentials
- Use environment variables
- Rotate keys regularly
- Limit access

### 4. Database Security
- Use strong passwords
- Enable encryption
- Regular backups

### 5. Network Security
- Use VPN for admin access
- Firewall rules
- Rate limiting

---

## 📈 Scaling Considerations

### When You Need to Scale

- More than 100 workflows
- More than 1000 daily executions
- Workflows taking > 30 seconds
- Team of 5+ people

### Scaling Options

1. **Upgrade infrastructure** - More CPU/RAM
2. **Queue mode** - Separate execution workers
3. **Multiple instances** - Load balanced
4. **Managed database** - PostgreSQL on cloud

---

## 🛠️ Common Deployments

### Small Business (< 10 workflows)
**Recommended:** n8n Cloud or single Docker container

### Medium Business (10-100 workflows)
**Recommended:** Docker on cloud platform with external database

### Large Enterprise (100+ workflows)
**Recommended:** Queue mode with multiple workers, managed database

---

## 🐛 Troubleshooting

### Issue 1: Cannot Connect
Check network, firewall, DNS settings

### Issue 2: Database Errors
Verify database running, credentials correct

### Issue 3: Slow Performance
Check CPU/RAM, database performance, workflow complexity

### Issue 4: Execution Failures
Check logs, verify configurations, test workflows

---

## 📚 Monitoring and Alerts

### What to Monitor

- Workflow execution success rate
- Execution time
- Error rates
- Database performance
- Server resources

### How to Monitor

- n8n execution logs
- Application performance monitoring (APM)
- Cloud provider monitoring
- Custom alerts

---

## 🔄 Backup and Recovery

### Backup Strategy

1. **Database backups** - Daily full backups
2. **Workflow exports** - Backup workflow JSON
3. **Configuration backups** - Backup environment files

### Recovery Process

1. Restore database
2. Restart n8n
3. Verify workflows running

---

## 🚀 Migration Guide

### Migrate from Cloud to Self-Hosted

1. Export workflows from cloud
2. Set up self-hosted n8n
3. Import workflows
4. Reconfigure credentials
5. Test all workflows

### Migrate Between Servers

1. Export workflows
2. Export database
3. Set up new server
4. Restore database
5. Import workflows
6. Test

---

## 🔗 Related Sections

- [Best Practices](../best-practices/) - Guidelines
- [Environment Variables](environment-variables.md) - Configuration
- [Installation](../getting-started/installation.md) - Setup guide

---

## 📝 Deployment Templates

### Docker Compose
See [Docker Deployment](docker.md)

### AWS CloudFormation
See [Self-Hosting Options](self-hosting.md)

### Kubernetes
See [Self-Hosting Options](self-hosting.md)

---

## 📚 Resources

- [n8n Hosting Docs](https://docs.n8n.io/hosting/)
- [Docker Hub](https://hub.docker.com/r/n8nio/n8n)
- [Cloud Pricing](https://n8n.io/cloud/)

---

**Level:** Intermediate to Advanced  
**Time:** Varies by deployment method  
**Prerequisites:** n8n basics and system administration knowledge
