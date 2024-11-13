# BimarzNet - Global Freedom Access Platform

A comprehensive VPN management and distribution platform built with modern technologies and focused on providing unrestricted internet access globally. BimarzNet combines advanced payment systems, intelligent resource management, and user-friendly interfaces to create a robust and reliable VPN service.

## Vision & Mission

BimarzNet aims to break down digital barriers and provide unrestricted internet access to users worldwide, with special consideration for regions with strict internet controls. Our platform combines cutting-edge technology with user-friendly interfaces to deliver:

- Unrestricted and secure internet access
- Multiple connection methods (Direct and Tunneled)
- Region-specific optimizations
- Intelligent resource management
- Community-driven growth

## Core Values

1. **Freedom of Access**
   - Unrestricted internet access as a fundamental right
   - Breaking down digital barriers
   - Supporting global connectivity

2. **Security & Privacy**
   - End-to-end encryption
   - No-logs policy
   - Anonymous payment options
   - Protected user identities

3. **User Empowerment**
   - Educational resources
   - Community support
   - Technical guidance
   - Transparent operations

4. **Community Growth**
   - Referral rewards
   - Daily engagement rewards
   - Community challenges
   - Shared success

## Key Features

### 1. Dual Connection Methods
- **Direct Connections**
  - SSH-based tunneling
  - Optimized for unrestricted regions
  - Maximum speed and efficiency
  - Automatic server selection

- **Tunneled Connections**
  - V2ray implementation
  - Multi-hop routing
  - Region-specific optimization
  - Automatic fallback systems

### 2. Advanced Payment Systems
- **Multiple Providers**
  - Cryptomus integration
  - Telegram Payment API
  - TON blockchain support
  - Foton integration

- **Regional Solutions**
  - Card-to-Card automation
  - SMS-based verification
  - Manual processing backup
  - Multi-stage verification

### 3. User Management
- **Role-Based System**
  ```typescript
  enum UserRole {
    TRIAL = "trial",
    NORMAL = "normal",
    RESELLER = "reseller",
    SUPPORT = "support",
    ADMIN = "admin",
    MASTER = "master"
  }
  ```

- **Permission System**
  ```typescript
  interface Permission {
    resource: string;
    action: "read" | "write" | "delete" | "manage";
    conditions?: Record<string, unknown>;
  }
  ```

### 4. Reward Systems
- **Daily Gifts**
  ```typescript
  interface DailyReward {
    day: number;
    bandwidth: number;
    duration: number;
    multiplier: number;
    bonusFeatures?: string[];
  }
  ```

- **Referral System**
  ```typescript
  interface ReferralReward {
    directBonus: number;
    indirectBonus: number;
    lifetimePercent: number;
    minimumPurchase: number;
  }
  ```

### 5. Family Plans
- **Shared Resources**
  ```typescript
  interface FamilyPool {
    totalBandwidth: number;
    memberLimit: number;
    transferLimit: number;
    durationPool: number;
  }
  ```

- **Management Features**
  - Resource allocation
  - Usage monitoring
  - Member management
  - Shared payments

### 6. AI Integration
- **Content Generation**
  - Marketing materials
  - SEO optimization
  - Multi-language support
  - Brand consistency

- **Analytics**
  - Usage patterns
  - Performance metrics
  - Growth predictions
  - Optimization suggestions

  ## Technical Architecture

### Technology Stack
```
Runtime: Deno
Language: TypeScript
Database: SQLite + Redis
OS: Ubuntu 22.04
VPN: V2ray (Marzneshin) & Direct SSH
Core: sing-box
```

### System Components

#### 1. Core Server
```typescript
interface ServerConfig {
  port: number;
  host: string;
  workers: number;
  timeout: number;
  maxConnections: number;
}

class BimarzServer {
  private config: ServerConfig;
  private handlers: Map<string, RequestHandler>;
  private middleware: Middleware[];
  
  async initialize(): Promise<void>;
  async handleRequest(req: Request): Promise<Response>;
  async shutdown(): Promise<void>;
}
```

