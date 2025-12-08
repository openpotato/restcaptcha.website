In diesem Kapitel beschreiben wir die Konfigurationsmöglichkeiten des RESTCaptcha-Server. 

Die gesamte Konfiguration des RESTCaptcha-Server befindet sich in der JSON-Datei `appsettings.Production.json`, die Du beim Installieren bereits angelegt hast. 

## RestCaptcha

Alle RESTCaptcha-spezifischen Konfigurationsdaten befinden sich unter `RestCaptcha`. Der folgende Konfigurationsausschnitt zeigt die Standardwerte für `RestCaptcha`: 

``` json
{
  ...
  "RestCaptcha": {
    "Sites": [
      ...
    ],
    "BehaviorMap": [
      ...
    ],
    "HMACKey": "<Generierter Wert>",
    "NonceMaxTTL": "00:30:00",
    "VerificationMinDelay": "00:00:02",
    "IpReputationCheck": {
      ...
    },
    "HealthCheck": {
      ...
    }
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`Sites`**

:   Konfigurationsdaten der registrierten Webseiten (siehe weiter unten).

**`BehaviorMap`**

:   Definiert, wie RESTCaptcha auf unterschiedliche Risikowerte einer Anfrage reagieren soll. Je höher der Risiko-Score, desto strenger sollte die Maßnahme ausfallen. Der Risiko-Score liegt zwischen 0 (sehr geringes Risiko) und 100 (sehr hohes Risiko). Mögliche Reaktionen sind entweder eine Captcha-Challenge mit unterschiedlicher Schwierigkeit oder Blocken der Anfrage.

**`HMACKey`**

:   Geheimer Schlüssel für HMAC (Hash-based Message Authentication Code). Der generierte Standardwert bei Nichtangabe ist nicht kryptografisch sicher. Bitte verwende einen Zufallsgenerator wie z.B. [1Password - Password Generator](https://1password.com/password-generator).

**`NonceMaxTTL`**

:   Maximale Lebensdauer der eindeutigen Nonce. Der Standardwert ist `"00:30:00"` (30 Minuten).

**`VerificationMinDelay`**

:   Mindestverzögerung zwischen dem Empfang der Challenge-Anfrage und der serverseitigen Überprüfung. Der Standardwert ist `"00:00:02"` (2 Sekunden).

**`IpReputationCheck`**

:   Konfigurationsdaten für die Durchführung von IP-Reputationsüberprüfungen. IP-Reputationsüberprüfungen sind entscheidend für die Erstellung eines Risiko-Score.

**`HealthCheck`**

:   Konfigurationsdaten zum HealthCheck-Endpunkt (siehe weiter unten).

## RestCaptcha.Sites

Damit Webseiten mit dem RESTCaptcha-Server interagieren können, müssen Sie registriert werden. Die folgende Konfigurationsausschnitt zeigt eine typische Registrierung: 

``` json
{
  ...
  "RestCaptcha": {
    "Sites": [
      {
        "name": "Mein Site",
        "description": "Nur zum Testen",
        "siteKey": "Mein-Site-Schlüssel",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": ["www.beispiel.de"]
      }
    ]
    ...
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

## RestCaptcha.BehaviorMap[]

Ein typischer Inhalt in der BehaviorMap sieht wie folgt aus:

``` json
{
  ...
  "RestCaptcha": {
    ...
    "BehaviorMap": [
    {
      "minRiskScore": 0,
      "maxRiskScore": 25,
      "action": "challenge",
      "ChallengeType": {
        "type": "proofOfWork",
        "algorithm": "hash-sha-256",
        "difficulty": 4
      }
    },
    {
      "minRiskScore": 25,
      "maxRiskScore": 75,
      "action": "challenge",
      "ChallengeType": {
        "type": "proofOfWork",
        "algorithm": "hash-sha-512",
        "difficulty": 6
      }
    },
    {
      "minRiskScore": 75,
      "maxRiskScore": 100,
      "action": "block"
    }
  ],
  ...
}  
```

Jeder Eintrag definiert eine Reaktion für einen bestimmten Risiko-Score-Berreich. Die Eigenschaften haben folgende Bedeutung:

**`minRiskScore`**

:   Der minimale Risiko-Score, ab dem die unter `action` definierte Aktion ausgeführt werden soll.

**`maxRiskScore`**

:   Der maximale Risiko-Score, bis zu dem die unter `action` definierte Aktion ausgeführt werden soll.

**`action`**

:   Die Aktion, die mit dem definierten Risiko-Score-Berreich vernüpft ist:

    Wert        | Beschreibung
    ------------| ------------
    `challenge` | Es wird eine Challenge generiert und zurückgeliefert.
    `block`     | Die Anfrage wird geblockt.

**`ChallengeType`**

:   Ist als Aktion `challenge` spezifiziert, wird hier der gewünschte Challenge-Typ definiert.

## RestCaptcha.BehaviorMap[].ChallengeType

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
  "ChallengeType": {
    "type": "proofOfWork",
    "algorithm": "hash-sha-256",
    "difficulty": 4
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

## RestCaptcha.IpReputationCheck

RESTCaptcha kann auf Wunsch die Reputation der Client-IP überprüfen, bevor es entscheidet, welöche Challnge zurückgesgesndet werden soll. Dabei kann auf die folgednen zwei Reputation-Dienste zurückgegriffen werden:

+ [AbuseDBIP](https://www.abuseipdb.com/): Eine öffentlich zugängliche Datenbank, die dabei hilft, schädliche IP-Adressen zu identifizieren. Nutzer und Sicherheitssysteme können dort verdächtige IPs melden. Die Plattform sammelt diese Meldungen, bewertet die IPs anhand eines *Abuse Confidence Score* und bietet eine API, über die Entwickler automatisiert prüfen können, ob eine IP-Adresse als gefährlich eingestuft wird. Für die kostenlose Nutzung der API wird ein API-Key benötigt.

+ [Spamhaus](https://www.spamhaus.org/): Eine international anerkannte Organisation, die Daten über Spam, Malware-Verteilung, Botnet-Aktivitäten und andere Sicherheitsbedrohungen sammelt und bereitstellt. Ihre Datenbanken (z. B. SBL, XBL, PBL, DROP/EDROP) werden weltweit von E-Mail-Servern, Firewalls und Sicherheitsdiensten genutzt, um schädliche IP-Adressen, Domains und Botnet-Infrastrukturen zu erkennen und zu blockieren. Eine kostenlose Abfrage ist via DNS Query möglich.

Standardmäßig ist nur die Abfrage via Spamhaus aktiviert, da für AbuseDBIP ein individueller Api-Key benötigt wird. Der folgende Konfigurationsausschnitt zeigt die Standardwerte für `RestCaptcha.IpReputationCheck`: 

``` json
{
  ...
  "RestCaptcha": {
    ...    
    "IpReputationCheck": {
      "AbuseIPDB": {
        "enabled": false,
        "apiKey": "my-api-key"
      },
      "Spamhaus": {
        "enabled": true
      }
    },
    ...
  }
}
```

Die Eigenschaften haben folgende Bedeutung:

**`AbuseIPDB.enabled`**

:   Soll eine IP-Reputationsabfrage via AbuseIPDB durchgeführt werden?

**`AbuseIPDB.apiKey`**

:   API-Key für AbuseIPDB.

**`Spamhaus.enabled`**

:   Soll eine IP-Reputationsabfrage via Spamhaus durchgeführt werden?

!!! note "Risiko-Score"

    Ist die IP-Reputationsabfrage komplett deaktiviert oder ist eine Client-IP in keinem der beiden Dienste verzeichnet, ist der Risiko-Score immer 0.

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

Standardmäßig ist der HealthCheck-Endpunkt ohne Einschränkungen erreichbar, kann aber jederzeit eingeschränkt oder ganz deaktiviert werden. Der folgende Konfigurationsausschnitt zeigt die Standardwerte für `RestCaptcha.HealthCheck`: 

``` json
{
  ...
  "RestCaptcha": {
    ...
    "HealthCheck": {
      "Enabled": true,
      "PrivateOnly": false,
      "AllowLocal": false,
      "AllowCidrs": [],
      "Keys": []
    }
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

    Typische Beispiele für IPv4:

    CIDR             | Bedeutung                                      | Anzahl der IPs        | Verwendung
    ---------------- | ---------------------------------------------- | --------------------- | ----------
    192.168.0.0/24   | Die ersten 24 Bits sind Netzwerkbits           | 256 IPs (254 nutzbar) | Typisches Home-LAN
    10.0.0.0/8       | Netzwerk = 10.x.x.x                            | 16.777.216 IPs        | Große private Netzwerke
    172.16.0.0/12    | Netzwerk = 172.16.0.0–172.31.255.255           | 1.048.576 IPs         | Private Netzwerke
    192.168.1.128/25 | Netzwerk aufgeteilt in die oberen 128 Adressen | 128 IPs               | Subnetting / Lastenausgleich
    192.168.1.0/30   | Nur 4 IPs (2 nutzbar)                          | 4 IPs                 | Punkt-zu-Punkt-Verbindungen

    Typische Beispiele für IPv6:

    CIDR                     | Bedeutung
    ------------------------ | ---------
    2001:db8::/32            | Dokumentationspräfix (großer Block)
    2001:db8:abcd::/48       | Typische Zuweisung für ISP-Kunden
    2001:db8:abcd\:1234::/64 | Standard-IPv6-LAN-Subnetz
    fe80::/10                | Link-Local-Adressbereich

**`Keys`**

:   Eine Liste von möglichen API-Schlüsseln für die Authorisierung des HealthCheck-Endpunkts.

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
