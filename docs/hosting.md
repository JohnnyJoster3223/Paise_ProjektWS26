# Hosting & Deployment Specification: Smart Task Manager

> **Document:** `docs/hosting.md`  
> **Phase:** Phase 2 – Software Architecture & Infrastructure Specification  
> **Target Application:** Smart Task Manager (Client-side React 18+ / TypeScript / Vite SPA)  
> **Primary Deployment Platform:** Cloudflare Pages  
> **Secondary / Fallback Platform:** GitHub Pages via GitHub Actions (OIDC)  
> **Status:** Approved / Specification Complete  
> **Related Documents:** [Architecture Specification](architecture.md), [Research: Hosting Options](research/hosting-options.md), [Requirements](requirements.md), [ADR-0001: Architecture Variant Selection](adr/0001-architecture-variant-selection.md)

---

## 1. Executive Summary

This specification establishes the production hosting, edge delivery, and Continuous Integration / Continuous Deployment (CI/CD) architecture for the **Smart Task Manager** web application.

As established in [ADR-0001](adr/0001-architecture-variant-selection.md), the Smart Task Manager is implemented as a **Pure Client-Side Single-Page Application (SPA)** built with React, TypeScript, and Vite, utilizing client-local storage (IndexedDB via Dexie.js) for persistent state. Because the system contains no server-side application logic or database containers for the MVP, the hosting infrastructure must provide high-availability static asset delivery, deterministic Single-Page Application routing, automated branch preview deployments, and zero operational cost.

Following an exhaustive comparative evaluation of leading static edge hosting platforms (**Cloudflare Pages**, **GitHub Pages**, and **Vercel**), **Cloudflare Pages** has been selected as the **primary production hosting platform**, with **GitHub Pages** specified as the **zero-external-dependency fallback platform**.

```
                           DEVELOPMENT LIFECYCLE & DEPLOYMENT TOPOLOGY
                           
 [ Developer Workstation ] ──> git push ──> [ GitHub Repository (JohnnyJoster3223/Paise_ProjektWS26) ]
                                                            │
                       ┌────────────────────────────────────┴────────────────────────────────────┐
                       ▼                                                                         ▼
           [ Pull Request Trigger ]                                                    [ Main Branch Push ]
                       │                                                                         │
                       ▼                                                                         ▼
          GitHub Actions CI Workflow                                                GitHub Actions CI Workflow
          • npm ci                                                                   • npm ci
          • npm run lint && npm run test                                             • npm run lint && npm run test
          • npm run build (Vite -> dist/)                                            • npm run build (Vite -> dist/)
                       │                                                                         │
                       ▼                                                                         ▼
         Cloudflare Pages Wrangler Action                                           Cloudflare Pages Wrangler Action
         (Branch Preview Environment)                                               (Production Environment)
                       │                                                                         │
                       ▼                                                                         ▼
       URL: <branch>.<project>.pages.dev                                          URL: <project>.pages.dev
       (Automated Bot Comment on PR)                                              (High-Availability Global Edge CDN)
```

---

## 2. Architectural Hosting Requirements & Constraints

The selection of a hosting and deployment strategy for the Smart Task Manager is governed by five core architectural constraints:

### 2.1 Native SPA Routing (HTML5 History API Support)
Single-Page Applications rely on the browser's HTML5 History API (`pushState`, `replaceState`) to navigate between views without triggering full page reloads. When a user navigates directly to a client-side route (e.g., `/tasks/new`, `/calendar`, `/settings`) or refreshes their browser, the hosting web server must rewrite the request and return `/index.html` with an **HTTP `200 OK`** response code. 
* A server that returns an HTTP `404 Not Found` breaks user bookmarks, harms web standards compliance, causes asset caching anomalies, and prevents clean link sharing.

### 2.2 Cost Ceiling & Zero Bill-Shock Guarantee
As an academic semester project, the hosting solution must have a guaranteed **$0.00/month operational cost**. The platform must enforce hard limits or unmetered static tiers that prevent unexpected financial liabilities (e.g., from distributed denial of service attacks, scraping bots, or infinite build loops). Solutions requiring an active credit card with post-threshold billing risks are disqualified.