#### 2. Database Architecture
```typescript
interface DatabaseConfig {
  main: {
    path: string;
    poolSize: number;
    timeout: number;
  };
  cache: {
    host: string;
    port: number;
    db: number;
  };
}

interface DBManager {
  initialize(): Promise<void>;
  query<T>(sql: string, params?: unknown[]): Promise<T>;
  transaction<T>(callback: (tx: Transaction) => Promise<T>): Promise<T>;
  close(): Promise<void>;
}
```

#### 3. VPN Management
```typescript
interface VPNConfig {
  type: "direct" | "tunneled";
  protocol: "ssh" | "v2ray";
  server: {
    host: string;
    port: number;
    credentials: Record<string, string>;
  };
  routing: {
    entry: string[];
    exit: string[];
    fallback: string[];
  };
}

class VPNManager {
  async createConnection(config: VPNConfig): Promise<Connection>;
  async monitorConnection(id: string): Promise<ConnectionStatus>;
  async optimizeRoute(connection: Connection): Promise<void>;
}
```

#### 4. Payment Processing
```typescript
interface PaymentProcessor {
  provider: string;
  settings: Record<string, unknown>;
  hooks: {
    onInitiate: (payment: Payment) => Promise<void>;
    onVerify: (payment: Payment) => Promise<boolean>;
    onComplete: (payment: Payment) => Promise<void>;
    onFail: (payment: Payment, error: Error) => Promise<void>;
  };
}

class PaymentManager {
  private processors: Map<string, PaymentProcessor>;
  
  async processPayment(request: PaymentRequest): Promise<PaymentResult>;
  async verifyPayment(id: string): Promise<boolean>;
  async refundPayment(id: string): Promise<boolean>;
}
```

#### 5. Caching System
```typescript
interface CacheConfig {
  driver: "redis" | "memory";
  prefix: string;
  ttl: number;
  serializer?: (data: unknown) => string;
  deserializer?: (data: string) => unknown;
}

class CacheManager {
  async get<T>(key: string): Promise<T | null>;
  async set<T>(key: string, value: T, ttl?: number): Promise<void>;
  async delete(key: string): Promise<void>;
  async clear(): Promise<void>;
}
```

#### 6. Monitoring System
```typescript
interface MetricsCollector {
  measure(metric: string, value: number): void;
  increment(counter: string): void;
  decrement(counter: string): void;
  timing(operation: string, duration: number): void;
}

class MonitoringSystem {
  private collectors: MetricsCollector[];
  private alerts: AlertManager;
  
  async track(): Promise<void>;
  async alert(condition: AlertCondition): Promise<void>;
  async report(): Promise<MetricsReport>;
}
```

### Database Schema

#### Users Table
```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  telegram_id INTEGER UNIQUE NOT NULL,
  role TEXT NOT NULL DEFAULT 'normal',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  metadata JSON,
  settings JSON,
  permissions JSON,
  status TEXT DEFAULT 'active'
);
```

#### Subscriptions Table
```sql
CREATE TABLE subscriptions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id),
  plan_id INTEGER REFERENCES plans(id),
  bandwidth_limit INTEGER NOT NULL,
  bandwidth_used INTEGER DEFAULT 0,
  starts_at TIMESTAMP NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  status TEXT DEFAULT 'active',
  metadata JSON
);
```

#### Connections Table
```sql
CREATE TABLE connections (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER REFERENCES users(id),
  subscription_id INTEGER REFERENCES subscriptions(id),
  type TEXT NOT NULL,
  protocol TEXT NOT NULL,
  server_id INTEGER REFERENCES servers(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_used TIMESTAMP,
  metadata JSON
);
```

## Installation and Configuration

### Prerequisites

#### System Requirements
- Ubuntu 22.04 LTS
- Minimum 2GB RAM
- 20GB SSD Storage
- Stable internet connection

#### Software Requirements
- Deno 1.37 or higher
- Node.js 18+ (for specific tools)
- SQLite 3.37+
- Redis 6.2+
- sing-box core

### Installation Steps

1. **System Preparation**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y git curl unzip build-essential

# Install Redis
sudo apt install redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

