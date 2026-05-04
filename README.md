# JE Scoring Calculator

Hosted Job Evaluation (JE) scoring calculator for HR role calibration.

This package is intended to be handed to another technical team so they can deploy the app into their own Azure account and add it to Microsoft Teams.

## Current Implementation

The active implementation is a static HTML app hosted with Azure Static Web Apps:

```text
hosted/index.html
-> Azure Static Web Apps
-> Microsoft Teams Website tab
-> optional Power Apps launcher button
```

There is no backend, database, build step, or server-side code in the current version.

## Live Pilot URL

The current pilot deployment is:

```text
https://wonderful-coast-01304d900.7.azurestaticapps.net
```

This URL belongs to the original pilot Azure resource. A receiving team should create their own Azure Static Web App and replace this URL in Teams, Power Apps, and documentation.

## Current Pilot Azure Resource

| Item | Value |
|---|---|
| Resource group | `rg-je-scoring-calculator` |
| Static Web App | `je-scoring-calculator-v1t0yf` |
| Region | `East Asia` |
| SKU | `Free` |
| Auth status | Disabled for pilot/testing |

## What The App Does

The app lets HR/admin users:

- Enter a role title.
- Enter a recipient email address for result sharing.
- Select a role category: `Technical`, `Operational`, `Sales`, or hybrid variants.
- Select team accountability: individual contributor, guides others, or manages a team.
- Score five JE criteria from 1 to 5:
  - Scope & impact
  - Complexity
  - Autonomy
  - Stakeholder
  - Expertise
- See a projected JE score after at least 3 criteria are scored.
- See an indicative grade band and profile.
- See team-management and hybrid-role notes where relevant.
- Print or save the result as PDF.
- Open an email draft with the scoring result.
- Open the integrated JA/JE Framework guide.

The app has two top-level tabs:

| Tab | Purpose |
|---|---|
| `Calculator` | Main scoring workflow. |
| `JA/JE Framework` | Reference guide for classification, category-specific scoring ranges, hiring-manager questions, and grade-band mapping. |

The current visual styling is aligned toward Enfrasys branding using the public site palette:

```text
#06377b deep navy
#0066b3 blue
#0095da cyan
#8dd8f8 light sky
```

## Repository Structure

| Path | Purpose |
|---|---|
| `hosted/index.html` | Production app. This is the main file to deploy. |
| `hosted/staticwebapp.config.json` | Azure Static Web Apps config, including Teams iframe/security headers. |
| `hosted/AUTH_RESTORE.md` | Instructions for re-enabling Microsoft Entra sign-in later. |
| `ja_je_framework_by_category.html` | Original standalone JA/JE framework source/reference. Its content has been integrated into `hosted/index.html`. |
| `powerapps/Azure_Static_Web_Apps_Hosting.md` | Additional Azure, Teams, and Power Apps hosting notes. |
| `powerapps/Option_2_Host_And_Link.md` | Earlier decision path for hosting/linking instead of rebuilding in Power Apps. |
| `powerapps/Alternative_Canvas_Rebuild_Guide.md` | Alternative only: native Power Apps rebuild guide. Not the active implementation. |
| `legacy/` | Older fragments/original references kept for audit/history. |

## Recommended Handoff Deployment

The receiving team should deploy the `hosted` folder to their own Azure Static Web App.

### Prerequisites

- Azure subscription access.
- Permission to create an Azure Static Web App.
- Azure CLI installed.
- Node.js/npm installed if using the SWA CLI deployment command.
- Optional: GitHub repository access if using GitHub Actions deployment.

Check local tooling:

```powershell
az --version
node --version
npm --version
```

## Option A - Deploy From Azure Portal + GitHub

This is the best long-term option for the receiving team.

1. Push this repository to the receiving team's GitHub repository.
2. In the Azure Portal, create a new **Static Web App**.
3. Choose the receiving team's Azure subscription and resource group.
4. Choose **GitHub** as the deployment source.
5. Select the repository and branch.
6. Set **App location** to:

```text
hosted
```

7. Leave **API location** blank.
8. Leave **Output location** blank because this is plain HTML.
9. Create the Static Web App.
10. Azure will create a GitHub Actions workflow.
11. Wait for the workflow to complete.
12. Open the generated Azure Static Web Apps URL.

The new URL will look like:

```text
https://<generated-name>.azurestaticapps.net
```

Use that new URL in Teams and Power Apps.

## Option B - Deploy Manually With SWA CLI

Use this when deploying directly from a local machine.

Set these values for the receiving team's Azure resource:

```powershell
$staticWebAppName = "<their-static-web-app-name>"
$resourceGroupName = "<their-resource-group-name>"
```

Retrieve the deployment token:

```powershell
$env:SWA_CLI_DEPLOYMENT_TOKEN = az staticwebapp secrets list `
  --name $staticWebAppName `
  --resource-group $resourceGroupName `
  --query properties.apiKey `
  --output tsv
```

Deploy:

```powershell
npx @azure/static-web-apps-cli@2.0.9 deploy .\hosted --env production --deployment-token $env:SWA_CLI_DEPLOYMENT_TOKEN
```

Verify:

```powershell
curl.exe -I https://<their-static-web-app-url>
```

Expected result:

