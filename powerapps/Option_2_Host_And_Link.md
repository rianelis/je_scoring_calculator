# Option 2 - Host Current HTML App and Open It from Power Apps

This option does not rebuild the calculator in Power Apps. It keeps the calculator as an HTML/JavaScript web page and uses Power Apps as the launcher.

## What To Host

Use this file:

```text
hosted/index.html
```

It is a complete standalone version of the current calculator, with the same scoring logic and a clean HTML page wrapper for hosting.

## Hosting Choices

Recommended:

- **SharePoint document library** if this is internal HR-only and your tenant allows opening HTML files in browser.
- **Azure Static Web Apps** if you want the cleanest web hosting URL.
- **Internal company web server** if your IT team already has one.
- **Teams tab** if users mainly access it in Teams.

Avoid:

- The Power Apps HTML text control. It is not meant to run this JavaScript app.
- Uploading the raw `.html` into Power Apps and expecting it to execute.

## Power Apps Button

In your Canvas app, add a button named:

```text
btnOpenCalculator
```

Set `btnOpenCalculator.Text` to:

```powerfx
"Open JE Scoring Calculator"
```

Set `btnOpenCalculator.OnSelect` to:

```powerfx
Launch("https://YOUR-HOSTED-URL-HERE/index.html")
```

Replace the URL with the actual hosted URL.

## What This Gives You

- Uses the current calculator file.
- No manual Power Apps rebuild.
- No PCF/code component setup.
- Fastest route to make the calculator available from Power Apps.
- Includes a browser print/save-as-PDF result export.

## Limitation

The calculator runs as a separate hosted web page. Power Apps will open it, but Power Apps will not automatically receive or save the scoring result.

If result saving is needed later, add one of these:

- A Power Automate flow endpoint called by the HTML app.
- A SharePoint/Dataverse save API behind Azure Functions.
- A later PCF version when deeper Power Apps integration is required.
