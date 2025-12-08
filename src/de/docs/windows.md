In diesem Kapitel beschreiben wir beispielhaft die Installation von RESTCaptcha unter [Windows Server 2025](https://www.microsoft.com/windows-server) mit IIS 10 als Hosting-Umgebung. Wir arbeiten dabei ausschließlich mit PowerShell. 

## Installation IIS

Öffne die PowerShell als Administrator und überprüfe, ob IIS 10 bereits installiert ist:

``` powershell
Get-WindowsFeature -Name Web-Server, Web-Scripting-Tools
```

Sieht das Ergebnis wie folgt aus, musst Du nichts weiter machen und kannst gleich zum nächsten Abschnitt "Installation ASP\.NET" springen:

``` powershell
Display Name                                            Name                       Install State
------------                                            ----                       -------------
[X] Web Server (IIS)                                    Web-Server                     Installed
        [X] IIS Management Scripts and Tools            Web-Scripting-Tools            Installed
```

Sollte da irgendwo ein Kreuzer fehlen, musst Du die Windows-Features nachinstallieren:

``` powershell
Install-WindowsFeature -Name Web-Server, Web-Scripting-Tools
```

Teste anschließend, ob Du lokal die Standardwebseite von IIS 10 aufrufen kannst:

``` powershell
Invoke-WebRequest -Uri "http://localhost" -UseBasicParsing
``` 

!!! note "IIS-Standardwebseite"

    Die Standardwebseite von IIS 10 wird für RESTCaptcha nicht benötigt. Du kannst und solltest sie deaktivieren:

    ``` powershell
    Import-Module WebAdministration
    Stop-Website -Name "Default Web Site"
    Set-ItemProperty "IIS:\Sites\Default Web Site" serverAutoStart False
    ```

## Installation ASP\.NET

RESTCaptcha benötigt das [ASP\.NET-Framework](https://dotnet.microsoft.com/apps/aspnet) als Abhängigkeit.

Öffne die PowerShell als Administrator und tippe folgenden Befehl ein, um das [ASP.NET Core 10.0 Runtime - Windows Hosting Bundle](https://dotnet.microsoft.com/download/dotnet/10.0) zu installieren:

``` powershell
winget install --id=Microsoft.DotNet.HostingBundle.10 -e --accept-package-agreements --accept-source-agreements
```

## Installation RESTCaptcha

### Binaries kopieren

Lege einen neuen Ordner für RESTCaptcha (Beispiel: `C:\Sites\RestCaptcha`) an und kopiere die Binaries des [aktuellen Release](https://github.com/openpotato/restcaptcha/releases/latest) dort hinein. Der folgende PowerShell-One-Liner macht das für Dich:

``` powershell
$r=Invoke-RestMethod "https://api.github.com/repos/openpotato/restcaptcha/releases/latest" -Headers @{ "User-Agent"="PS" };$a=$r.assets|?{ $_.name -like "*.zip"}|select -f 1;$zip="$env:TEMP\$($a.name)";Invoke-WebRequest $a.browser_download_url -OutFile $zip -Headers @{ "User-Agent"="PS" };Expand-Archive $zip -DestinationPath "C:\Sites\RestCaptcha" -Force;Remove-Item $zip
```

Kopiere in diesem Ordner die Datei `appsettings.json` und nenne sie `appsettings.Production.json`:

```powershell
Copy-Item C:\Sites\RestCaptcha\appsettings.json C:\Sites\RestCaptcha\appsettings.Production.json
```

Ändere den Inhalt von `appsettings.Production.json` wie folgt ab:

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
        "name": "Meine Site",
        "description": "Meine Site, die RESTCaptcha nutzt",
        "siteKey": "Mein-Site-Schlüssel",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": []
      }
    ],
    "IpReputationCheck": {
      "AbuseIPDB": {
        "enabled": false,
        "apiKey": "Mein-AbuseIPDB-API-Schlüssel"
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
      "apiKeys": [ "Mein-API-Schlüssel" ]
    }
  }
}
```

### Eine neue IIS Site anlegen

Zunächst laden wir das **IIS Management Module** in unsere PowerShell-Session:

``` powershell
Import-Module WebAdministration
```

Dann erstellen wir einen passenden [IIS App Pool](https://learn.microsoft.com/iis/configuration/system.applicationhost/applicationpools/) für unseren ASP.NET-Dienst und konfigurieren ihn:

``` powershell
New-WebAppPool -Name "AspNetCorePool"
Set-ItemProperty "IIS:\AppPools\AspNetCorePool" -Name managedRuntimeVersion -Value ""
Set-ItemProperty "IIS:\AppPools\AspNetCorePool" -Name enable32BitAppOnWin64 -Value False
```

Dann erstellen wir eine neue [IIS Site](https://learn.microsoft.com/iis/configuration/system.applicationhost/sites/), die auf unseren RESTCaptcha-Ordner `C:\Sites\RestCaptcha` verweist:

``` powershell
New-Item "IIS:\Sites\RestCaptcha" `
   -bindings @(
     @{protocol="http";  bindingInformation="*:8080:"},
     @{protocol="https"; bindingInformation="*:443:captcha.beispiel.de"}
   ) `
   -physicalPath "C:\Sites\RestCaptcha"
Set-ItemProperty "IIS:\Sites\RestCaptcha" -Name applicationPool -Value "AspNetCorePool"
``` 

Wir haben jetzt zwei Bindungen, eine lokale **http**-Bindung auf Port **8080** und eine öffentliche **https**-Bindung auf Port **443**.

Lege im Ordner `C:\Sites\RestCaptcha` eine neue Datei mit Namen `web.config` an und kopiere folgenden XML-Code hinein:

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

Die Datei [web.config](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/web-config?view=aspnetcore-10.0) konfiguriert das Hosting von RESTCaptcha als ASP\.NET-Applikation, also die Art und Weise, wie einkommende Anfragen an RESTCaptcha weitergeleitet werden. In unserem Fall läuft RESTCaptcha innerhalb des IIS-Arbeitsprozesses (in-process hosting model).

Wir sind mit der Basiskonfiguration fertig. Starte Deine IIS-Site neu:

```powershell
Restart-WebAppPool "AspNetCorePool"; 
Restart-WebItem "IIS:\Sites\RestCaptcha"
```

Der folgende lokale Test ruft den Health-Endpunkt von RESTCaptcha auf und sollte erfolgreich sein:

``` powershell
Invoke-WebRequest -Uri "http://localhost:8080/health" -UseBasicParsing
``` 

Für einen öffentlichen Test fehlt noch ein **TLS-Zertifikat** für unsere **https**-Bindung. Eine gute Möglichkeit ist die Nutzung von [Let's Encrypt](https://letsencrypt.org/) zum Erstellen von kostenlosne TLS-Zertifikaten. 

Du musst dafür einen ACME-Client installieren. Einer der besten Clients für Windows ist [simple-acme](https://simple-acme.com/). Installiere ihn, starte die Kommandozeilenanwendung. Sind alle Fragen beantwortet, kommuniziert simple-acme mit Let’s Encrypt und erfragt ein TLS-Zertifikat für die Domäne `captcha.beispiel.de`.

Ein Test in einem Web-Browser mit `https://captcha.beispiel.de/health` sollte jetzt auch erfolgreich funktionieren.

!!! note "Let’s Encrypt und IIS 10"

    Eine ausführliche Einführung in das Thema findest Du in folgendem Blog-Post: [HTTPS unter IIS 10](https://blog.stueber.de/posts/letsencrypt-unter-iis-10).