2. **Install Deno**
```bash
# Download Deno installer
curl -fsSL https://deno.land/x/install/install.sh | sh

# Add Deno to PATH
echo 'export DENO_INSTALL="/root/.deno"' >> ~/.bashrc
echo 'export PATH="$DENO_INSTALL/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

3. **Clone Repository**
```bash
# Clone BimarzNet repository
git clone https://github.com/yourusername/BimarzNet.git
cd BimarzNet

# Copy example configuration
cp config.example.json config.json
```

4. **Configure Environment**
```bash
# Copy example environment file
cp .env.example .env

# Generate secret keys
deno task generate-keys
```

5. **Initialize Database**
```bash
# Create database directory
mkdir -p data/db

# Run migrations
deno task db:migrate

# Seed initial data
deno task db:seed
```

### Configuration Files

#### 1. Environment Variables
```env
# Core Settings
APP_ENV=production
APP_PORT=8000
APP_KEY=your-secret-key

# Database
DB_PATH=./data/db/bimarz.db
REDIS_URL=redis://localhost:6379

# Payment APIs
CRYPTOMUS_API_KEY=your-key
CRYPTOMUS_MERCHANT_ID=your-id
TON_API_KEY=your-key
FOTON_API_KEY=your-key

# OpenAI
OPENAI_API_KEY=your-key

# Telegram
BOT_TOKEN=your-token
PAYMENT_TOKEN=your-payment-token

# VPN Settings
MARZBAN_API_URL=http://localhost:8000
MARZBAN_USERNAME=admin
MARZBAN_PASSWORD=your-password
```

#### 2. Server Configuration
```json
{
  "server": {
    "host": "0.0.0.0",
    "port": 8000,
    "workers": 4,
    "timeout": 30000
  },
  "database": {
    "main": {
      "path": "./data/db/bimarz.db",
      "poolSize": 10
    },
    "cache": {
      "host": "localhost",
      "port": 6379,
      "db": 0
    }
  },
  "vpn": {
    "protocols": ["ssh", "v2ray"],
    "defaultRoute": "direct",
    "fallbackRoute": "tunneled",
    "monitoring": {
      "interval": 60000,
      "timeout": 5000
    }
  }
}
```

#### 3. sing-box Configuration
```json
{
  "log": {
    "level": "info",
    "timestamp": true
  },
  "dns": {
    "servers": [
      {
        "tag": "google",
        "address": "8.8.8.8"
      },
      {
        "tag": "local",
        "address": "223.5.5.5",
        "detour": "direct"
      }
    ],
    "rules": [
      {
        "domain": "localhost",
        "server": "local"
      }
    ]
  },
  "inbounds": [
    {
      "type": "mixed",
      "tag": "mixed-in",
      "listen": "::",
      "listen_port": 2080
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "block",
      "tag": "block"
    }
  ]
}
```

### Post-Installation Steps

1. **Security Configuration**
```bash
# Configure firewall
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 8000
sudo ufw enable

# Set up SSL
sudo certbot --nginx -d yourdomain.com
```

2. **Service Configuration**
```bash
# Create systemd service
sudo nano /etc/systemd/system/bimarznet.service

[Unit]
Description=BimarzNet VPN Service
After=network.target

[Service]
Type=simple
User=bimarz
WorkingDirectory=/opt/bimarznet
ExecStart=/usr/local/bin/deno task start
Restart=always
Environment=PATH=/usr/local/bin:/usr/bin:/bin
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

3. **Start Services**
```bash
# Reload systemd
sudo systemctl daemon-reload

# Start BimarzNet
sudo systemctl enable bimarznet
sudo systemctl start bimarznet

# Check status
sudo systemctl status bimarznet
```

4. **Verify Installation**
```bash
# Check server status
curl http://localhost:8000/health

# Check logs
sudo journalctl -u bimarznet -f
```

## Detailed Feature Documentation

### User Management System

