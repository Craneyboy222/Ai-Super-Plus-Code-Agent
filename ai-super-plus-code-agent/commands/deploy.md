---
name: /deploy
description: >
  Generate deployment configuration. Dockerfile, CI/CD pipeline, environment configs,
  monitoring setup, health checks, rollback plan. Produces complete deployment infrastructure
  ready for production.
---

# Deploy Command

## Purpose
Generate production-ready deployment configuration with containerization, CI/CD, and infrastructure setup.

## When to Use
- First deployment of a project
- Setting up production environment
- Migrating to new infrastructure
- Implementing CI/CD pipeline
- Containerizing application
- Setting up monitoring and alerting

## Execution Steps

### 1. Dockerfile Generation
- **Base Image**: Appropriate base image (node:18-alpine, python:3.10, etc)
- **Build Stage**: Multi-stage build with dependencies
- **Runtime Stage**: Minimal runtime image
- **User Setup**: Non-root user for security
- **Workdir**: Application working directory
- **Dependencies**: Copy and install dependencies
- **Build**: Compile/build application
- **Expose Port**: Expose application port
- **Health Check**: HEALTHCHECK instruction
- **Entrypoint**: Command to start application
- **Optimize**: Minimize layer size and cache misses

### 2. Docker Compose Configuration
- **Services**: Application, database, cache, other services
- **Networking**: Service discovery and communication
- **Volumes**: Persistent data storage
- **Environment**: Environment variables
- **Ports**: Port mapping for local development
- **Dependencies**: Service startup order
- **Health Checks**: Service health verification
- **Logging**: Log configuration and drivers

### 3. Kubernetes Manifests
- **Deployment**: Pod replicas, rolling updates, resource limits
- **Service**: Internal and external service exposure
- **ConfigMap**: Non-secret configuration
- **Secrets**: Encrypted sensitive data
- **Ingress**: HTTP routing and TLS
- **HPA**: Horizontal pod autoscaling
- **PDB**: Pod disruption budgets for availability
- **Network Policies**: Network access control
- **RBAC**: Service account and role bindings

### 4. CI/CD Pipeline Setup

**GitHub Actions**
- **Build Job**: Lint, test, build steps
- **Publish Job**: Docker image build and push
- **Deploy Job**: Deployment to staging/production
- **Rollback Job**: Automatic rollback on failure
- **Secrets**: Secure credential management
- **Notifications**: Slack/email notifications
- **Caching**: Dependency caching for speed
- **Artifacts**: Build artifact storage

**GitLab CI**
- **.gitlab-ci.yml**: Pipeline stages and jobs
- **Build Stage**: Lint, test, build
- **Test Stage**: Automated testing
- **Deploy Stage**: Staging and production deployment
- **Variables**: CI/CD variable management
- **Runners**: Runner configuration
- **Artifacts**: Build and test artifacts
- **Cache**: Dependency caching

### 5. Infrastructure as Code

**Terraform**
- **Provider Config**: AWS/GCP/Azure configuration
- **VPC**: Virtual network setup
- **Load Balancer**: Load balancing configuration
- **Database**: RDS/Cloud SQL setup
- **Cache**: ElastiCache/Memorystore setup
- **Storage**: S3/Cloud Storage setup
- **Monitoring**: CloudWatch/Stackdriver setup
- **Security Groups**: Firewall rules
- **Auto Scaling**: Scaling policies
- **Modules**: Reusable infrastructure modules

**Helm Charts**
- **Chart Structure**: Chart.yaml, values.yaml, templates
- **Deployment Template**: Kubernetes deployment manifest
- **Service Template**: Kubernetes service manifest
- **ConfigMap**: Configuration management
- **Values**: Default configuration values
- **Dependencies**: Chart dependencies
- **Hooks**: Deployment hooks (pre-install, post-deploy)

### 6. Environment Configuration
- **.env.example**: Template with all variables
- **.env.production**: Production configuration
- **.env.staging**: Staging configuration
- **.env.local**: Local development (git ignored)
- **Secrets Management**: Environment variables for secrets
- **Config Override**: Environment-specific overrides
- **Validation**: Configuration validation on startup

