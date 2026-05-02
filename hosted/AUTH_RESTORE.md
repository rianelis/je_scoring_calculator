# Restore Microsoft Sign-In Later

Authentication is disabled for the pilot.

To require Microsoft Entra sign-in again, add these top-level sections back into `staticwebapp.config.json`:

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

Keep the existing `globalHeaders` section for Teams iframe support.
