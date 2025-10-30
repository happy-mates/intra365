# Intra365 Architecture

## High-Level System Architecture

The intra365 integration framework is built on a distributed, event-driven architecture using NATS as the central communication backbone.

## Core Components

### 1. Security & Token Service (STS)

**Purpose**: Central authentication and authorization hub that issues tokens for Mate-to-Mate communication

**Capabilities**:
- Issues JWT tokens for authenticated Mates
- Integrates with external identity providers:
  - Microsoft Entra ID (Azure AD)
  - GitHub OAuth
  - Google Workspace
  - Custom OIDC providers
- Token validation and refresh
- Role-based access control (RBAC) for Mate capabilities
- API key management for service-to-service authentication

**Technology**:
- NATS authentication using NATS Auth (NKeys, JWT)
- Integration adapters for external IdPs
- Token store in JetStream for revocation and audit

### 2. NATS Infrastructure

**Purpose**: Central nervous system for all Mate-to-Mate communication

**Components**:

#### NATS Core
- Pub/Sub messaging between Mates
- Request/Reply patterns for synchronous communication
- Subject-based routing (e.g., `intra365.mate.{mate-id}.{action}`)
- Multi-tenancy support through account isolation

#### NATS JetStream
- Persistent message storage
- Event sourcing for system state
- Message replay capabilities
- Stream processors for data transformation
- Key-Value store for configuration and state
- Object store for large payloads

#### NATS Service Mesh
- Service discovery and registration
- Load balancing across Mate instances
- Health checking
- Circuit breaking and retry logic

### 3. Consent Management & Policy System

**Purpose**: Manage user consent and data sharing policies across Mates

**Capabilities**:
- User consent collection and storage
- Granular permission management per Mate
- Data access policy enforcement
- Consent revocation and updates
- Compliance with GDPR, CCPA, and other regulations
- Consent audit trail

**Features**:
- Purpose-based consent (e.g., analytics, personalization, data sharing)
- Consent scopes per Mate and capability
- Time-bound consent with expiration
- Consent delegation and inheritance
- User preference dashboard
- Automated consent renewal workflows

**Policy Engine**:
- Attribute-based access control (ABAC)
- Policy decision point (PDP) for real-time evaluation
- Policy enforcement point (PEP) at Gateway and Mate level
- Policy templates for common scenarios
- Conflict resolution for overlapping policies

**Storage**:
- PostgreSQL database for all persistent data
  - User accounts and profiles
  - Consent records with full history
  - Policy definitions and versions
  - Audit logs and compliance data
- JetStream KV for active consent cache (read performance)
- Event stream for consent change events (real-time notifications)
- Encrypted storage for sensitive consent data

**Data Model**:
```json
{
  "consentId": "uuid",
  "userId": "user-id",
  "mateId": "mate-id",
  "purposes": ["analytics", "personalization"],
  "scopes": ["profile.read", "messages.send"],
  "granted": true,
  "grantedAt": "2025-10-29T10:00:00Z",
  "expiresAt": "2026-10-29T10:00:00Z",
  "revokedAt": null,
  "metadata": {
    "ipAddress": "192.168.1.1",
    "userAgent": "Mozilla/5.0...",
    "consentMethod": "explicit"
  }
}
```

**Policy Model**:
```json
{
  "policyId": "uuid",
  "name": "Healthcare Data Sharing Policy",
  "type": "data-access",
  "rules": [
    {
      "effect": "allow",
      "subjects": ["mate:healthcare-ai"],
      "resources": ["user:*/health-records"],
      "actions": ["read"],
      "conditions": {
        "consent.granted": true,
        "consent.purpose": "medical-analysis",
        "user.country": "US"
      }
    }
  ],
  "priority": 100,
  "active": true
}
```

### 4. Mate Registry & Discovery Service

**Purpose**: Central catalog of all available Mates and their capabilities

**Features**:
- Mate registration with capability metadata
- Service discovery API
- Capability matching and recommendation
- Version management
- Dependency tracking
- Required consent and data access declarations

**Storage**:
- JetStream KV for real-time registry
- Event stream for registration history
- PostgreSQL for Mate metadata and historical data

**Data Model**:
```json
{
  "mateId": "uuid",
  "name": "My AI Assistant",
  "type": "ai-agent",
  "capabilities": ["text-generation", "image-analysis"],
  "version": "1.0.0",
  "endpoints": {
    "nats.subjects": ["intra365.mate.ai-assistant.*"]
  },
  "authentication": {
    "required": true,
    "methods": ["jwt", "api-key"]
  },
  "dataAccess": {
    "requiredScopes": ["profile.read", "messages.send"],
    "purposes": ["ai-assistance", "analytics"],
    "dataRetention": "90-days"
  },
  "status": "active"
}
```

### 5. Observability Stack

**Purpose**: Monitor, trace, and analyze all Mate interactions

**Components**:

#### Metrics Collection
- NATS monitoring endpoints
- Prometheus exporters for:
  - Message throughput
  - Token issuance/validation rates
  - Mate health status
  - JetStream storage metrics
- Custom business metrics from Mates

#### Distributed Tracing
- OpenTelemetry integration
- Trace context propagation via NATS headers
- Jaeger or Tempo backend
- End-to-end request tracing across Mates

#### Logging
- Centralized log aggregation (Loki, Elasticsearch)
- Structured logging with correlation IDs
- NATS server logs
- Mate application logs
- Security audit logs

#### Dashboards & Alerting
- Grafana dashboards for system health
- Alert rules for anomalies
- SLA monitoring
- Cost tracking

### 5. Event Processing Layer

**Purpose**: Handle complex event patterns and data orchestration

**Features**:
- Event filtering and routing
- Data transformation pipelines
- Aggregation and enrichment
- Scheduled jobs and workflows
- Dead letter queues

**Implementation**:
- JetStream consumers with filtering
- Stream processors
- NATS request/reply for transformations

### 6. Gateway & API Layer

**Purpose**: External access point for Mates and clients

**Features**:
- REST API for management operations
- WebSocket support for browser-based Mates
- GraphQL API for flexible queries
- Rate limiting and quotas
- API documentation (OpenAPI/Swagger)

**Technology**:
- NATS as backend
- JWT validation via STS
- Request proxying to appropriate Mates

### 7. PostgreSQL Database

**Purpose**: Central persistent storage for all system data

**Schema Design**:

#### Users & Authentication
- `users` - User accounts and profiles
- `user_identities` - External identity provider mappings (Entra ID, GitHub, etc.)
- `user_sessions` - Active user sessions
- `api_keys` - Service account API keys

#### Consent Management
- `consent_records` - User consent grants and revocations
- `consent_history` - Complete audit trail of all consent changes
- `consent_purposes` - Defined purposes for data usage
- `consent_scopes` - Available permission scopes

#### Policy Management
- `policies` - Policy definitions and rules
- `policy_versions` - Version history of policies
- `policy_evaluations` - Cached policy evaluation results

#### Mate Registry
- `mates` - Registered Mates and their metadata
- `mate_capabilities` - Capabilities offered by each Mate
- `mate_versions` - Version history and deployment tracking
- `mate_dependencies` - Dependencies between Mates

#### LLM Integration
- `llm_providers` - Configured LLM providers (Azure AI Foundry, Anthropic, Google Gemini)
- `llm_models` - Available models per provider
- `llm_requests` - Request history and metadata
- `llm_responses` - Response cache and audit trail
- `llm_usage` - Token usage and cost tracking per user/mate
- `llm_fine_tuning` - Custom model training jobs and configurations

#### Observability & Audit
- `audit_logs` - Security and compliance audit trail
- `data_access_logs` - Record of all data access events
- `system_events` - System-level events and state changes
- `metrics_aggregates` - Pre-aggregated metrics for reporting

#### Token Management
- `tokens` - Active tokens and their metadata
- `token_revocations` - Revoked tokens list
- `refresh_tokens` - Refresh token storage

**Features**:
- JSONB columns for flexible metadata storage
- Full-text search for discovery and audit queries
- Row-level security (RLS) for multi-tenancy
- Partitioning for large tables (audit logs, events)
- Replication for high availability
- Point-in-time recovery (PITR) for disaster recovery
- Connection pooling via PgBouncer

**Integration with NATS**:
- Write-through cache: PostgreSQL is source of truth
- JetStream KV for read-heavy data (consent, registry)
- Change Data Capture (CDC) via logical replication
- Event publishing on data changes to JetStream
- Eventual consistency between PostgreSQL and JetStream

**Performance Optimization**:
- Indexes on frequently queried columns
- Materialized views for complex reporting queries
- Query optimization and EXPLAIN analysis
- Prepared statements for common queries
- Batch inserts for high-throughput scenarios

### 8. LLM Integration Layer

**Purpose**: Unified interface for Large Language Model providers

**Supported Providers**:

#### Azure AI Foundry
- Azure OpenAI Service integration
- GPT-4, GPT-4 Turbo, GPT-3.5 models
- Custom model deployments
- Content filtering and safety
- Azure AD authentication
- Regional deployment support
- Cost management and quotas

#### Anthropic
- Claude 3.5 Sonnet, Claude 3 Opus, Claude 3 Haiku
- Claude API integration
- Extended context windows (200K tokens)
- Constitutional AI safety features
- Streaming responses
- Function calling support

#### Google Gemini
- Gemini Pro, Gemini Ultra models
- Vertex AI integration
- Multimodal capabilities (text, images, video)
- Grounding with Google Search
- Safety settings and filters
- Google Cloud authentication

**Features**:

#### Unified API
- Provider-agnostic interface for Mates
- Automatic provider selection based on capability
- Fallback and retry logic across providers
- Load balancing between providers
- Cost optimization routing

#### Request Management
- Request queuing and rate limiting
- Priority queues for critical requests
- Batch processing for efficiency
- Response caching to reduce costs
- Request deduplication

#### Safety & Compliance
- Content filtering and moderation
- PII detection and redaction
- Consent verification before LLM access
- Output validation and safety checks
- Bias detection and mitigation

#### Observability
- Request/response logging
- Token usage tracking
- Cost attribution per Mate/user
- Performance metrics (latency, throughput)
- Error rate monitoring
- Model performance evaluation

#### Cost Management
- Per-Mate budget limits
- Cost allocation and chargeback
- Provider cost comparison
- Optimization recommendations
- Usage forecasting

**Integration Pattern**:

```
Mate → intra365.llm.request → LLM Integration Layer
                             ↓
                   [Provider Selection]
                             ↓
          ┌──────────────────┼──────────────────┐
          ↓                  ↓                  ↓
    Azure AI Foundry    Anthropic        Google Gemini
          ↓                  ↓                  ↓
          └──────────────────┼──────────────────┘
                             ↓
                       [Response Cache]
                             ↓
                      PostgreSQL + JetStream
```

**NATS Subject Design**:
```
intra365.llm.
├── request.{provider}.{model}
├── response.{request-id}
├── stream.{request-id}       # Streaming responses
├── cache.query
└── usage.report
```

**Configuration Storage**:
- Provider credentials in PostgreSQL (encrypted)
- Model configurations and parameters
- Rate limits and quotas per provider
- Routing rules and fallback policies
- Cost limits and budget alerts

### 9. Backup & Recovery System