#### Role-Based Access Control
```typescript
interface UserPermissions {
  canManageUsers: boolean;
  canManageServers: boolean;
  canViewAnalytics: boolean;
  canProcessRefunds: boolean;
  canModifySettings: boolean;
  maxSubAccounts: number;
}

const rolePermissions: Record<UserRole, UserPermissions> = {
  TRIAL: {
    canManageUsers: false,
    canManageServers: false,
    canViewAnalytics: false,
    canProcessRefunds: false,
    canModifySettings: false,
    maxSubAccounts: 0
  },
  NORMAL: {
    canManageUsers: false,
    canManageServers: false,
    canViewAnalytics: false,
    canProcessRefunds: false,
    canModifySettings: true,
    maxSubAccounts: 1
  },
  RESELLER: {
    canManageUsers: true,
    canManageServers: false,
    canViewAnalytics: true,
    canProcessRefunds: true,
    canModifySettings: true,
    maxSubAccounts: 100
  }
  // ... other roles
};
```

#### User Reward System

##### Daily Login Rewards
```typescript
interface DailyReward {
  day: number;
  bandwidth: {
    amount: number;   // MB
    multiplier: number;
  };
  duration: {
    hours: number;
    multiplier: number;
  };
  maxStreak: number; // 15 days
  streakReset: {
    resetTo: number;
    keepRewards: boolean;
  };
}

const rewardConfig: DailyReward = {
  day: 1,
  bandwidth: {
    amount: 100,     // Start with 100MB
    multiplier: 1.5  // Increase by 50% each day
  },
  duration: {
    hours: 2,        // Start with 2 hours
    multiplier: 1.25 // Increase by 25% each day
  },
  maxStreak: 15,
  streakReset: {
    resetTo: 1,
    keepRewards: true
  }
};
```

##### Family Plan System
```typescript
interface FamilyPlan {
  core: {
    maxMembers: number;
    adminRoles: string[];
    memberRoles: string[];
  };
  resources: {
    sharedBandwidth: number;
    individualLimits: {
      daily: number;
      monthly: number;
    };
    transferLimits: {
      minimum: number;
      maximum: number;
      cooldown: number; // hours
    };
  };
  features: {
    monitoring: boolean;
    restrictions: boolean;
    sharing: boolean;
    reporting: boolean;
  };
  billing: {
    shared: boolean;
    splitOptions: string[];
    autoRenew: boolean;
  };
}
```

### Payment Processing System

#### Multi-Provider Integration
```typescript
interface PaymentProvider {
  name: string;
  enabled: boolean;
  config: {
    apiKey: string;
    webhook: string;
    successUrl: string;
    failureUrl: string;
  };
  limits: {
    minimum: number;
    maximum: number;
    currency: string;
  };
  features: {
    refunds: boolean;
    partial: boolean;
    recurring: boolean;
  };
}

const providers: Record<string, PaymentProvider> = {
  cryptomus: {
    name: "Cryptomus",
    enabled: true,
    config: {
      apiKey: process.env.CRYPTOMUS_API_KEY,
      webhook: "/webhooks/cryptomus",
      successUrl: "/payment/success",
      failureUrl: "/payment/failure"
    },
    limits: {
      minimum: 1,
      maximum: 1000,
      currency: "USD"
    },
    features: {
      refunds: true,
      partial: false,
      recurring: true
    }
  }
  // ... other providers
};
```

#### SMS-Based Verification System
```typescript
interface SMSVerification {
  provider: string;
  templates: {
    success: string;
    pending: string;
    failure: string;
  };
  timeouts: {
    initial: number;  // seconds
    retry: number;    // seconds
    maximum: number;  // seconds
  };
  matching: {
    patterns: string[];
    extractors: Record<string, RegExp>;
    validators: Record<string, (value: string) => boolean>;
  };
}

const smsConfig: SMSVerification = {
  provider: "local",
  templates: {
    success: "Payment of {amount} received from {sender}",
    pending: "Pending payment of {amount}",
    failure: "Failed payment attempt of {amount}"
  },
  timeouts: {
    initial: 30,
    retry: 60,
    maximum: 3600
  },
  matching: {
    patterns: [
      "card to card",
      "transfer received",
      "payment received"
    ],
    extractors: {
      amount: /(\d+,?\d*)/,
      sender: /from\s+(\w+)/
    },
    validators: {
      amount: (value: string) => parseFloat(value) > 0,
      sender: (value: string) => value.length >= 3
    }
  }
};
```

### VPN Connection Management

