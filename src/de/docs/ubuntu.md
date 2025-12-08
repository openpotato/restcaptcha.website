In diesem Kapitel beschreiben wir beispielhaft die Installation von RESTCaptcha unter [Ubuntu Server 24.04 LTS](https://ubuntu.com/server) mit [nginx](https://nginx.org/) als Reverse-Proxy. 

## Installation nginx

Überprüfe, ob Du nginx bereits installiert hast:

``` bash
nginx -v
```

Sieht das Ergebnis wie folgt aus, musst Du nichts weiter machen und kannst gleich zum nächsten Abschnitt "Installation ASP\.NET" springen:

``` text
nginx version: nginx/1.28.0
```

Sollte stattdessen `Command 'nginx' not found` erscheinen, musst Du nginx noch installieren:

``` bash
sudo apt update && sudo apt install nginx
```

Zum Starten von nginx tippe folgendes ein:

``` bash
sudo systemctl start nginx
```

Für einen Autostart beim Booten Deines Servers tippe folgendes ein:

``` bash
sudo systemctl enable nginx
```

Teste anschließend, ob Du lokal die Standardwebseite von nginx aufrufen kannst:

``` bash
wget -S --spider localhost
``` 

!!! note "nginx-Standardwebseite"

    Die Standardwebseite von nginx wird für RESTCaptcha nicht benötigt. Du kannst und solltest sie deaktivieren:

    ``` bash
    sudo unlink /etc/nginx/sites-enabled/default
    sudo systemctl reload nginx
    ```

## Installation ASP\.NET

RESTCaptcha benötigt das [ASP\.NET-Framework](https://dotnet.microsoft.com/apps/aspnet) als Abhängigkeit.

Die Installation von ASP.NET Core 10 wird in der [Microsoft-Dokumentation](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?pivots=os-linux-ubuntu-2404&tabs=dotnet10) ausführlich beschrieben. Hier die Kurzfassung:

Zunächst musst Du das Paket-Repository hinzufügen:

``` bash
sudo add-apt-repository ppa:dotnet/backports
```

Dann installierst Du die ASP.NET Core-Runtime:

``` bash
sudo apt-get update && sudo apt-get install -y apt-transport-https 
sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-10.0
```

## Installation RESTCaptcha

### Binaries kopieren

Lege einen neues Verzeichnis für RESTCaptcha (Beispiel: `/usr/share/restcaptcha`) an und kopiere die Binaries des [aktuellen Release](https://github.com/openpotato/restcaptcha/releases/latest) dort hinein. 

Der folgende Bash-One-Liner macht das für Dich:

``` bash
curl -sL -H "User-Agent: curl" https://api.github.com/repos/openpotato/restcaptcha/releases/latest | jq -r '.assets[] | select(.name | endswith(".zip")) | .browser_download_url' | head -n1 | xargs -I{} bash -c 'tmp=$(mktemp /tmp/restcaptcha.XXXX.zip); curl -L -H "User-Agent: curl" -o "$tmp" "{}"; sudo mkdir -p /usr/share/restcaptcha; sudo unzip -o "$tmp" -d /usr/share/restcaptcha; rm "$tmp"'
```

Jetzt konfigurieren wir Eigentümer und Zugriffsrechte für die Verzeichnisse:

``` bash
sudo chown -R nginx:nginx /usr/share/restcaptcha
sudo chmod -R u=rwX,g=rX,o= /usr/share/restcaptcha
```

Kopiere unter `/usr/share/restcaptcha` die Datei `appsettings.json` und nenne sie `appsettings.Production.json`:

``` bash
sudo cp /usr/share/restcaptcha/appsettings.json /usr/share/restcaptcha/appsettings.Production.json
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
          "path": "/var/log/restcaptcha/log-.txt",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 14,
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

Für den API-Webservice müssen wir eine Service-Datei anlegen:

``` bash
sudo nano /etc/systemd/system/restcaptcha.service
```

Da kommt folgender Inhalt rein:

``` text
[Unit]
Description=RESTCaptcha Web Service

[Service]
WorkingDirectory=/usr/share/restcaptcha
ExecStart=/usr/bin/dotnet RestCaptcha.WebService.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=restcaptcha
User=nginx
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false
Environment=ASPNETCORE_URLS=http://localhost:5030

[Install]
WantedBy=multi-user.target
```

Anschließend wird der API-Webservice gestartet

``` bash
sudo systemctl daemon-reload
sudo systemctl enable restcaptcha.service
sudo systemctl start restcaptcha.service
```

Die folgende Überprüfung sollte erfolgreich sein:

``` bash
sudo systemctl status restcaptcha.service
```

### Nginx als Reverse Proxy

Erstelle eine neue Konfigurationsdatei für Nginx und füge folgenden Inhalt hinzu:

``` nginx
server {
    server_name  localhost;
    listen       127.0.0.1:8080;
    listen       [::1]:8080;
    root         /usr/share/restcaptcha.webservice;

    # Proxy request to Kestrel
    location / {
        proxy_pass         http://127.0.0.1:5030;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }

    # redirect server error pages to the static page /50x.html
    error_page 500 502 504  /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}

server {
    server_name  captcha.beispiel.de;
    listen       443 ssl; 
    root         /usr/share/restcaptcha.webservice;

    # Proxy request to Kestrel
    location / {
        proxy_pass         http://127.0.0.1:5030;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }

    # redirect server error pages to the static page /50x.html
    error_page 500 502 504  /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    # Turn on http2 protocol
    http2 on;
}
```

Starte anschließend nginx neu:

``` bash
sudo systemctl restart nginx
```

Der lokale folgende Test ruft den Health-Endpunkt von RESTCaptcha auf und sollte erfolgreich sein:

``` powershell
Invoke-WebRequest -Uri "http://localhost:8080/health" -UseBasicParsing
``` 

Für einen öffentlichen Test fehlt noch ein **TLS-Zertifikat** für unsere **https**-Bindung. Eine gute Möglichkeit ist die Nutzung von [Let's Encrypt](https://letsencrypt.org/) zum Erstellen von kostenlosne TLS-Zertifikaten. 

Du musst dafür einen ACME-Client installieren. Einer der besten Clients für Linux ist [Certbot](https://certbot.eff.org/). Installiere und starte Certbot. Sind alle Fragen beantwortet, kommuniziert Certbot mit Let’s Encrypt, erfragt ein TLS-Zertifikat für die Domäne `captcha.beispiel.de` und führt die notwendigen Änderungen in Deiner nginx-Konfigurationsdatei durch.

Ein Test in einem Web-Browser mit `https://captcha.beispiel.de/health` sollte jetzt erfolgreich funktionieren.

!!! note "Let’s Encrypt und nginx"

    Eine ausführliche Einführung in das Thema findest Du in folgendem Blog-Post: [HTTPS unter nginx/Ubuntu](https://blog.stueber.de/posts/letsencrypt-unter-nginx/).
