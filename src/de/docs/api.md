Diese Referenz dokumentiert die Server-API von RESTCaptcha.

## Allgemein

Die vollständige URL der API-Aufruffe ergibt sich aus der konfigurierten Domäne Deines Servers, auf dem RESTCaptcha installiert ist. 

In dieser Dokumentation gehen wir beispielhaft von folgender Domäne aus:

```
captcha.beispiel.de
```

API-Fehler werden als [RFC 9457 Problem Details](https://datatracker.ietf.org/doc/html/rfc9457) zurückgegeben.

## CAPTCHA-API

### Challenge anfordern

Fordert eine neue Challenge an.

#### Request

```
GET https://captcha.beispiel.de/v1/challenge?siteKey={Mein-Site-Schlüssel}
```

Die Query-Parameter bedeuten:

`siteKey` *(string, required)*

:    Öffentlicher Site-Schlüssel

#### Responses

`200 OK` *(application/json)*

:   Liefert ein JSON-Objekt mit Informationen zur Challenge zurück.  

    Beispiel:

    ```json
    {
      "challengeType": {
        "type": "proofOfWork",
        "algorithm": "hash-sha-256",
        "difficulty": 4
      },
      "token": "2c8f4b2d-7ab6-4f3a-9c2a-2e6b3a1c9e8f"
    }
    ```

    Die Eigenschaften bedeuten:
    
    `challengeType.type`:
    
    :     Challenge-Typ, im Moment immer `proofOfWork`
    
    `challengeType.algorithm`:
    
    :     Der zu nutzende Hash-Algorithmus:

          Wert           | Beschreibung
          -------------- | ------------
          `hash-sha-256` | SHA 256-basierter Hash-Algorithmus
          `hash-sha-384` | SHA 384-basierter Hash-Algorithmus
          `hash-sha-512` | SHA 512-basierter Hash-Algorithmus

    `challengeType.difficulty`:
    
    :     Schwierigkeitsgrad der Challenge
    
    `token`
    
    :    Eine eindeutige, zufällige Zeichenfolge (Nonce).

Mögliche Fehler *(application/problem+json)*

+ `400 Bad Request`
+ `403 Forbidden`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Beispiel

```bash
curl -sS "https://captcha.beispiel.de/v1/challenge?siteKey=<Mein-Site-Schlüssel>" \
  -H "Accept: application/json"
```

---

###  Lösung validieren

Validiert die vom Client übermittelte Lösung zu einer Challenge.

#### Request

```
POST https://captcha.beispiel.de/v1/verify?siteKey={Mein-Site-Schlüssel}
```

Die Query-Parameter bedeuten:

`siteKey` *(string, required)*

:    Öffentlicher Site-Schlüssel

Der Request-Body (`application/json`)

Beispiel:

```json
{
  "callerIp": "203.0.113.42",
  "siteSecret": "<Mein-Site-Secret>",
  "solution": "<Meine-Challenge-Lösung>",
  "token": "<Mein-Challenge-Token>"
}
```

Die Eigenschaften bedeuten:

`siteSecret` *(string, required)*

:    Geheimer Schlüssel der Website

`solution` *(string, required)*

:    Gelöste Antwort des Clients

`token` *(string, required)*

:    Token (Nonce) aus `/v1/challenge`

`callerIp` *(string, optional)*

:    IP des Anfragenden (falls relevant)

#### Responses

`200 OK` *(application/json)*

:   Liefert ein JSON-Objekt mit Statusinformationen zur Verifikation zurück.  

    Beispiel:

     ```json
     {
       "hostName": "www.beispiel.de",
       "status": "success"
     }
     ```

    Die Eigenschaften bedeuten:
    
    `hostName`:
    
    :    Hostname
    
    `status`
    
    :    Das Ergebnis der Verifikation:

         Wert               | Bedeutung
         ------------------ | ---------
         `success`          | Alles OK
         `invalid-token`    | Der Token ist ungültig
         `invalid-solution` | Die Lösung ist ungültig

Mögliche Fehler (*application/problem+json*)

+ `400 Bad Request`
+ `401 Unauthorized`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Beispiel

```bash
curl -sS "https://captcha.beispiel.de/v1/verify?siteKey=<Mein-Site-Schlüssel>" \
  -H "Content-Type: application/json" \
  -d '{
        "callerIp": "203.0.113.42",
        "siteSecret": "<Mein-Site-Secret>",
        "solution": "<Meine-Challenge-Lösung>",
        "token": "<Mein-Challenge-Token>"
      }'
```

### Challenge zurücksetzen

Macht ein bestehendes Token ungültig, damit eine Challenge neu gestartet werden kann.

#### Request

```
POST https://captcha.beispiel.de/v1/reset?siteKey={Mein-Site-Schlüssel}
```

Die Query-Parameter bedeuten:

`siteKey` *(string, required)*

:    Öffentlicher Site-Schlüssel

Der Request-Body (`application/json`)

Beispiel:

```json
{
  "token": "<ChallengeToken>"
}
```

Die Eigenschaften bedeuten:

`token` *(string, required)*

:   Token (Nonce) aus `/v1/challenge`

#### Responses

`200 OK` *(application/json)*

:   Kein Reponse-Body.  

Mögliche Fehler (*application/problem+json*):

+ `400 Bad Request`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Beispiel

```bash
curl -sS "https://captcha.beispiel.de/v1/reset?siteKey=<Mein-Site-Schlüssel>" \
  -H "Content-Type: application/json" \
  -d '{ "token": "<Mein-Challenge-Token>" }'
```

## Health-API

Die Health-API liefert Informationen über den Zustand des RESTCaptcha-Server.

#### Request

```
GET /health
```

Die Health-API kann optional über einen API-Schlüssel abgesichert werden. Der API-Schlüssel kann entweder als `X-API-KEY`-Header oder als `Authorization`-Header übergeben werden.

`X-API-KEY`

:    Dein API-Schlüssel als `X-API-KEY`-Header.

     Beispiel:

     ```
     X-API-KEY: <Mein-API-Schlüssel>
     ```

`Authorization`

:    Dein API-Schlüssel als `Authorization`-Header.

     Beispiel:

     ```
     Authorization: ApiKey <Mein-API-Schlüssel>
     ```

#### Responses

`200 OK` *(application/json)*

:   Liefert ein JSON-Objekt mit Statusinformationen zurück.  

    Beispiel:

     ```json
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

Mögliche Fehler (*application/problem+json*):

+ `401 Unauthorized`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Beispiel

``` bash
curl -sS https://captcha.beispiel.de/health \
  -H "X-API-KEY: <Mein-API-Schlüssel>" \
  -H "Accept: application/json"
```
