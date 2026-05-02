# Host the JE Calculator with Azure Static Web Apps

Use this when you want a reliable HTTPS URL for Teams or Power Apps.

## File To Publish

The hosted app is:

```text
hosted/index.html
```

## Recommended Deployment Path

1. Put this project in a GitHub repository.
2. In Azure Portal, create a new **Static Web App**.
3. Choose **GitHub** as the deployment source.
4. Select the repository and branch.
5. Set the app location to:

```text
hosted
```

6. Leave API location blank.
7. Leave output/build location blank, because this is plain HTML.
8. Create the Static Web App.
9. Azure will add a GitHub Actions workflow.
10. After deployment finishes, open the generated Static Web Apps URL.

The URL will look similar to:

```text
https://your-app-name.azurestaticapps.net
```

## Add To Power Apps

In Power Apps, add a button and set `OnSelect` to:

```powerfx
Launch("https://wonderful-coast-01304d900.7.azurestaticapps.net")
```

## Add To Teams

In Teams:

```text
Channel > + Add a tab > Website > paste the Azure Static Web Apps URL
```

Use this URL:

```text
https://wonderful-coast-01304d900.7.azurestaticapps.net
```

## Result Export

The live calculator includes a **Print / Save PDF** button.

Users can click it and choose:

```text
Destination > Save as PDF
```

## Why Azure Static Web Apps

- It serves the HTML page as a real website.
- It gives you HTTPS automatically.
- It works well with Teams website tabs.
- It works well with Power Apps `Launch(...)`.
- No Power Apps rebuild is required.

## Access Control

The deployed app includes:

```text
hosted/staticwebapp.config.json
```

Authentication is currently disabled for pilot/testing, but the restore settings are preserved in:

```text
hosted/AUTH_RESTORE.md
```

To re-enable Microsoft sign-in later, add this route back to the top-level config:

```json
{
  "routes": [
    {
      "route": "/*",
      "allowedRoles": ["authenticated"]
    }
  ],
  "responseOverrides": {
    "401": {
      "statusCode": 302,
      "redirect": "/.auth/login/aad"
    }
  }
}
```

The app also sets a Teams-friendly frame policy:

```text
Content-Security-Policy: frame-ancestors 'self' https://teams.microsoft.com https://*.teams.microsoft.com;
```

## Deployed Resource

| Item | Value |
|---|---|
| Resource group | `rg-je-scoring-calculator` |
| Static Web App | `je-scoring-calculator-v1t0yf` |
| Region | `East Asia` |
| SKU | `Free` |
| URL | `https://wonderful-coast-01304d900.7.azurestaticapps.net` |
