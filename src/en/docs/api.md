This reference documents the RESTCaptcha server API.

## General

The full URL for API requests depends on the configured domain of your RESTCaptcha installation.

In this documentation, we’ll assume the following example domain:

```
captcha.example.com
```

API errors are returned using [RFC 9457 Problem Details](https://datatracker.ietf.org/doc/html/rfc9457).

## CAPTCHA API

### Requesting a challenge

Requests a new challenge.

#### Request

```
GET https://captcha.example.com/v1/challenge?siteKey={My-Site-Key}
```

Query parameters:

`siteKey` *(string, required)*

:    Public site key.

#### Responses

`200 OK` *(application/json)*

:    Returns a JSON object containing the challenge information.

     Example:

     ``` json
     {
       "challengeType": {
         "type": "proofOfWork",
         "algorithm": "hash-sha-256",
         "difficulty": 4
       },
       "token": "2c8f4b2d-7ab6-4f3a-9c2a-2e6b3a1c9e8f"
     }
     ```

    Property descriptions:

    `challengeType.type`

    :    Challenge type — currently always `proofOfWork`.

    `challengeType.algorithm`

    :    The hash algorithm used:

         Value          | Description
         -------------- | -----------
         `hash-sha-256` | SHA-256–based hash algorithm
         `hash-sha-384` | SHA-384–based hash algorithm
         `hash-sha-512` | SHA-512–based hash algorithm

    `challengeType.difficulty`

    :     Difficulty level of the challenge.

    `token`

    :     A unique random string (nonce).

Possible errors *(application/problem+json)*:

+ `400 Bad Request`
+ `403 Forbidden`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Example

```bash
curl -sS "https://captcha.example.com/v1/challenge?siteKey=<My-Site-Key>" \
  -H "Accept: application/json"
```

### Validating a solution

Validates the client-submitted solution for a given challenge.

#### Request

```
POST https://captcha.example.com/v1/verify?siteKey={My-Site-Key}
```

Query parameters:

`siteKey` *(string, required)*

:    Public site key.

Request body (`application/json`):

Example:

``` json
{
  "callerIp": "203.0.113.42",
  "siteSecret": "<My-Site-Secret>",
  "solution": "<My-Challenge-Solution>",
  "token": "<My-Challenge-Token>"
}
```

Property descriptions:

`siteSecret` *(string, required)*

:    Website’s secret key.

`solution` *(string, required)*

:    Client’s computed challenge solution.

`token` *(string, required)*

:    Token (nonce) obtained from `/v1/challenge`.

`callerIp` *(string, optional)*

:    IP address of the requester (optional, useful for logging or rate-limiting).

#### Responses

`200 OK` *(application/json)*

:    Returns a JSON object with verification status.

Example:

```json
{
  "hostName": "www.example.com",
  "status": "success"
}
```

Property descriptions:

`hostName`

:    Hostname.

`status`

:    Verification result:

     Value              | Description
     ------------------ | -----------
     `success`          | Verification OK
     `invalid-token`    | Token is invalid
     `invalid-solution` | Solution is invalid

Possible errors (*application/problem+json*):

+ `400 Bad Request`
+ `401 Unauthorized`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Example

```bash
curl -sS "https://captcha.example.com/v1/verify?siteKey=<My-Site-Key>" \
  -H "Content-Type: application/json" \
  -d '{
        "callerIp": "203.0.113.42",
        "siteSecret": "<My-Site-Secret>",
        "solution": "<My-Challenge-Solution>",
        "token": "<My-Challenge-Token>"
      }'
```

### Resetting a challenge

Invalidates an existing token so that a new challenge can be started.

#### Request

```
POST https://captcha.example.com/v1/reset?siteKey={My-Site-Key}
```

Query parameters:

`siteKey` *(string, required)*

:    Public site key.

Request body (`application/json`):

Example:

```json
{
  "token": "<ChallengeToken>"
}
```

Property descriptions:

`token` *(string, required)*

:    Token (nonce) obtained from `/v1/challenge`.

#### Responses

`200 OK` *(application/json)*

:    No response body.

Possible errors (*application/problem+json*):

* `400 Bad Request`
* `404 Not Found`
* `500 Internal Server Error`

#### Example

```bash
curl -sS "https://captcha.example.com/v1/reset?siteKey=<My-Site-Key>" \
  -H "Content-Type: application/json" \
  -d '{ "token": "<My-Challenge-Token>" }'
```

## Health API

The Health API provides information about the operational status of the RESTCaptcha server.

#### Request

```
GET /health
```

The Health API can optionally be protected with an API key, which may be passed either as an `X-API-KEY` header or as an `Authorization` header.

`X-API-KEY`

:   Your API key in an `X-API-KEY` header.

    Example:

    ```
    X-API-KEY: <My-API-Key>
    ```

`Authorization`

:   Your API key in an `Authorization` header.

    Example:

    ```
    Authorization: ApiKey <My-API-Key>
    ```

#### Responses

`200 OK` *(application/json)*

:   Returns a JSON object with status information.

    Example:

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

Possible errors (*application/problem+json*):

+ `401 Unauthorized`
+ `404 Not Found`
+ `500 Internal Server Error`

#### Example

```bash
curl -sS https://captcha.example.com/health \
  -H "X-API-KEY: <My-API-Key>" \
  -H "Accept: application/json"
```
