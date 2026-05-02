# JE Scoring Calculator

Hosted job evaluation scoring calculator for HR role calibration.

The current production approach is:

```text
Static HTML calculator
-> Azure Static Web Apps hosting
-> Teams Website tab or Power Apps launcher
```

## Live URL

```text
https://wonderful-coast-01304d900.7.azurestaticapps.net
```

## Current Azure Resource

| Item | Value |
|---|---|
| Resource group | `rg-je-scoring-calculator` |
| Static Web App | `je-scoring-calculator-v1t0yf` |
| Region | `East Asia` |
| SKU | `Free` |

## Important Files

| Path | Purpose |
|---|---|
| `hosted/index.html` | Production calculator page. Deploy this folder to Azure Static Web Apps. |
| `hosted/staticwebapp.config.json` | Azure Static Web Apps config, including Teams iframe/security headers. |
| `hosted/AUTH_RESTORE.md` | Notes for restoring Microsoft Entra sign-in later. |
| `powerapps/Azure_Static_Web_Apps_Hosting.md` | Deployment and usage notes for Azure, Teams, and Power Apps. |
| `powerapps/Option_2_Host_And_Link.md` | Original decision path for hosting/linking instead of rebuilding. |
| `powerapps/Alternative_Canvas_Rebuild_Guide.md` | Alternative only: native Power Apps rebuild guide. Not the active implementation. |
| `legacy/` | Original exported/fragments kept for reference. |

## What The App Does

The calculator lets users enter role details, score five criteria from 1 to 5, and receive:

- Projected JE score
- Grade band
- Role profile
- Team-management note where relevant
- Hybrid-role note where relevant
- Print/save-as-PDF export
- Email-result action using a recipient email input

The scoring questions/criteria are in `hosted/index.html`.

## Deploy Updates

Prerequisites:

- Azure CLI signed into the correct subscription
- Node.js/npm available

Deploy the current `hosted` folder:

```powershell
$env:SWA_CLI_DEPLOYMENT_TOKEN = az staticwebapp secrets list --name je-scoring-calculator-v1t0yf --resource-group rg-je-scoring-calculator --query properties.apiKey --output tsv
npx @azure/static-web-apps-cli@2.0.9 deploy .\hosted --env production --deployment-token $env:SWA_CLI_DEPLOYMENT_TOKEN
```

Verify:

```powershell
curl.exe -I https://wonderful-coast-01304d900.7.azurestaticapps.net
```

Expected result while auth is disabled:

```text
HTTP/1.1 200 OK
```

## Teams Setup

Add as a Teams Website tab:

```text
Team/channel > + Add a tab > Website
```

Tab name:

```text
JE Scoring Calculator
```

URL:

```text
https://wonderful-coast-01304d900.7.azurestaticapps.net
```

## Power Apps Setup

Power Apps is optional for this implementation.

If needed, create a button with:

```powerfx
Launch("https://wonderful-coast-01304d900.7.azurestaticapps.net")
```

For Teams-only use, the Power Apps launcher is not required.

## Authentication Status

Microsoft sign-in is currently disabled for pilot/testing.

The restore instructions are in:

```text
hosted/AUTH_RESTORE.md
```

## Notes For Future Work

Recommended next improvements:

- Add a custom domain.
- Package as a proper Teams app wrapper if a more native Teams rollout is needed.
- Add result saving to SharePoint, Dataverse, or a Power Automate endpoint.
- Re-enable Microsoft Entra sign-in for production if the app contains sensitive HR content.
