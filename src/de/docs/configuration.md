In diesem Kapitel beschreiben wir die Konfigurationsmöglichkeiten des RESTCaptcha-Server. 

Die gesamte Konfiguration des RESTCaptcha-Server befindet sich in der JSON-Datei `appsettings.Production.json`, die Du beim Installieren bereits angelegt hast. 

## RestCaptcha

Alle RESTCaptcha-spezifischen Konfigurationsdaten befinden sich unter `RestCaptcha`. Der folgende Konfigurationsausschnitt zeigt die Standardwerte für `RestCaptcha`: 

``` json
{
  ...
  "RestCaptcha": {
    "ChallengeType": {
      ...
    },
    "HMACKey": "<Generierter Wert>",
    "NonceMaxTTL": "00:30:00",
    "VerificationMinDelay": "00:00:02",
    "HealthCheck": {
      ...
    },
    "Sites": [
      ...
    ]
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`ChallengeType`**

:   Konfigurationsdaten zum Challenge-Typ, den eine Formularseite bei Anfrage zurückgesendet bekommt (siehe weiter unten). Der Challenge-Typ definiert, was der Nutzer einer Formularseite bzw. die Formularseite selbst machen muss, um eine gültige Lösung zu generieren.

**`HMACKey`**

:   Geheimer Schlüssel für HMAC (Hash-based Message Authentication Code). Der generierte Standardwert bei Nichtangabe ist nicht kryptografisch sicher. Bitte verwende einen
Zufallsgenerator wie z.B. [1Password - Password Generator](https://1password.com/password-generator).

**`NonceMaxTTL`**

:   Maximale Lebensdauer der eindeutigen Nonce. Der Standardwert ist `"00:30:00"` (30 Minuten).

**`VerificationMinDelay`**

:   Mindestverzögerung zwischen dem Empfang der Challenge-Anfrage und der serverseitigen Überprüfung. Der Standardwert ist `"00:00:02"` (2 Sekunden).

**`HealthCheck`**

:   Konfigurationsdaten zum HealthCheck-Endpunkt (siehe weiter unten).

**`Sites`**

:   Konfigurationsdaten der registrierten Webseiten (siehe weiter unten).

## RestCaptcha.ChallengeType

RESTCaptcha ist dazu ausgelegt, mit unterschiedlichen Challenge-Typen zu arbeiten (momentan ist aber nur einer implementiert 😊).

**`type`**

:   Der Challenge-Typ, den eine Formularseite bei Anfrage zurückgesendet bekommt:

    Wert          | Beschreibung
    ------------- | ------------
    `proofOfWork` | Proof of Work (zu Deutsch: Arbeitsnachweis)

### ProofOfWork

Der folgende Konfigurationsausschnitt zeigt die Standardwerte des Challenge-Typs `proofOfWork`: 

``` json
{
  ...
  "RestCaptcha": {
    "ChallengeType": {
      "type": "proofOfWork",
      "algorithm": "hash-sha-256",
      "difficulty": 4
    },
    ...
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`algorithm`**

:   Der zu nutzende Hash-Algorithmus:

    Wert           | Beschreibung
    -------------- | ------------
    `hash-sha-256` | SHA 256-basierter Hash-Algorithmus
    `hash-sha-384` | SHA 384-basierter Hash-Algorithmus
    `hash-sha-512` | SHA 512-basierter Hash-Algorithmus

**`difficulty`**

:   Schwierigkeitsgrad der Challenge. Der Standardwert ist `4`.

## RestCaptcha.HealthCheck

Der RESTCaptcha-Server implementiert einen HealthCheck-Endpunkt, der optional durch einen API-Schlüssel geschützt werden kann. Der HealthCheck-Endpunkt gibt auf Anfrage Metriken zum Zustand des RESTCaptcha-Server zurück. 

Eine typsche Antwort sieht wie folgt aus:

``` json
{
  "status": "pass",
  "version": "0.0.1.20175",
  "serviceId": "restcaptcha",
  "checks": {
    "uptime": [
      {
        "componentType": "system",
        "observedValue": 0,
        "observedUnit": "s",
        "time": "2025-10-16T14:05:11.5262654+00:00"
      }
    ]
  }
}
```

Standardmäßig ist der HealthCheck-Endpunkt ohne Einschränkungen erreichbar, kann aber jederzeit eingeschränkt oder ganz deaktiviert werden. Die folgende Konfigurationsausschnitt zeigt die Standardwerte für `RestCaptcha.HealthCheck`: 

``` json
{
  ...
  "RestCaptcha": {
    "HealthCheck": {
      "Enabled": true,
      "PrivateOnly": false,
      "AllowLocal": false,
      "AllowCidrs": [],
      "Keys": []
    },
    "Sites": [
      ...
    ]
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`Enabled`**

:   Ein boolscher Wert:

    Wert    | Beschreibung
    ------- | ------------
    `true`  | HealthCheck-Endpunkt ist aktiviert
    `false` | HealthCheck-Endpunkt ist deaktiviert

**`PrivateOnly`**

:   Steuert, ob nur Endpunkt privat ist. In diesem Fall sind nur Anfragen von Loopback-Adressen und von den unter `AllowCidrs` definierte Netzwerkbereiche zugelassen. API-Schlüssel werden ignoriert und öffentliche Anfragen abgelehnt.

    Wert    | Beschreibung
    ------- | ------------
    `true`  | Privater Endpunkt
    `false` | Öffentlicher Endpunkt

**`AllowLocal`**

:   Steuert, ob Anfragen von lokalen Loopback-Adressen zugelassen werden sollen. Hinter einem Proxy (z.B. nginx) muss sichergestellt werden, dass die weitergeleiteten Header so konfiguriert sind, dass die effektive Client-IP übergeben wird.

    Wert    | Beschreibung
    ------- | ------------
    `true`  | Lokale Loopback-Adressen zulassen
    `false` | Lokale Loopback-Adressen ablehnen

**`AllowCidrs`**

:   Eine Liste vertrauenswürdiger Netzwerkbereiche in CIDR-Notation ([Classless Inter-Domain Routing](https://datatracker.ietf.org/doc/html/rfc4632)). Ist `PrivateOnly = false`, sind Anfragen aus diesen Bereichen ohne API-Schlüssel erlaubt. Ist `PrivateOnly = true`, sind nur Anfragen aus diesen Bereichen (plus Loopback, falls aktiviert) zulässig.

**`Keys`**

:   Eine Liste von möglichen API-Schlüsseln für die Authorisierung des HealthCheck-Endpunkts.

## RestCaptcha.Sites

Damit Webseiten mit dem RESTCaptcha-Server interagieren können, müssen Sie registriert werden. Die folgende Konfigurationsausschnitt zeigt eine typische Registrierung: 

``` json
{
  ...
  "RestCaptcha": {
    ...
    "Sites": [
      {
        "name": "Mein Site",
        "description": "Nur zum Testen",
        "siteKey": "Mein-Site-Schlüssel",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": ["www.beispiel.de"]
      }
    ]
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`name`**

:   Name der registrierten Webseite. Dient nur zu Dokumentationszwecken.

**`description`**

:   Zusätzliche Beschreibung der registrierten Webseite. Dient nur zu Dokumentationszwecken.

**`siteKey`**

:   Schlüssel für die Zuordnung von Anfragen zu einer registrierten Webseite.

**`siteSecret`**

:   Kennwort für die Authorisierung von Anfragen für die registrierte Webseite.

**`validHostNames`**

:   Eine optionale Liste von gültigen Domainnamen, die mit dieser Website verknüpft sind. Ist die Liste gefüllt, werden nur Anfragen entgegenneommen, die von diesen Domänen stammen. Hinter einem Proxy (z.B. nginx) muss sichergestellt werden, dass der ursprüngliche Hostname weitergeleitet wird.

## SeriLog

Der RESTCaptcha-Server nutzt für das Logging die .NET-Bibliothek [Serilog](https://github.com/serilog/serilog). RESTCaptcha implementiert die folgenden Log-Ausgaben:

+ [Konsolenausgabe](https://github.com/serilog/serilog-sinks-console)

    Beispiel:

    ``` json
    {
      "Serilog": {
        "Using": [
          "Serilog.Sinks.Console",
        ],
        "WriteTo": [{"Name": "Console"}]
      }
      ...
    }
    ```

+ [Dateiausgabe](https://github.com/serilog/serilog-sinks-file)

    Beispiel:

    ``` json
    {
      "Serilog": {
        "Using": [
          "Serilog.Sinks.File",
        ],
        "WriteTo": [
          "Name": "File",
          "Args": {
            "path": "/var/log/restcaptcha/log-.txt",
            "rollingInterval": "Day",
            "retainedFileCountLimit": 7,
            "fileSizeLimitBytes": 10000000,
            "rollOnFileSizeLimit": true
          }          
        ]
      }
      ...
    }
    ```

+ [OpenTelemetry-Ausgabe](https://github.com/serilog/serilog-sinks-opentelemetry)
