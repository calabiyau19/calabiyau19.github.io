---
layout: post
title: "Ghostfolio Complete Setup Guide"  
draft: false
date: 2025-12-01
description: "Complete guide for installing Ghostfolio using Docker on Ubuntu 24.04, setting up paper trading portfolios and real investment tracking, and migrating to the latest version."
---

## Ghostfolio Installation and Setup Guide

Complete guide for installing Ghostfolio using Docker on Ubuntu 24.04, setting up paper trading portfolios and real investment tracking, and migrating to the latest version.

---

## Table of Contents

1. [Initial Installation (Working Version)](#initial-installation-working-version)
2. [Setting Up Paper Trading Portfolio](#setting-up-paper-trading-portfolio)
3. [Setting Up Real Investment Tracking](#setting-up-real-investment-tracking)
4. [Adding Cryptocurrency Holdings](#adding-cryptocurrency-holdings)
5. [Migrating to Latest Version](#migrating-to-latest-version)
6. [Troubleshooting](#troubleshooting)
7. [Maintenance and Updates](#maintenance-and-updates)

---

## Initial Installation (Working Version)

This installation uses a pinned version (2.109.0) to ensure stability while getting started.

### Prerequisites

- Ubuntu 24.04 server
- Docker and Docker Compose installed
- Port 3333 available

### Step 1: Create Working Directory

```sh
mkdir -p ~/ghostfolio
cd ~/ghostfolio
```

### Step 2: Generate Secure Passwords

Generate passwords using hex encoding to avoid special character issues:

```sh
echo "REDIS_PASSWORD=$(openssl rand -hex 32)"
echo "POSTGRES_PASSWORD=$(openssl rand -hex 32)"
echo "ACCESS_TOKEN_SALT=$(openssl rand -hex 32)"
echo "JWT_SECRET_KEY=$(openssl rand -hex 32)"
```

**Important:** Save these passwords - you'll need them in the next step.

### Step 3: Create Environment File

Create `.env` file with your generated passwords:

```sh
nano .env
```

Add the following content (replace placeholders with your generated passwords):

```
# Ghostfolio Environment Configuration
REDIS_PASSWORD=YOUR_REDIS_PASSWORD_HERE
POSTGRES_PASSWORD=YOUR_POSTGRES_PASSWORD_HERE
ACCESS_TOKEN_SALT=YOUR_ACCESS_TOKEN_HERE
JWT_SECRET_KEY=YOUR_JWT_SECRET_HERE

# Database Configuration
POSTGRES_DB=ghostfolio-db
POSTGRES_USER=user
```

Save and exit (Ctrl+X, then Y, then Enter).

### Step 4: Create Docker Compose File

Create `docker-compose.yml`:

```sh
nano docker-compose.yml
```

Add the following content:

```yaml
name: ghostfolio

services:
  ghostfolio:
    image: docker.io/ghostfolio/ghostfolio:2.109.0
    container_name: ghostfolio
    restart: unless-stopped
    init: true
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?connect_timeout=300&sslmode=prefer
      NODE_ENV: production
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: ${REDIS_PASSWORD}
      ACCESS_TOKEN_SALT: ${ACCESS_TOKEN_SALT}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
    ports:
      - 3333:3333
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  postgres:
    image: docker.io/library/postgres:15-alpine
    container_name: gf-postgres
    restart: unless-stopped
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_READ_SEARCH
      - FOWNER
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}']
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres:/var/lib/postgresql/data

  redis:
    image: docker.io/library/redis:alpine
    container_name: gf-redis
    restart: unless-stopped
    cap_drop:
      - ALL
    cap_add:
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    command: ['redis-server', '--requirepass', '${REDIS_PASSWORD}']
    healthcheck:
      test: ['CMD-SHELL', 'redis-cli --pass $${REDIS_PASSWORD} ping | grep PONG']
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - redis:/data

volumes:
  postgres:
  redis:
```

Save and exit.

### Step 5: Deploy Ghostfolio

```sh
cd ~/ghostfolio
docker compose up -d
```

Wait for all containers to start (about 30 seconds).

### Step 6: Verify Installation

Check container status:

```sh
docker compose ps
```

All three containers should show as "running (healthy)".

View logs:

```sh
docker compose logs -f
```

Press Ctrl+C to exit logs.

### Step 7: Initial Setup

1. Access Ghostfolio in your browser: `http://YOUR_SERVER_IP:3333`
2. Click **"Get Started"**
3. Create your admin account
4. **IMPORTANT:** Save the security token displayed - you'll need this for API access

### Step 8: Configure Firewall (if using UFW)

```sh
sudo ufw allow 3333/tcp comment 'Ghostfolio'
sudo ufw reload
```

---

## Setting Up Paper Trading Portfolio

Paper trading allows you to track simulated investments without real money.

### Step 1: Create Trading Account

1. Navigate to **Accounts** in Ghostfolio
2. Click **Add Account**
3. Configure:
   - **Name:** Paper Trading Portfolio (or any descriptive name)
   - **Platform:** Leave blank or set to "Paper Trading"
   - **Currency:** USD
   - **Initial Balance:** Leave at $0 (we'll manage cash through transactions)

### Step 2: Determine Portfolio Allocation

Example: $6,000 portfolio split across 12 stocks at ~$500 each.

For each stock, calculate shares to buy:

```
Shares = $500 / Current Stock Price
```

Round down to whole shares (or use fractional if supported by your broker simulation).

### Step 3: Add Stock Purchases

For each stock in your portfolio:

1. Go to **Activities** → **Add Activity**
2. Select your paper trading account
3. Search for stock ticker (e.g., NVDA, PANW, etc.)
4. Configure transaction:
   - **Type:** BUY
   - **Date:** Your simulated purchase date
   - **Quantity:** Number of shares calculated
   - **Unit Price:** Current stock price
   - **Fee:** 0 (or realistic broker fee like $0-10)
   - **✓ CHECK "Update Cash Balance"** ← This is critical!
5. Click **Save**

Repeat for all stocks in your portfolio.

### Step 4: Verify Portfolio

1. Go to **Accounts** → Select your paper trading account
2. Verify:
   - **Total value** matches your intended investment
   - **Cash balance** shows remaining funds
   - **Equity** shows total stock value
   - All holdings appear in the list

### Example Transaction Entry

```
Stock: NVDA (Nvidia Corporation)
Price: $177.00
Allocation: $500
Shares: 2 (calculated as $500 / $177 = 2.82, rounded to 2)
Total Cost: $354.00
Cash Remaining: Updated automatically
```

---

## Setting Up Real Investment Tracking

Track actual investments from brokerages like Fidelity, Schwab, etc.

### Step 1: Create Investment Account

1. Navigate to **Accounts**
2. Click **Add Account**
3. Configure:
   - **Name:** Fidelity Index Funds (or your brokerage name)
   - **Platform:** Fidelity (or leave blank)
   - **Currency:** USD
   - **Initial Balance:** $0

### Step 2: Add Existing Holdings

For holdings you already own (not new purchases):

1. Go to **Activities** → **Add Activity**
2. Select your investment account
3. Search for fund ticker (e.g., FXAIX, FSKAX)
4. Configure transaction:
   - **Type:** BUY
   - **Date:** Your original purchase date (for accurate cost basis)
   - **Quantity:** Shares you currently own
   - **Unit Price:** Your original purchase price per share
   - **Fee:** 0 (already paid in the past)
   - **❌ UNCHECK "Update Cash Balance"** ← Don't deduct cash for existing holdings
5. Click **Save**

### Step 3: Verify Tracking

1. Navigate to your investment account
2. Verify all holdings appear with correct share counts
3. Check that performance calculations show your actual gains/losses

### Checking if Your Fund Works

Before adding, verify the fund is available:

1. Go to Yahoo Finance: `https://finance.yahoo.com`
2. Search for your fund's ticker symbol
3. If found: ✓ Ghostfolio will auto-update prices
4. If not found: You'll need manual price updates

### Common Fidelity Index Funds

These work automatically in Ghostfolio:

- **FXAIX** - Fidelity 500 Index Fund
- **FSKAX** - Fidelity Total Market Index Fund
- **FTIHX** - Fidelity Total International Index Fund
- **FXNAX** - Fidelity U.S. Bond Index Fund

---

## Adding Cryptocurrency Holdings

Track Bitcoin, Ethereum, and other cryptocurrencies.

### Step 1: Create Crypto Account

1. Navigate to **Accounts**
2. Click **Add Account**
3. Configure:
   - **Name:** Cryptocurrency Portfolio
   - **Platform:** Leave blank or "Paper Trading"
   - **Currency:** USD
   - **Initial Balance:** Amount you're allocating to crypto

### Step 2: Calculate Fractional Cryptocurrency

Most cryptocurrencies require fractional purchases due to high prices.

Example: Bitcoin at $85,773 per BTC

```
Investment Amount: $861.40
BTC Price: $85,773
Fractional BTC = $861.40 / $85,773 = 0.01004 BTC
```

### Step 3: Add Crypto Purchase

1. Go to **Activities** → **Add Activity**
2. Select your crypto account
3. Search for **BTC** or **Bitcoin** (pulls BTC-USD from Yahoo Finance)
4. Configure transaction:
   - **Type:** BUY
   - **Date:** Purchase date
   - **Quantity:** 0.01004 (fractional amount calculated)
   - **Unit Price:** 85773
   - **Fee:** 0
   - **✓ CHECK "Update Cash Balance"**
5. Click **Save**

### Supported Cryptocurrencies

Ghostfolio supports major cryptocurrencies via Yahoo Finance:

- Bitcoin (BTC-USD)
- Ethereum (ETH-USD)
- Cardano (ADA-USD)
- Polkadot (DOT-USD)
- And many more - search by symbol

---

## Migrating to Latest Version

Upgrade from pinned version (2.109.0) to latest version with proper configuration.

### Why Migrate?

**Benefits:**
- Automatic updates using `latest` tag
- Newest features and bug fixes
- Cleaner configuration with hex passwords

**When to Migrate:**
- After you're comfortable with Ghostfolio (1-2 weeks)
- When you want automatic updates
- If you need a new feature from latest version

### What Gets Preserved

✓ All portfolio data (stocks, funds, crypto)
✓ All transaction history
✓ Account configurations
✓ Performance tracking history
✓ User accounts and settings

### What Changes

- Docker image: `2.109.0` → `latest`
- Passwords: New hex-encoded passwords generated
- Configuration: Cleaner environment variable structure

### Migration Process

#### Step 1: Create Migration Script

```sh
cd ~/ghostfolio
nano migrate-ghostfolio.sh
```

Paste the following script:

```sh
#!/bin/bash
#
# Ghostfolio Migration Script
# Upgrades to latest version with proper hex password configuration
#

set -e

echo "========================================="
echo "Ghostfolio Migration to Latest Tag"
echo "========================================="
echo ""

cd ~/ghostfolio || { echo "ERROR: ~/ghostfolio not found!"; exit 1; }

echo "Step 1: Generating new hex passwords..."
REDIS_PW=$(openssl rand -hex 32)
POSTGRES_PW=$(openssl rand -hex 32)
ACCESS_TOKEN=$(openssl rand -hex 32)
JWT_SECRET=$(openssl rand -hex 32)
echo "✓ Generated secure passwords"
echo ""

echo "Step 2: Backing up current configuration..."
if [ -f .env ]; then
    cp .env .env.backup-$(date +%Y%m%d-%H%M%S)
    echo "✓ Backed up existing .env"
fi

if [ -f docker-compose.yml ]; then
    cp docker-compose.yml docker-compose.yml.backup-$(date +%Y%m%d-%H%M%S)
    echo "✓ Backed up existing docker-compose.yml"
fi
echo ""

echo "Step 3: Creating new .env file..."
cat > .env << EOF
# Ghostfolio Environment Configuration
# Generated: $(date)
# Using hex-encoded passwords (no special characters)

REDIS_PASSWORD=${REDIS_PW}
POSTGRES_PASSWORD=${POSTGRES_PW}
ACCESS_TOKEN_SALT=${ACCESS_TOKEN}
JWT_SECRET_KEY=${JWT_SECRET}

# Database Configuration
POSTGRES_DB=ghostfolio-db
POSTGRES_USER=user
EOF
echo "✓ Created new .env file"
echo ""

echo "Step 4: Creating new docker-compose.yml..."
cat > docker-compose.yml << 'COMPOSE'
name: ghostfolio

services:
  ghostfolio:
    image: docker.io/ghostfolio/ghostfolio:latest
    container_name: ghostfolio
    restart: unless-stopped
    init: true
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?connect_timeout=300&sslmode=prefer
      NODE_ENV: production
      REDIS_HOST: redis
      REDIS_PORT: 6379
      REDIS_PASSWORD: ${REDIS_PASSWORD}
      ACCESS_TOKEN_SALT: ${ACCESS_TOKEN_SALT}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
    ports:
      - 3333:3333
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ['CMD-SHELL', 'curl -f http://localhost:3333/api/v1/health']
      interval: 10s
      timeout: 5s
      retries: 5

  postgres:
    image: docker.io/library/postgres:15-alpine
    container_name: gf-postgres
    restart: unless-stopped
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_READ_SEARCH
      - FOWNER
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}']
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres:/var/lib/postgresql/data

  redis:
    image: docker.io/library/redis:alpine
    container_name: gf-redis
    restart: unless-stopped
    cap_drop:
      - ALL
    cap_add:
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    command: ['redis-server', '--requirepass', '${REDIS_PASSWORD}']
    healthcheck:
      test: ['CMD-SHELL', 'redis-cli --pass $${REDIS_PASSWORD} ping | grep PONG']
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - redis:/data

volumes:
  postgres:
  redis:
COMPOSE
echo "✓ Created new docker-compose.yml"
echo ""

echo "Step 5: Stopping old containers..."
docker compose down
echo "✓ Stopped containers"
echo ""

echo "Step 6: Starting new containers..."
docker compose up -d
echo "✓ Started containers"
echo ""

echo "========================================="
echo "Migration Complete!"
echo "========================================="
echo ""
echo "Your Ghostfolio is now running on the 'latest' tag"
echo "Access it at: http://YOUR_SERVER_IP:3333"
echo ""
echo "IMPORTANT:"
echo "  - Your admin account still works"
echo "  - All portfolio data is preserved"
echo "  - Passwords have been changed (check .env for new values)"
echo "  - Backups saved with timestamp in filename"
echo ""
echo "View logs: docker compose logs -f"
echo "Check status: docker compose ps"
echo ""
```

Save and exit (Ctrl+X, then Y, then Enter).

#### Step 2: Make Script Executable

```sh
chmod +x migrate-ghostfolio.sh
```

#### Step 3: Run Migration

```sh
./migrate-ghostfolio.sh
```

The script will:
1. Generate new hex passwords
2. Backup your current configuration
3. Create new docker-compose.yml with `latest` tag
4. Stop old containers
5. Start new containers
6. Preserve all your data

#### Step 4: Verify Migration

```sh
docker compose ps
```

All containers should be running and healthy.

Access Ghostfolio and verify:
- All your accounts are present
- All holdings are intact
- Transaction history is preserved
- Performance tracking continues

#### Step 5: Check New Configuration

View your new passwords:

```sh
cat .env
```

View backups created:

```sh
ls -la *.backup-*
```

---

## Troubleshooting

### Container Won't Start

Check logs:

```sh
docker compose logs ghostfolio
docker compose logs postgres
docker compose logs redis
```

Common issues:
- Port 3333 already in use
- Database connection issues (check password encoding)
- Redis authentication failures

### Reset Installation

If you need to start completely fresh:

```sh
cd ~/ghostfolio
docker compose down -v  # WARNING: Deletes all data!
# Then re-run installation from Step 4
```

### Backup Database

Create manual backup:

```sh
docker run --rm \
  -v ghostfolio_postgres:/source:ro \
  -v ~/backups:/backup \
  alpine tar czf /backup/ghostfolio-db-$(date +%Y%m%d).tar.gz -C /source .
```

### Restore Database

Restore from backup:

```sh
docker run --rm \
  -v ghostfolio_postgres:/target \
  -v ~/backups:/backup \
  alpine sh -c "cd /target && tar xzf /backup/ghostfolio-db-YYYYMMDD.tar.gz"
```

### Check Container Health

```sh
docker inspect ghostfolio | grep -A 10 Health
docker inspect gf-postgres | grep -A 10 Health
docker inspect gf-redis | grep -A 10 Health
```

### View Real-time Logs

```sh
docker compose logs -f --tail=50
```

Press Ctrl+C to exit.

---

## Maintenance and Updates

### Price Update Frequency

Ghostfolio updates stock prices **once per day** during off-market hours using Yahoo Finance data.

Manual price update (as admin):
1. Go to **Admin Control Panel**
2. Navigate to **Market Data**
3. Click **"Gather All Data"**

### Updating to Latest Version

After migrating to `latest` tag:

```sh
cd ~/ghostfolio
docker compose pull
docker compose up -d
```

Ghostfolio automatically runs database migrations on startup.

### Container Management

Start containers:

```sh
docker compose up -d
```

Stop containers:

```sh
docker compose down
```

Restart containers:

```sh
docker compose restart
```

View container status:

```sh
docker compose ps
```

### Disk Space Management

Check Docker disk usage:

```sh
docker system df
```

Clean up unused images:

```sh
docker image prune -a
```

### Security Best Practices

1. **Never commit .env to version control**
2. **Keep passwords secure** - they're in `.env` file
3. **Regular backups** - backup PostgreSQL volume weekly
4. **Firewall configuration** - only expose port 3333 if needed
5. **Use reverse proxy** - consider nginx or Caddy for HTTPS
6. **Update regularly** - when using `latest` tag

### Monitoring

Basic health check:

```sh
curl http://localhost:3333/api/v1/health
```

Should return healthy status.

---

## Additional Resources

- **Official Documentation:** https://github.com/ghostfolio/ghostfolio
- **Docker Hub:** https://hub.docker.com/r/ghostfolio/ghostfolio
- **Issue Tracker:** https://github.com/ghostfolio/ghostfolio/issues
- **Community Slack:** Check GitHub for invite link

---

## Summary

**Current Installation:**
- Version: 2.109.0 (pinned for stability)
- Location: `~/ghostfolio`
- Access: `http://YOUR_SERVER_IP:3333`
- Data stored in Docker volumes (persistent across restarts)

**Portfolio Setup:**
- Paper trading: Check "Update Cash Balance" when adding transactions
- Real investments: Uncheck "Update Cash Balance" for existing holdings
- Crypto: Use fractional amounts for high-price assets

**Migration to Latest:**
- Run migration script when ready (1-2 weeks after setup)
- All data preserved automatically
- Enables automatic updates going forward

**Maintenance:**
- Daily automatic price updates from Yahoo Finance
- Manual updates available via Admin Control Panel
- Regular backups recommended for PostgreSQL volume