### 2.3 Unrestricted Multi-Author Team Collaboration
The development team consists of multiple student collaborators committing to a shared Git repository. The hosting platform must support automated CI builds triggered by any authenticated repository contributor without licensing walls or commit-author paywalls on private repositories.

### 2.4 Ephemeral Pull Request Previews
To support continuous integration, code review, and automated testing, the hosting platform should generate isolated, immutable preview deployments for every pull request opened against `main`. Reviewers must be able to test features and UI changes in a live browser environment before code integration.

### 2.5 Security, Global Performance & HTTPS
The hosting platform must provide automated, zero-maintenance SSL/TLS certificate lifecycle management (TLS 1.3), enforce HTTPS redirects, and deliver assets via a globally distributed Content Delivery Network (CDN) with edge caching for immutable static assets.

---

## 3. Comparative Evaluation of Hosting Options

Three industry-standard static hosting platforms were evaluated against the project's requirements: **Cloudflare Pages**, **GitHub Pages**, and **Vercel (Hobby)**.

### 3.1 Feature & Constraint Matrix

| Evaluation Dimension | Cloudflare Pages (Free) | GitHub Pages | Vercel (Hobby) |
| :--- | :--- | :--- | :--- |
| **Primary Architectural Fit** | **Optimal (Primary Target)** | **Viable (Zero-Setup Fallback)** | **Disqualified (Collaboration Blocker)** |
| **Client-Side Routing / SPA** | **Native 200 OK Rewrites** (`_redirects` or automatic fallback) | ❌ HTTP 404 fallback hack (`dist/404.html`) or `HashRouter` | **Native 200 OK Rewrites** (`vercel.json` rewrites) |
| **Monthly Hosting Cost** | **$0.00** | **$0.00** | **$0.00** (Personal use only) |
| **Bandwidth Allowance** | **Unlimited / Unmetered** | 100 GB / month (Soft limit) | 100 GB / month (Fast Data Transfer) |
| **Edge Request Allowance** | **Unlimited** | Unmetered (Standard usage) | 1,000,000 requests / month |
| **Build Allowance** | 500 builds/month on Cloudflare CI *(Unlimited via GitHub Actions + Wrangler)* | Unlimited via GitHub Actions runners (2,000 mins/mo free pool) | 6,000 build minutes / month (Max 100 deployments / day) |
| **Team Collaboration on Free Tier** | **Unlimited collaborators**; no Git author restrictions | **Full repo collaborators**; native GitHub permissions | ❌ **Blocks non-owner commits** on private repos (`Deployments from other users are blocked`) |
| **Pull Request Previews** | **First-class native** (`<branch>.<project>.pages.dev` + PR comment) | ❌ None native (requires bloated branch hacks) | **First-class native** (`<project>-git-<branch>.vercel.app`) |
| **Base URL Structure** | Root domain (`/`) | Subpath domain (`/<repo-name>/`) | Root domain (`/`) |
| **SSL/TLS & Custom Domains** | Free Universal Edge SSL (TLS 1.3); up to 100 custom domains | Free Let's Encrypt SSL; 1 custom domain | Free Let's Encrypt SSL; up to 50 custom domains |
| **Credit Card Required?** | **No** | **No** | **No** |

---

### 3.2 Detailed Analysis of Candidates

#### Candidate A: Cloudflare Pages (Selected Primary Target)
* **Strengths:**
  1. *Native SPA Routing:* Serves `/index.html` with HTTP 200 OK for any unmatched route via a simple `_redirects` file in `public/`.
  2. *Predictable Zero-Cost Tier:* Bandwidth and static requests are completely unmetered. There is zero risk of exceeding limits or incurring cloud bills.
  3. *Unrestricted Team Collaboration:* Cloudflare does not inspect or restrict Git commit author emails on GitHub repositories. All team members can push branches and open pull requests that trigger builds immediately.
  4. *Automated Staging Environments:* Every pull request generates an ephemeral, isolated preview deployment with an automated status check and comment in GitHub.
  5. *Zero Base URL Complications:* Assets are always served from the root domain (`https://<project>.pages.dev/`), matching Vite's default `base: '/'`.
* **Weaknesses:**
  - Requires creating a single free Cloudflare account and setting up repository secrets for GitHub Actions.