**Purpose**: Comprehensive backup and disaster recovery for all intra365 components

**Components**:

#### Backup Manager Service
- Centralized orchestration of all backup operations
- Backup scheduling and automation
- Retention policy management
- Backup verification and validation
- Recovery point objective (RPO) tracking
- Recovery time objective (RTO) monitoring
- Backup inventory and catalog

**Technology**:
- Go-based service with gRPC/HTTP APIs
- Integration with PostgreSQL, NATS, Kubernetes
- S3-compatible storage for backup artifacts
- Encryption at rest and in transit

#### Database Backups (PostgreSQL)

**CloudNativePG Integrated Backups**:
- Continuous WAL archiving to S3-compatible storage
- Automated base backups (daily, configurable)
- Point-in-time recovery (PITR) support
- Backup compression (gzip, lz4, zstd)
- Retention policies (30 days default, configurable)
- Cross-region backup replication
- Backup encryption with customer-managed keys

**Backup Types**:
- **Full Backups**: Complete database snapshot (scheduled daily)
- **Incremental Backups**: WAL segments (continuous streaming)
- **Logical Backups**: pg_dump for specific schemas/tables (weekly)
- **Snapshot Backups**: Volume snapshots for rapid recovery

**Configuration**:
```yaml
backup:
  barmanObjectStore:
    destinationPath: s3://intra365-backups/postgres
    s3Credentials:
      secretName: postgres-backup-s3-credentials
    wal:
      compression: gzip
      maxParallel: 4
    data:
      compression: gzip
      jobs: 4
  retentionPolicy: "30d"
  schedule: "0 2 * * *"  # 2 AM daily
```

#### NATS JetStream Backups

**Stream and KV Store Backups**:
- JetStream stream snapshots
- Key-Value store exports
- Object store backups
- Stream configuration backups
- Consumer state preservation

**Backup Strategy**:
- Stream snapshots every 6 hours
- KV store exports every hour for critical data
- Configuration backups on every change
- Stored in S3-compatible storage
- Compressed and encrypted

**NATS Backup Tool**:
```bash
# Backup all streams
nats stream backup --all --output s3://intra365-backups/nats/

# Backup specific stream
nats stream backup EVENTS --output s3://intra365-backups/nats/streams/

# Backup KV store
nats kv backup CONFIG --output s3://intra365-backups/nats/kv/
```

#### Kubernetes Resource Backups

**Velero Integration**:
- Automated Kubernetes resource backups
- Persistent volume snapshots
- Namespace-level backup and restore
- Application-consistent snapshots
- Cross-cluster migration support
- Backup hooks for data consistency

**Backup Schedule**:
```yaml
# Daily full cluster backup
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 1 * * *"  # 1 AM daily
  template:
    includedNamespaces:
      - intra365-core
      - intra365-infra
      - intra365-observability
      - intra365-llm
    storageLocation: default
    volumeSnapshotLocations:
      - default
    ttl: 720h  # 30 days
```

**Resource Types Backed Up**:
- Deployments, StatefulSets, DaemonSets
- ConfigMaps and Secrets (encrypted)
- Services, Ingresses, NetworkPolicies
- PersistentVolumeClaims and data
- Custom Resource Definitions (CRDs)
- RBAC policies

#### Secrets Backup (HashiCorp Vault)

**Vault Backup Strategy**:
- Automated Vault snapshots using Raft storage backend
- Raft snapshot API for consistent backups
- Encryption keys backup (sealed/unsealed state)
- Audit log archival
- Policy and configuration backups

**Backup Frequency**:
- Raft snapshots: Every 6 hours
- Audit logs: Continuous archival to S3
- Configuration: On every change
- Retention: 90 days minimum

**Vault Backup Commands**:
```bash
# Take Raft snapshot
vault operator raft snapshot save backup.snap

# Upload to S3
aws s3 cp backup.snap s3://intra365-backups/vault/$(date +%Y%m%d-%H%M%S).snap

# Verify snapshot
vault operator raft snapshot inspect backup.snap
```

#### Application State Backups

**Consent and Policy Data**:
- PostgreSQL backups include consent records
- JetStream event stream backups preserve history
- Audit trail preservation
- Compliance data retention

**LLM Usage Data**:
- Request/response history backups
- Token usage tracking data
- Cost allocation data
- Fine-tuning model checkpoints

**Mate Registry Data**:
- Mate metadata and configurations
- Capability definitions
- Version history
- Dependency graphs

#### Backup Verification & Testing

**Automated Verification**:
- Daily backup integrity checks
- Periodic restore tests (monthly)
- Backup completeness validation
- Corruption detection
- Recovery time testing

**Backup Testing Schedule**:
- **Daily**: Checksum verification of backup files
- **Weekly**: Sample restore of small dataset
- **Monthly**: Full restore to isolated environment
- **Quarterly**: Disaster recovery drill

**Verification Process**:
```yaml
# Backup verification job
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-verification
  namespace: intra365-infra
spec:
  schedule: "0 3 * * *"  # 3 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: verify
            image: intra365/backup-manager:latest
            command:
            - /bin/backup-verify
            args:
            - --type=all
            - --s3-bucket=intra365-backups
            - --notification-webhook=${SLACK_WEBHOOK}
```

#### Disaster Recovery Procedures

**Recovery Scenarios**:

1. **Database Failure**:
   - CloudNativePG automatic failover to replica
   - If cluster lost: Restore from PITR backup
   - RTO: < 5 minutes (failover), < 30 minutes (full restore)
   - RPO: < 5 minutes (WAL streaming)

2. **NATS Cluster Failure**:
   - JetStream replica takeover
   - If cluster lost: Restore streams from backup
   - RTO: < 2 minutes (replica), < 15 minutes (restore)
   - RPO: < 1 minute (replication)

3. **Namespace Deletion**:
   - Velero restore from latest backup
   - RTO: < 1 hour
   - RPO: Based on backup schedule (1-24 hours)

4. **Full Cluster Loss**:
   - Provision new Kubernetes cluster
   - Restore infrastructure components (NATS, PostgreSQL, Vault)
   - Restore application workloads via Velero
   - Validate data integrity and consistency
   - RTO: < 4 hours
   - RPO: < 24 hours

5. **Regional Disaster**:
   - Cross-region backup replication
   - Failover to secondary region
   - DNS update for traffic routing
   - RTO: < 2 hours
   - RPO: < 1 hour

**Recovery Runbooks**:
- Step-by-step recovery procedures in `chef/docs/runbooks/`
- Automated recovery scripts where possible
- On-call engineer access to recovery tools
- Regular DR drill documentation and updates

#### Backup Storage & Retention

**Storage Locations**:
- Primary: S3-compatible object storage (MinIO, AWS S3, Azure Blob, GCS)
- Secondary: Cross-region replication for critical backups
- Tertiary: Tape/cold storage for long-term archival (optional)

**Retention Policies**:
- **Hot Backups** (frequent access): 7 days in standard storage
- **Warm Backups** (occasional access): 30 days in infrequent access storage
- **Cold Backups** (compliance): 90-365 days in archival storage
- **Compliance Backups**: 7 years for regulatory requirements

**Storage Optimization**:
- Deduplication for incremental backups
- Compression (gzip, lz4, zstd)
- Encryption at rest (AES-256)
- Lifecycle policies for automatic tiering
- Cost monitoring and optimization

#### Backup Monitoring & Alerting

**Metrics Collected**:
- Backup success/failure rate
- Backup duration and size
- Storage utilization
- Recovery time metrics
- Backup age (last successful backup)
- Verification test results

**Alerts**:
- Backup failure (critical)
- Backup age > 48 hours (warning)
- Storage capacity > 80% (warning)
- Verification test failure (critical)
- RPO/RTO SLA breach (critical)

**Dashboard**:
- Grafana dashboard showing backup health
- Backup history and trends
- Storage consumption forecasting
- Recovery point and time objectives tracking

#### Backup Service API

**Endpoints**:
```
POST   /api/v1/backup/trigger          # Trigger manual backup
GET    /api/v1/backup/status/:id       # Get backup status
GET    /api/v1/backup/list             # List all backups
POST   /api/v1/backup/restore          # Initiate restore
GET    /api/v1/backup/verify/:id       # Verify backup integrity
DELETE /api/v1/backup/:id              # Delete old backup
GET    /api/v1/backup/metrics          # Get backup metrics
```

**NATS Subjects**:
```
intra365.backup.
├── trigger.{component}              # Trigger backup
├── status.{backup-id}               # Backup status updates
├── completed.{component}            # Backup completion event
├── failed.{component}               # Backup failure event
├── restore.request                  # Restore request
├── restore.status.{restore-id}      # Restore progress
└── verify.{backup-id}               # Verification request
```

**Backup Metadata**:
```json
{
  "backupId": "uuid",
  "component": "postgresql",
  "type": "full",
  "startTime": "2025-10-29T02:00:00Z",
  "endTime": "2025-10-29T02:15:00Z",
  "duration": 900,
  "size": 52428800,
  "compression": "gzip",
  "encryption": "aes-256",
  "storage": {
    "location": "s3://intra365-backups/postgres/2025-10-29/full.tar.gz",
    "checksum": "sha256:abc123...",
    "replicas": [
      "s3://intra365-backups-eu/postgres/2025-10-29/full.tar.gz"
    ]
  },
  "retentionPolicy": "30d",
  "expiresAt": "2025-11-28T02:00:00Z",
  "verified": true,
  "verifiedAt": "2025-10-29T03:00:00Z",
  "status": "completed"
}
```

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Intra365 Ecosystem                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│  │  Mate 1  │  │  Mate 2  │  │  Mate 3  │  │  Mate N  │        │
│  │ (AI Bot) │  │(Web App) │  │(Service) │  │(IoT Dev) │        │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│       │             │              │             │               │
│       └─────────────┴──────────────┴─────────────┘               │
│                          │                                        │
│                          ▼                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Gateway & API Layer                           │ │
│  │  - REST/GraphQL API  - WebSocket  - Rate Limiting         │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│                         ▼                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │         Security & Token Service (STS)                     │ │
│  │  - Token Issuance    - Entra ID Integration               │ │
│  │  - Token Validation  - GitHub OAuth                       │ │
│  │  - RBAC             - API Key Management                  │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│  ┌──────────────────────┴─────────────────────────────────────┐ │
│  │         Consent Management & Policy System                 │ │
│  │  - User Consent Storage    - Policy Engine (ABAC)         │ │
│  │  - Permission Management   - Compliance (GDPR/CCPA)       │ │
│  │  - Consent Audit Trail     - Real-time Policy Evaluation  │ │
│  └──────────────────────┬─────────────────────────────────────┘ │
│                         │                                        │
│  ┌──────────────────────┴─────────────────────────────────────┐ │
│  │                   NATS Infrastructure                      │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │            NATS Core (Pub/Sub/Request)             │  │ │
│  │  │  Subject: intra365.{mate-id}.{capability}.{action} │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │              NATS JetStream                        │  │ │
│  │  │  - Event Streams  - KV Store  - Object Store      │  │ │
│  │  │  - Persistent Messaging  - Message Replay          │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │            Service Discovery                       │  │ │
│  │  │  - Mate Registry  - Health Checks  - Load Balance │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                         │                                        │
│  ┌──────────────────────┼─────────────────────────────────────┐ │
│  │                      ▼                                      │ │
│  │              Event Processing Layer                        │ │
│  │  - Filtering  - Transformation  - Orchestration           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              LLM Integration Layer                         │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │ │
│  │  │  Azure AI    │  │  Anthropic   │  │Google Gemini │    │ │
│  │  │   Foundry    │  │    Claude    │  │              │    │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘    │ │
│  │  - Unified API  - Cost Management  - Safety Controls     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Observability Stack                           │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │ │
│  │  │ Metrics  │  │ Tracing  │  │ Logging  │  │Dashboards│  │ │
│  │  │Prometheus│  │   Jaeger │  │   Loki   │  │ Grafana  │  │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              Persistence Layer                             │ │
│  │  - PostgreSQL Database (Source of Truth)                  │ │
│  │  - JetStream Streams  - KV Store (Cache)                  │ │
│  │  - Object Store       - CDC to JetStream                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

