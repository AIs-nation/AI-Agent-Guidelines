# LMS Production Deployment Guide

## Table of Contents
1. [Production Deployment Overview](#production-deployment-overview)
2. [Containerization with Docker](#containerization-with-docker)
3. [Kubernetes Orchestration](#kubernetes-orchestration)
4. [CI/CD Pipeline Implementation](#cicd-pipeline-implementation)
5. [Cloud Infrastructure Setup](#cloud-infrastructure-setup)
6. [Database Deployment & Scaling](#database-deployment--scaling)
7. [Monitoring & Observability](#monitoring--observability)
8. [Security & Compliance](#security--compliance)
9. [Performance & Scalability](#performance--scalability)
10. [Disaster Recovery](#disaster-recovery)

## Production Deployment Overview

### 1. LMS Deployment Architecture

**✅ Good: Comprehensive LMS Production Architecture**
```yaml
# Production LMS architecture overview
production_architecture:
  frontend:
    technology: "Next.js React Application"
    deployment_strategy: "Static site generation with API routes"
    hosting: "CDN + Edge computing"
    scaling: "Horizontal auto-scaling"
    
  backend:
    technology: "Node.js Express API"
    deployment_strategy: "Microservices architecture"
    hosting: "Container orchestration"
    scaling: "Auto-scaling based on load"
    
  database:
    primary: "PostgreSQL with Prisma ORM"
    caching: "Redis for session and content caching"
    analytics: "ClickHouse for learning analytics"
    file_storage: "AWS S3 / Google Cloud Storage"
    
  infrastructure:
    container_runtime: "Docker containers"
    orchestration: "Kubernetes (EKS/GKE/AKS)"
    service_mesh: "Istio for inter-service communication"
    ingress: "NGINX Ingress Controller"
    
  monitoring:
    application: "New Relic / DataDog"
    infrastructure: "Prometheus + Grafana"
    logging: "ELK Stack (Elasticsearch, Logstash, Kibana)"
    error_tracking: "Sentry"
    
  security:
    secrets_management: "HashiCorp Vault / AWS Secrets Manager"
    certificate_management: "Let's Encrypt with cert-manager"
    network_security: "VPC with security groups"
    compliance: "SOC 2, FERPA, GDPR ready"
```

### 2. Deployment Environment Strategy

**✅ Good: Multi-Environment Deployment Pipeline**
```javascript
// Environment configuration for LMS deployment
const DEPLOYMENT_ENVIRONMENTS = {
  DEVELOPMENT: {
    purpose: 'Feature development and initial testing',
    infrastructure: {
      compute: 'Local Docker containers or lightweight cloud instances',
      database: 'PostgreSQL container with test data',
      storage: 'Local file system or dev bucket',
      monitoring: 'Basic logging and health checks'
    },
    deployment_frequency: 'Continuous (on every commit)',
    data_sources: 'Synthetic test data and anonymized production samples',
    access_control: 'Developer access only'
  },

  STAGING: {
    purpose: 'Integration testing and UAT',
    infrastructure: {
      compute: 'Production-like Kubernetes cluster (smaller scale)',
      database: 'PostgreSQL with production-like data volume',
      storage: 'Cloud storage with staging data',
      monitoring: 'Full monitoring stack with alerting'
    },
    deployment_frequency: 'Daily or per feature completion',
    data_sources: 'Anonymized production data subset',
    access_control: 'Developers, QA, and stakeholders'
  },

  PRODUCTION: {
    purpose: 'Live educational platform serving real users',
    infrastructure: {
      compute: 'Multi-zone Kubernetes cluster with auto-scaling',
      database: 'High-availability PostgreSQL with read replicas',
      storage: 'Redundant cloud storage with CDN',
      monitoring: 'Comprehensive monitoring with 24/7 alerting'
    },
    deployment_frequency: 'Scheduled releases with blue-green deployment',
    data_sources: 'Live educational data with real-time analytics',
    access_control: 'Limited production access with audit logging'
  }
};

// Environment-specific configuration management
class EnvironmentConfig {
  constructor(environment) {
    this.env = environment;
    this.config = this.loadEnvironmentConfig(environment);
  }

  loadEnvironmentConfig(env) {
    const configs = {
      development: {
        DATABASE_URL: process.env.DEV_DATABASE_URL,
        REDIS_URL: process.env.DEV_REDIS_URL,
        API_BASE_URL: 'http://localhost:3001',
        FRONTEND_URL: 'http://localhost:3000',
        LOG_LEVEL: 'debug',
        RATE_LIMIT: false,
        ANALYTICS_ENABLED: false
      },
      
      staging: {
        DATABASE_URL: process.env.STAGING_DATABASE_URL,
        REDIS_URL: process.env.STAGING_REDIS_URL,
        API_BASE_URL: 'https://api-staging.lms.example.com',
        FRONTEND_URL: 'https://staging.lms.example.com',
        LOG_LEVEL: 'info',
        RATE_LIMIT: true,
        ANALYTICS_ENABLED: true
      },
      
      production: {
        DATABASE_URL: process.env.PROD_DATABASE_URL,
        REDIS_URL: process.env.PROD_REDIS_URL,
        API_BASE_URL: 'https://api.lms.example.com',
        FRONTEND_URL: 'https://lms.example.com',
        LOG_LEVEL: 'warn',
        RATE_LIMIT: true,
        ANALYTICS_ENABLED: true,
        MONITORING_ENABLED: true
      }
    };

    return configs[env] || configs.development;
  }

  validateConfig() {
    const requiredVars = [
      'DATABASE_URL', 'REDIS_URL', 'API_BASE_URL', 'FRONTEND_URL'
    ];

    const missing = requiredVars.filter(key => !this.config[key]);
    
    if (missing.length > 0) {
      throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
    }

    return true;
  }
}
```

## Containerization with Docker

### 1. Docker Configuration for LMS Components

**✅ Good: Optimized Docker Setup for Educational Platform**
```dockerfile
# Frontend Dockerfile (Next.js React application)
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build the application
ENV NEXT_TELEMETRY_DISABLED 1
RUN yarn build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy public assets
COPY --from=builder /app/public ./public

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]

# Backend Dockerfile (Node.js Express API)
FROM node:18-alpine AS backend-base

WORKDIR /app

# Install system dependencies
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    && ln -sf python3 /usr/bin/python

# Copy package files
COPY package*.json ./
COPY prisma ./prisma/

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Generate Prisma client
RUN npx prisma generate

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership
RUN chown -R nodejs:nodejs /app
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

EXPOSE 3001

CMD ["npm", "start"]
```

### 2. Docker Compose for Development

**✅ Good: Complete Development Environment**
```yaml
# docker-compose.yml - Development environment
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    container_name: lms-postgres
    environment:
      POSTGRES_DB: lms_development
      POSTGRES_USER: lms_user
      POSTGRES_PASSWORD: lms_password
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U lms_user -d lms_development"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - lms-network

  # Redis for caching and sessions
  redis:
    image: redis:7-alpine
    container_name: lms-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - lms-network

  # Backend API service
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: backend-base
    container_name: lms-backend
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://lms_user:lms_password@postgres:5432/lms_development
      REDIS_URL: redis://redis:6379
      JWT_SECRET: development-jwt-secret
      CLAUDE_API_KEY: ${CLAUDE_API_KEY}
      OPENAI_API_KEY: ${OPENAI_API_KEY}
    ports:
      - "3001:3001"
    volumes:
      - ./backend:/app
      - /app/node_modules
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - lms-network
    restart: unless-stopped

  # Frontend React application
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: base
    container_name: lms-frontend
    environment:
      NODE_ENV: development
      NEXT_PUBLIC_API_URL: http://localhost:3001
      NEXT_PUBLIC_WS_URL: ws://localhost:3001
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - /app/.next
    depends_on:
      - backend
    networks:
      - lms-network
    restart: unless-stopped

  # NGINX reverse proxy
  nginx:
    image: nginx:alpine
    container_name: lms-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend
    networks:
      - lms-network
    restart: unless-stopped

  # Monitoring and observability
  prometheus:
    image: prom/prometheus:latest
    container_name: lms-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    networks:
      - lms-network

  grafana:
    image: grafana/grafana:latest
    container_name: lms-grafana
    ports:
      - "3003:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./monitoring/grafana/datasources:/etc/grafana/provisioning/datasources:ro
    depends_on:
      - prometheus
    networks:
      - lms-network

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  lms-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

## Kubernetes Orchestration

### 1. Kubernetes Manifests for LMS

**✅ Good: Production-Ready Kubernetes Configuration**
```yaml
# namespace.yaml - LMS namespace
apiVersion: v1
kind: Namespace
metadata:
  name: lms-production
  labels:
    name: lms-production
    environment: production

---
# configmap.yaml - Application configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: lms-config
  namespace: lms-production
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"
  RATE_LIMIT_ENABLED: "true"
  ANALYTICS_ENABLED: "true"
  FRONTEND_URL: "https://lms.example.com"
  API_BASE_URL: "https://api.lms.example.com"

---
# secret.yaml - Sensitive configuration
apiVersion: v1
kind: Secret
metadata:
  name: lms-secrets
  namespace: lms-production
type: Opaque
data:
  database-url: <base64-encoded-database-url>
  redis-url: <base64-encoded-redis-url>
  jwt-secret: <base64-encoded-jwt-secret>
  claude-api-key: <base64-encoded-claude-key>
  openai-api-key: <base64-encoded-openai-key>

---
# backend-deployment.yaml - Backend API deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lms-backend
  namespace: lms-production
  labels:
    app: lms-backend
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: lms-backend
  template:
    metadata:
      labels:
        app: lms-backend
        version: v1
    spec:
      containers:
      - name: backend
        image: lms/backend:latest
        ports:
        - containerPort: 3001
          name: http
        env:
        - name: NODE_ENV
          valueFrom:
            configMapKeyRef:
              name: lms-config
              key: NODE_ENV
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: lms-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: lms-secrets
              key: redis-url
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: lms-secrets
              key: jwt-secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3001
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
      terminationGracePeriodSeconds: 30

---
# backend-service.yaml - Backend service
apiVersion: v1
kind: Service
metadata:
  name: lms-backend-service
  namespace: lms-production
  labels:
    app: lms-backend
spec:
  selector:
    app: lms-backend
  ports:
  - port: 80
    targetPort: 3001
    protocol: TCP
    name: http
  type: ClusterIP

---
# frontend-deployment.yaml - Frontend deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lms-frontend
  namespace: lms-production
  labels:
    app: lms-frontend
    version: v1
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: lms-frontend
  template:
    metadata:
      labels:
        app: lms-frontend
        version: v1
    spec:
      containers:
      - name: frontend
        image: lms/frontend:latest
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NEXT_PUBLIC_API_URL
          valueFrom:
            configMapKeyRef:
              name: lms-config
              key: API_BASE_URL
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /api/health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
# frontend-service.yaml - Frontend service  
apiVersion: v1
kind: Service
metadata:
  name: lms-frontend-service
  namespace: lms-production
  labels:
    app: lms-frontend
spec:
  selector:
    app: lms-frontend
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
    name: http
  type: ClusterIP

---
# hpa.yaml - Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: lms-backend-hpa
  namespace: lms-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: lms-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max

---
# ingress.yaml - Ingress configuration
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lms-ingress
  namespace: lms-production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
spec:
  tls:
  - hosts:
    - lms.example.com
    - api.lms.example.com
    secretName: lms-tls-secret
  rules:
  - host: lms.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lms-frontend-service
            port:
              number: 80
  - host: api.lms.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lms-backend-service
            port:
              number: 80
```

### 2. Database Deployment in Kubernetes

**✅ Good: High-Availability Database Setup**
```yaml
# postgres-statefulset.yaml - PostgreSQL StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: lms-production
spec:
  serviceName: postgres-service
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: lms_production
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        ports:
        - containerPort: 5432
          name: postgres
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        - name: postgres-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - $(POSTGRES_USER)
            - -d
            - $(POSTGRES_DB)
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - $(POSTGRES_USER)
            - -d
            - $(POSTGRES_DB)
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: postgres-config
        configMap:
          name: postgres-config
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 100Gi

---
# postgres-service.yaml - PostgreSQL service
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: lms-production
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  clusterIP: None  # Headless service for StatefulSet

---
# redis-deployment.yaml - Redis deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: lms-production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-storage
          mountPath: /data
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
        livenessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - redis-cli
            - ping
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: redis-storage
        persistentVolumeClaim:
          claimName: redis-pvc

---
# redis-pvc.yaml - Redis persistent volume claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: lms-production
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi
```

This comprehensive deployment guide provides production-ready configurations for deploying an LMS platform with proper containerization, orchestration, monitoring, and scalability considerations. 