#### Candidate B: GitHub Pages (Selected Fallback Target)
* **Strengths:**
  1. *Zero External Service Footprint:* Hosted directly within GitHub infrastructure; requires no external accounts, third-party dashboards, or API credentials.
  2. *Secure Native CI/CD:* Deploys seamlessly via official GitHub Actions (`actions/deploy-pages@v4`) using ephemeral OpenID Connect (OIDC) tokens.
* **Weaknesses:**
  1. *No Native SPA 200 Rewrites:* GitHub Pages is designed for static static-site generators (Jekyll). Requesting `/calendar` emits an HTTP `404 Not Found`. While copying `dist/index.html` to `dist/404.html` allows the browser to execute React scripts, the HTTP 404 status code breaks web standards, degrades caching, and can cause browser quirks.
  2. *Subpath Base URL Gotchas:* Deploys to `https://<owner>.github.io/<repo-name>/`, requiring Vite configuration to specify `base: '/<repo-name>/'`. If configured incorrectly, all asset paths fail with 404s.
  3. *No Native PR Previews:* GitHub Pages supports only a single deployment environment per repository. Testing PRs on live URLs requires complex, brittle branch commits into subdirectories.

#### Candidate C: Vercel (Rejected)
* **Strengths:**
  1. Excellent developer experience and visual feedback toolbars for pull requests.
  2. Native SPA routing via `vercel.json`.
* **Disqualifying Weakness:**
  - **Git Commit Author Verification Blocker:** Vercel's Hobby tier strictly forbids multi-user collaboration on private repositories. If any student team member pushes a commit whose Git email does not match the Vercel account owner, Vercel aborts the deployment with the error:
    > *"Deployments from other users are blocked on Hobby accounts. Upgrade to Pro to collaborate."*
  - Bypassing this restriction requires upgrading to the **$20/month per user Pro plan** (unacceptable for an academic project) or routing deployments through a shared GitHub Actions secret, which defeats Vercel's zero-config premise.

---

### 3.3 Weighted Decision Matrix

| Evaluation Criteria | Weight | Cloudflare Pages | GitHub Pages | Vercel (Hobby) | Rationale |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **SPA Routing Fidelity (HTTP 200)** | 25% | **5 / 5** | 2 / 5 | 4 / 5 | Cloudflare provides deterministic 200 rewrites; GitHub Pages relies on 404 workarounds. |
| **Team Collaboration on Free Tier** | 20% | **5 / 5** | **5 / 5** | 1 / 5 | Vercel blocks non-owner commits on private repos without $20/user/mo Pro seats. |
| **Bandwidth & Request Quotas** | 15% | **5 / 5** | 4 / 5 | 3 / 5 | Cloudflare offers unmetered static bandwidth; others enforce 100 GB caps. |
| **Pull Request Previews** | 15% | **5 / 5** | 1 / 5 | **5 / 5** | Cloudflare and Vercel generate isolated preview URLs; GitHub Pages has no native PR previews. |
| **Configuration Simplicity** | 15% | **5 / 5** | 3 / 5 | 3 / 5 | Cloudflare uses root path `base: '/'`. GitHub Pages requires subpath and 404 hacks. |
| **Zero Cost & Financial Safety** | 10% | **5 / 5** | **5 / 5** | 4 / 5 | No credit card required; zero overage bill-shock risk on Cloudflare and GitHub Pages. |
| **Weighted Total** | **100%** | **4.95 / 5.0** | **3.15 / 5.0** | **3.35 / 5.0** | **Cloudflare Pages is the decisive architectural choice.** |

---

## 4. Architecture Decision & Rationale

### 4.1 Primary Deployment Target: Cloudflare Pages
**Decision:** The project adopts **Cloudflare Pages** as the canonical hosting platform for the Smart Task Manager.

**Justification:**
1. **Clean Web Standards:** Guarantees genuine HTTP 200 responses across all client-side routes, preventing routing edge cases and caching problems.
2. **Inclusive Team Workflow:** Allows every team member to push code, trigger builds, and review pull requests without commercial license barriers.
3. **Automated Quality Gates:** Gives students and evaluators instantaneous access to live preview environments for every pull request before merging to `main`.
4. **Resilience & Zero Maintenance:** Unmetered global edge delivery ensures the app remains accessible 24/7 with zero hosting overhead.

