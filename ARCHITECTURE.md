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
| Connection Pooling | PgBouncer |
| Token Service | Custom service with NATS JWT |
| Identity Integration | Entra ID SDK, GitHub OAuth |
| LLM Providers | Azure AI Foundry, Anthropic Claude, Google Gemini |
| Service Discovery | NATS Service API |
| Observability | Prometheus, Grafana, Jaeger, Loki |
| API Gateway | Custom (Go/Node.js) + NATS |
| Persistence | PostgreSQL (primary), JetStream KV (cache) |
| Languages | Go (core services), TypeScript/Node.js (Mates), Python (ML/AI) |

## Deployment Considerations

### Infrastructure
- Kubernetes recommended for orchestration
- NATS Helm charts for deployment
- StatefulSets for JetStream persistence
- StatefulSets for PostgreSQL with persistent volumes
- PostgreSQL Operator for automated management
- ConfigMaps for NATS configuration

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

## Next Steps

1. **Proof of Concept**: Implement basic NATS infrastructure with token service
2. **Reference Implementation**: Create sample Mates demonstrating patterns
3. **SDK Development**: Build client libraries for popular languages
4. **Documentation**: Detailed guides for Mate developers
5. **Testing**: Load testing and security audits
6. **Community**: Open-source release and developer onboarding
