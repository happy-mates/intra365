# Intra365 Project Instructions

## Project Overview

**Intra365** is an enterprise integration framework enabling "Mates" (applications, services, AI agents) to communicate within the Happy Mates ecosystem. This is a **multi-workspace project** with four related repositories:

- **happy-mates-intra365**: Core integration framework specs (current repo)
- **happy-mates-chef**: GitOps deployment orchestration for AKS
- **happy-mates-web**: Next.js marketing/web application
- **happy-mates-knowledge**: Docusaurus knowledge base

## Architecture Philosophy

### Zero-Trust Security Model
Every access request is verified. No implicit trust based on network location or prior authentication.

**Key Principles**:
- Never trust, always verify
- Assume breach (minimize blast radius)
- Continuous verification with risk-based adaptive authentication
- Behavioral biometrics and anomaly detection

**Risk Scoring**: 0-100 scale determines authentication requirements:
- 0-30 (Low): Standard auth
- 31-60 (Medium): TOTP/Push MFA required
- 61-80 (High): Hardware key + biometric required  
- 81-100 (Critical): Block + security review

### Core Technologies
- **NATS**: Central messaging backbone for Mate-to-Mate communication
- **NATS JetStream**: Persistent messaging, event sourcing, KV store
- **PostgreSQL**: Primary data store for consent, policies, user data
- **JWT Tokens**: Authentication with device fingerprinting
- **OpenTelemetry**: Distributed tracing across all services

## Repository Structure & File Organization

### happy-mates-intra365 (Specifications)
```
happy-mates-intra365/
├── ARCHITECTURE.md              # 3200+ lines: Complete system architecture
├── FUNCTIONAL_REQUIREMENTS.md   # 630+ lines: Technology-agnostic functional specs
├── NON_FUNCTIONAL_REQUIREMENTS.md  # 700+ lines: Performance/quality metrics
├── ZERO_TRUST_SECURITY.md       # 800+ lines: Security architecture
└── references/
    ├── RAISE2.0_Implementation_Guide.md  # DoD DevSecOps guide
    └── images/                  # Extracted PDF figures
```

**Purpose**: This repo contains **NO CODE** - only comprehensive specifications defining the "what" (functional requirements) and "how well" (non-functional requirements) without implementation details.

### happy-mates-chef (Deployment Orchestration)
GitOps-based Kubernetes deployment using Azure AKS with convention-over-configuration:
- Repository name = Azure Key Vault name = Application name
- Automatic secret discovery via Key Vault CSI driver
- Multi-domain support (MSOPS, INTRA)
- GitHub Actions reusable workflows

### happy-mates-web (Marketing Site)
Next.js 15.5+ with App Router:
- **Stack**: React 19, TypeScript, Tailwind CSS 4, shadcn/ui (Radix UI)
- **Deploy**: Vercel with auto-sync from v0.app
- **Package Manager**: pnpm 10.14.0
- **Components**: `/components/` for features, `/components/ui/` for shadcn/ui primitives

### happy-mates-knowledge (Documentation)
Docusaurus 3 knowledge base with structured learning paths:
- **Numbering**: Directories use 3-digit prefixes (010-, 020-, etc.)
- **File Naming**: kebab-case only (`my-article-title.md`)
- **Categories**: 10 top-level sections from subjects to accessibility

## Key Design Patterns

### 1. Consent Management
**Three-layer data strategy** for consent records:
- PostgreSQL: Source of truth (full history, audit trail)
- JetStream KV: Active consent cache (performance)
- Event Stream: Real-time consent change notifications

**Adaptive Consent**: Risk score determines granularity
- High-risk sessions → More specific consent required
- New devices → Explicit re-consent for sensitive data
- Anomalous behavior → Purpose-specific consent only

### 2. Anomaly Detection (4 Categories)
1. **Device**: New devices, fingerprint changes, jailbroken devices
2. **Location**: Impossible travel, new geographies, VPN/proxy use
3. **Behavioral**: Unusual time/frequency, typing/mouse patterns, API usage
4. **Threat Intelligence**: Known malicious IPs, credential stuffing, bot behavior

**Performance Targets** (NON_FUNCTIONAL_REQUIREMENTS.md):
- Login anomaly detection: <50ms
- Behavioral analysis: <100ms  
- False positive rate: <5%
- False negative rate: <1%

### 3. Service Repository Pattern
Each service is self-contained in its own repo:
- `intra365-sts`: Security & Token Service
- `intra365-consent-manager`: Consent & Policy Engine
- `intra365-mate-registry`: Mate Registry & Discovery
- `intra365-gateway`: API Gateway
- `intra365-llm-proxy`: LLM Integration Layer