#### Direct Connection Setup
```typescript
interface DirectConnection {
  type: "ssh" | "v2ray";
  server: {
    host: string;
    port: number;
    username?: string;
    privateKey?: string;
  };
  client: {
    allowedIPs: string[];
    dns: string[];
    mtu: number;
  };
  routing: {
    allowedPorts: number[];
    blockedPorts: number[];
    allowedProtocols: string[];
  };
  monitoring: {
    interval: number;
    timeout: number;
    retries: number;
  };
}
```

#### Tunneled Connection Setup
```typescript
interface TunneledConnection {
  type: "v2ray";
  entry: {
    host: string;
    port: number;
    protocol: string;
  };
  exit: {
    host: string;
    port: number;
    protocol: string;
  };
  intermediate: {
    enabled: boolean;
    nodes: string[];
    randomize: boolean;
  };
  optimization: {
    mtu: number;
    buffer: number;
    congestion: string;
  };
  security: {
    encryption: string;
    certificates: string[];
    verifyPeers: boolean;
  };
}
```

## API Documentation

### RESTful API Endpoints

#### Authentication
```typescript
// POST /api/auth/login
interface LoginRequest {
  telegramId: number;
  authToken: string;
}

interface LoginResponse {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
  user: UserDetails;
}

// POST /api/auth/refresh
interface RefreshRequest {
  refreshToken: string;
}

// POST /api/auth/logout
interface LogoutRequest {
  refreshToken: string;
}
```

#### User Management
```typescript
// GET /api/users
interface ListUsersRequest {
  page?: number;
  limit?: number;
  role?: UserRole;
  status?: UserStatus;
}

// POST /api/users
interface CreateUserRequest {
  telegramId: number;
  role: UserRole;
  metadata?: Record<string, unknown>;
}

// GET /api/users/{id}
interface GetUserResponse {
  user: UserDetails;
  subscriptions: Subscription[];
  connections: Connection[];
  statistics: UserStatistics;
}

// PATCH /api/users/{id}
interface UpdateUserRequest {
  role?: UserRole;
  status?: UserStatus;
  metadata?: Record<string, unknown>;
}
```

#### Subscription Management
```typescript
// POST /api/subscriptions
interface CreateSubscriptionRequest {
  userId: number;
  planId: number;
  duration: number;
  bandwidth: number;
  metadata?: Record<string, unknown>;
}

// GET /api/subscriptions/{id}
interface GetSubscriptionResponse {
  subscription: Subscription;
  usage: UsageStatistics;
  connections: Connection[];
}

// PATCH /api/subscriptions/{id}
interface UpdateSubscriptionRequest {
  duration?: number;
  bandwidth?: number;
  status?: SubscriptionStatus;
}
```

#### Payment Processing
```typescript
// POST /api/payments/initiate
interface InitiatePaymentRequest {
  amount: number;
  currency: string;
  provider: string;
  metadata?: Record<string, unknown>;
}

interface InitiatePaymentResponse {
  paymentId: string;
  redirectUrl?: string;
  qrCode?: string;
  expiresIn: number;
}

// POST /api/payments/verify
interface VerifyPaymentRequest {
  paymentId: string;
  transactionId?: string;
  metadata?: Record<string, unknown>;
}

// POST /api/payments/refund
interface RefundRequest {
  paymentId: string;
  amount?: number;
  reason: string;
}
```

### WebSocket API

#### Connection
```typescript
// WS /ws/v1
interface WebSocketMessage {
  type: string;
  payload: unknown;
  timestamp: number;
}

interface WebSocketError {
  code: number;
  message: string;
  details?: unknown;
}
```

#### Real-time Updates
```typescript
// Connection Status Updates
interface ConnectionUpdate {
  type: "connection_update";
  payload: {
    userId: number;
    connectionId: string;
    status: ConnectionStatus;
    timestamp: number;
    metadata?: Record<string, unknown>;
  };
}

// Usage Updates
interface UsageUpdate {
  type: "usage_update";
  payload: {
    userId: number;
    subscriptionId: number;
    bandwidth: {
      used: number;
      remaining: number;
    };
    duration: {
      used: number;
      remaining: number;
    };
    timestamp: number;
  };
}

// System Updates
interface SystemUpdate {
  type: "system_update";
  payload: {
    serverLoad: number;
    activeConnections: number;
    bandwidth: {
      inbound: number;
      outbound: number;
    };
    timestamp: number;
  };
}
```

