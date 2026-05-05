# Falcon Courier Services - Deployment Guide

This guide covers deploying the Falcon Courier Services application to production using Cloudflare Workers and GitHub.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Database Configuration](#database-configuration)
4. [Cloudflare Workers Deployment](#cloudflare-workers-deployment)
5. [GitHub Integration](#github-integration)
6. [Post-Deployment](#post-deployment)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before deploying, ensure you have:

- **Node.js** 18+ and pnpm installed
- **Cloudflare Account** with Workers enabled
- **GitHub Account** for version control
- **MySQL/TiDB Database** (cloud-hosted or self-managed)
- **Manus OAuth Credentials** (provided by platform)

---

## Environment Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/falconsl-website.git
cd falconsl-website
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Configure Environment Variables

Create a `.env.production` file in the project root:

```env
# Database
DATABASE_URL=mysql://user:password@host:port/database_name

# OAuth
VITE_APP_ID=your_manus_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://oauth.manus.im

# JWT
JWT_SECRET=your_secure_jwt_secret_key_here

# Owner Info
OWNER_OPEN_ID=your_owner_open_id
OWNER_NAME=Your Name

# Manus APIs
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_forge_api_key
VITE_FRONTEND_FORGE_API_KEY=your_frontend_forge_key
VITE_FRONTEND_FORGE_API_URL=https://api.manus.im

# Analytics (optional)
VITE_ANALYTICS_ENDPOINT=https://analytics.example.com
VITE_ANALYTICS_WEBSITE_ID=your_website_id
```

---

## Database Configuration

### 1. Create Database Schema

```bash
# Generate migrations (if needed)
pnpm drizzle-kit generate

# Apply migrations to your database
pnpm drizzle-kit migrate
```

### 2. Seed Demo Data

```bash
# Run the seed script to populate demo data
node server/seed.mjs
```

This creates:
- 3 staff users (2 couriers, 1 admin)
- 20 sample shipments
- 50+ tracking events

### 3. Verify Database Connection

Test the connection:

```bash
node -e "
import { getDb } from './server/db.ts';
const db = await getDb();
console.log('✅ Database connected');
"
```

---

## Cloudflare Workers Deployment

### 1. Install Wrangler CLI

```bash
npm install -g wrangler
```

### 2. Authenticate with Cloudflare

```bash
wrangler login
```

### 3. Create Wrangler Configuration

Create `wrangler.toml` in the project root:

```toml
name = "falconsl-website"
type = "javascript"
account_id = "your_account_id"
workers_dev = true
route = "example.com/*"
zone_id = "your_zone_id"

[env.production]
name = "falconsl-website-prod"
route = "courier.example.com/*"
zone_id = "your_production_zone_id"

[[env.production.kv_namespaces]]
binding = "KV_STORE"
id = "your_kv_namespace_id"

[build]
command = "pnpm build"
cwd = "./"
main = "dist/index.js"

[build.upload]
format = "service-worker"
```

### 4. Build and Deploy

```bash
# Build the project
pnpm build

# Deploy to Cloudflare Workers
wrangler deploy --env production
```

### 5. Set Environment Variables on Cloudflare

```bash
# Set secrets for production
wrangler secret put DATABASE_URL --env production
wrangler secret put JWT_SECRET --env production
wrangler secret put VITE_APP_ID --env production
# ... repeat for all sensitive env vars
```

---

## GitHub Integration

### 1. Create GitHub Repository

```bash
# Initialize git (if not already done)
git init
git add .
git commit -m "Initial commit: Falcon Courier Services"

# Add remote
git remote add origin https://github.com/your-org/falconsl-website.git
git branch -M main
git push -u origin main
```

### 2. Create GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Cloudflare Workers

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install pnpm
        run: npm install -g pnpm
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run tests
        run: pnpm test
      
      - name: Build project
        run: pnpm build
      
      - name: Deploy to Cloudflare
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          environment: production
```

### 3. Add GitHub Secrets

In your GitHub repository settings, add:

- `CLOUDFLARE_API_TOKEN`: Your Cloudflare API token
- `CLOUDFLARE_ACCOUNT_ID`: Your Cloudflare account ID

---

## Post-Deployment

### 1. Verify Deployment

```bash
# Check deployment status
wrangler deployments list --env production

# Test the endpoint
curl https://courier.example.com/
```

### 2. Configure Custom Domain

1. Go to Cloudflare Dashboard
2. Navigate to Workers > Routes
3. Add route: `courier.example.com/*`
4. Point to your worker

### 3. Set Up SSL/TLS

Cloudflare automatically provides SSL/TLS. Verify:

1. Dashboard > SSL/TLS > Overview
2. Set to "Full (strict)" for best security

### 4. Enable Caching

Configure caching rules in Cloudflare:

1. Caching > Cache Rules
2. Add rule: Cache static assets (CSS, JS, images)
3. Set TTL to 1 month for versioned assets

### 5. Monitor Performance

- Cloudflare Dashboard > Analytics
- Check request volume, error rates, latency
- Set up alerts for anomalies

---

## Database Backups

### 1. Automated Backups

For cloud databases (TiDB, AWS RDS):
- Enable automated backups (daily)
- Set retention to 30 days
- Test restore procedures monthly

### 2. Manual Backup

```bash
# MySQL backup
mysqldump -u user -p database_name > backup_$(date +%Y%m%d).sql

# Restore from backup
mysql -u user -p database_name < backup_20260504.sql
```

---

## Scaling Considerations

### 1. Database

- Monitor query performance
- Add indexes for frequently queried columns
- Consider read replicas for high traffic

### 2. Workers

- Cloudflare Workers auto-scales
- Monitor CPU time and memory usage
- Optimize hot paths in code

### 3. Caching

- Cache tracking data (read-heavy)
- Use KV store for session data
- Implement cache invalidation on updates

---

## Security Best Practices

### 1. Secrets Management

- Never commit `.env` files
- Use Cloudflare Secrets for sensitive data
- Rotate API keys quarterly

### 2. CORS Configuration

Configure in `server/_core/index.ts`:

```typescript
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", process.env.FRONTEND_URL);
  res.header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
  res.header("Access-Control-Allow-Headers", "Content-Type, Authorization");
  next();
});
```

### 3. Rate Limiting

Implement rate limiting for public endpoints:

```typescript
import rateLimit from "express-rate-limit";

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use("/api/", limiter);
```

### 4. Input Validation

All tRPC procedures validate input using Zod schemas.

---

## Troubleshooting

### Issue: Database Connection Fails

**Solution:**
```bash
# Verify connection string
echo $DATABASE_URL

# Test connection
mysql -h host -u user -p -e "SELECT 1"

# Check firewall rules
# Ensure Cloudflare Workers IP is whitelisted
```

### Issue: OAuth Not Working

**Solution:**
1. Verify `VITE_APP_ID` matches Manus OAuth app
2. Check redirect URI: `https://courier.example.com/api/oauth/callback`
3. Ensure `OAUTH_SERVER_URL` is correct

### Issue: Slow Queries

**Solution:**
```bash
# Enable query logging
mysql -e "SET GLOBAL slow_query_log = 'ON';"
mysql -e "SET GLOBAL long_query_time = 2;"

# Analyze slow queries
mysqldumpslow /var/log/mysql/slow.log | head -20
```

### Issue: Workers Timeout

**Solution:**
- Optimize database queries
- Implement caching
- Break long operations into smaller tasks
- Check CPU time in Cloudflare logs

---

## Monitoring & Alerts

### 1. Cloudflare Analytics

- Dashboard > Analytics & Logs
- Monitor: Requests, Errors, Latency, Bandwidth

### 2. Database Monitoring

```bash
# Monitor connections
SHOW PROCESSLIST;

# Check table sizes
SELECT table_name, ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.TABLES
WHERE table_schema = 'database_name';
```

### 3. Application Logs

Logs are available at:
- Cloudflare Workers > Logs
- GitHub Actions > Workflow runs

---

## Rollback Procedure

If deployment causes issues:

```bash
# View deployment history
wrangler deployments list --env production

# Rollback to previous version
wrangler rollback --env production --message "Rollback to stable version"
```

---

## Support & Resources

- **Manus Documentation**: https://docs.manus.im
- **Cloudflare Workers Docs**: https://developers.cloudflare.com/workers/
- **Drizzle ORM Docs**: https://orm.drizzle.team/
- **tRPC Docs**: https://trpc.io/

---

## Maintenance Schedule

| Task | Frequency | Owner |
|------|-----------|-------|
| Security updates | Weekly | DevOps |
| Database optimization | Monthly | DBA |
| Backup verification | Monthly | DevOps |
| Performance review | Weekly | Engineering |
| Security audit | Quarterly | Security |

---

## Contact

For deployment issues or questions:
- Email: ops@falcon.lk
- Slack: #falcon-deployment
- On-call: Check PagerDuty

---

**Last Updated**: May 2026
**Version**: 1.0
