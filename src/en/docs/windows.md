In this chapter, we’ll demonstrate how to install RESTCaptcha on [Windows Server 2025](https://www.microsoft.com/windows-server) using IIS 10 as the hosting environment. All steps are performed using PowerShell.

## Installing IIS

Open PowerShell as an administrator and check whether IIS 10 is already installed:

``` powershell
Get-WindowsFeature -Name Web-Server, Web-Scripting-Tools
```

If the result looks like this, you’re good to go and can proceed straight to the next section, *“Installing ASP.NET”*:

``` powershell
Display Name                                            Name                       Install State
------------                                            ----                       -------------
[X] Web Server (IIS)                                    Web-Server                     Installed
        [X] IIS Management Scripts and Tools            Web-Scripting-Tools            Installed
```

If one of these components is missing, install the Windows features manually:

``` powershell
Install-WindowsFeature -Name Web-Server, Web-Scripting-Tools
```

Then check whether you can access the IIS 10 default website locally:

``` powershell
Invoke-WebRequest -Uri "http://localhost" -UseBasicParsing
```

!!! note "IIS default website"

    The default IIS 10 website is not required for RESTCaptcha. You can—and should—disable it:

    ``` powershell
    Import-Module WebAdministration
    Stop-Website -Name "Default Web Site"
    Set-ItemProperty "IIS:\Sites\Default Web Site" serverAutoStart False
    ```

## Installing ASP.NET

RESTCaptcha requires the [ASP.NET framework](https://dotnet.microsoft.com/apps/aspnet) as a dependency.

Open PowerShell as an administrator and run the following command to install the [ASP.NET Core 10.0 Runtime – Windows Hosting Bundle](https://dotnet.microsoft.com/download/dotnet/10.0):

``` powershell
winget install --id=Microsoft.DotNet.HostingBundle.10 -e --accept-package-agreements --accept-source-agreements
```

## Installing RESTCaptcha

### Copying the binaries

Create a new folder for RESTCaptcha (for example, `C:\Sites\RestCaptcha`) and copy the binaries from the [latest release](https://github.com/openpotato/restcaptcha/releases/latest) into it. The following PowerShell one-liner does this automatically:

``` powershell
$r=Invoke-RestMethod "https://api.github.com/repos/openpotato/restcaptcha/releases/latest" -Headers @{ "User-Agent"="PS" };$a=$r.assets|?{ $_.name -like "*.zip"}|select -f 1;$zip="$env:TEMP\$($a.name)";Invoke-WebRequest $a.browser_download_url -OutFile $zip -Headers @{ "User-Agent"="PS" };Expand-Archive $zip -DestinationPath "C:\Sites\RestCaptcha" -Force;Remove-Item $zip
```

Copy the `appsettings.json` file within this folder and rename it to `appsettings.Production.json`:

``` powershell
Copy-Item C:\Sites\RestCaptcha\appsettings.json C:\Sites\RestCaptcha\appsettings.Production.json
```

Modify the contents of `appsettings.Production.json` as follows:

``` json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.File"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.AspNetCore.Hosting.Diagnostics": "Warning",
        "Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware": "Information"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "File",
        "Args": {
          "path": "c:\\Sites\\RestCaptcha\\logs\\log-.txt",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 7,
          "fileSizeLimitBytes": 10000000,
          "rollOnFileSizeLimit": true
        }
      }
    ]
  },
  "RestCaptcha": {
    "Sites": [
      {
        "name": "My Site",
        "description": "My site using RESTCaptcha",
        "siteKey": "my-site-key",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": []
      }
    ],
    "IpReputationCheck": {
      "AbuseIPDB": {
        "enabled": false,
        "apiKey": "my-abusedbip-api-key"
      },
      "Spamhaus": {
        "enabled": true
      }
    },
    "BehaviorMap": [
      {
        "minRiskScore": 0,
        "maxRiskScore": 75,
        "action": "challenge",
        "ChallengeType": {
          "type": "proofOfWork",
          "algorithm": "hash-sha-256",
          "difficulty": 4
        }
      },
      {
        "minRiskScore": 75,
        "maxRiskScore": 100,
        "action": "block"
      }
    ],
    "HealthCheck": {
      "enabled": true,
      "allowLocal": false,
      "apiKeys": [ "my-api-key" ]
    }
  }
}
```

### Creating a new IIS site

First, load the **IIS Management Module** into your PowerShell session:

``` powershell
Import-Module WebAdministration
```

Next, create a dedicated [IIS App Pool](https://learn.microsoft.com/iis/configuration/system.applicationhost/applicationpools/) for the ASP.NET service and configure it:

``` powershell
New-WebAppPool -Name "AspNetCorePool"
Set-ItemProperty "IIS:\AppPools\AspNetCorePool" -Name managedRuntimeVersion -Value ""
Set-ItemProperty "IIS:\AppPools\AspNetCorePool" -Name enable32BitAppOnWin64 -Value False
```

Now create a new [IIS Site](https://learn.microsoft.com/iis/configuration/system.applicationhost/sites/) pointing to your RESTCaptcha folder `C:\Sites\RestCaptcha`:

``` powershell
New-Item "IIS:\Sites\RestCaptcha" `
   -bindings @(
     @{protocol="http";  bindingInformation="*:8080:"},
     @{protocol="https"; bindingInformation="*:443:captcha.example.com"}
   ) `
   -physicalPath "C:\Sites\RestCaptcha"
Set-ItemProperty "IIS:\Sites\RestCaptcha" -Name applicationPool -Value "AspNetCorePool"
```

You now have two bindings: a local **HTTP** binding on port **8080** and a public **HTTPS** binding on port **443**.

In the folder `C:\Sites\RestCaptcha`, create a new file named `web.config` and insert the following XML:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet"
                arguments=".\RestCaptcha.WebService.dll"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout"
                hostingModel="inprocess">
      <environmentVariables>
        <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />
      </environmentVariables>
    </aspNetCore>
  </system.webServer>
</configuration>
```

The [web.config](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/web-config?view=aspnetcore-10.0) file configures RESTCaptcha as an ASP.NET application, defining how incoming requests are handled. In this case, RESTCaptcha runs within the IIS worker process (*in-process hosting model*).

That completes the basic configuration. Restart your IIS site:

``` powershell
Restart-WebAppPool "AspNetCorePool"; 
Restart-WebItem "IIS:\Sites\RestCaptcha"
```

You can now test the RESTCaptcha health endpoint locally:

``` powershell
Invoke-WebRequest -Uri "http://localhost:8080/health" -UseBasicParsing
```

For public access, you still need a **TLS certificate** for your **HTTPS** binding. A good option is to use [Let’s Encrypt](https://letsencrypt.org/) to generate free TLS certificates.

To do this, install an ACME client. One of the best for Windows is [simple-acme](https://simple-acme.com/). Install and run the command-line tool — once you’ve answered the prompts, simple-acme will communicate with Let’s Encrypt, request a TLS certificate for the domain `captcha.example.com`, and configure it automatically.

A test in your web browser at `https://captcha.example.com/health` should now work successfully.