External Identity Providers:
┌──────────┐  ┌──────────┐  ┌──────────┐
│Entra ID  │  │  GitHub  │  │  Custom  │
│(Azure AD)│  │  OAuth   │  │   OIDC   │
└──────────┘  └──────────┘  └──────────┘

External LLM Providers:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Azure AI    │  │  Anthropic   │  │Google Gemini │
│   Foundry    │  │    Claude    │  │   Vertex AI  │
└──────────────┘  └──────────────┘  └──────────────┘
```

## Communication Patterns

### 1. Authentication Flow

```
1. Mate requests token from STS
   → Subject: intra365.auth.token.request
   → Payload: { provider: "entra-id", credentials: {...} }

2. STS validates with external IdP (Entra ID/GitHub)

3. STS issues JWT token
   → Response: { token: "jwt...", expiresIn: 3600 }

4. Mate uses token in NATS auth headers for all communications
```

### 2. Mate Registration

```
1. Mate connects to NATS with valid token

2. Mate registers capabilities
   → Subject: intra365.registry.register
   → Payload: { mateId, capabilities, endpoints }

3. Registry stores in JetStream KV

4. Registry publishes event
   → Subject: intra365.events.mate.registered
   → Other Mates can discover new capabilities
```

### 3. Mate-to-Mate Communication

```
1. Mate A discovers Mate B via Registry
   → Request: intra365.registry.discover
   → Query: { capability: "image-analysis" }

2. Mate A verifies user consent for data sharing
   → Request: intra365.consent.verify
   → Payload: { userId, mateId: "mate-b", scopes: ["image.analyze"], purpose: "ai-assistance" }
   → Response: { allowed: true, consentId: "uuid" }

3. Mate A sends request to Mate B (if consent granted)
   → Subject: intra365.mate.{mate-b-id}.analyze-image
   → Headers: { 
       Authorization: "Bearer jwt...",
       X-Consent-Id: "uuid",
       X-User-Id: "user-id"
     }
   → Payload: { imageUrl: "...", options: {...} }

4. Mate B validates consent before processing
   → Checks consent scope and expiration
   → Logs data access for audit

5. Mate B processes and responds
   → Reply with results
   → Logs consent usage

6. Interaction logged to observability with consent context
```

### 4. Event Stream Processing

```
1. Mate publishes event
   → Subject: intra365.events.{domain}.{event-type}
   → Stored in JetStream stream

2. Subscribers consume events
   → Durable consumers with filters
   → At-least-once delivery guarantee

3. Event processors transform/route
   → Enrichment, aggregation, routing

4. Metrics collected for monitoring
```

## Subject Namespace Design

```
intra365.
├── auth.
│   ├── token.request
│   ├── token.validate
│   └── token.refresh
├── consent.
│   ├── grant
│   ├── revoke
│   ├── verify
│   ├── query
│   └── audit
├── policy.
│   ├── evaluate
│   ├── create
│   ├── update
│   └── delete
├── registry.
│   ├── register
│   ├── deregister
│   ├── discover
│   └── heartbeat
├── mate.{mate-id}.
│   ├── {capability}.{action}
│   ├── status
│   └── config
├── llm.
│   ├── request.{provider}.{model}
│   ├── response.{request-id}
│   ├── stream.{request-id}
│   ├── cache.query
│   └── usage.report
├── events.
│   ├── mate.{registered|deregistered|updated}
│   ├── consent.{granted|revoked|expired}
│   ├── policy.{created|updated|deleted}
│   ├── llm.{request|response|error}
│   ├── system.{started|stopped|error}
│   └── {domain}.{event-type}
└── observability.
    ├── metrics
    ├── traces
    └── logs
```

## Security Considerations

### Multitenancy Architecture

The intra365 platform is designed for **strong multitenancy** with complete isolation between tenants at multiple levels.

**Tenancy Model**:

1. **Tenant Definition**: A tenant represents an organization, workspace, or isolated group of users
2. **Tenant ID**: Unique identifier embedded in all requests and data
3. **Isolation Levels**: Network, data, compute, and logical isolation
4. **Tenant Onboarding**: Automated provisioning of tenant-specific resources

**Multi-Tenant Isolation Layers**:

```
┌─────────────────────────────────────────────────────────────┐
│                    Tenant Isolation Stack                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: Network Isolation (Caddy + Network Policies)      │
│  ┌────────────────────────────────────────────────────┐    │
│  │  tenant1.intra365.io  │  tenant2.intra365.io       │    │
│  │         ↓                        ↓                  │    │
│  │  Namespace: tenant-1    Namespace: tenant-2        │    │
│  │  NetworkPolicy: Deny cross-namespace traffic       │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  Layer 2: NATS Account Isolation                            │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Account: tenant-1  │  Account: tenant-2           │    │
│  │  Subjects:          │  Subjects:                   │    │
│  │  intra365.t1.*      │  intra365.t2.*               │    │
│  │  (No cross-account messaging without export)       │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  Layer 3: Database Row-Level Security (PostgreSQL RLS)     │
│  ┌────────────────────────────────────────────────────┐    │
│  │  SELECT * FROM mates WHERE tenant_id = current_tenant│   │
│  │  (Enforced at database level, cannot be bypassed)   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  Layer 4: Application-Level Tenant Context                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │  All requests carry tenant_id in JWT token         │    │
│  │  Middleware validates tenant access on every call  │    │
│  │  Audit logs include tenant context                 │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### 1. Network-Level Isolation (Caddy + Kubernetes)

**Caddy Ingress Configuration**:

```caddyfile
# Caddyfile for multi-tenant routing

# Tenant 1
tenant1.intra365.io {
    reverse_proxy /api/* {
        to gateway-service.tenant-1.svc.cluster.local:3000
        header_up X-Tenant-ID "tenant-1"
        header_up X-Tenant-Name "Acme Corp"
    }
    
    reverse_proxy /ws/* {
        to gateway-service.tenant-1.svc.cluster.local:3001
        header_up X-Tenant-ID "tenant-1"
    }
    
    # Automatic HTTPS
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}

# Tenant 2
tenant2.intra365.io {
    reverse_proxy /api/* {
        to gateway-service.tenant-2.svc.cluster.local:3000
        header_up X-Tenant-ID "tenant-2"
        header_up X-Tenant-Name "TechStart Inc"
    }
    
    reverse_proxy /ws/* {
        to gateway-service.tenant-2.svc.cluster.local:3001
        header_up X-Tenant-ID "tenant-2"
    }
    
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}

# Wildcard for dynamic tenant domains
*.intra365.io {
    # Extract tenant from subdomain
    reverse_proxy /api/* {
        to gateway-service.intra365-core.svc.cluster.local:3000
        header_up X-Tenant-ID {labels.0}
    }
    
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
        on_demand
    }
}

# Shared services (cross-tenant)
admin.intra365.io {
    reverse_proxy {
        to admin-service.intra365-core.svc.cluster.local:8080
    }
    
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}
```

**Kubernetes Network Policies**:

```yaml
# Deny all cross-tenant traffic by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: tenant-isolation
  namespace: tenant-1
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  # Allow from same namespace
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-1
  # Allow from shared infrastructure
  - from:
    - namespaceSelector:
        matchLabels:
          role: infrastructure
  egress:
  # Allow to same namespace
  - to:
    - namespaceSelector:
        matchLabels:
          tenant: tenant-1
  # Allow to shared infrastructure (NATS, PostgreSQL, Vault)
  - to:
    - namespaceSelector:
        matchLabels:
          role: infrastructure
  # Allow external egress (LLM providers, etc.)
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
```

**Namespace Strategy**:

```yaml
# Tenant-specific namespaces
- tenant-1                  # Tenant 1 services
- tenant-2                  # Tenant 2 services
- tenant-N                  # Additional tenants

# Shared infrastructure namespaces
- intra365-core            # Core services (STS, consent, registry)
- intra365-infra           # NATS, PostgreSQL, Vault, Caddy
- intra365-observability   # Monitoring stack
- intra365-llm             # LLM proxy (handles multi-tenancy internally)
```

#### 2. NATS Account Isolation

**Multi-Tenant NATS Configuration**:

```yaml
# NATS Accounts per tenant
accounts:
  TENANT_1:
    jetstream: enabled
    limits:
      max_connections: 1000
      max_subscriptions: 10000
      max_payload: 1048576
      max_data: 100GB
    exports:
      # Allow controlled sharing with other tenants
      - stream: intra365.t1.public.*
        accounts: [TENANT_2]
    imports:
      # Import from shared services
      - stream: intra365.shared.llm.*
        account: SHARED
    
  TENANT_2:
    jetstream: enabled
    limits:
      max_connections: 1000
      max_subscriptions: 10000
      max_payload: 1048576
      max_data: 100GB
  
  SHARED:
    # Shared services account for cross-tenant functionality
    jetstream: enabled
    exports:
      - stream: intra365.shared.llm.*
        accounts: [TENANT_1, TENANT_2]
      - stream: intra365.shared.registry.*
        accounts: [TENANT_1, TENANT_2]
```

**Subject Namespace per Tenant**:

```
Tenant 1:
  intra365.t1.mate.{mate-id}.*
  intra365.t1.auth.*
  intra365.t1.consent.*
  intra365.t1.events.*

Tenant 2:
  intra365.t2.mate.{mate-id}.*
  intra365.t2.auth.*
  intra365.t2.consent.*
  intra365.t2.events.*

Shared (cross-tenant):
  intra365.shared.llm.*
  intra365.shared.registry.*
  intra365.shared.backup.*
```

#### 3. Database Row-Level Security (PostgreSQL RLS)

**Multi-Tenant Database Schema**:

All tables include a `tenant_id` column with Row-Level Security policies:

```sql
-- Enable RLS on all tenant-specific tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE mates ENABLE ROW LEVEL SECURITY;
ALTER TABLE consent_records ENABLE ROW LEVEL SECURITY;
ALTER TABLE policies ENABLE ROW LEVEL SECURITY;
ALTER TABLE llm_requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

-- Create RLS policies for tenant isolation
CREATE POLICY tenant_isolation_policy ON users
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

CREATE POLICY tenant_isolation_policy ON mates
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

CREATE POLICY tenant_isolation_policy ON consent_records
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

CREATE POLICY tenant_isolation_policy ON policies
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

CREATE POLICY tenant_isolation_policy ON llm_requests
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

CREATE POLICY tenant_isolation_policy ON audit_logs
  USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Shared resources (cross-tenant with visibility control)
CREATE POLICY tenant_visibility_policy ON mates
  USING (
    tenant_id = current_setting('app.current_tenant')::uuid
    OR (visibility = 'public' AND status = 'published')
  );
```

