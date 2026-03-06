# Deployment Guide — IIS

## Prerequisites

- Windows Server with IIS installed and enabled
- [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) (if using SPA routing)
- .NET Hosting Bundle installed (if applicable)
- A publish-ready build of the application

---

## 1. Build the Application

```bash
dotnet publish -c Release -o ./publish
```

Or for a static/front-end project, run your build tool:

```bash
npm run build
```

---

## 2. Create the IIS Site

1. Open **IIS Manager** (`inetmgr`)
2. Right-click **Sites** → **Add Website**
3. Fill in:
   - **Site name:** `AI-Webpage`
   - **Physical path:** Point to your publish/build output folder
   - **Binding:** Set hostname, IP, and port (e.g. port `80` / `443`)
4. Click **OK**

---

## 3. Application Pool

1. In IIS Manager, go to **Application Pools**
2. Select the pool created for this site
3. Set **.NET CLR version** to:
   - `No Managed Code` — for .NET Core / .NET 5+ apps
   - `v4.0` — for .NET Framework apps
4. Ensure **Identity** has read access to the site folder

---

## 4. Folder Permissions

Grant the IIS app pool identity read (and write if needed) access to the site folder:

```powershell
icacls "C:\inetpub\wwwroot\AI-Webpage" /grant "IIS AppPool\AI-Webpage:(OI)(CI)RX"
```

---

## 5. HTTPS / SSL

1. In IIS Manager, select the site → **Bindings** → **Add**
2. Type: `https`, Port: `443`
3. Select your SSL certificate (use [Let's Encrypt via win-acme](https://www.win-acme.com/) for free certs)

---

## 6. URL Rewrite (SPA / Single Page Apps)

If the app uses client-side routing, add a `web.config` to redirect all requests to `index.html`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="SPA Fallback" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

---

## 7. Verify Deployment

- Browse to `http://localhost` (or your configured hostname)
- Check **IIS Manager → Sites → AI-Webpage** for status
- Review logs at: `C:\inetpub\logs\LogFiles\`

---

## Troubleshooting

| Issue | Fix |
|---|---|
| 500.19 error | Check `web.config` syntax; ensure URL Rewrite is installed |
| 403 Forbidden | Check folder permissions for the app pool identity |
| 502 Bad Gateway | App process not starting — check Event Viewer |
| Blank page (SPA) | Add URL Rewrite rule (see step 6) |
