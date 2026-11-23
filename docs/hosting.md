# Hosting & Deployment Specification: Smart Task Manager

> **Document:** `docs/hosting.md`  
> **Phase:** Phase 2 – Software Architecture & Infrastructure Specification  
> **Target Application:** Smart Task Manager (Client-side React 18+ / TypeScript / Vite SPA)  
> **Primary Deployment Platform:** GitHub Pages  
> **Evaluated Alternatives:** Cloudflare Pages, Vercel  
> **Status:** Approved / Specification Complete  
> **Last Updated:** 2026-11-21  
> **Related Documents:** [Architecture Specification](architecture.md), [Requirements](requirements.md), [ADR-0001: Architecture Variant Selection](adr/0001-architecture-variant-selection.md)

---

## 1. Executive Summary

This specification establishes the production hosting, delivery model, and deployment strategy for the **Smart Task Manager** web application.

As established in [ADR-0001](adr/0001-architecture-variant-selection.md), the Smart Task Manager is implemented as a **Pure Client-Side Single-Page Application (SPA)** built with React, TypeScript, and Vite, utilizing client-local storage (IndexedDB via Dexie.js) for persistent state. Because the system contains no server-side application logic or database containers for the MVP, the hosting infrastructure must provide high-availability static asset delivery, deterministic client-side routing support, and guaranteed zero operational cost.

Following a comparative evaluation of leading static edge hosting platforms (**GitHub Pages**, **Cloudflare Pages**, and **Vercel**), **GitHub Pages** has been selected as the **primary deployment platform**. Because the project codebase already resides in a GitHub repository, GitHub Pages provides an optimal zero-friction workflow: automated deployments via native GitHub Actions without third-party account registration, external API token management, or team authorization roadblocks.

---

## 2. Architectural Hosting Requirements & Constraints

The selection of a hosting and deployment strategy for the Smart Task Manager is governed by five core constraints:

### 2.1 Client-Side SPA Routing
Single-Page Applications manage routing in the browser via the HTML5 History API (`pushState`, `replaceState`). The static host must serve the SPA entrypoint (`index.html`) across client-side view paths so direct navigations and refreshes execute without breaking navigation.

### 2.2 Guaranteed Zero Operational Cost
As an academic semester project, hosting must cost **$0.00/month** with zero risk of overage charges. Solutions requiring an active credit card with post-threshold billing risks are disqualified.

### 2.3 Unrestricted Multi-Author Collaboration
The repository is developed collaboratively by student teammates. The hosting platform must support automated deployments triggered by any repository contributor without per-seat licensing walls or commit-author paywalls.

### 2.4 Zero-Friction Operational Footprint
The deployment pipeline must remain simple and self-contained within the existing repository infrastructure, avoiding third-party dashboards, external API secrets, or cross-platform permission management.

### 2.5 Security & HTTPS Delivery
The hosting environment must provide automated SSL/TLS certificate management (TLS 1.3), enforce HTTPS redirects, and deliver assets reliably via global CDN caching.

---

## 3. Comparative Evaluation of Hosting Options

Three industry-standard static hosting platforms were evaluated against the project's requirements: **GitHub Pages**, **Cloudflare Pages**, and **Vercel (Hobby)**.

### 3.1 Feature & Constraint Matrix

| Evaluation Dimension | GitHub Pages | Cloudflare Pages (Free) | Vercel (Hobby) |
| :--- | :--- | :--- | :--- |
| **Architectural Fit** | **Selected Primary Choice** | **Evaluated Alternative** | **Disqualified** |
| **Repository Integration** | **Native** (Same platform, zero external setup) | External (Requires Cloudflare account & API keys) | External (Requires Vercel account linking) |
| **Monthly Hosting Cost** | **$0.00** | **$0.00** | **$0.00** (Personal use only) |
| **Team Collaboration on Free Tier** | **Full repo collaborators**; native GitHub permissions | Unlimited collaborators; no author restrictions | ❌ **Blocks non-owner commits** on private repos |
| **SPA Routing Support** | Handled via standard fallback (`dist/404.html`) | Native HTTP 200 rewrites (`_redirects`) | Native HTTP 200 rewrites (`vercel.json`) |
| **Bandwidth Allowance** | 100 GB / month (Soft limit, ample for MVP) | Unmetered static bandwidth | 100 GB / month |
| **Build & Deploy Mechanism** | Official GitHub Actions (`deploy-pages`) via OIDC | GitHub Actions + Wrangler or Cloudflare CI | Vercel CI |
| **Base URL Structure** | Subpath domain (`/<repo-name>/`) | Root domain (`/`) | Root domain (`/`) |
| **Credit Card Required?** | **No** | **No** | **No** |

---

### 3.2 Analysis of Candidates

#### Candidate A: GitHub Pages (Selected Primary Choice)
* **Strengths:**
  1. *Zero External Footprint:* Runs entirely within existing GitHub infrastructure; requires no third-party accounts, external dashboards, or shared secret maintenance.
  2. *Secure Native CI/CD:* Deploys directly via official GitHub Actions (`actions/deploy-pages`) using short-lived OpenID Connect (OIDC) tokens.
  3. *Unrestricted Team Workflow:* Any authorized student collaborator pushing to the repository can trigger CI verification and deployment.
  4. *Zero Financial Risk:* Standard academic usage is completely free without credit card requirements.
