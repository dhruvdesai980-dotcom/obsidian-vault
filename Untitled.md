You've made the port change correctly, and I can see you're working on the `fix/windows-IIS-configuration` branch.

Next, we need to configure IIS to launch your Node.js application on Windows App Service.

Your current setup is:

- `config.js` now preserves `process.env.PORT`.
    
- `start.js` retrieves secrets from Vault and launches `server.js`.
    
- The Windows App Service contains your deployed application files.
    
- The deployed root does not contain `web.config`.
    

We'll keep the Linux deployment unchanged and introduce a Windows-specific startup configuration.

## Step 1 — Create a Windows entry point

Inside your `issue-tracker` directory, create a file named `app.js`.

```
issue-tracker/
├── app.js              ← NEW
├── web.config          ← NEW (Step 2)
├── client/
├── server/
└── package.json
```

Add this to `app.js`:

```
// Windows Azure App Service entry point
require("./server/src/scripts/start.js");
```

This delegates startup to your existing script, preserving the Vault secret retrieval and child-process initialization.

Your existing Linux startup command remains unchanged.

## Step 2 — Create `web.config`

In the same `issue-tracker` directory, create `web.config`:

```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>

    <handlers>
      <add
        name="iisnode"
        path="app.js"
        verb="*"
        modules="iisnode"
      />
    </handlers>

    <rewrite>
      <rules>
        <rule name="NodeJS" stopProcessing="true">
          <match url=".*" />
          <action
            type="Rewrite"
            url="app.js"
          />
        </rule>
      </rules>
    </rewrite>

    <iisnode
      loggingEnabled="true"
      devErrorsEnabled="true"
    />

  </system.webServer>
</configuration>
```

This is a basic IISNode configuration based on the Windows App Service hosting pattern documented by Microsoft.

![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=32)

Microsoft Learn

+1

It directs incoming requests to `app.js`, which invokes your existing startup script.

Important: This is an initial configuration for troubleshooting. Because your application uses Socket.IO, we may need to adjust IIS WebSocket handling after basic HTTP requests work. Diagnostic error details should also be disabled once testing is complete.

## Step 3 — Check application dependencies

Your previous Azure directory listing did not show `server/node_modules`.

Before redeploying, inspect how the TD pipeline installs or packages dependencies. Your existing application has:

```
"install:all": "cd server && npm install && cd ../client && npm install"
```

If the deployment excludes `node_modules`, the server will not start unless dependency installation occurs during deployment.

Microsoft notes that Windows App Service can install production dependencies during supported build-automation deployments, but your custom TD packaging process may handle this differently.

![](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com&sz=32)

Microsoft Learn

## Step 4 — Make sure both new files are deployed

This is particularly important for your TD framework.

Both components use:

```
buildContext: issue-tracker
```

But we have not confirmed whether the packaging process includes `app.js` and `web.config`.

After deployment, check:

```
cd C:\home\site\wwwroot
dir app.js
dir web.config
```

Both files need to be present at the application root.

Also verify that the Windows deployment has its required dependencies and frontend build.

## Step 5 — Test the Windows application

Once you've verified the packaged files and redeployed, test:

```
https://d4080-eastus2-as-binbl-issue-tracker-dev-plan2.azurewebsites.net/
```

And:

```
https://d4080-eastus2-as-binbl-issue-tracker-dev-plan2.azurewebsites.net/health
```

If the 403 changes to an HTTP 500 error, that would indicate progress in configuring the IIS request handler, but we would then need to inspect IISNode logs to diagnose startup.

One additional thing to watch: your `start.js` spawns `server.js` as a child process. The IISNode named-pipe value must reach that child unchanged. Your updated `config.port` supports that, but the complete process chain still needs to be tested.

For now, implement Steps 1 and 2 locally, but don't commit yet. First, we should verify how your TD packaging process handles the new files and dependencies, so that we don't unnecessarily affect the working Linux deployment.