### Telegram Bot API

#### Commands
```typescript
interface BotCommand {
  command: string;
  description: string;
  role: UserRole[];
  handler: (ctx: Context) => Promise<void>;
}

const commands: BotCommand[] = [
  {
    command: "start",
    description: "Start the bot",
    role: ["*"],
    handler: handleStart
  },
  {
    command: "subscribe",
    description: "Subscribe to a plan",
    role: ["normal", "reseller"],
    handler: handleSubscribe
  },
  {
    command: "status",
    description: "Check subscription status",
    role: ["*"],
    handler: handleStatus
  },
  {
    command: "daily",
    description: "Claim daily reward",
    role: ["normal", "reseller"],
    handler: handleDaily
  }
];
```

#### Callbacks
```typescript
interface CallbackQuery {
  action: string;
  data: Record<string, unknown>;
  messageId: number;
  userId: number;
}

interface CallbackHandler {
  pattern: RegExp;
  handler: (query: CallbackQuery) => Promise<void>;
  role: UserRole[];
}
```

## Development and Deployment

### Development Setup

#### Local Development Environment
```bash
# Clone repository
git clone https://github.com/yourusername/BimarzNet.git
cd BimarzNet

# Install dependencies
deno cache deps.ts

# Copy example configuration
cp config.example.json config.json
cp .env.example .env

# Start development server
deno task dev
```

#### Testing
```bash
# Run all tests
deno test

# Run specific test suite
deno test tests/api/
deno test tests/integration/
deno test tests/unit/

# Run tests with coverage
deno test --coverage
```

#### Code Quality
```bash
# Format code
deno fmt

# Lint code
deno lint

# Type check
deno check src/**/*.ts
```

### Deployment

#### Production Deployment
```bash
# Build production assets
deno task build

# Run database migrations
deno task db:migrate

# Start production server
deno task start
```

#### Docker Deployment
```dockerfile
FROM denoland/deno:1.37.0

WORKDIR /app

# Cache dependencies
COPY deps.ts .
RUN deno cache deps.ts

# Copy source code
COPY . .

# Compile TypeScript
RUN deno cache src/main.ts

# Expose port
EXPOSE 8000

# Start server
CMD ["task", "start"]
```

#### Docker Compose
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=production
      - DB_PATH=/data/bimarz.db
    volumes:
      - ./data:/app/data
    depends_on:
      - redis

  redis:
    image: redis:6.2-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

### Monitoring and Maintenance

#### Health Checks
```typescript
interface HealthCheck {
  service: string;
  status: "up" | "down" | "degraded";
  latency: number;
  lastCheck: Date;
  details?: Record<string, unknown>;
}

async function performHealthCheck(): Promise<HealthCheck[]> {
  return [
    await checkDatabase(),
    await checkRedis(),
    await checkVPNServers(),
    await checkPaymentProviders()
  ];
}
```

#### Backup System
```typescript
interface BackupConfig {
  schedule: string;  // Cron expression
  retention: {
    days: number;
    copies: number;
  };
  storage: {
    local: string;
    remote?: string;
  };
  compression: boolean;
  encryption: boolean;
}

async function performBackup(config: BackupConfig): Promise<void> {
  // Backup database
  await backupDatabase();
  
  // Backup configurations
  await backupConfigs();
  
  // Cleanup old backups
  await cleanupOldBackups();
}
```

### Contributing

#### Getting Started
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

#### Code Style
- Use TypeScript strict mode
- Follow Deno style guide
- Write comprehensive tests
- Document all public APIs
- Use meaningful commit messages

#### Pull Request Process
1. Update documentation
2. Add/update tests
3. Update changelog
4. Request review
5. Address feedback

### Security

#### Security Policies
- Regular security audits
- Dependency scanning
- Code analysis
- Penetration testing
- Vulnerability reporting

#### Reporting Vulnerabilities
- Email: security@bimarz.net
- Bug bounty program
- Responsible disclosure
- Security advisories

### License

MIT License

Copyright (c) 2024 BimarzNet

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use