### 7. Monitoring & Alerting Setup
- **Prometheus Config**: Metrics scraping
- **Grafana Dashboards**: Visualization dashboards
- **Alert Rules**: Threshold-based alerts
- **Alert Routing**: Route alerts to on-call
- **Slack Integration**: Slack notifications
- **PagerDuty**: On-call escalation
- **Custom Metrics**: Application-specific metrics
- **Dashboard**: Real-time status dashboard

### 8. Logging Configuration
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Log Format**: Structured JSON logging
- **Log Aggregation**: Centralized log collection
- **Log Retention**: 30-day minimum retention
- **Index Templates**: Elasticsearch templates
- **Kibana Dashboards**: Log visualization
- **Alerting**: Alert on error patterns
- **Archival**: Long-term log storage

### 9. Health Checks & Readiness
- **Liveness Probe**: Container restart on failure
- **Readiness Probe**: Pod ready for traffic
- **Health Endpoint**: /health endpoint implementation
- **Database Check**: Database connectivity check
- **Cache Check**: Cache connectivity check
- **External Services**: External API health
- **Graceful Shutdown**: SIGTERM handling
- **Drain Period**: Pod graceful termination

### 10. Rollback Strategy
- **Blue-Green Deployment**: Two production versions
- **Canary Deployment**: Gradual rollout to subset
- **Rollback Procedure**: Steps to rollback
- **Database Rollback**: Migration rollback procedures
- **Version Pinning**: Lock previous working version
- **Health Checks**: Verification of successful deployment
- **Automated Rollback**: Automatic rollback on failure
- **Manual Rollback**: Runbook for manual rollback

### 11. Backup & Disaster Recovery
- **Automated Backups**: Daily backup schedule
- **Backup Verification**: Regular restore testing
- **Retention Policy**: 30+ day retention minimum
- **Encryption**: Encrypted backups
- **Geographic**: Multi-region backup storage
- **Recovery Time**: <1 hour RTO target
- **Recovery Point**: <15 min RPO target
- **Runbook**: Recovery procedures documented

### 12. Documentation
- **Deployment Guide**: Step-by-step deployment
- **Runbook**: Production runbook
- **Troubleshooting**: Common issues and fixes
- **Architecture Diagram**: Deployment architecture
- **Infrastructure Diagram**: Cloud architecture
- **Monitoring Guide**: Dashboard interpretation
- **Incident Response**: Incident handling procedures
- **Maintenance**: Regular maintenance tasks

## Quality Criteria

- Docker image builds without errors
- Image scans clean for vulnerabilities
- Kubernetes manifests validate successfully
- CI/CD pipeline tests pass
- Application starts in <30 seconds
- Health checks pass consistently
- Monitoring shows all metrics
- Alerts fire for test conditions
- Rollback completes in <5 minutes
- Backups verify successfully
- TLS configured correctly
- No hardcoded secrets

## Output Expectations

```
/deploy
  /docker
    Dockerfile
    .dockerignore
    docker-compose.yml
  /kubernetes
    /base
      deployment.yaml
      service.yaml
      configmap.yaml
    /overlays
      /staging
        kustomization.yaml
      /production
        kustomization.yaml
  /terraform
    main.tf
    variables.tf
    outputs.tf
    vpc.tf
    rds.tf
    eks.tf
  /.github/workflows
    ci.yml
    cd.yml
  /.gitlab-ci.yml
  /helm
    Chart.yaml
    values.yaml
    values-staging.yaml
    values-production.yaml
    templates/
  /monitoring
    prometheus.yml
    grafana-dashboards.json
    alert-rules.yaml
  /logging
    logstash.conf
    kibana-dashboards.json
  .env.example
  /docs
    deployment.md
    troubleshooting.md
    runbook.md
```

## Success Indicators

- Docker image builds in <2 minutes
- Kubernetes manifests deploy successfully
- Application ready within 30 seconds
- Health checks pass
- Monitoring shows metrics
- CI/CD pipeline runs automatically
- Rollback completes in <5 minutes
- Backups work and verify
- Logs centralized and searchable
- TLS working on all endpoints
- No manual deployment steps