**Tenant Context Setting**:

```go
// Go code to set tenant context for database session
func SetTenantContext(db *sql.DB, tenantID string) error {
    _, err := db.Exec("SET app.current_tenant = $1", tenantID)
    return err
}

// Middleware that sets tenant context from JWT
func TenantContextMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        tenantID := extractTenantFromJWT(r)
        
        // Set tenant context for this request's database connections
        db := getDBConnection()
        if err := SetTenantContext(db, tenantID); err != nil {
            http.Error(w, "Tenant context error", http.StatusInternalServerError)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}
```

**Tenant-Specific Database Users**:

```yaml
# CloudNativePG configuration with tenant-specific users
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: intra365-postgres
spec:
  # ... other config ...
  
  bootstrap:
    initdb:
      postInitSQL:
        - CREATE ROLE tenant_1_app WITH LOGIN PASSWORD 'generated_by_vault';
        - GRANT USAGE ON SCHEMA public TO tenant_1_app;
        - GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO tenant_1_app;
        - ALTER ROLE tenant_1_app SET app.current_tenant = 'tenant-1-uuid';
        
        - CREATE ROLE tenant_2_app WITH LOGIN PASSWORD 'generated_by_vault';
        - GRANT USAGE ON SCHEMA public TO tenant_2_app;
        - GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO tenant_2_app;
        - ALTER ROLE tenant_2_app SET app.current_tenant = 'tenant-2-uuid';
```

#### 4. Application-Level Tenant Context

**JWT Token with Tenant Claim**:

```json
{
  "sub": "user-id",
  "tenant_id": "tenant-1-uuid",
  "tenant_name": "Acme Corp",
  "roles": ["admin", "mate-creator"],
  "permissions": [
    "mates.create",
    "mates.read",
    "consent.manage"
  ],
  "iat": 1698518400,
  "exp": 1698522000
}
```

**Gateway Tenant Extraction**:

```typescript
// TypeScript middleware in API Gateway
export function extractTenantContext(req: Request, res: Response, next: NextFunction) {
  // Extract from JWT
  const token = extractJWT(req);
  const tenantId = token.tenant_id;
  
  // Or extract from subdomain
  const host = req.hostname;
  const subdomain = host.split('.')[0];
  
  // Or extract from header (set by Caddy)
  const headerTenant = req.headers['x-tenant-id'];
  
  // Validate tenant and attach to request context
  req.context = {
    tenantId: tenantId,
    tenantName: token.tenant_name,
    userId: token.sub,
    roles: token.roles,
    permissions: token.permissions
  };
  
  // Set tenant context for downstream services
  req.headers['x-tenant-id'] = tenantId;
  
  next();
}
```

**Service-Level Tenant Validation**:

```go
// Go service validates tenant on every operation
func (s *ConsentService) GrantConsent(ctx context.Context, req *GrantConsentRequest) error {
    tenantID := middleware.GetTenantID(ctx)
    
    // Validate tenant matches resource
    if req.TenantID != tenantID {
        return ErrTenantMismatch
    }
    
    // Validate user belongs to tenant
    if !s.validateUserTenant(req.UserID, tenantID) {
        return ErrUnauthorized
    }
    
    // All database queries automatically filtered by RLS
    // All NATS messages published to tenant-specific subjects
    
    return s.repo.GrantConsent(ctx, req)
}
```

#### 5. Tenant Resource Quotas

**Kubernetes ResourceQuota per Tenant**:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-1-quota
  namespace: tenant-1
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    requests.storage: 500Gi
    persistentvolumeclaims: "10"
    pods: "50"
    services: "20"
    configmaps: "50"
    secrets: "50"
```

**NATS JetStream Limits per Tenant**:

```yaml
# Per-tenant stream limits
stream_limits:
  tenant_1:
    max_bytes: 107374182400  # 100 GB
    max_age: 2592000          # 30 days
    max_msg_size: 1048576     # 1 MB
    max_consumers: 100
  
  tenant_2:
    max_bytes: 107374182400
    max_age: 2592000
    max_msg_size: 1048576
    max_consumers: 100
```

**LLM Budget Limits per Tenant**:

```sql
CREATE TABLE tenant_llm_budgets (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id),
    monthly_budget_usd DECIMAL(10,2) NOT NULL,
    current_month_spend DECIMAL(10,2) DEFAULT 0,
    budget_reset_date DATE NOT NULL,
    alert_threshold_pct INTEGER DEFAULT 80,
    hard_limit_enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Trigger to check budget before LLM request
CREATE OR REPLACE FUNCTION check_tenant_llm_budget()
RETURNS TRIGGER AS $$
BEGIN
    DECLARE
        budget RECORD;
    BEGIN
        SELECT * INTO budget FROM tenant_llm_budgets 
        WHERE tenant_id = NEW.tenant_id;
        
        IF budget.hard_limit_enabled AND 
           budget.current_month_spend >= budget.monthly_budget_usd THEN
            RAISE EXCEPTION 'Tenant LLM budget exceeded';
        END IF;
        
        RETURN NEW;
    END;
END;
$$ LANGUAGE plpgsql;
```

#### 6. Tenant Data Isolation in Backups

**Velero Tenant-Specific Backups**:

```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: tenant-1-daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *"
  template:
    includedNamespaces:
      - tenant-1
    labelSelector:
      matchLabels:
        tenant: tenant-1
    storageLocation: tenant-1-backups
    volumeSnapshotLocations:
      - tenant-1-snapshots
    ttl: 720h
```

**PostgreSQL Tenant Data Export**:

```sql
-- Export tenant-specific data for compliance/backup
COPY (
    SELECT * FROM users WHERE tenant_id = 'tenant-1-uuid'
) TO '/backups/tenant-1/users.csv' WITH CSV HEADER;

COPY (
    SELECT * FROM consent_records WHERE tenant_id = 'tenant-1-uuid'
) TO '/backups/tenant-1/consent_records.csv' WITH CSV HEADER;

-- Encrypted backup with tenant key from Vault
```

#### 7. Tenant Onboarding Automation

**AI Agent Prompt for Tenant Provisioning**:

```
"Create a new tenant named 'Acme Corp' with subdomain 'acme':
- Provision Kubernetes namespace 'tenant-acme'
- Create NATS account with 100GB JetStream limit
- Set up PostgreSQL user with RLS policies
- Configure Caddy route for acme.intra365.io
- Create Vault secrets path /tenants/acme
- Set resource quotas: 10 CPUs, 20GB RAM, 200GB storage
- Configure network policies for isolation
- Enable monthly LLM budget of $500
- Send welcome email to admin@acme.com"
```

**Tenant Metadata Storage**:

```sql
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(100) UNIQUE NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    plan_tier VARCHAR(50) DEFAULT 'standard',
    
    -- Resource limits
    max_users INTEGER DEFAULT 100,
    max_mates INTEGER DEFAULT 50,
    max_cpu_cores INTEGER DEFAULT 10,
    max_memory_gb INTEGER DEFAULT 20,
    max_storage_gb INTEGER DEFAULT 200,
    
    -- LLM configuration
    llm_monthly_budget_usd DECIMAL(10,2) DEFAULT 500,
    llm_providers_enabled TEXT[] DEFAULT ARRAY['azure', 'anthropic'],
    
    -- Billing
    billing_email VARCHAR(255),
    billing_plan VARCHAR(50),
    stripe_customer_id VARCHAR(255),
    
    -- Metadata
    custom_domain VARCHAR(255),
    logo_url VARCHAR(500),
    primary_contact_email VARCHAR(255),
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP
);