### 4. RAISE 2.0 DevSecOps (references/)
DoD/Navy container security framework:
- **RPOC**: RAISE Platform of Choice certification
- **Security Gates**: SAST, DAST, CSS, SBOM, secrets detection
- **Residual Risk**: Must not exceed Moderate
- **Quarterly Reviews**: AO, SCA, RPOC ISSM continuous authorization

## Development Workflow

### Requirements Documents (This Repo)
When updating specs, maintain separation:
- **FUNCTIONAL_REQUIREMENTS.md**: What the system must do (technology-agnostic)
- **NON_FUNCTIONAL_REQUIREMENTS.md**: Performance targets, SLAs, quality metrics
- **ZERO_TRUST_SECURITY.md**: Security architecture patterns and workflows

**Never mix**: Keep "what" (FR) separate from "how well" (NFR) and "how secure" (ZERO_TRUST).

### Cross-Workspace References
When referencing implementation patterns:
- Chef deployments → `happy-mates-chef/`
- Web application → `happy-mates-web/`
- Documentation → `happy-mates-knowledge/docs/`

### File Naming Conventions
- **Knowledge Base**: kebab-case with number prefixes (`01-introduction.md`)
- **Specifications**: SCREAMING_SNAKE_CASE.md for major docs
- **Maximum filename length**: 50 characters
- **No underscores in knowledge base**: Use hyphens only

### Package Management
- **Web/Knowledge**: pnpm 10.14.0 (NOT npm or yarn)
- **Monorepo**: Turborepo for build orchestration (future)
- **TypeScript**: Version 5+ across all projects

## Important Context

### User Profile & Accessibility
This is a **neurodiversity-friendly project** (Happy Mates focuses on autism employment):
- Clear, structured documentation without ambiguity
- Step-by-step instructions with explicit outcomes
- Visual diagrams for complex flows
- Accessibility considerations in all design decisions

### Security-First Approach
Every feature must consider:
1. How can this be abused?
2. What's the blast radius if compromised?
3. How do we detect anomalies?
4. What consent is required?
5. How do we audit access?

### Real-World Reference: intra365 @ Nexi
Implementation patterns reference the production intra365 system at Nexi:
- `intra365-chef` (formerly supermate-chef)
- `intra365-web` (formerly support-agent-supermate)
- `intra365-browser` (formerly supermate-browser)
- `intra365-office` (formerly office-addin-mate)

## Common Commands

### happy-mates-web
```bash
pnpm dev          # Start Next.js dev server
pnpm build        # Production build
pnpm lint         # ESLint
```

### happy-mates-knowledge
```bash
pnpm start        # Start Docusaurus dev server
pnpm build        # Build static site
pnpm serve        # Serve production build
```

### happy-mates-intra365
No build commands - specifications only. When extracting images from PDFs:
```bash
pdftotext document.pdf output.txt        # Extract text
pdfimages -png document.pdf images/fig   # Extract images
```

## Critical Files to Reference

**Architecture & Design**:
- `ARCHITECTURE.md` lines 1-200: High-level system architecture
- `ZERO_TRUST_SECURITY.md` lines 1-150: Zero-trust principles & auth flow

**Requirements**:
- `FUNCTIONAL_REQUIREMENTS.md` lines 1-150: FR-1 (Auth) through FR-4 (Discovery)
- `NON_FUNCTIONAL_REQUIREMENTS.md`: NFR-4.1.2 through NFR-4.4.5 (Security performance)

**Implementation Patterns**:
- `happy-mates-chef/HAPPY_MATES_FRAMEWORK.md`: Monorepo structure (lines 32-69)
- `happy-mates-chef/IMPLEMENTATION_PLAN.md`: 48-week roadmap (lines 178-250)

## What NOT to Do

❌ Don't implement code in `happy-mates-intra365` - it's specs only
❌ Don't use npm/yarn - always pnpm in web projects
❌ Don't mix functional and non-functional requirements
❌ Don't skip anomaly detection in authentication flows
❌ Don't assume trust - verify explicitly every time
❌ Don't use underscores in knowledge base filenames
❌ Don't create generic content - reference actual project patterns

## Getting Immediate Context

**For security questions**: Read `ZERO_TRUST_SECURITY.md` lines 1-300
**For auth requirements**: Read `FUNCTIONAL_REQUIREMENTS.md` FR-1.1 through FR-1.8
**For performance targets**: Read `NON_FUNCTIONAL_REQUIREMENTS.md` NFR-4
**For deployment patterns**: Check `happy-mates-chef/HAPPY_MATES_FRAMEWORK.md`
**For DoD security**: Review `references/RAISE2.0_Implementation_Guide.md`
