# LMS DevOps & CI/CD Deployment Automation Guide

## Table of Contents
1. [DevOps Strategy for Educational Platforms](#devops-strategy-for-educational-platforms)
2. [CI/CD Pipeline Architecture](#cicd-pipeline-architecture)
3. [Infrastructure as Code](#infrastructure-as-code)
4. [Automated Testing Integration](#automated-testing-integration)
5. [Deployment Strategies](#deployment-strategies)
6. [Monitoring & Observability](#monitoring--observability)
7. [Rollback & Recovery Procedures](#rollback--recovery-procedures)
8. [Security in DevOps](#security-in-devops)
9. [Educational Platform Considerations](#educational-platform-considerations)
10. [Performance Optimization](#performance-optimization)

## DevOps Strategy for Educational Platforms

### 1. Educational DevOps Requirements

**✅ Good: LMS-Specific DevOps Considerations**
```yaml
# Educational platform DevOps requirements
educational_devops_strategy:
  high_availability:
    requirement: "99.9% uptime during academic periods"
    considerations:
      - "Peak usage during exam periods"
      - "Global student access across time zones"
      - "Critical learning deadlines"
      - "Assessment submission windows"
    
  scalability:
    patterns:
      - "Predictable seasonal traffic (semester starts)"
      - "Sudden spikes during assignment deadlines"
      - "Video streaming peaks during live sessions"
      - "Assessment period load surges"
    
  data_sensitivity:
    compliance_requirements:
      - "FERPA (Family Educational Rights and Privacy Act)"
      - "GDPR for international students"
      - "SOC 2 Type II compliance"
      - "Educational data privacy laws by region"
    
  release_timing:
    considerations:
      - "No deployments during exam periods"
      - "Feature freezes during critical academic periods"
      - "Coordinated releases with academic calendar"
      - "Emergency patches for security issues only"

  stakeholder_communication:
    notification_channels:
      - "Student notification systems"
      - "Faculty communication channels"
      - "IT administrator alerts"
      - "Emergency maintenance procedures"
```

### 2. Development Workflow for Educational Teams

**✅ Good: Academic Development Process**
```javascript
// Educational development workflow configuration
const EDUCATIONAL_DEVOPS_WORKFLOW = {
  BRANCHING_STRATEGY: {
    main: {
      description: 'Production-ready code',
      protection: 'Protected branch with required reviews',
      deployment: 'Automated to production during maintenance windows'
    },
    
    develop: {
      description: 'Integration branch for features',
      protection: 'Automated testing required',
      deployment: 'Continuous deployment to staging'
    },
    
    feature: {
      pattern: 'feature/TICKET-ID-description',
      description: 'Individual feature development',
      requirements: 'Unit tests and documentation',
      review_process: 'Peer review required'
    },
    
    hotfix: {
      pattern: 'hotfix/CRITICAL-ID-description',
      description: 'Emergency fixes for production',
      process: 'Fast-track with abbreviated testing',
      approval: 'Senior developer and platform owner approval'
    },
    
    academic: {
      pattern: 'academic/semester-YYYY-term',
      description: 'Academic period specific features',
      timing: 'Deployed before semester start',
      rollback: 'Must be revertible during academic period'
    }
  },

  RELEASE_SCHEDULE: {
    major_releases: {
      frequency: 'Quarterly, aligned with academic calendar',
      timing: 'Between semesters or during breaks',
      features: 'New educational features and major updates'
    },
    
    minor_releases: {
      frequency: 'Monthly during non-critical periods',
      timing: 'Weekends or low-usage periods',
      features: 'Bug fixes and minor enhancements'
    },
    
    hotfixes: {
      frequency: 'As needed for critical issues',
      timing: 'Emergency deployment any time',
      scope: 'Security fixes and critical bug fixes only'
    }
  },

  ACADEMIC_CALENDAR_INTEGRATION: {
    exam_periods: {
      deployment_policy: 'Feature freeze - hotfixes only',
      monitoring: 'Enhanced monitoring and support coverage',
      rollback_readiness: 'Immediate rollback capability required'
    },
    
    semester_start: {
      preparation: 'Load testing and capacity planning',
      support: '24/7 support coverage',
      monitoring: 'Real-time performance tracking'
    },
    
    break_periods: {
      opportunity: 'Major feature deployments',
      maintenance: 'Infrastructure updates and improvements',
      testing: 'Comprehensive testing with reduced user impact'
    }
  }
};
```

## CI/CD Pipeline Architecture

### 1. Educational Platform Pipeline Design

**✅ Good: Comprehensive CI/CD for LMS**
```yaml
# GitHub Actions CI/CD Pipeline for LMS
name: LMS Educational Platform CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * *'  # Daily security scans

env:
  NODE_VERSION: '18'
  POSTGRES_VERSION: '15'
  REDIS_VERSION: '7'

jobs:
  # Code Quality and Security Scanning
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for SonarQube
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: |
          npm ci
          npm run install:backend
          npm run install:frontend
      
      - name: Run ESLint
        run: npm run lint:ci
      
      - name: Run Prettier check
        run: npm run format:check
      
      - name: TypeScript type checking
        run: npm run type-check
      
      - name: Security audit
        run: |
          npm audit --audit-level=moderate
          npm run security:scan
      
      - name: SAST Security Scanning
        uses: github/super-linter@v4
        env:
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          VALIDATE_JAVASCRIPT_ES: true
          VALIDATE_TYPESCRIPT_ES: true
          VALIDATE_DOCKER: true
      
      - name: SonarQube Scan
        uses: sonarqube-quality-gate-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Educational Content Validation
  content-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Validate Learning Objectives
        run: npm run validate:learning-objectives
      
      - name: Check Educational Compliance
        run: npm run validate:educational-standards
      
      - name: Accessibility Testing
        run: |
          npm run test:accessibility
          npm run lighthouse:accessibility
      
      - name: Content Quality Checks
        run: |
          npm run validate:course-content
          npm run check:broken-links

  # Unit and Integration Testing
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: lms_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    strategy:
      matrix:
        test-suite: [frontend, backend, integration, educational-logic]

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Setup test database
        run: |
          npm run db:migrate:test
          npm run db:seed:test
      
      - name: Run tests - ${{ matrix.test-suite }}
        run: npm run test:${{ matrix.test-suite }} -- --coverage
        env:
          DATABASE_URL: postgresql://postgres:test_password@localhost:5432/lms_test
          REDIS_URL: redis://localhost:6379
          NODE_ENV: test
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
          flags: ${{ matrix.test-suite }}

  # Performance Testing
  performance-testing:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup test environment
        run: docker-compose -f docker-compose.test.yml up -d
      
      - name: Wait for services
        run: |
          sleep 30
          curl --retry 10 --retry-delay 10 http://localhost:3000/health
          curl --retry 10 --retry-delay 10 http://localhost:3001/health
      
      - name: Run performance tests
        run: |
          npm run test:performance
          npm run test:load
      
      - name: Educational load testing
        run: |
          # Simulate concurrent student logins
          npm run test:concurrent-users
          # Test video streaming performance
          npm run test:video-streaming
          # Test assessment submission load
          npm run test:assessment-load
      
      - name: Generate performance report
        run: npm run performance:report
      
      - name: Upload performance artifacts
        uses: actions/upload-artifact@v3
        with:
          name: performance-report
          path: performance-report/

  # Build and Package
  build:
    runs-on: ubuntu-latest
    needs: [code-quality, test, content-validation]
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ vars.CONTAINER_REGISTRY }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ vars.CONTAINER_REGISTRY }}/lms
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix=sha-
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push Frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ steps.meta.outputs.tags }}-frontend
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push Backend
        id: build
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ steps.meta.outputs.tags }}-backend
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Security Scanning
  security-scan:
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image-tag }}-backend
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    if: github.ref == 'refs/heads/develop'
    
    environment:
      name: staging
      url: https://staging.lms.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'v1.28.0'
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Deploy to staging
        run: |
          envsubst < k8s/staging/deployment.yaml | kubectl apply -f -
          kubectl rollout status deployment/lms-backend -n lms-staging
          kubectl rollout status deployment/lms-frontend -n lms-staging
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
      
      - name: Run smoke tests
        run: |
          npm run test:smoke -- --env=staging
          npm run test:e2e:critical -- --env=staging
      
      - name: Notify stakeholders
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#lms-deployments'
          message: 'Staging deployment completed for LMS platform'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Deploy to Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build, security-scan]
    if: github.ref == 'refs/heads/main'
    
    environment:
      name: production
      url: https://lms.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Check deployment window
        run: |
          # Check if current time is within allowed deployment window
          npm run check:deployment-window
          npm run check:academic-calendar
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'v1.28.0'
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG_PROD }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Pre-deployment checks
        run: |
          npm run check:database-migrations
          npm run check:breaking-changes
          npm run validate:production-readiness
      
      - name: Create deployment backup
        run: |
          kubectl create job backup-$(date +%Y%m%d%H%M%S) \
            --from=cronjob/database-backup -n lms-production
      
      - name: Blue-Green Deployment
        run: |
          # Deploy to green environment
          envsubst < k8s/production/deployment-green.yaml | kubectl apply -f -
          kubectl rollout status deployment/lms-backend-green -n lms-production
          kubectl rollout status deployment/lms-frontend-green -n lms-production
          
          # Run health checks
          npm run test:health-check -- --env=green
          
          # Switch traffic to green
          kubectl patch service lms-service -p '{"spec":{"selector":{"version":"green"}}}' -n lms-production
          
          # Monitor for 5 minutes
          sleep 300
          npm run monitor:error-rate -- --duration=300
          
          # Clean up blue environment
          kubectl delete deployment lms-backend-blue lms-frontend-blue -n lms-production
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
      
      - name: Post-deployment validation
        run: |
          npm run test:smoke -- --env=production
          npm run test:critical-paths -- --env=production
          npm run validate:learning-analytics
      
      - name: Update monitoring dashboards
        run: |
          npm run update:grafana-dashboards
          npm run update:alerting-rules
      
      - name: Notify success
        uses: 8398a7/action-slack@v3
        with:
          status: success
          channel: '#lms-deployments'
          message: |
            🚀 Production deployment successful!
            Image: ${{ needs.build.outputs.image-tag }}
            Commit: ${{ github.sha }}
            Author: ${{ github.actor }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Rollback on failure
  rollback:
    runs-on: ubuntu-latest
    if: failure() && github.ref == 'refs/heads/main'
    needs: deploy-production
    
    steps:
      - name: Emergency rollback
        run: |
          echo "${{ secrets.KUBE_CONFIG_PROD }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
          
          # Get previous successful deployment
          PREVIOUS_TAG=$(kubectl get deployment lms-backend -o jsonpath='{.metadata.annotations.previous-image}' -n lms-production)
          
          # Rollback to previous version
          kubectl set image deployment/lms-backend backend=$PREVIOUS_TAG -n lms-production
          kubectl set image deployment/lms-frontend frontend=$PREVIOUS_TAG -n lms-production
          
          # Wait for rollback completion
          kubectl rollout status deployment/lms-backend -n lms-production
          kubectl rollout status deployment/lms-frontend -n lms-production
      
      - name: Notify rollback
        uses: 8398a7/action-slack@v3
        with:
          status: custom
          custom_payload: |
            {
              "attachments": [{
                "color": "danger",
                "fields": [{
                  "title": "🚨 Emergency Rollback Executed",
                  "value": "Production deployment failed and has been rolled back",
                  "short": false
                }]
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### 2. Infrastructure as Code with Terraform

**✅ Good: Educational Platform Infrastructure**
```hcl
# terraform/main.tf - LMS Infrastructure
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
  }
  
  backend "s3" {
    bucket = "lms-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-west-2"
  }
}

# Variables for educational platform
variable "environment" {
  description = "Environment name (staging, production)"
  type        = string
}

variable "academic_year" {
  description = "Academic year for resource tagging"
  type        = string
  default     = "2024-2025"
}

variable "peak_students" {
  description = "Expected peak concurrent students"
  type        = number
  default     = 10000
}

# EKS Cluster for educational workloads
module "eks" {
  source = "./modules/eks"
  
  cluster_name    = "lms-${var.environment}"
  cluster_version = "1.28"
  
  # Node groups sized for educational load
  node_groups = {
    educational_workloads = {
      instance_types = ["t3.large", "t3.xlarge"]
      min_size       = 3
      max_size       = 20
      desired_size   = 5
      
      labels = {
        workload-type = "educational"
        environment   = var.environment
      }
      
      taints = [{
        key    = "educational-workload"
        value  = "true"
        effect = "NO_SCHEDULE"
      }]
    }
    
    video_streaming = {
      instance_types = ["c5.large", "c5.xlarge"]
      min_size       = 2
      max_size       = 10
      desired_size   = 3
      
      labels = {
        workload-type = "streaming"
        environment   = var.environment
      }
    }
  }
  
  tags = {
    Environment    = var.environment
    AcademicYear   = var.academic_year
    Purpose        = "Educational Platform"
    Compliance     = "FERPA"
  }
}

# RDS PostgreSQL for educational data
resource "aws_db_instance" "educational_db" {
  identifier = "lms-${var.environment}-db"
  
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = var.environment == "production" ? "db.r5.xlarge" : "db.t3.medium"
  
  allocated_storage     = var.environment == "production" ? 500 : 100
  max_allocated_storage = var.environment == "production" ? 1000 : 200
  storage_encrypted     = true
  
  db_name  = "lms_${var.environment}"
  username = "lms_admin"
  password = random_password.db_password.result
  
  # Educational data backup requirements
  backup_retention_period = var.environment == "production" ? 30 : 7
  backup_window          = "03:00-04:00"  # During low usage hours
  maintenance_window     = "sun:04:00-sun:05:00"
  
  # High availability for production
  multi_az = var.environment == "production"
  
  # Security for educational data
  vpc_security_group_ids = [aws_security_group.educational_db.id]
  db_subnet_group_name   = aws_db_subnet_group.educational.name
  
  # Compliance and monitoring
  enabled_cloudwatch_logs_exports = ["postgresql"]
  monitoring_interval             = 60
  monitoring_role_arn            = aws_iam_role.rds_monitoring.arn
  
  # Educational data protection
  deletion_protection = var.environment == "production"
  skip_final_snapshot = var.environment != "production"
  final_snapshot_identifier = var.environment == "production" ? "lms-final-snapshot-${formatdate("YYYY-MM-DD-hhmm", timestamp())}" : null
  
  tags = {
    Name           = "LMS Educational Database"
    Environment    = var.environment
    DataType       = "Educational Records"
    Compliance     = "FERPA, GDPR"
    BackupRequired = "true"
  }
}

# ElastiCache Redis for session management
resource "aws_elasticache_replication_group" "educational_cache" {
  replication_group_id         = "lms-${var.environment}-cache"
  description                  = "Redis cache for LMS sessions and content"
  
  node_type            = var.environment == "production" ? "cache.r6g.large" : "cache.t3.micro"
  port                 = 6379
  parameter_group_name = "default.redis7"
  
  num_cache_clusters = var.environment == "production" ? 3 : 1
  
  # Educational session requirements
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token                 = random_password.redis_auth.result
  
  # High availability for educational continuity
  automatic_failover_enabled = var.environment == "production"
  multi_az_enabled          = var.environment == "production"
  
  # Backup for session recovery
  snapshot_retention_limit = 5
  snapshot_window         = "03:00-05:00"
  
  subnet_group_name = aws_elasticache_subnet_group.educational.name
  security_group_ids = [aws_security_group.educational_cache.id]
  
  tags = {
    Name        = "LMS Educational Cache"
    Environment = var.environment
    Purpose     = "Session Management"
  }
}

# S3 buckets for educational content
resource "aws_s3_bucket" "educational_content" {
  bucket = "lms-${var.environment}-educational-content-${random_id.bucket_suffix.hex}"
  
  tags = {
    Name        = "Educational Content Storage"
    Environment = var.environment
    ContentType = "Course Materials, Videos, Documents"
    Compliance  = "FERPA"
  }
}

resource "aws_s3_bucket_versioning" "educational_content" {
  bucket = aws_s3_bucket.educational_content.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_encryption" "educational_content" {
  bucket = aws_s3_bucket.educational_content.id
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

# CloudWatch monitoring for educational metrics
resource "aws_cloudwatch_dashboard" "educational_platform" {
  dashboard_name = "LMS-Educational-Platform-${var.environment}"
  
  dashboard_body = jsonencode({
    widgets = [
      {
        type   = "metric"
        width  = 12
        height = 6
        
        properties = {
          metrics = [
            ["AWS/EKS", "cluster_active_nodes", "ClusterName", module.eks.cluster_name],
            ["AWS/ApplicationELB", "TargetResponseTime", "LoadBalancer", aws_lb.educational_alb.arn_suffix],
            ["AWS/RDS", "DatabaseConnections", "DBInstanceIdentifier", aws_db_instance.educational_db.id],
            ["AWS/ElastiCache", "CacheHits", "CacheClusterId", aws_elasticache_replication_group.educational_cache.id]
          ]
          period = 300
          stat   = "Average"
          region = data.aws_region.current.name
          title  = "Educational Platform Infrastructure Metrics"
        }
      },
      {
        type   = "log"
        width  = 12
        height = 6
        
        properties = {
          query   = "SOURCE '/aws/eks/lms-${var.environment}/cluster' | fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 100"
          region  = data.aws_region.current.name
          title   = "Educational Platform Error Logs"
        }
      }
    ]
  })
}

# Auto Scaling for educational load patterns
resource "aws_autoscaling_policy" "educational_scale_up" {
  name                   = "lms-${var.environment}-scale-up"
  scaling_adjustment     = 2
  adjustment_type        = "ChangeInCapacity"
  cooldown              = 300
  autoscaling_group_name = module.eks.node_groups["educational_workloads"].asg_name
}

resource "aws_cloudwatch_metric_alarm" "high_student_load" {
  alarm_name          = "lms-${var.environment}-high-student-load"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = "120"
  statistic           = "Average"
  threshold           = "70"
  alarm_description   = "This metric monitors EC2 cpu utilization during high student load"
  alarm_actions       = [aws_autoscaling_policy.educational_scale_up.arn]
  
  dimensions = {
    AutoScalingGroupName = module.eks.node_groups["educational_workloads"].asg_name
  }
  
  tags = {
    Environment = var.environment
    Purpose     = "Educational Load Management"
  }
}

# Outputs for application configuration
output "database_endpoint" {
  description = "RDS PostgreSQL endpoint for educational data"
  value       = aws_db_instance.educational_db.endpoint
  sensitive   = true
}

output "redis_endpoint" {
  description = "ElastiCache Redis endpoint for sessions"
  value       = aws_elasticache_replication_group.educational_cache.primary_endpoint_address
  sensitive   = true
}

output "content_bucket" {
  description = "S3 bucket for educational content"
  value       = aws_s3_bucket.educational_content.bucket
}

output "eks_cluster_endpoint" {
  description = "EKS cluster endpoint for educational workloads"
  value       = module.eks.cluster_endpoint
  sensitive   = true
}
```

This comprehensive DevOps and CI/CD guide provides specific automation strategies for educational platforms, ensuring reliable deployments, monitoring, and maintenance while considering the unique requirements of learning management systems. 