CREATE INDEX idx_tenants_subdomain ON tenants(subdomain);
CREATE INDEX idx_tenants_status ON tenants(status);
```

### Summary of Multitenancy Features

1. **Network Isolation**: Caddy subdomain routing + Kubernetes NetworkPolicies
2. **Messaging Isolation**: NATS accounts with separate subject namespaces
3. **Data Isolation**: PostgreSQL Row-Level Security enforced at database level
4. **Application Isolation**: JWT tenant claims + middleware validation
5. **Resource Isolation**: Kubernetes ResourceQuotas per tenant namespace
6. **Budget Isolation**: Per-tenant LLM spending limits and quotas
7. **Backup Isolation**: Tenant-specific backup schedules and storage
8. **Automated Onboarding**: AI agent-based tenant provisioning

All isolation layers work together to provide defense-in-depth security, ensuring complete tenant separation while allowing controlled cross-tenant functionality where needed (e.g., public Mate discovery).

### Authentication
- All Mates must authenticate via STS
- Token-based auth using JWT with short expiration
- Support for token refresh
- API keys for service accounts

### Authorization
- RBAC for Mate capabilities
- Subject-level permissions in NATS
- Account isolation for multi-tenancy
- Rate limiting per Mate
- Consent-based access control for user data
- Policy-driven authorization (ABAC)

### Consent & Privacy
- Explicit user consent required for data sharing
- Granular consent scopes per Mate and purpose
- Real-time consent verification before data access
- Consent audit trail for compliance
- Automated consent expiration and renewal
- Right to revoke consent at any time
- GDPR, CCPA, and other privacy regulation compliance

### Data Protection
- TLS encryption for all NATS connections
- Message encryption for sensitive payloads
- Token encryption at rest in JetStream
- PostgreSQL encryption at rest (disk encryption)
- PostgreSQL SSL/TLS connections required
- Column-level encryption for sensitive data (PII, tokens)
- Database access audit logging
- Audit logging for compliance

### Threat Mitigation
- DDoS protection via rate limiting
- Circuit breakers for cascading failures
- Input validation at Gateway
- Token revocation support

## Scalability Design

### Horizontal Scaling
- NATS cluster for high availability
- Multiple STS instances with shared JetStream state
- Stateless Gateway nodes behind load balancer
- Mate instances can scale independently

### Performance Optimization
- Subject partitioning for high-throughput
- JetStream replica configuration
- Message batching where appropriate
- Caching for frequently accessed data (KV store)

### Resource Management
- Per-Mate quotas and limits
- JetStream storage limits and retention policies
- Connection limits and backpressure
- Priority queues for critical messages

## Technology Stack Summary

| Component | Technology |
|-----------|------------|
| Message Broker | NATS Core |
| Persistent Messaging | NATS JetStream |
| Database | PostgreSQL 16+ |
| Database Operator | CloudNativePG |
| Connection Pooling | PgBouncer |
| Token Service | Custom service with NATS JWT |
| Identity Integration | Entra ID SDK, GitHub OAuth |
| LLM Providers | Azure AI Foundry, Anthropic Claude, Google Gemini |
| Service Discovery | NATS Service API |
| Secrets Management | Self-hosted HashiCorp Vault |
| Observability | Prometheus, Grafana, Jaeger, Loki |
| Ingress Controller | Caddy (with automatic HTTPS and HTTP/3) |
| API Gateway | Custom (Go/Node.js) + NATS |
| Persistence | PostgreSQL (primary), JetStream KV (cache) |
| Backup & Recovery | Velero, Restic, CloudNativePG Backups |
| Languages | Go (core services), TypeScript/Node.js (Mates), Python (ML/AI) |
| Version Control | GitHub (git, task management, security) |
| Infrastructure Deployment | AI Agent-based (NO Terraform/Pulumi) |
| IaC Philosophy | Prompt-based desired state with resilient reconciliation |
| Multitenancy | Network, NATS account, PostgreSQL RLS, application-level isolation |

## Repository Structure

The intra365 platform follows a **single-repository-per-service** architecture, where each component maintains its own repository with independent release cycles.

### GitHub Platform Features

**All repositories utilize GitHub's comprehensive platform features**:

#### Version Control & Collaboration
- **GitHub Git**: Distributed version control with branch protection rules
- **Pull Requests**: Code review workflow with required approvals
- **Code Owners**: Automatic reviewer assignment based on CODEOWNERS file
- **Branch Protection**: Enforce status checks, require reviews, prevent force pushes
- **Merge Queue**: Automated PR merging with CI validation

#### Task Management & Project Planning
- **GitHub Issues**: Task tracking, bug reports, feature requests
- **GitHub Projects**: Kanban boards, roadmaps, sprint planning
- **Milestones**: Group issues and PRs by release version
- **Labels**: Categorize issues (bug, enhancement, documentation, etc.)
- **Issue Templates**: Standardized bug reports and feature requests
- **Discussions**: Community Q&A, RFCs, design discussions

#### Security Features
- **GitHub Advanced Security**: Comprehensive security scanning and analysis
  - **Code Scanning**: Automated CodeQL analysis for vulnerabilities
  - **Secret Scanning**: Detect exposed credentials and tokens
  - **Dependency Review**: Review dependency changes in PRs
  - **Security Advisories**: Private vulnerability reporting and disclosure
  
- **Dependabot**: Automated dependency updates and security patches
  - Version updates for npm, Go modules, Python packages, Docker images
  - Automated PR creation for dependency updates
  - Security vulnerability alerts and auto-fixes
  
- **Supply Chain Security**:
  - Dependency graph visualization
  - Software Bill of Materials (SBOM)
  - Vulnerable dependency alerts
  - License compliance tracking

#### Automation & CI/CD
- **GitHub Actions**: CI/CD workflows for build, test, deploy
- **Workflows**: Automated testing, linting, security scanning
- **Environments**: Deployment targets (dev, staging, production)
- **Secrets Management**: Encrypted secrets for workflows
- **Self-hosted Runners**: Custom build agents for sensitive workloads

#### Code Quality & Review
- **Required Reviews**: Minimum number of approvals before merge
- **Status Checks**: CI must pass before merge
- **Automated Testing**: Unit, integration, security tests on PR
- **Code Coverage**: Track test coverage with Codecov integration
- **Performance Testing**: Automated benchmarks on PR

#### Release Management
- **GitHub Releases**: Semantic versioned releases with changelogs
- **Release Notes**: Auto-generated from commit messages and PRs
- **Tags**: Git tags for version tracking
- **Pre-release**: Beta/RC releases for early testing
- **Asset Distribution**: Binary and container image distribution

### Service Repositories

Each service repository is self-contained and responsible for:
- Source code
- Dockerfile and container build configuration
- Unit and integration tests
- Service-specific documentation
- CI/CD pipeline for building and releasing artifacts
- Semantic versioning and release management
- GitHub Issues for task tracking
- GitHub Projects for sprint planning
- GitHub Advanced Security scanning

**Core Service Repositories**:
- `happy-mates/intra365-sts` - Security & Token Service
- `happy-mates/intra365-consent-manager` - Consent Management & Policy Engine
- `happy-mates/intra365-mate-registry` - Mate Registry & Discovery
- `happy-mates/intra365-gateway` - API Gateway
- `happy-mates/intra365-llm-proxy` - LLM Integration Layer
- `happy-mates/intra365-event-processor` - Event Processing Service
- `happy-mates/intra365-backup-manager` - Backup & Recovery Service
- `happy-mates/intra365-infra-agent` - Infrastructure AI Agent (IAA) for prompt-based deployment

**Library/SDK Repositories**:
- `happy-mates/intra365-sdk-go` - Go SDK for Mate development
- `happy-mates/intra365-sdk-typescript` - TypeScript/JavaScript SDK
- `happy-mates/intra365-sdk-python` - Python SDK

**Infrastructure Repositories**:
- `happy-mates/intra365` - Architecture documentation and specifications (this repo)
- `happy-mates/intra365-vault` - Self-hosted Vault configuration and policies

### Chef Configuration Repository

**Repository**: `happy-mates/happy-mates-chef`

The central configuration repository that holds all deployment and orchestration configurations:

**Purpose**:
- Single source of truth for all infrastructure configurations
- Kubernetes manifests and Helm values for all environments
- GitOps configuration for ArgoCD/FluxCD
- Environment-specific settings (dev, staging, production)
- Shared configuration templates and policies

**Structure**:
```
happy-mates-chef/
├── README.md
├── environments/
│   ├── dev/
│   │   ├── values.yaml                    # Development environment values
│   │   ├── secrets.yaml                   # Vault-encrypted secrets
│   │   └── kustomization.yaml             # Environment-specific overlays
│   ├── staging/
│   │   ├── values.yaml
│   │   ├── secrets.yaml
│   │   └── kustomization.yaml
│   └── production/
│       ├── values.yaml
│       ├── secrets.yaml
│       └── kustomization.yaml
├── kubernetes/
│   ├── base/                              # Base Kubernetes manifests
│   │   ├── namespaces/
│   │   ├── network-policies/
│   │   ├── resource-quotas/
│   │   └── rbac/
│   ├── services/                          # Service-specific manifests
│   │   ├── sts/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── hpa.yaml
│   │   │   └── configmap.yaml
│   │   ├── consent-manager/
│   │   ├── mate-registry/
│   │   ├── gateway/
│   │   └── llm-proxy/
│   ├── infrastructure/                    # Infrastructure components
│   │   ├── nats/
│   │   │   └── helm-values.yaml
│   │   ├── postgresql/
│   │   │   └── cnpg-cluster.yaml          # CloudNativePG cluster definition
│   │   ├── vault/
│   │   │   ├── vault-helm-values.yaml
│   │   │   └── vault-policies/
│   │   └── ingress/
│   │       └── ingress-nginx-values.yaml
│   └── observability/                     # Monitoring stack
│       ├── prometheus/
│       ├── grafana/
│       ├── jaeger/
│       └── loki/
├── helm/
│   └── intra365-platform/                 # Umbrella Helm chart
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── templates/
│       └── charts/                        # Sub-charts
├── argocd/
│   ├── applications/                      # ArgoCD Application definitions
│   │   ├── core-services.yaml
│   │   ├── infrastructure.yaml
│   │   └── observability.yaml
│   └── projects/                          # ArgoCD Projects
│       └── intra365.yaml
├── vault/
│   ├── policies/                          # Vault policies
│   │   ├── sts-policy.hcl
│   │   ├── consent-manager-policy.hcl
│   │   └── llm-proxy-policy.hcl
│   ├── auth-methods/                      # Vault auth configurations
│   │   └── kubernetes-auth.yaml
│   └── secrets-engines/                   # Secret engine configurations
│       ├── database.yaml                  # Dynamic DB credentials
│       └── pki.yaml                       # Certificate management
├── scripts/
│   ├── bootstrap-cluster.sh               # Initial cluster setup
│   ├── deploy-service.sh                  # Service deployment helper
│   ├── vault-init.sh                      # Vault initialization
│   └── backup-restore.sh                  # Backup/restore procedures
├── docs/
│   ├── deployment-guide.md
│   ├── runbooks/
│   └── architecture-decisions/
└── .github/
    └── workflows/
        ├── validate-manifests.yml         # CI for manifest validation
        └── sync-environments.yml          # Sync configurations
```

**Workflow**:
1. Each service repository builds and publishes container images to registry
2. Service releases trigger updates to `happy-mates-chef` repository
3. Chef repository updates image tags in appropriate environment configurations
4. ArgoCD/FluxCD detects changes and synchronizes to Kubernetes cluster
5. Vault provides secrets dynamically at runtime

**Benefits**:
- **Separation of Concerns**: Service code separate from infrastructure configuration
- **Independent Releases**: Services can release independently without affecting others
- **Centralized Configuration**: Single place to manage all deployment configurations
- **Environment Consistency**: Same configuration patterns across all environments
- **GitOps Ready**: Git history provides audit trail and rollback capability
- **Security**: Vault integration keeps secrets out of Git

**CloudNativePG Configuration Example**:
```yaml
# kubernetes/infrastructure/postgresql/cnpg-cluster.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: intra365-postgres
  namespace: intra365-infra
spec:
  instances: 3
  imageName: ghcr.io/cloudnative-pg/postgresql:16
  
  postgresql:
    parameters:
      max_connections: "500"
      shared_buffers: "2GB"
      effective_cache_size: "6GB"
      work_mem: "32MB"
  
  bootstrap:
    initdb:
      database: intra365
      owner: intra365
      secret:
        name: intra365-db-credentials
  
  storage:
    size: 500Gi
    storageClass: premium-ssd
  
  backup:
    barmanObjectStore:
      destinationPath: s3://intra365-backups/postgres
      s3Credentials:
        secretName: postgres-backup-s3-credentials
      wal:
        compression: gzip
        maxParallel: 4
    retentionPolicy: "30d"
  
  pooler:
    enabled: true
    type: pgbouncer
    instances: 3
    pgbouncer:
      poolMode: transaction
      parameters:
        max_client_conn: "1000"
        default_pool_size: "25"
```

**HashiCorp Vault Configuration Example**:
```yaml
# vault/policies/llm-proxy-policy.hcl
path "secret/data/intra365/llm/*" {
  capabilities = ["read"]
}

path "database/creds/llm-proxy" {
  capabilities = ["read"]
}

