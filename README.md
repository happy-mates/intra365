# Happy Mates - Intra365

> Reference architecture and production patterns from the intra365 platform

## Overview

This repository contains reference implementations and patterns extracted from the production intra365 platform at Nexi. These patterns serve as proven blueprints for building enterprise-grade applications in the Happy Mates ecosystem.

## What's Inside

The intra365 reference architecture demonstrates:

- **Multi-application Workspace**: Turborepo monorepo managing web apps, Office add-ins, browser extensions, and SharePoint integrations
- **Modern Authentication**: Dual-layer auth with MSAL + JWT httpOnly cookies
- **GitOps Deployment**: Convention-based Kubernetes deployments with Azure Key Vault CSI integration
- **OpenTelemetry Observability**: Full-stack distributed tracing and monitoring
- **Next.js 15 + React 19**: Latest framework patterns with Server Components and Server Actions
- **Enterprise Integration**: SharePoint Framework (SPFx), Office.js, and browser extension patterns

## Key Patterns

### 1. GitOps Deployment (intra365-chef)
- Convention-based secret discovery (Key Vault name = repository name)
- YAML-based configuration management
- Azure Key Vault CSI driver integration
- Multi-domain support (MSOPS/INTRA)

### 2. Web Application (intra365-web)
- Next.js 15 with App Router
- MSAL + JWT authentication
- Prisma ORM with PostgreSQL
- IndexedDB for client-side persistence
- OpenTelemetry integration

### 3. Browser Extension (intra365-browser)
- Manifest V3 with @crxjs/vite-plugin
- React 19 + TypeScript
- Cross-browser compatibility

### 4. Office Add-ins (intra365-office)
- Vite-based development
- Office.js integration
- MSAL authentication
- Modern React patterns

### 5. SharePoint Integration
- SPFx web parts and extensions
- Modern SharePoint patterns
- Fluent UI integration

### 6. OpenTelemetry Observability
- Server-side: @vercel/otel
- Client-side: Custom browser telemetry
- Correlation ID tracking
- Error capture and user context

## Usage

This repository serves as a reference for implementing the [Happy Mates Framework](../chef). Refer to specific patterns when building components of the Happy Mates ecosystem.

## Related Repositories

- [happy-mates/chef](../chef) - GitOps deployment orchestration
- [happy-mates/knowledge](../knowledge) - Documentation and knowledge base

## Technology Stack

- **Frontend**: React 19, Next.js 15, TypeScript 5
- **Backend**: Node.js, Prisma, PostgreSQL
- **Build Tools**: Turborepo, Vite, pnpm
- **Cloud**: Azure (AKS, Key Vault, Container Registry)
- **Observability**: OpenTelemetry, Application Insights
- **Auth**: MSAL, JWT (jose)

## License

This is a public reference repository for the Happy Mates initiative.

---

**Part of the Happy Mates Initiative** - Connecting young people with autism and IT interests to companies through innovative bootcamps.