* **Trade-Offs & Mitigations:**
  1. *SPA Routing:* GitHub Pages defaults to file-system matching and returns a 404 for deep paths. Mitigated cleanly for the client-side SPA by copying `dist/index.html` to `dist/404.html` during the build step, allowing the browser to load the SPA and resolve the route client-side.
  2. *Subpath Base URL:* Deploys to `https://<owner>.github.io/<repo-name>/`. Mitigated by configuring Vite's `base` property to match the repository name (`base: '/Paise_ProjektWS26/'`).

#### Candidate B: Cloudflare Pages (Evaluated Alternative)
* **Strengths:**
  1. Provides native HTTP 200 rewrites via `_redirects` and serves from root paths (`<project>.pages.dev`).
  2. Generates ephemeral pull request preview URLs.
* **Trade-Offs:**
  1. Requires team members to register an external Cloudflare account, configure API tokens, and store sensitive secrets in the GitHub repository.
  2. Adds unnecessary operational complexity when the code already lives natively on GitHub.

#### Candidate C: Vercel (Rejected)
* **Disqualifying Factor:** Vercel's free Hobby plan blocks builds from commits authored by team members other than the account owner on private repositories. Removing this barrier requires a $20/user/month Pro plan, which violates our zero-cost requirement.

---

## 4. Architecture Decision & Rationale

**Selected Platform:** **GitHub Pages** (formally recorded in [ADR-0001](adr/0001-architecture-variant-selection.md)).

**Rationale:**
1. **Direct Repository Integration:** Deploying on the same platform that hosts the source code eliminates external accounts, third-party dashboards, and API key management.
2. **Inclusive Student Collaboration:** All team members can push code, create branches, and deploy without encountering collaborator licensing gates.
3. **Guaranteed Zero Cost:** Complete financial predictability with zero risk of overage charges or subscription fees.
4. **Clean Decoupling:** Because the Vite build produces standard static client assets (`dist/`), migrating to an edge host (such as Cloudflare Pages) in the future requires only redirecting deployment output.

---

## 5. Technical Configuration Specification

### 5.1 Vite Application Configuration (`vite.config.ts`)
To support GitHub Pages hosting under the repository subpath, Vite is configured with the matching base path:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  // GitHub Pages subpath deployment
  base: '/Paise_ProjektWS26/',
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### 5.2 SPA Routing Fallback
Because GitHub Pages serves static files, requesting a route such as `/tasks` directly would result in a GitHub 404 page. The deployment pipeline copies `dist/index.html` to `dist/404.html` post-build. When a user requests a direct subpath, GitHub Pages serves `404.html` (the SPA application), which initializes React Router / application routing and renders the correct view seamlessly.

---

## 6. Continuous Integration & Deployment Pipeline

Deployments are automated through a streamlined GitHub Actions pipeline. Rather than maintaining complex bespoke deployment scripts, the workflow standardizes on standard stages:

### 6.1 Pipeline Stages Overview

| Stage | Action | Tooling | Purpose |
| :--- | :--- | :--- | :--- |
| **1. Verification** | Linting & Typechecking | TypeScript (`tsc`) & ESLint | Enforce code quality, style standards, and type safety |
| **2. Testing** | Unit & Domain Tests | Vitest | Validate priority scoring and greedy scheduling algorithms |
| **3. Build** | Static SPA Bundling | Vite | Compile React and TypeScript assets into `dist/` |
| **4. SPA Routing Setup** | Fallback Page Generation | Shell (`cp dist/index.html dist/404.html`) | Enable client-side route handling on GitHub Pages |
| **5. Deployment** | Publish to GitHub Pages | `actions/deploy-pages@v4` | Deploy static bundle to GitHub Pages via secure OIDC |

### 6.2 Trigger Policy
* **Pushes to `main`:** Automatically execute verification, testing, and deployment to the live GitHub Pages environment.
* **Pull Requests:** Automatically execute verification and testing stages to validate code health before merging, keeping the `main` branch continuously deployable.

---

## 7. Security & Data Isolation

Operating as a client-side application on GitHub Pages provides clear security benefits:

1. **Zero Server Data Exposure:** GitHub Pages serves pre-compiled static assets only. No user tasks, availability windows, or completion data are ever transmitted to or stored on server infrastructure.
2. **Client Sandbox Isolation:** All task data resides exclusively in the user's browser IndexedDB, isolated by the browser origin policy.
3. **Automated HTTPS:** GitHub Pages automatically provisions and renews TLS certificates, enforcing encrypted HTTPS delivery across all sessions.

---

## 8. Operational Runbook

### 8.1 Repository Setup (One-Time)
1. In the GitHub repository, navigate to **Settings** > **Pages**.
2. Under **Build and deployment** > **Source**, select **GitHub Actions**.
3. Pushes to `main` will automatically build and publish the site to:
   `https://<username>.github.io/Paise_ProjektWS26/`

### 8.2 Verifying Deployment Health
* Check the **Actions** tab in GitHub to inspect build and test logs for each commit.
* Open the deployment URL to verify that the SPA loads, state persists across browser reloads (IndexedDB), and navigation functions properly.