### 4.2 Contingency Plan: GitHub Pages Fallback
**Decision:** If team access to Cloudflare is unavailable or restricted by administrative network policies, the project maintains an automated **GitHub Pages** deployment workflow as a zero-external-dependency fallback.

**Fallback Configuration Protocol:**
1. Configure `vite.config.ts` with `base: '/Paise_ProjektWS26/'`.
2. Add a post-build step in CI: `cp dist/index.html dist/404.html`.
3. Enable GitHub Pages in repository settings: **Settings > Pages > Build and deployment > Source: GitHub Actions**.

---

## 5. Technical Configuration Specification

### 5.1 Vite Application Configuration (`vite.config.ts`)
For Cloudflare Pages, Vite operates with standard root-relative asset resolution. No subpath prefixing is necessary:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  // Cloudflare Pages serves from root domain
  base: '/',
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        // Deterministic cache-busting chunk hashes for CDN caching
        chunkFileNames: 'assets/[name]-[hash].js',
        entryFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',
      },
    },
  },
});
```

### 5.2 SPA Rewrite Rule (`public/_redirects`)
To instruct Cloudflare Pages to rewrite all client-side requests back to `/index.html` with an HTTP 200 OK status, place a `_redirects` file in the `public/` directory:

```text
# Cloudflare Pages SPA rewrite configuration
# Rewrites all requests to /index.html with HTTP 200 OK
/*    /index.html   200
```

When Vite compiles the project, assets in `public/` are copied directly to `dist/`, ensuring `dist/_redirects` is published on every deployment.

### 5.3 Edge Caching & Security Headers (`public/_headers`)
Static assets generated by Vite include content-hash identifiers in their filenames, making them safe for aggressive browser and CDN caching. HTML documents must never be cached to ensure users receive application updates immediately.

Configure `public/_headers`:

```text
# Strict caching rules for static assets
/assets/*
  Cache-Control: public, max-age=31536000, immutable

# Prevent stale HTML caching
/index.html
  Cache-Control: public, max-age=0, must-revalidate

# Global Security Headers
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), camera=(), microphone=()
  Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self';
```

---

## 6. Continuous Integration & Deployment Workflows

Automated deployments are orchestrated via GitHub Actions.

### 6.1 Primary Workflow: Cloudflare Pages Deployment (`.github/workflows/deploy-cloudflare.yml`)

This workflow executes quality verification (linting, testing, and typechecking) followed by automated deployment to Cloudflare Pages. It deploys production builds on pushes to `main` and branch preview builds on pull requests.

```yaml
name: Deploy to Cloudflare Pages

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  verify-and-build:
    name: Lint, Test & Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Typecheck
        run: npm run typecheck --if-present

      - name: Linting
        run: npm run lint --if-present

      - name: Unit & Domain Tests
        run: npm test --if-present

      - name: Build SPA
        run: npm run build

      - name: Verify Redirects Artifact
        run: |
          if [ ! -f dist/_redirects ]; then
            echo "Creating dist/_redirects fallback..."
            echo "/* /index.html 200" > dist/_redirects
          fi

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist-artifact
          path: dist/
          retention-days: 1

  deploy-cloudflare:
    name: Deploy to Cloudflare Pages
    needs: verify-and-build
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write # For posting preview URL comments on PRs
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Download Build Artifact
        uses: actions/download-artifact@v4
        with:
          name: dist-artifact
          path: dist

      - name: Deploy to Cloudflare Pages
        id: cloudflare_deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy dist --project-name=smart-task-manager --branch=${{ github.head_ref || github.ref_name }}

      - name: Post PR Preview URL Comment
        if: github.event_name == 'pull_request' && steps.cloudflare_deploy.outputs.deployment-url != ''
        uses: actions/github-script@v7
        with:
          script: |
            const deploymentUrl = '${{ steps.cloudflare_deploy.outputs.deployment-url }}';
            const issueNumber = context.issue.number;
            const commentBody = [
              '### 🚀 Cloudflare Pages Preview Deployment',
              '',
              `**Preview URL:** [${deploymentUrl}](${deploymentUrl})`,
              `**Commit:** \`${context.sha.substring(0, 7)}\``,
              `**Branch:** \`${context.payload.pull_request.head.ref}\``,
              '',
              '*Reviewers can interact with this preview deployment directly in the browser.*'
            ].join('\n');

            // Find existing preview comment if present to avoid spamming
            const { data: comments } = await github.rest.issues.listComments({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: issueNumber,
            });

            const botComment = comments.find(c => c.body.includes('Cloudflare Pages Preview Deployment'));

            if (botComment) {
              await github.rest.issues.updateComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                comment_id: botComment.id,
                body: commentBody,
              });
            } else {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issueNumber,
                body: commentBody,
              });
            }
```

---

### 6.2 Secondary Workflow: GitHub Pages Fallback (`.github/workflows/deploy-gh-pages.yml`)

Included as a disaster recovery and zero-secret deployment path:

```yaml
name: Deploy to GitHub Pages (Fallback)

on:
  workflow_dispatch: # Triggered manually or when Cloudflare is disabled

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: 'github-pages'
  cancel-in-progress: false

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build Vite SPA
        run: npm run build

      - name: SPA Routing Fallback (Copy index to 404)
        run: cp dist/index.html dist/404.html

      - name: Setup GitHub Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 7. Security, Privacy & Data Isolation

Because the Smart Task Manager MVP operates under a client-local architecture, the hosting layer has distinct security advantages:

1. **Zero Server-Side Data Exposure:** The hosting tier delivers static JavaScript and CSS assets exclusively. No user task titles, availability windows, or completion statuses are ever transmitted to or stored on Cloudflare or GitHub servers.
2. **Confidentiality by Design:** All user planning data resides within the client browser's IndexedDB database, sandboxed under the Origin policy (`https://<project>.pages.dev`).
3. **Automated HTTPS Enforcement:** Cloudflare Pages enforces TLS 1.3 encryption across all requests by default. Insecure HTTP requests are automatically upgraded via 301 redirects.
4. **Supply Chain Integrity:** Build workflows pin GitHub Action versions (`@v4`, `@v3`) and enforce `npm ci` using reproducible package-lock manifests to eliminate untrusted dependency injection.

---

## 8. Operational Runbook & Setup Guide

### 8.1 Initial Project Provisioning (One-Time Setup)

To activate automated deployment for the repository:

1. **Cloudflare Account & Project Setup:**
   * Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/).
   * Navigate to **Workers & Pages** > **Create application** > **Pages**.
   * Create a project named `smart-task-manager`.
2. **Generate API Token:**
   * Go to **My Profile** > **API Tokens** > **Create Token**.
   * Use the **Edit Cloudflare Workers/Pages** template.
   * Restrict token scope to the student account and Pages resources.
3. **Configure GitHub Repository Secrets:**
   * Navigate to the GitHub repository: **Settings** > **Secrets and variables** > **Actions**.
   * Create Repository Secret: `CLOUDFLARE_API_TOKEN` (the generated API token).
   * Create Repository Secret: `CLOUDFLARE_ACCOUNT_ID` (found in the Cloudflare dashboard right-hand sidebar).
4. **Deploy:**
   * Push code to `main` or open a pull request. The workflow `.github/workflows/deploy-cloudflare.yml` will trigger and publish the application.

### 8.2 Troubleshooting & FAQ

* **Q: Why does navigating directly to `/calendar` show a blank screen on other hosts?**  
  *A:* Static hosts default to file-system matching. Without `_redirects` returning `/index.html` with HTTP 200, the server reports 404. Ensure `dist/_redirects` contains `/* /index.html 200`.
* **Q: Are PR preview links accessible to external evaluators/instructors?**  
  *A:* Yes. Cloudflare Pages preview URLs are publicly accessible on the internet unless Cloudflare Access (SSO) is explicitly enabled. Evaluators can inspect features directly.
* **Q: What happens if our team exceeds the 500 monthly builds limit on Cloudflare?**  
  *A:* The GitHub Actions workflow deploys pre-built assets using the `wrangler pages deploy` command. This uses Cloudflare's direct upload API and does **not** consume Cloudflare CI build minutes.