```text
HTTP/1.1 200 OK
```

## Teams Setup

Add the deployed URL as a Teams Website tab:

```text
Team/channel > + Add a tab > Website
```

Recommended tab name:

```text
JE Scoring Calculator
```

URL:

```text
https://<their-static-web-app-url>
```

The app is configured to allow Teams iframe embedding through this response header:

```text
Content-Security-Policy: frame-ancestors 'self' https://teams.microsoft.com https://*.teams.microsoft.com https://*.cloud.microsoft https://*.skype.com;
```

The `*.cloud.microsoft` entry is important for New Teams/Microsoft 365 hosts. Without it, Teams may show:

```text
There was a problem reaching this app
```

If Teams still shows that message after deployment:

- Confirm the app URL opens in a normal browser.
- Confirm `curl.exe -I <url>` returns `200 OK`.
- Confirm the live `Content-Security-Policy` header includes `https://*.cloud.microsoft`.
- Click `Retry` in Teams.
- Remove and re-add the Teams Website tab if Teams cached the failed load.
- Make sure the tab URL is the new team's Azure URL, not the old pilot URL.

## Power Apps Setup

Power Apps is optional. The recommended implementation is still the hosted web app.

If a Power Apps launcher is needed, create a button and set `OnSelect` to:

```powerfx
Launch("https://<their-static-web-app-url>")
```

For Teams-only use, this Power Apps launcher is not required.

Avoid trying to paste this app into a Power Apps HTML text control. Power Apps HTML text controls are not intended to run a full JavaScript application.

## Authentication

Authentication is currently disabled for pilot/testing.

If HR content or scoring results are considered sensitive, the receiving team should re-enable Microsoft Entra sign-in before production use.

Restore instructions are in:

```text
hosted/AUTH_RESTORE.md
```

At a high level, add `routes` and `responseOverrides` back into `hosted/staticwebapp.config.json` while preserving the existing `globalHeaders` section.

## Result Storage

The current app does not save results to a database, SharePoint, or Dataverse.

Current result sharing options:

- Print / Save PDF from the browser.
- Email result using the user's local email client via `mailto:`.

If persistent storage is needed later, recommended options are:

- Power Automate HTTP endpoint.
- Azure Function API writing to SharePoint, Dataverse, or SQL.
- Dataverse-backed Power Apps rebuild.
- Custom Teams app with Graph/SharePoint integration.

## Updating The App

Most logic and content are inside:

```text
hosted/index.html
```

Key areas:

| Area | Location |
|---|---|
| Visual theme and layout | `<style>` block near the top of `hosted/index.html` |
| Score criteria | `const CRITERIA = [...]` |
| Grade mapping | `const GRADES = [...]` |
| Calculator behavior | `calc()`, `resetAll()`, `emailResult()` |
| Framework guide content | `view-framework` section in the HTML |
| Teams iframe policy | `hosted/staticwebapp.config.json` |

After changing the app:

```powershell
git diff --check
node -e "const fs=require('fs'); const html=fs.readFileSync('hosted/index.html','utf8'); const scripts=[...html.matchAll(/<script>([\s\S]*?)<\/script>/g)].map(m=>m[1]); for (const s of scripts) new Function(s); console.log('script syntax ok:', scripts.length);"
```

Then redeploy using Option A or Option B above.

## Creating A Handoff ZIP

From the repository root:

```powershell
Compress-Archive -Path README.md, hosted, powerapps, legacy, ja_je_framework_by_category.html -DestinationPath .\je-scoring-calculator-handoff.zip -Force
```

The ZIP should exclude `.git`.

## Current Git State At Handoff

Branch:

```text
master
```

To inspect the latest handoff history:

```powershell
git log -5 --oneline
git status --short
```

Important app commits to retain in the history:

```text
25cf25d Allow New Teams frame host
1b62b01 Add JA JE framework guide and Enfrasys styling
4a31ea4 Show email result button by default
5063f35 Add email result action
c896db5 Prepare JE scoring calculator handoff
```

## Production Readiness Checklist

Before production rollout, the receiving team should confirm:

- The app is deployed to their own Azure subscription.
- The Teams tab uses their new Static Web Apps URL.
- The app loads inside New Teams.
- The app loads in a normal browser.
- The `Content-Security-Policy` header includes Teams and `*.cloud.microsoft`.
- Authentication requirements are reviewed.
- Result storage requirements are reviewed.
- HR/legal stakeholders approve the scoring framework and grade-band mapping.
- Any Enfrasys branding requirements are approved.
- A support owner is assigned for future changes.

## Known Limitations

- Grade bands are indicative and should be treated as HR calibration support, not an automatic final decision.
- Results are not stored.
- Email uses `mailto:`, so behavior depends on the user's configured email client.
- The app is a static website, not a native Teams app package.
- There is no role-based access control unless Microsoft Entra auth is re-enabled in Azure Static Web Apps.

## Recommended Future Improvements

- Add a custom domain.
- Re-enable Microsoft Entra sign-in for production.
- Add result saving to SharePoint, Dataverse, or another approved HR data store.
- Package as a proper Teams app if deeper Teams integration is required.
- Add audit/version metadata to each scoring result.
- Add automated browser tests for Teams-sized viewports.