path "pki/issue/intra365" {
  capabilities = ["create", "update"]
}
```

## Deployment Considerations

### Container Architecture

**Docker Containers**:

All intra365 components are containerized for consistent deployment across environments:

#### Core Services
- `intra365/sts` - Security & Token Service
  - Base: Go Alpine
  - Exposes: gRPC (50051), HTTP (8080)
  - Health checks: /health, /ready
  
- `intra365/consent-manager` - Consent Management & Policy Engine
  - Base: Go Alpine
  - Exposes: gRPC (50052), HTTP (8081)
  - Dependencies: PostgreSQL, NATS
  
- `intra365/mate-registry` - Mate Registry & Discovery
  - Base: Go Alpine
  - Exposes: gRPC (50053), HTTP (8082)
  - Dependencies: PostgreSQL, NATS, JetStream
  
- `intra365/gateway` - API Gateway
  - Base: Node.js Alpine
  - Exposes: HTTP (3000), WebSocket (3001)
  - Dependencies: NATS, Redis (optional cache)
  
- `intra365/llm-proxy` - LLM Integration Layer
  - Base: Python Slim
  - Exposes: gRPC (50054), HTTP (8083)
  - Dependencies: PostgreSQL, NATS
  - Environment: Azure SDK, Anthropic SDK, Google Cloud SDK
  
- `intra365/event-processor` - Event Processing Service
  - Base: Go Alpine
  - Exposes: Metrics (9090)
  - Dependencies: NATS, JetStream

- `intra365/infra-agent` - Infrastructure AI Agent
  - Base: Python Slim
  - Exposes: gRPC (50056), HTTP (8085)
  - Dependencies: PostgreSQL, NATS, Kubernetes API, Vault API
  - LLM Integration: Azure OpenAI, Anthropic Claude
  - Capabilities: Prompt interpretation, reconciliation engine, specialized execution agents

#### Infrastructure Services
- `nats:latest` - NATS Server with JetStream
  - Cluster mode: 3+ replicas
  - Persistent volumes for JetStream
  
- `postgres:16-alpine` - PostgreSQL Database
  - Persistent volumes for data
  - Replication via streaming replication
  
- `pgbouncer:latest` - PostgreSQL Connection Pooler
  - Sits between services and PostgreSQL

- `caddy:latest` - Ingress Controller
  - Automatic HTTPS with Let's Encrypt
  - HTTP/3 support
  - Dynamic multi-tenant routing
  - On-demand TLS certificate provisioning
  - Reverse proxy for all services
  
#### Observability
- `prom/prometheus:latest` - Metrics collection
- `grafana/grafana:latest` - Dashboards and visualization
- `jaegertracing/all-in-one:latest` - Distributed tracing
- `grafana/loki:latest` - Log aggregation

#### Backup & Recovery
- `intra365/backup-manager` - Centralized backup orchestration service
  - Base: Go Alpine
  - Exposes: gRPC (50055), HTTP (8084)
  - Dependencies: PostgreSQL, NATS, S3-compatible storage
  - Features: Scheduled backups, retention policies, backup verification
  
- `velero/velero:latest` - Kubernetes cluster backup and restore
  - Backup Kubernetes resources, volumes, and application state
  - Disaster recovery for entire namespaces
  - Cross-cluster migration support
  
- `restic/restic:latest` - Backup utility for file-level backups
  - Encrypted, deduplicated backups
  - Support for multiple storage backends (S3, Azure Blob, GCS)
  - Incremental backups with efficient storage

**Container Registry**:
- Private registry: `registry.happymates.io`
- Public images: Docker Hub or GitHub Container Registry
- Multi-architecture builds: AMD64, ARM64

**Image Optimization**:
- Multi-stage builds to minimize image size
- Non-root user execution for security
- Distroless or Alpine base images
- Layer caching for faster builds
- Security scanning with Trivy/Snyk

### Infrastructure

**Kubernetes Deployment**:
- Kubernetes 1.28+ recommended for orchestration
- NATS Helm charts for deployment
- CloudNativePG Operator for PostgreSQL management
  - Declarative PostgreSQL cluster management
  - Automated failover and self-healing
  - Continuous backup and point-in-time recovery
  - Connection pooling with PgBouncer integration
- StatefulSets for JetStream persistence
- ConfigMaps for NATS and application configuration
- Self-hosted key vault for secrets management (HashiCorp Vault)
  - Centralized secret storage and rotation
  - Dynamic secret generation
  - Encryption as a service
  - Audit logging for secret access
  - Kubernetes authentication integration

**Kubernetes Resources**:

#### Namespaces
```yaml
- intra365-core        # Core services (STS, consent, registry)
- intra365-infra       # Infrastructure (NATS, PostgreSQL, Redis)
- intra365-observability  # Monitoring stack
- intra365-llm         # LLM integration services
```

#### Deployments
- API Gateway: Deployment with HPA (2-10 replicas)
- STS: Deployment with HPA (2-5 replicas)
- Consent Manager: Deployment with HPA (2-5 replicas)
- Mate Registry: Deployment with HPA (2-5 replicas)
- LLM Proxy: Deployment with HPA (2-10 replicas)
- Event Processor: Deployment (2-3 replicas)

#### StatefulSets
- NATS Cluster: StatefulSet (3 replicas minimum)
- PostgreSQL: Managed by CloudNativePG Operator
  - Cluster resource with 1 primary + 2 replicas
  - Automated backup to S3-compatible storage
  - PgBouncer pooler integrated
- JetStream: Integrated with NATS StatefulSet

#### Services
- Type: ClusterIP for internal services
- Type: LoadBalancer for API Gateway (external access)
- Headless services for StatefulSets (NATS, PostgreSQL)

#### Ingress
- Ingress Controller: **Caddy** (preferred)
  - Automatic HTTPS with Let's Encrypt
  - HTTP/3 support out of the box
  - Dynamic configuration without restarts
  - Built-in reverse proxy with load balancing
  - Simple configuration with Caddyfile
- TLS termination at ingress
- Multi-tenant routing with host-based and path-based rules
- Path-based routing:
  - `/api/*` → API Gateway (with tenant context extraction)
  - `/ws/*` → WebSocket Gateway
  - `/health/*` → Health check endpoints
- Host-based routing for tenant isolation:
  - `tenant1.intra365.io` → Tenant 1 namespace
  - `tenant2.intra365.io` → Tenant 2 namespace
  - `*.intra365.io` → Wildcard with tenant extraction

#### Storage
- StorageClass: High-performance SSD (gp3, Premium SSD)
- PersistentVolumeClaims:
  - NATS JetStream: 100GB per node (expandable)
  - PostgreSQL: 500GB per node (expandable)
  - Volume snapshots for backups

#### ConfigMaps
- NATS configuration (clustering, JetStream, limits)
- Application settings (feature flags, endpoints)
- Logging and observability configuration

#### Secrets
- Managed by self-hosted HashiCorp Vault
- Database credentials (dynamically generated by Vault)
- NATS authentication tokens
- LLM provider API keys (Azure, Anthropic, Google)
- TLS certificates
- OAuth client secrets
- Kubernetes service account authentication to Vault
- Automatic secret rotation and lease renewal

#### Resource Limits
```yaml
API Gateway:
  requests: { cpu: 100m, memory: 128Mi }
  limits: { cpu: 500m, memory: 512Mi }

STS / Consent / Registry:
  requests: { cpu: 200m, memory: 256Mi }
  limits: { cpu: 1000m, memory: 1Gi }

LLM Proxy:
  requests: { cpu: 500m, memory: 512Mi }
  limits: { cpu: 2000m, memory: 2Gi }

NATS:
  requests: { cpu: 500m, memory: 1Gi }
  limits: { cpu: 2000m, memory: 4Gi }

PostgreSQL:
  requests: { cpu: 1000m, memory: 2Gi }
  limits: { cpu: 4000m, memory: 8Gi }
```

#### Autoscaling
- Horizontal Pod Autoscaler (HPA):
  - Metric: CPU utilization > 70%
  - Metric: Memory utilization > 80%
  - Metric: Custom (request rate, queue depth)
- Vertical Pod Autoscaler (VPA): For right-sizing
- Cluster Autoscaler: For node scaling

#### Health Checks
- Liveness probes: Ensure container is running
- Readiness probes: Ensure service can handle traffic
- Startup probes: For slow-starting containers

#### Network Policies
- Default deny all ingress/egress
- Allow traffic between specific namespaces
- Allow traffic to PostgreSQL only from app namespaces
- Allow traffic to NATS from all app namespaces
- Egress to external LLM providers (Azure, Anthropic, Google)

#### Service Mesh (Optional)
- Istio or Linkerd for advanced traffic management
- mTLS between services
- Traffic splitting for canary deployments
- Enhanced observability

**Helm Charts**:
- Custom Helm chart: `intra365/intra365-platform`
- Sub-charts:
  - `intra365/core-services`
  - `intra365/infrastructure`
  - `intra365/observability`
- Values files for different environments (dev, staging, prod)

**GitOps Deployment**:
- ArgoCD or FluxCD for continuous deployment
- Git repository as source of truth
- Automated sync from Git to Kubernetes
- Rollback capabilities
- Progressive delivery strategies

**Docker Compose (Local Development)**:

For local development and testing, a simplified stack using Docker Compose:

```yaml
version: '3.9'

services:
  # Infrastructure
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: intra365
      POSTGRES_USER: intra365
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U intra365"]
      interval: 10s
      timeout: 5s
      retries: 5

  nats:
    image: nats:latest
    command: 
      - "--jetstream"
      - "--store_dir=/data"
      - "--cluster_name=intra365"
    ports:
      - "4222:4222"   # Client
      - "8222:8222"   # HTTP monitoring
      - "6222:6222"   # Cluster
    volumes:
      - nats_data:/data
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8222/healthz"]
      interval: 10s
      timeout: 5s
      retries: 3

  pgbouncer:
    image: pgbouncer/pgbouncer:latest
    environment:
      DATABASES_HOST: postgres
      DATABASES_PORT: 5432
      DATABASES_USER: intra365
      DATABASES_PASSWORD: dev_password
      DATABASES_DBNAME: intra365
    ports:
      - "6432:6432"
    depends_on:
      postgres:
        condition: service_healthy

  # Core Services
  sts:
    image: intra365/sts:dev
    build:
      context: ./services/sts
      dockerfile: Dockerfile
    environment:
      NATS_URL: nats://nats:4222
      DATABASE_URL: postgresql://intra365:dev_password@pgbouncer:6432/intra365
    ports:
      - "8080:8080"
      - "50051:50051"
    depends_on:
      - nats
      - pgbouncer
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/health"]
      interval: 10s
      timeout: 5s
      retries: 3

  consent-manager:
    image: intra365/consent-manager:dev
    build:
      context: ./services/consent-manager
      dockerfile: Dockerfile
    environment:
      NATS_URL: nats://nats:4222
      DATABASE_URL: postgresql://intra365:dev_password@pgbouncer:6432/intra365
    ports:
      - "8081:8081"
      - "50052:50052"
    depends_on:
      - nats
      - pgbouncer
      - sts

  mate-registry:
    image: intra365/mate-registry:dev
    build:
      context: ./services/mate-registry
      dockerfile: Dockerfile
    environment:
      NATS_URL: nats://nats:4222
      DATABASE_URL: postgresql://intra365:dev_password@pgbouncer:6432/intra365
    ports:
      - "8082:8082"
      - "50053:50053"
    depends_on:
      - nats
      - pgbouncer
      - sts

  gateway:
    image: intra365/gateway:dev
    build:
      context: ./services/gateway
      dockerfile: Dockerfile
    environment:
      NATS_URL: nats://nats:4222
      STS_URL: http://sts:8080
    ports:
      - "3000:3000"
      - "3001:3001"
    depends_on:
      - nats
      - sts

  llm-proxy:
    image: intra365/llm-proxy:dev
    build:
      context: ./services/llm-proxy
      dockerfile: Dockerfile
    environment:
      NATS_URL: nats://nats:4222
      DATABASE_URL: postgresql://intra365:dev_password@pgbouncer:6432/intra365
      AZURE_OPENAI_ENDPOINT: ${AZURE_OPENAI_ENDPOINT}
      AZURE_OPENAI_KEY: ${AZURE_OPENAI_KEY}
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      GOOGLE_CLOUD_PROJECT: ${GOOGLE_CLOUD_PROJECT}
    ports:
      - "8083:8083"
      - "50054:50054"
    depends_on:
      - nats
      - pgbouncer
      - consent-manager

  # Observability
  prometheus:
    image: prom/prometheus:latest
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_USERS_ALLOW_SIGN_UP: false
    ports:
      - "3001:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./config/grafana/dashboards:/etc/grafana/provisioning/dashboards
    depends_on:
      - prometheus

  jaeger:
    image: jaegertracing/all-in-one:latest
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: :9411
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"  # UI
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"

volumes:
  postgres_data:
  nats_data:
  prometheus_data:
  grafana_data:
```

**Development Commands**:
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Rebuild and restart a service
docker-compose up -d --build [service-name]

# Stop all services
docker-compose down

# Clean up volumes (warning: data loss)
docker-compose down -v

# Scale a service
docker-compose up -d --scale gateway=3
```

**CI/CD Pipeline**:

Container build and deployment pipeline using GitHub Actions:

```yaml
# .github/workflows/build-and-deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: go, javascript, python
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
      
      - name: Run Dependency Review
        uses: actions/dependency-review-action@v4
        if: github.event_name == 'pull_request'
      
      - name: Check for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

  test:
    name: Test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [sts, consent-manager, mate-registry, gateway, llm-proxy, backup-manager]
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21'
      
      - name: Run tests
        run: |
          cd services/${{ matrix.service }}
          go test -v -race -coverprofile=coverage.out ./...
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: ./services/${{ matrix.service }}/coverage.out
          flags: ${{ matrix.service }}

  build:
    name: Build and Push
    needs: [security-scan, test]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write
    strategy:
      matrix:
        service: [sts, consent-manager, mate-registry, gateway, llm-proxy, backup-manager]
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/happy-mates/intra365-${{ matrix.service }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: ./services/${{ matrix.service }}
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true
          sbom: true
      
      - name: Security scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/happy-mates/intra365-${{ matrix.service }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
          category: trivy-${{ matrix.service }}
      
      - name: Sign container image
        uses: sigstore/cosign-installer@v3
      
      - name: Sign the images with GitHub OIDC
        run: |
          cosign sign --yes ghcr.io/happy-mates/intra365-${{ matrix.service }}:${{ github.sha }}

  update-chef:
    name: Update Chef Configuration
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
        with:
          repository: happy-mates/happy-mates-chef
          token: ${{ secrets.CHEF_REPO_TOKEN }}
      
      - name: Update image tags
        run: |
          ./scripts/update-image-tags.sh \
            --service=${{ matrix.service }} \
            --tag=${{ github.sha }} \
            --environment=dev
      
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.CHEF_REPO_TOKEN }}
          commit-message: "chore: update ${{ matrix.service }} to ${{ github.sha }}"
          title: "Deploy ${{ matrix.service }} ${{ github.sha }} to dev"
          body: |
            Automated image tag update from service deployment
            
            - Service: ${{ matrix.service }}
            - Image: ghcr.io/happy-mates/intra365-${{ matrix.service }}:${{ github.sha }}
            - Source: ${{ github.repository }}@${{ github.sha }}
            - Triggered by: @${{ github.actor }}
          branch: deploy/${{ matrix.service }}-${{ github.sha }}
          labels: deployment, automated

  deploy:
    name: Deploy to Kubernetes
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://intra365.happymates.io
    steps:
      - uses: actions/checkout@v4
      
      - name: Install kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: Update image tags
        run: |
          kubectl set image deployment/sts sts=ghcr.io/happy-mates/intra365-sts:${{ github.sha }} -n intra365-core
          kubectl set image deployment/consent-manager consent-manager=ghcr.io/happy-mates/intra365-consent-manager:${{ github.sha }} -n intra365-core
          kubectl set image deployment/mate-registry mate-registry=ghcr.io/happy-mates/intra365-mate-registry:${{ github.sha }} -n intra365-core
          kubectl set image deployment/gateway gateway=ghcr.io/happy-mates/intra365-gateway:${{ github.sha }} -n intra365-core
          kubectl set image deployment/llm-proxy llm-proxy=ghcr.io/happy-mates/intra365-llm-proxy:${{ github.sha }} -n intra365-llm
          kubectl set image deployment/backup-manager backup-manager=ghcr.io/happy-mates/intra365-backup-manager:${{ github.sha }} -n intra365-infra
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/sts -n intra365-core --timeout=10m
          kubectl rollout status deployment/consent-manager -n intra365-core --timeout=10m
          kubectl rollout status deployment/mate-registry -n intra365-core --timeout=10m
          kubectl rollout status deployment/gateway -n intra365-core --timeout=10m
          kubectl rollout status deployment/llm-proxy -n intra365-llm --timeout=10m
          kubectl rollout status deployment/backup-manager -n intra365-infra --timeout=10m
      
      - name: Run smoke tests
        run: |
          ./scripts/smoke-tests.sh
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          text: |
            Deployment ${{ job.status }}
            Commit: ${{ github.sha }}
            Actor: ${{ github.actor }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}

  backup-verification:
    name: Verify Backup System
    needs: deploy
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Trigger backup verification
        run: |
          curl -X POST https://intra365.happymates.io/api/v1/backup/verify \
            -H "Authorization: Bearer ${{ secrets.BACKUP_API_TOKEN }}" \
            -H "Content-Type: application/json" \
            -d '{"type":"all","notify":true}'
```

**Additional GitHub Workflows**:

```yaml
# .github/workflows/dependabot-auto-merge.yml
name: Dependabot Auto-Merge

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    permissions:
      pull-requests: write
      contents: write
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Enable auto-merge for Dependabot PRs
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

```yaml
# .github/workflows/backup-schedule.yml
name: Scheduled Backup Verification

on:
  schedule:
    - cron: '0 3 * * *'  # 3 AM daily
  workflow_dispatch:

jobs:
  verify-backups:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Verify PostgreSQL backups
        run: |
          ./scripts/verify-postgres-backups.sh
      
      - name: Verify NATS backups
        run: |
          ./scripts/verify-nats-backups.sh
      
      - name: Verify Velero backups
        run: |
          ./scripts/verify-velero-backups.sh
      
      - name: Create issue on failure
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Backup verification failed',
              body: 'Automated backup verification failed. Please investigate immediately.',
              labels: ['backup', 'critical', 'ops']
            })
```

**Container Best Practices**:
- Semantic versioning for image tags
- Immutable tags (SHA-based) for production
- Regular base image updates for security patches
- Automated vulnerability scanning
- Resource limits and requests defined
- Non-root user execution
- Read-only root filesystem where possible
- Minimal attack surface (distroless/alpine)

### High Availability
- NATS cluster with 3+ nodes
- JetStream replication (R3)
- Multi-zone deployment
- PostgreSQL replication (primary + standby)
- Automated failover for database
- Backup and disaster recovery

### Monitoring
- Health check endpoints
- Prometheus scraping
- Distributed tracing
- Log aggregation
- Alert management

## Infrastructure Deployment Philosophy

### AI Agent-Based Infrastructure Engineering

The intra365 infrastructure deployment **does NOT use traditional Infrastructure as Code (IaC) tools** like Terraform, Pulumi, or CloudFormation. Instead, deployment is managed through **AI agent-based prompt engineering** with intelligent reconciliation.

**Core Principles**:

1. **Prompt-Based Desired State**: Infrastructure requirements are expressed as natural language prompts describing the desired state
2. **AI Agent Orchestration**: Specialized AI agents interpret prompts and execute necessary actions
3. **Resilient Reconciliation**: Continuous reconciliation loop ensures actual state matches desired state
4. **Self-Healing**: Agents detect drift and automatically remediate without human intervention
5. **Conversational Infrastructure**: Changes requested through natural language, not code

**Architecture**:

```
┌─────────────────────────────────────────────────────────────┐
│              Infrastructure Control Plane                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │         Infrastructure AI Agent (IAA)              │    │
│  │  - Natural language prompt interpreter             │    │
│  │  - Desired state reasoning                         │    │
│  │  - Multi-step plan generation                      │    │
│  │  - Context-aware decision making                   │    │
│  └────────────────┬───────────────────────────────────┘    │
│                   │                                          │
│  ┌────────────────┴───────────────────────────────────┐    │
│  │         Reconciliation Engine                      │    │
│  │  - Continuous state observation                    │    │
│  │  - Drift detection (desired vs actual)             │    │
│  │  - Intelligent remediation planning                │    │
│  │  - Conflict resolution                             │    │
│  │  - Rollback on failure                             │    │
│  └────────────────┬───────────────────────────────────┘    │
│                   │                                          │
│  ┌────────────────┴───────────────────────────────────┐    │
│  │         Execution Layer                            │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │ kubectl  │  │   NATS   │  │  Vault   │        │    │
│  │  │  Agent   │  │  Agent   │  │  Agent   │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │   DB     │  │ Storage  │  │ Network  │        │    │
│  │  │  Agent   │  │  Agent   │  │  Agent   │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └────────────────────────────────────────────────────┘    │
│                   │                                          │
│  ┌────────────────┴───────────────────────────────────┐    │
│  │         State Store (PostgreSQL + NATS)            │    │
│  │  - Desired state history                           │    │
│  │  - Actual state snapshots                          │    │
│  │  - Reconciliation events                           │    │
│  │  - Agent decision logs                             │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
   Kubernetes            PostgreSQL              NATS
     Cluster          CloudNativePG           JetStream
```

**How It Works**:

#### 1. Prompt-Based Requests

Infrastructure changes are expressed as natural language prompts:

```
"Deploy the intra365 consent-manager service with 3 replicas, 
allocate 512MB memory per pod, enable autoscaling from 3 to 10 
replicas based on 70% CPU, expose on port 8081, connect to the 
intra365-postgres database, and ensure it has access to NATS 
in the intra365-infra namespace."
```

```
"Scale the PostgreSQL cluster to 5 instances for high availability,
enable point-in-time recovery with 60-day retention, set up 
cross-region backup replication to eu-west-1, and configure 
PgBouncer pooler with 100 max connections per instance."
```

```
"Create a new NATS JetStream with name 'consent-events', 
configure 3 replicas, set retention policy to 30 days or 
100GB max, enable deduplication with 5-minute window, 
and create consumers for the consent-manager and audit services."
```

#### 2. AI Agent Processing

The Infrastructure AI Agent (IAA):

1. **Interprets** the prompt using LLM reasoning
2. **Validates** the request against current state and policies
3. **Plans** a multi-step execution sequence
4. **Estimates** impact and resource requirements
5. **Generates** executable actions for specialized agents
6. **Monitors** execution progress
7. **Handles** errors with intelligent retry and rollback

**Agent Capabilities**:
- Context-aware understanding of infrastructure relationships
- Knowledge of best practices and anti-patterns
- Ability to ask clarifying questions when ambiguous
- Learn from past reconciliation patterns
- Predict potential issues before execution

#### 3. Resilient Reconciliation Loop

**Continuous Reconciliation**:
```
┌─────────────────────────────────────────────────┐
│  1. Observe Current State                       │
│     - Query Kubernetes API                      │
│     - Check PostgreSQL cluster status           │
│     - Verify NATS configuration                 │
│     - Read Vault secrets state                  │
└───────────────────┬─────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────┐
│  2. Compare with Desired State                  │
│     - Semantic diff (not just text comparison)  │
│     - Understand intent behind configuration    │
│     - Identify meaningful drift vs noise        │
└───────────────────┬─────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────┐
│  3. Detect Drift                                │
│     - Missing resources                         │
│     - Configuration mismatches                  │
│     - Scale discrepancies                       │
│     - Security policy violations                │
└───────────────────┬─────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────┐
│  4. Plan Remediation                            │
│     - Determine minimal change set              │
│     - Order operations by dependency            │
│     - Evaluate blast radius                     │
│     - Plan rollback strategy                    │
└───────────────────┬─────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────┐
│  5. Execute Changes                             │
│     - Apply changes incrementally               │
│     - Wait for health checks                    │
│     - Validate each step                        │
│     - Emit progress events                      │
└───────────────────┬─────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────────┐
│  6. Verify & Report                             │
│     - Confirm desired state achieved            │
│     - Update state store                        │
│     - Notify stakeholders                       │
│     - Learn from outcome                        │
└───────────────────┬─────────────────────────────┘
                    │
                    └──────────► Loop (every 30s)
```

**Resilience Features**:

- **Graceful Degradation**: If agent cannot fully reconcile, it achieves partial state and reports
- **Conflict Resolution**: Multiple concurrent changes are merged intelligently
- **Failure Recovery**: Automatic rollback on error, with human approval for risky operations
- **Rate Limiting**: Prevents thrashing from rapid reconciliation loops
- **Circuit Breaking**: Pauses reconciliation after repeated failures
- **Idempotency**: Safe to run multiple times without side effects

#### 4. Specialized Execution Agents

Each infrastructure domain has a specialized agent:

**Kubernetes Agent**:
- Manages deployments, services, configmaps, secrets
- Understands pod lifecycles and rollout strategies
- Handles HPA, VPA, PDB configurations
- Applies network policies and RBAC

**Database Agent** (CloudNativePG):
- Creates and scales PostgreSQL clusters
- Manages backup schedules and retention
- Configures replication and failover
- Handles connection pooling (PgBouncer)
- Executes schema migrations (when requested)

**NATS Agent**:
- Configures NATS servers and clustering
- Manages JetStream streams and consumers
- Sets up KV stores and Object stores
- Handles subject permissions and ACLs

**Vault Agent**:
- Creates and rotates secrets
- Manages policies and authentication methods
- Configures dynamic secret engines
- Handles encryption and PKI

**Storage Agent**:
- Provisions persistent volumes
- Manages storage classes and snapshots
- Handles volume expansions
- Configures backup destinations

**Network Agent**:
- Manages ingress controllers and routes
- Configures load balancers
- Sets up DNS records
- Handles TLS certificate management

#### 5. State Management

**Desired State Store**:
```json
{
  "requestId": "uuid",
  "timestamp": "2025-10-29T10:00:00Z",
  "prompt": "Deploy consent-manager with 3 replicas...",
  "interpretation": {
    "resource": "deployment",
    "name": "consent-manager",
    "namespace": "intra365-core",
    "replicas": 3,
    "container": {
      "image": "ghcr.io/happy-mates/intra365-consent-manager:v1.2.0",
      "resources": {
        "memory": "512Mi",
        "cpu": "200m"
      }
    },
    "autoscaling": {
      "enabled": true,
      "minReplicas": 3,
      "maxReplicas": 10,
      "targetCPU": 70
    },
    "dependencies": ["postgresql", "nats"]
  },
  "executionPlan": [
    {"step": 1, "action": "verify_dependencies", "agent": "kubectl"},
    {"step": 2, "action": "create_configmap", "agent": "kubectl"},
    {"step": 3, "action": "create_deployment", "agent": "kubectl"},
    {"step": 4, "action": "create_service", "agent": "kubectl"},
    {"step": 5, "action": "create_hpa", "agent": "kubectl"},
    {"step": 6, "action": "wait_for_ready", "agent": "kubectl"}
  ],
  "status": "completed",
  "appliedAt": "2025-10-29T10:05:00Z"
}
```

**Reconciliation Events**:
```json
{
  "eventId": "uuid",
  "timestamp": "2025-10-29T11:30:00Z",
  "type": "drift_detected",
  "resource": "deployment/consent-manager",
  "namespace": "intra365-core",
  "drift": {
    "field": "spec.replicas",
    "desired": 3,
    "actual": 2,
    "reason": "pod_evicted"
  },
  "remediation": {
    "action": "scale_up",
    "agent": "kubectl",
    "status": "in_progress"
  }
}
```

#### 6. Human Interaction

**Approval Workflows**:
- **Auto-Apply**: Low-risk changes (scaling within limits, config updates)
- **Review Required**: Medium-risk changes (new services, storage expansion)
- **Explicit Approval**: High-risk changes (database migrations, cluster upgrades)

**Conversational Interface**:
```
User: "Why is the consent-manager only running 2 replicas?"

IAA: "I detected that one pod was evicted due to node pressure at 
11:28 AM. I'm currently scaling back to 3 replicas. The new pod 
is in 'Pending' state waiting for node resources. ETA: 2 minutes."

User: "Can you increase the HPA max to 15?"

IAA: "I can do that. This will allow the consent-manager to scale 
up to 15 replicas under high load. Current resource quotas support 
this. Shall I proceed?"

User: "Yes"

IAA: "Done. Updated HPA maxReplicas to 15. Change will take effect 
on next scaling event."
```

**Observability**:
- Real-time status dashboard showing reconciliation state
- Event stream of all agent actions
- Audit trail of prompts and executions
- Metrics on reconciliation loop performance
- Alerts when manual intervention needed

#### 7. Integration with GitHub

**GitOps Augmentation** (Not Replacement):
- Chef repository still holds configuration **context**
- Git commits trigger reconciliation events
- AI agent interprets configuration changes as prompts
- Pull requests include AI-generated impact analysis
- Rollback via conversation: "Revert the last change to consent-manager"

**GitHub Actions Integration**:
```yaml
- name: AI Agent Deployment
  run: |
    # Submit prompt to Infrastructure AI Agent
    curl -X POST https://infra-agent.intra365.io/api/v1/apply \
      -H "Authorization: Bearer ${{ secrets.IAA_TOKEN }}" \
      -H "Content-Type: application/json" \
      -d '{
        "prompt": "Deploy version ${{ github.sha }} of ${{ matrix.service }} to production",
        "context": {
          "commit": "${{ github.sha }}",
          "actor": "${{ github.actor }}",
          "pr": "${{ github.event.pull_request.number }}"
        },
        "approval": "auto"
      }'
    
    # Watch reconciliation progress
    ./scripts/watch-reconciliation.sh
```

### Benefits of AI Agent Approach

1. **Natural Language**: No need to learn Terraform/Pulumi syntax
2. **Intelligent Adaptation**: Agent understands intent, not just configuration
3. **Self-Healing**: Continuous reconciliation without manual intervention
4. **Contextual Decisions**: Agent considers broader system state
5. **Reduced Toil**: Conversational changes instead of YAML engineering
6. **Learning System**: Improves over time from past reconciliations
7. **Accessible**: Operations teams without deep IaC knowledge can manage infrastructure
8. **Flexible**: Adapts to changing requirements without code refactoring

### Implementation Stack

**AI Agent Core**:
- LLM Backend: Azure OpenAI GPT-4, Claude 3.5 Sonnet (for complex reasoning)
- Agent Framework: LangChain or custom Go-based agent with LLM integration
- Vector DB: For infrastructure knowledge base and past decision retrieval
- Prompt Templates: Curated prompts for common infrastructure patterns

**Execution**:
- Kubernetes Client SDK (Go)
- CloudNativePG Operator API
- NATS Client
- Vault API
- AWS/Azure/GCP SDKs (for cloud resources)

**State & Observability**:
- PostgreSQL: Desired state, reconciliation history
- NATS JetStream: Real-time event stream
- Prometheus: Agent performance metrics
- Grafana: Reconciliation dashboards

**Security**:
- Role-based agent permissions
- Approval gates for sensitive operations
- Audit logging of all agent decisions
- Secrets in Vault (not in prompts)
- Rate limiting to prevent abuse

## GitHub Platform Integration Summary

The intra365 project leverages GitHub's complete platform capabilities:

### Development Workflow
- **Git Repository**: All code, configurations, and documentation version controlled
- **Branch Protection**: Main branch requires PR reviews, passing CI, and code owner approval
- **Pull Requests**: All changes go through peer review before merging
- **Code Owners**: Automatic assignment of reviewers based on file paths
- **Merge Queue**: Automated PR merging with CI validation to prevent broken builds

### Security & Compliance
- **GitHub Advanced Security**: 
  - CodeQL for automated vulnerability detection in Go, TypeScript, Python code
  - Secret scanning prevents accidental credential commits
  - Dependency scanning alerts on vulnerable packages
  - Security advisories for responsible vulnerability disclosure
- **Dependabot**: Automated dependency updates with auto-merge for patch releases
- **Supply Chain Security**: SBOM generation, dependency graph, license compliance
- **Container Signing**: Cosign with GitHub OIDC for container image provenance

### Task Management
- **GitHub Issues**: All bugs, features, and tasks tracked as issues
- **GitHub Projects**: Sprint planning, roadmaps, and release tracking
- **Milestones**: Group work by version releases
- **Labels**: Categorize by type (bug, enhancement, security, backup, ops)
- **Issue Templates**: Standardized formats for bug reports, feature requests, security issues

### Automation & CI/CD
- **GitHub Actions**: 
  - Automated testing on every PR
  - Security scanning (CodeQL, Trivy, TruffleHog)
  - Multi-architecture container builds (AMD64, ARM64)
  - Automated deployment to Kubernetes
  - Scheduled backup verification
  - Dependabot auto-merge for patches
- **GitHub Container Registry**: Store all container images at `ghcr.io/happy-mates/`
- **Environments**: Protected deployment targets with approval gates

### Observability
- **Code Coverage**: Codecov integration tracks test coverage trends
- **Security Alerts**: Centralized view of vulnerabilities across all repos
- **Deployment Status**: GitHub Deployments API tracks release history
- **Audit Log**: Complete history of all repository actions

### Collaboration
- **GitHub Discussions**: RFCs, design discussions, community Q&A
- **Wiki**: Additional documentation and guides
- **Release Notes**: Auto-generated changelogs from commits and PRs
- **Notifications**: Email, Slack, webhooks for important events

## Next Steps

1. **Infrastructure AI Agent**: Build the core Infrastructure AI Agent (IAA) service with prompt interpretation and reconciliation engine
2. **Proof of Concept**: Deploy basic NATS infrastructure using IAA with natural language prompts
3. **Specialized Agents**: Implement Kubernetes, Database, NATS, and Vault execution agents
4. **Token Service**: Implement STS for authentication foundation
5. **Reference Implementation**: Create sample Mates demonstrating patterns
6. **SDK Development**: Build client libraries for popular languages
7. **Documentation**: Detailed guides for Mate developers and IAA prompt patterns
8. **Testing**: Load testing, security audits, and IAA resilience testing
9. **Backup System**: Set up Velero, configure CloudNativePG backups, implement backup-manager service
10. **GitHub Setup**: Configure branch protection, enable Advanced Security, set up project boards
11. **Community**: Open-source release and developer onboarding
