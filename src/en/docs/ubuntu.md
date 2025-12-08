In this chapter, we’ll demonstrate how to install RESTCaptcha on [Ubuntu Server 24.04 LTS](https://ubuntu.com/server) using [nginx](https://nginx.org/) as a reverse proxy.

## Installing nginx

Check whether nginx is already installed:

``` bash
nginx -v
```

If the result looks like this, you’re all set and can proceed to the next section, *“Installing ASP.NET”*:

``` text
nginx version: nginx/1.28.0
```

If instead you see `Command 'nginx' not found`, you need to install nginx:

``` bash
sudo apt update && sudo apt install nginx
```

To start nginx, enter the following:

``` bash
sudo systemctl start nginx
```

To enable automatic startup when your server boots:

``` bash
sudo systemctl enable nginx
```

Then test whether you can access nginx’s default webpage locally:

``` bash
wget -S --spider localhost
```

!!! note "nginx default website"

    The nginx default site is not required for RESTCaptcha. You can—and should—disable it:

    ```bash
    sudo unlink /etc/nginx/sites-enabled/default
    sudo systemctl reload nginx
    ```

## Installing ASP.NET

RESTCaptcha requires the [ASP.NET framework](https://dotnet.microsoft.com/apps/aspnet) as a dependency.

The installation of ASP.NET Core 10 is described in detail in the [Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?pivots=os-linux-ubuntu-2404&tabs=dotnet10). Here’s the short version:

First, add the package repository:

``` bash
sudo add-apt-repository ppa:dotnet/backports
```

Then install the ASP.NET Core runtime:

``` bash
sudo apt-get update && sudo apt-get install -y apt-transport-https 
sudo apt-get update && sudo apt-get install -y aspnetcore-runtime-10.0
```

## Installing RESTCaptcha

### Copying the binaries

Create a new directory for RESTCaptcha (e.g. `/usr/share/restcaptcha`) and copy the binaries from the [latest release](https://github.com/openpotato/restcaptcha/releases/latest) into it.

The following Bash one-liner will do this for you:

```bash
curl -sL -H "User-Agent: curl" https://api.github.com/repos/openpotato/restcaptcha/releases/latest | jq -r '.assets[] | select(.name | endswith(".zip")) | .browser_download_url' | head -n1 | xargs -I{} bash -c 'tmp=$(mktemp /tmp/restcaptcha.XXXX.zip); curl -L -H "User-Agent: curl" -o "$tmp" "{}"; sudo mkdir -p /usr/share/restcaptcha; sudo unzip -o "$tmp" -d /usr/share/restcaptcha; rm "$tmp"'
```

Now configure ownership and access permissions for the directory:

``` bash
sudo chown -R nginx:nginx /usr/share/restcaptcha
sudo chmod -R u=rwX,g=rX,o= /usr/share/restcaptcha
```

Copy the `appsettings.json` file under `/usr/share/restcaptcha` and rename it `appsettings.Production.json`:

``` bash
sudo cp /usr/share/restcaptcha/appsettings.json /usr/share/restcaptcha/appsettings.Production.json
```

Change the contents of `appsettings.Production.json` as follows:

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

Next, create a systemd service file for the API web service:

``` bash
sudo nano /etc/systemd/system/restcaptcha.service
```

Insert the following content:

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

Then start the API web service:

``` bash
sudo systemctl daemon-reload
sudo systemctl enable restcaptcha.service
sudo systemctl start restcaptcha.service
```

Verify that it’s running successfully:

``` bash
sudo systemctl status restcaptcha.service
```

### Nginx as a reverse proxy

Create a new nginx configuration file and add the following content:

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

    # Redirect server error pages to the static page /50x.html
    error_page 500 502 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}

server {
    server_name  captcha.example.com;
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

    # Redirect server error pages to the static page /50x.html
    error_page 500 502 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    # Enable HTTP/2 protocol
    http2 on;
}
```

Restart nginx:

``` bash
sudo systemctl restart nginx
```

You can now test the RESTCaptcha health endpoint locally:

``` powershell
Invoke-WebRequest -Uri "http://localhost:8080/health" -UseBasicParsing
```

For public access, you’ll still need a **TLS certificate** for your **HTTPS** binding. A good option is to use [Let’s Encrypt](https://letsencrypt.org/) to generate free TLS certificates.

To do so, install an ACME client. One of the best for Linux is [Certbot](https://certbot.eff.org/). Install and run Certbot; once you’ve answered all questions, it will communicate with Let’s Encrypt, request a TLS certificate for the domain `captcha.example.com`, and automatically update your nginx configuration.

A test in your web browser at `https://captcha.example.com/health` should now succeed.
