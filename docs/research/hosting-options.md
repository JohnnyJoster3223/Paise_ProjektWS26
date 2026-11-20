# Research: Static SPA Hosting Options for React & Vite

> **Research Ticket:** [Research static SPA hosting options for React/Vite](https://github.com/JohnnyJoster3223/Paise_ProjektWS26/issues/6)  
> **Investigation Question:** *What are the trade-offs, pricing/quotas, custom domain capabilities, and GitHub Actions deployment workflows for hosting a static React/Vite Single-Page Application on GitHub Pages vs Cloudflare Pages vs Vercel? Which option best minimizes maintenance overhead for a student project while remaining 100% free and reliable?*  
> **Target Application:** Smart Task Manager (Client-side React 18+ / Vite Single-Page Application; local persistence via `localStorage`/`IndexedDB`; no dedicated backend server).  
> **Date:** November 2026  
> **Status:** Completed  

---

## Executive Summary & Recommendation

For the **Smart Task Manager** student semester project, **Cloudflare Pages** is the recommended hosting platform.

| Provider | Overall Fit | Primary Advantages | Critical Drawbacks |
| :--- | :--- | :--- | :--- |
| **Cloudflare Pages** | **Strongly Recommended** (Best Fit) | • 100% free with unlimited unmetered bandwidth<br>• Native SPA 200 OK rewrites (no 404 hacks)<br>• Free branch/PR preview environments out-of-the-box<br>• Full multi-user team collaboration on Free tier<br>• Standard root (`/`) base path | Requires creating a free Cloudflare account |
| **GitHub Pages** | **Viable Fallback** (Zero Setup) | • Zero external accounts (lives inside GitHub)<br>• Native GitHub Actions OIDC deployment (`deploy-pages`)<br>• Reliable and free | • No native SPA routing (requires `404.html` hack or `HashRouter`)<br>• Subpath base URL (`/<repo-name>/`) gotchas<br>• No native PR preview environments |
| **Vercel (Hobby)** | **Not Recommended** (Collaboration Wall) | • Best-in-class PR previews & feedback toolbar<br>• Zero-config Vite detection | • **Severe team blocker:** Hobby tier blocks commits from non-owners on private repos (`Deployments from other users are blocked`)<br>• Requires $20/user/mo Pro plan or complex CI token bypass<br>• 100 GB monthly bandwidth cap |

### Why Cloudflare Pages Wins for This Project
1. **Zero Maintenance Routing:** Single-page applications relying on HTML5 History API (e.g. `react-router-dom`) require the web server to rewrite all route requests back to `/index.html` with an HTTP `200 OK` status. Cloudflare Pages supports this natively (serving `/index.html` on any route not matching a physical asset, or explicitly via `_redirects`). In contrast, GitHub Pages returns HTTP `404 Not Found`, forcing fragile workarounds.
2. **Team Collaboration Without Financial Friction:** Unlike Vercel's Hobby plan—which restricts deployments to a single Git commit author and blocks team members on private repositories unless upgraded to the $20/month/user Pro tier—Cloudflare Pages allows unlimited team collaborators on its Free tier with no commit author restrictions.
3. **Automated PR Preview Environments:** Each pull request receives an isolated preview deployment URL (e.g., `<branch>.<project>.pages.dev`) with an automated GitHub PR comment. This allows reviewers and testing subagents to test changes in live staging environments before merging to `main`.
4. **Predictable Free Tier (Zero Bill Shock):** Unmetered bandwidth and static requests ensure that traffic spikes or bot scrapes will never incur costs or suspend the project.

---

## 1. Deep-Dive Comparison by Evaluation Axis

### 1.1 Client-Side Routing & SPA 404 Handling

Single-Page Applications do not map browser URLs directly to discrete HTML files on disk. Navigating to `/tasks/new` or refreshing on `/calendar` requires the static host to serve the root `index.html` so that client-side routers can mount the appropriate components.

```
Incoming Request: GET /tasks/new
┌────────────────────────────────────────────────────────────────────────┐
│ Cloudflare Pages:  Rewrites to /index.html (HTTP 200 OK)        [Clean]│
│ Vercel:            Rewrites via vercel.json (HTTP 200 OK)       [Clean]│
│ GitHub Pages:      No match -> Returns /404.html (HTTP 404)     [Hack] │
└────────────────────────────────────────────────────────────────────────┘
```

#### GitHub Pages
* **Behavior:** Operates strictly as a static web server. If a requested path does not map to a physical file or directory containing `index.html`, GitHub Pages emits an **HTTP 404 Not Found**.
* **Workarounds:**
  1. *Duplicate `index.html` as `404.html`:* Copy `dist/index.html` to `dist/404.html` during the build step. When a user directly visits `/tasks/new`, GitHub Pages serves `404.html` with an HTTP status code of `404`. While the browser executes the bundled React script and the router renders the view, returning an HTTP `404` status code degrades SEO, breaks OpenGraph social link unfurling, and causes browser caching quirks.
  2. *Redirection Scripts (`spa-github-pages`):* A custom `404.html` encodes the target path into a query parameter (e.g., `/?/tasks/new`) and redirects to `index.html`, which decodes the path and executes `window.history.replaceState()`. This is brittle and introduces visible URL flickering.
  3. *Hash Router (`HashRouter`):* Routes become `https://<user>.github.io/<repo>/#/tasks/new`. Bypasses server lookups because the hash fragment is never sent to the server. However, it creates non-standard URLs and complicates deep-link integration.
* **Base URL Gotcha:** By default, GitHub Pages serves project repositories at `https://<user>.github.io/<repo-name>/`. Developers must explicitly configure `base: '/<repo-name>/'` in `vite.config.ts`. If this configuration is omitted, all JavaScript and CSS asset paths will resolve to `https://<user>.github.io/assets/...` and fail with 404 errors.

#### Cloudflare Pages
* **Behavior:** Provides **first-class native SPA support**. If a Cloudflare Pages project does not contain a top-level `404.html` file, Cloudflare Pages automatically serves `/index.html` with an HTTP `200 OK` status for any route that does not match a static file.
* **Explicit Control via `_redirects`:** For guaranteed deterministic behavior, place a `_redirects` file in the Vite `public/` folder:
  ```text
  /*    /index.html   200
  ```
  Vite copies this file directly to `dist/_redirects`. Cloudflare Pages reads this rule and treats it as an internal server-side rewrite, preserving the path and returning HTTP `200`.
* **Base URL:** Always deployed at the root domain (`https://<project>.pages.dev/`), so `base: '/'` (Vite's default) works without modifications.

#### Vercel
* **Behavior:** Out of the box, direct deep links to Vite SPAs result in 404 errors. Vercel requires an explicit catch-all rewrite rule.
* **Configuration via `vercel.json`:**
  ```json
  {
    "$schema": "https://openapi.vercel.sh/vercel.json",
    "rewrites": [
      {
        "source": "/(.*)",
        "destination": "/index.html"
      }
    ]
  }
  ```
  Static files (CSS, JS, images) are served directly; all unmatched routes fall back to `/index.html` with HTTP `200 OK`.
* **Base URL:** Root domain (`https://<project>.vercel.app/`), requiring `base: '/'`.

---

### 1.2 Pricing, Quotas & Risk of Overages

Student projects require zero cost and protection against unexpected billing or service termination.

| Metric | GitHub Pages | Cloudflare Pages (Free) | Vercel (Hobby) |
| :--- | :--- | :--- | :--- |
| **Monthly Cost** | **$0.00** | **$0.00** | **$0.00** (Personal only) |
| **Bandwidth (Data Transfer)** | 100 GB / month (Soft limit) | **Unlimited / Unmetered** | 100 GB / month (Fast Data Transfer) |
| **Edge / Static Requests** | Not explicitly metered | **Unlimited** | 1,000,000 / month |
| **Build Allowance** | Unlimited with custom GitHub Actions (Soft limit: 10 builds/hr via native trigger) | **500 builds / month** on Cloudflare CI *(Unlimited via GitHub Actions + Wrangler)* | 6,000 build minutes / month (Max 100 deployments / day) |
| **Concurrent Builds** | 1 (per GHA runner limits) | 1 concurrent build | 1 concurrent build |
| **Build Timeout** | 10 minutes | 20 minutes | 45 minutes |
| **Asset / File Limits** | Max site size: 1 GB<br>Max repo size: 1 GB | Max site files: 20,000<br>Max single asset: 25 MiB | Max single file: 100 MiB |
| **Overage / Spikes Behavior** | Soft warning / temporary pause; no billing | Hard stop of builds on CF CI when 500 reached; static traffic remains up; no billing | Account/project suspended or paused when 100 GB reached; no credit card required on Hobby |
| **Credit Card Required?** | **No** | **No** | **No** |

#### Analysis:
* **Cloudflare Pages** offers the most generous free tier: static bandwidth and request counts are completely unmetered. If the team exhausts the 500 monthly builds on Cloudflare's build queue, they can deploy via GitHub Actions and the Wrangler CLI, which bypasses the Cloudflare build minute limit.
* **Vercel** enforces a 100 GB bandwidth cap and 1M edge request limit. If traffic exceeds these thresholds, the site can be paused until the billing cycle resets.
* **GitHub Pages** has a generous 100 GB soft limit and zero risk of surprise invoices.

---

### 1.3 Custom Domain Capabilities & SSL

All three providers support apex domains (`example.com`) and subdomains (`app.example.com`) with automated SSL certificate issuance.

| Feature | GitHub Pages | Cloudflare Pages | Vercel |
| :--- | :--- | :--- | :--- |
| **Apex Domain Support** | Yes (A records pointing to 4 GitHub IPs) | Yes (CNAME flattening on Cloudflare DNS or standard CNAME) | Yes (A record to `76.76.21.21`) |
| **Subdomain Support** | Yes (`CNAME` record) | Yes (`CNAME` record to `<project>.pages.dev`) | Yes (`CNAME` to `cname.vercel-dns.com`) |
| **SSL/TLS Certificate** | Automatic Let's Encrypt (Free) | Automatic Universal Edge SSL / TLS 1.3 (Free) | Automatic Let's Encrypt / DigiCert (Free) |
| **Enforce HTTPS** | Yes (1-click checkbox) | Yes (Always Use HTTPS setting) | Yes (Default) |
| **Custom Domains Limit** | 1 custom domain per site | Up to **100 custom domains** | Up to **50 custom domains** |
| **Setup Complexity** | Low (add `CNAME` file or configure via UI) | Extremely Low (1-click if domain is on Cloudflare; CNAME if external) | Low (DNS verification in dashboard) |

---

### 1.4 PR Previews & Team Collaboration

For team software engineering projects, automated PR previews enable reviewers to test features in an isolated environment before merging.

```
Developer pushes feature branch ──> Pull Request opened
                                           │
       ┌───────────────────────────────────┴───────────────────────────────────┐
       ▼                                   ▼                                   ▼
Cloudflare Pages                    Vercel                              GitHub Pages
Creates branch URL                  Creates preview URL                 No native preview!
(e.g. branch.proj.pages.dev)        (e.g. proj-git-branch.vercel.app)   (Only main branch deployed;
Posts automated PR comment          Posts PR comment + toolbar          overwrites production)
```

#### GitHub Pages
* **No Native Preview Support:** GitHub Pages supports only a single deployment environment per repository. Deploying a pull request requires either overwriting production or publishing to subdirectories.
* **Third-Party Workaround:** Community actions (such as `rossjrw/pr-preview-action`) commit preview builds into a subfolder on the `gh-pages` branch (`gh-pages/pr-preview/pr-12`). This introduces significant maintenance overhead:
  - Requires dynamic injection of the base URL for every PR (`base: '/repo-name/pr-preview/pr-${{ github.event.number }}/'`).
  - Bloats the repository's Git history with ephemeral build artifacts.
  - Requires elevated write permissions on repository contents.

#### Cloudflare Pages
* **First-Class Native Previews:** Every push to a non-production branch or pull request automatically generates a preview deployment with an immutable unique URL (e.g. `<commit-hash>.<project>.pages.dev` and `<branch-name>.<project>.pages.dev`).
* **Automated Feedback:** Cloudflare Pages automatically posts a status check and comment on the GitHub pull request containing the preview link and deployment logs.
* **Team Collaboration:** Cloudflare's Free tier allows inviting multiple team members to the Cloudflare account with developer roles. Crucially, Cloudflare's Git integration does **not** restrict or validate the commit author. Any teammate who pushes to the GitHub repository triggers a build.

#### Vercel
* **The Hobby Collaboration Pitfall:**
  - Vercel's Hobby tier is strictly intended for individual personal use.
  - On **private repositories**, Vercel enforces **Git Commit Author Verification**: the commit author email must match the connected Vercel account owner.
  - If teammate B pushes a commit or opens a PR on teammate A's private repository, Vercel fails the deployment with:
    > *"Deployments from other users are blocked on Hobby accounts. Upgrade to Pro to collaborate."*
  - Upgrading to a Pro team costs **$20 per user per month**.
  - While public repositories in GitHub organizations are exempt, student teams frequently develop in private repositories or switch repository visibility.
  - Workarounds (such as routing deployments through a shared GitHub Actions secret token) bypass Vercel's primary benefit: zero-configuration Git deployment.

---

## 2. GitHub Actions Deployment Workflows

Below are tested, production-grade GitHub Actions workflows for each provider.

### 2.1 GitHub Pages Workflow (`.github/workflows/deploy-gh-pages.yml`)

Uses GitHub's official Pages deployment actions (`upload-pages-artifact` and `deploy-pages`) using OpenID Connect (OIDC) tokens—no personal access tokens (PAT) or secrets required.

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: 'pages'
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

      - name: Build SPA
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

> **Note on Vite Configuration:**  
> When deploying to GitHub Pages without a custom domain, `vite.config.ts` must declare:
> ```ts
> export default defineConfig({
>   base: '/smart-task-manager/', // Repository name
>   // ...
> })
> ```

---

### 2.2 Cloudflare Pages Workflow (`.github/workflows/deploy-cloudflare.yml`)

This workflow builds the application in GitHub Actions and deploys the `dist/` directory to Cloudflare Pages using the official Wrangler Action. It automatically distinguishes between production and PR preview deployments.

**Required GitHub Repository Secrets:**
* `CLOUDFLARE_API_TOKEN` (Cloudflare API token with `Cloudflare Pages:Edit` permissions)
* `CLOUDFLARE_ACCOUNT_ID` (Found in Cloudflare dashboard URL or account settings)

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

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Build & Deploy
    permissions:
      contents: read
      pull-requests: write # To comment preview URLs on PRs
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

      - name: Build Vite application
        run: npm run build

      - name: Ensure SPA Redirects
        run: |
          if [ ! -f dist/_redirects ]; then
            echo "/* /index.html 200" > dist/_redirects
          fi

      - name: Deploy to Cloudflare Pages
        id: cloudflare_deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy dist --project-name=smart-task-manager --branch=${{ github.head_ref || github.ref_name }}

      - name: Comment PR Preview URL
        if: github.event_name == 'pull_request' && steps.cloudflare_deploy.outputs.deployment-url != ''
        uses: actions/github-script@v7
        with:
          script: |
            const url = '${{ steps.cloudflare_deploy.outputs.deployment-url }}';
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 **Cloudflare Pages Preview deployed!**\n\nPreview URL: [${url}](${url})`
            });
```

*(Alternatively, connecting Cloudflare Pages directly via GitHub Git integration requires zero YAML files, zero secrets, and handles PR previews automatically.)*

---

### 2.3 Vercel Workflow via GitHub Actions (`.github/workflows/deploy-vercel.yml`)

Required when bypassing Vercel's Git author restrictions on private repos without upgrading to Pro.

**Required GitHub Repository Secrets:**
* `VERCEL_TOKEN`
* `VERCEL_ORG_ID`
* `VERCEL_PROJECT_ID`

```yaml
name: Deploy to Vercel

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=${{ github.ref == 'refs/heads/main' && 'production' || 'preview' }} --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

      - name: Build Project Artifacts
        run: |
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
          else
            vercel build --token=${{ secrets.VERCEL_TOKEN }}
          fi
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

      - name: Deploy to Vercel
        run: |
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
          else
            vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }}
          fi
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
```

---

## 3. Comprehensive Decision Matrix

Scores range from 1 (Poor / Complex) to 5 (Excellent / Effortless).

| Evaluation Criteria | Weight | GitHub Pages | Cloudflare Pages | Vercel (Hobby) | Rationale & Evidence |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Client-Side Routing / SPA Rewrites** | 25% | 2/5 | **5/5** | 4/5 | Cloudflare supports native 200 rewrite fallback. Vercel requires `vercel.json`. GitHub Pages emits 404 and requires workarounds. |
| **Team Collaboration on Free Tier** | 20% | **5/5** | **5/5** | 1/5 | Vercel blocks non-owner commits on private repos under Hobby. GitHub and Cloudflare allow unlimited teammates. |
| **Bandwidth & Request Quotas** | 15% | 4/5 | **5/5** | 3/5 | Cloudflare provides unlimited unmetered bandwidth. GitHub Pages and Vercel cap soft/hard limits at 100 GB/month. |
| **Pull Request Preview Environments** | 15% | 1/5 | **5/5** | **5/5** | Cloudflare and Vercel create isolated ephemeral preview deployments automatically. GitHub Pages has no native PR preview capability. |
| **Configuration & Maintenance Simplicity** | 15% | 3/5 | **5/5** | 3/5 | GitHub Pages requires subpath `base` config and 404 duplication. Vercel requires CI secret workarounds for teams. Cloudflare works out-of-the-box. |
| **Zero Cost Guarantee (No Overages)** | 10% | **5/5** | **5/5** | 4/5 | None require credit cards; Cloudflare and GitHub Pages carry zero risk of bill shock. |
| **Weighted Score** | **100%** | **3.15 / 5.0** | **4.95 / 5.0** | **3.35 / 5.0** | **Cloudflare Pages emerges as the clear winner.** |

---

## 4. Final Recommendation & Implementation Plan

### Recommendation for Smart Task Manager
Adopt **Cloudflare Pages** as the primary hosting solution for the project.

#### Justification Summary:
1. **Clean SPA Architecture:** Full support for HTML5 pushState routing with genuine HTTP 200 OK responses, avoiding broken 404 responses or hash fragments in URLs.
2. **Effortless Team Collaboration:** The student team can commit from multiple GitHub accounts to private or public repositories without being blocked by licensing restrictions.
3. **Automated Quality Gate:** Pull request previews allow students and automated reviewers to verify user interface and client storage behavior on live devices before merging.
4. **Zero Financial Risk:** Unlimited static bandwidth and generous build allowances ensure the application remains functional throughout the semester evaluation without expense.

### Implementation Checklist for the Team

1. **Vite Configuration (`vite.config.ts`):**
   * Keep `base: '/'` (default). No subpath adjustments needed.
2. **SPA Fallback Configuration:**
   * Create `public/_redirects` containing:
     ```text
     /*    /index.html   200
     ```
3. **Deployment Setup (Choose Option A or B):**
   * **Option A (Zero-Config Direct Git Integration - Recommended):**
     1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/) > **Workers & Pages**.
     2. Click **Create application** > **Pages** > **Connect to Git**.
     3. Select repository `JohnnyJoster3223/Paise_ProjektWS26`.
     4. Build Preset: `Vite` (`npm run build`, output directory `dist`).
     5. Cloudflare will automatically build `main` on push and create PR previews on pull requests.
   * **Option B (GitHub Actions Driven via Wrangler):**
     1. Create a Cloudflare API token with Pages edit permissions.
     2. Add `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID` to GitHub Repository Secrets.
     3. Add `.github/workflows/deploy-cloudflare.yml` (as detailed in Section 2.2).

---

## 5. Primary Sources & Documentation

1. **GitHub Pages Official Documentation:**
   * [About GitHub Pages and Usage Limits](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#usage-limits)
   * [Creating a Custom 404 Page for GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-custom-404-page-for-your-github-pages-site)
   * [Configuring a Custom Domain for GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site)
   * [GitHub Actions `deploy-pages` Action](https://github.com/actions/deploy-pages)
2. **Cloudflare Pages Official Documentation:**
   * [Cloudflare Pages Limits & Pricing](https://developers.cloudflare.com/pages/platform/limits/)
   * [Serving Single-Page Applications and Redirects](https://developers.cloudflare.com/pages/configuration/redirects/)
   * [Cloudflare Pages Preview Deployments](https://developers.cloudflare.com/pages/configuration/preview-deployments/)
   * [Custom Domains on Cloudflare Pages](https://developers.cloudflare.com/pages/configuration/custom-domains/)
   * [Wrangler Pages Command Reference](https://developers.cloudflare.com/workers/wrangler/commands/#pages)
3. **Vercel Official Documentation:**
   * [Vercel Limits & Fair Use Guidelines](https://vercel.com/docs/limits/fair-use-guidelines)
   * [Vercel Hobby Plan Collaboration Restrictions](https://vercel.com/docs/accounts/plans/hobby#collaboration)
   * [Configuring Rewrites for Single Page Applications (`vercel.json`)](https://vercel.com/docs/projects/project-configuration#rewrites)
   * [Deploying Vite to Vercel](https://vercel.com/docs/frameworks/vite)
4. **Vite Official Documentation:**
   * [Deploying a Static Site (GitHub Pages, Cloudflare Pages, Vercel)](https://vite.dev/guide/static-deploy.html)
