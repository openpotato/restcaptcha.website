In this chapter, we describe the configuration options available for the RESTCaptcha server.

All configuration settings are stored in the JSON file `appsettings.Production.json`, which you created during installation.

## RestCaptcha

All RESTCaptcha-specific configuration data is found under the `RestCaptcha` section.

The following snippet shows the default values for `RestCaptcha`:

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

The properties have the following meanings:

**`Sites`**

:   Configuration data for the registered websites (see below).

**`BehaviorMap`**

:   Defines how RESTCaptcha should respond to different risk values of a request. The higher the risk score, the stricter the measure should be. The risk score ranges from 0 (very low risk) to 100 (very high risk). Possible responses include either a captcha challenge with varying degrees of difficulty or blocking the request.

**`HMACKey`**

:   Secret key used for HMAC (Hash-based Message Authentication Code).
The generated default value (if unspecified) is *not* cryptographically secure.
Please use a secure random generator such as the [1Password Password Generator](https://1password.com/password-generator).

**`NonceMaxTTL`**

:   Maximum lifetime of the unique nonce.
Default value: `"00:30:00"` (30 minutes).

**`VerificationMinDelay`**

:   Minimum delay between receiving the challenge request and performing server-side verification.
Default value: `"00:00:02"` (2 seconds).

**`IpReputationCheck`**

:   Configuration data for performing IP reputation checks. IP reputation checks are crucial for generating a risk score.

**`HealthCheck`**

:   Configuration data for the HealthCheck endpoint (see below).

## RestCaptcha.Sites

For websites to interact with the RESTCaptcha server, they must be registered. The following configuration snippet shows a typical registration:

``` json
{
  ...
  "RestCaptcha": {
    ...
    "Sites": [
      {
        "name": "My Site",
        "description": "For testing purposes only",
        "siteKey": "My-Site-Key",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": ["www.example.com"]
      }
    ]
  }
}
```

**`name`**

:   Name of the registered website (for documentation purposes only).

**`description`**

:   Additional description of the registered website (for documentation purposes only).

**`siteKey`**

:   Key used to associate requests with the registered website.

**`siteSecret`**

:   Secret used to authorise requests for the registered website.

**`validHostNames`**

:   Optional list of valid domain names associated with the website. If specified, only requests originating from these domains will be accepted. When running behind a proxy (e.g. nginx), ensure that the original hostname is correctly forwarded.

## RestCaptcha.BehaviorMap[]

Typical content in the BehaviourMap looks like this:

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

Each entry defines a response for a specific risk score range. The properties have the following meanings:

**`minRiskScore`**

:   The minimum risk score at which the action defined under `action` should be executed.

**`maxRiskScore`**

:   The maximum risk score up to which the action defined under `action` should be executed.

**`action`**

:   The action associated with the defined risk score range:

    Value       | Description
    ------------| ------------
    `challenge` | A challenge is generated and returned.
    `block`     | The request is blocked.

**`ChallengeType`**

:   If specified as action `challenge`, the desired challenge type is defined here.

## RestCaptcha.BehaviorMap[].ChallengeType

RESTCaptcha is designed to support multiple challenge types (currently, only one is implemented 😊).

**`type`**

:   The challenge type returned when a form page requests one:

    Value         | Description
    ------------- | -----------
    `proofOfWork` | Proof of Work challenge

### ProofOfWork

The following snippet shows the default values for the `proofOfWork` challenge type:

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

The properties have the following meanings:

**`algorithm`**

:   The hash algorithm used for the proof-of-work challenge:

    Value          | Description
    -------------- | -----------
    `hash-sha-256` | SHA-256-based hash algorithm
    `hash-sha-384` | SHA-384-based hash algorithm
    `hash-sha-512` | SHA-512-based hash algorithm

**`difficulty`**

:   Difficulty level of the challenge. Default value: `4`.


## RestCaptcha.IpReputationCheck

RESTCaptcha can check the reputation of the client IP before deciding which challenge to send back. The following two reputation services can be used for this purpose:

+ [AbuseDBIP](https://www.abuseipdb.com/): A publicly accessible database that helps identify malicious IP addresses. Users and security systems can report suspicious IPs there. The platform collects these reports, evaluates the IPs using an Abuse Confidence Score, and provides an API that developers can use to automatically check whether an IP address is classified as dangerous. An API key is required to use the API free of charge.

+ [Spamhaus](https://www.spamhaus.org/): An internationally recognised organisation that collects and provides data on spam, malware distribution, botnet activity and other security threats. Its databases (e.g. SBL, XBL, PBL, DROP/EDROP) are used worldwide by email servers, firewalls and security services to detect and block malicious IP addresses, domains and botnet infrastructures. A free query is possible via DNS query.

By default, only queries via Spamhaus are enabled, as AbuseDBIP requires an individual API key. The following configuration excerpt shows the default values for `RestCaptcha.IpReputationCheck`: 

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

The properties have the following meanings:

**`AbuseIPDB.enabled`**

:   Should an IP reputation query be performed via AbuseIPDB?

**`AbuseIPDB.apiKey`**

:   API key for AbuseIPDB.

**`Spamhaus.enabled`**

:   Should an IP reputation query be performed via Spamhaus?

!!! note ‘Risk score’

    If the IP reputation query is completely deactivated or if a client IP is not listed in either of the two services, the risk score is always 0.

## RestCaptcha.HealthCheck

The RESTCaptcha server includes a HealthCheck endpoint that can optionally be protected by an API key.
This endpoint returns system metrics indicating the operational state of the RESTCaptcha server.

A typical response looks like this:

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

By default, the HealthCheck endpoint is publicly accessible, but it can be restricted or disabled at any time.
The following configuration snippet shows the default values for `RestCaptcha.HealthCheck`:

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

The properties have the following meanings:

**`Enabled`**

:   Boolean value controlling whether the endpoint is active.

    Value   | Description
    ------- | -----------
    `true`  | HealthCheck endpoint enabled
    `false` | HealthCheck endpoint disabled

**`PrivateOnly`**

:   Controls whether the endpoint is private. When enabled, only loopback addresses and networks listed in `AllowCidrs` are permitted. API keys are ignored, and public requests are denied.

    Value   | Description
    ------- | -----------
    `true`  | Private endpoint
    `false` | Public endpoint

**`AllowLocal`**

:   Determines whether requests from local loopback addresses are allowed. When running behind a proxy (e.g. nginx), ensure that forwarded headers are configured correctly so the actual client IP is passed through.

    Value   | Description
    ------- | -----------
    `true`  | Allow local loopback addresses
    `false` | Block local loopback addresses

**`AllowCidrs`**

:   A list of trusted network ranges in [CIDR notation](https://datatracker.ietf.org/doc/html/rfc4632). If `PrivateOnly = false`, requests from these ranges are allowed without an API key. If `PrivateOnly = true`, only requests from these ranges (and loopback, if enabled) are permitted.

    Typical examples for IPv4:

    CIDR             | Meaning                                | Number of IPs        | Usage
    ---------------- | -------------------------------------- | -------------------- | -----
    192.168.0.0/24   | First 24 bits are network bits         | 256 IPs (254 usable) | Typical home LAN
    10.0.0.0/8       | Network = 10.x.x.x                     | 16,777,216 IPs       | Large private networks
    172.16.0.0/12    | Network = 172.16.0.0–172.31.255.255    | 1,048,576 IPs        | Private networks
    192.168.1.128/25 | Network split into upper 128 addresses | 128 IPs              | Subnetting / load balancing
    192.168.1.0/30   | Only 4 IPs (2 usable)                  | 4 IPs                | Point-to-point links

    Typical examples for IPv6:

    CIDR                    | Meaning
    ----------------------- | -------
    2001:db8::/32           | Documentation prefix (large block)
    2001:db8:abcd::/48      | Typical ISP customer assignment
    2001:db8:abcd:1234::/64 | Standard IPv6 LAN subnet
    fe80::/10               | Link-local address range

**`Keys`**

:   A list of valid API keys that can be used to authorise access to the HealthCheck endpoint.

## RestCaptcha.Sites

For websites to interact with the RESTCaptcha server, they must be registered. The following configuration snippet shows a typical registration:

``` json
{
  ...
  "RestCaptcha": {
    ...
    "Sites": [
      {
        "name": "My Site",
        "description": "For testing purposes only",
        "siteKey": "My-Site-Key",
        "siteSecret": "AeyGWx3kQeyrDFCE5KDR",
        "validHostNames": ["www.example.com"]
      }
    ]
  }
}
```

**`name`**

:   Name of the registered website (for documentation purposes only).

**`description`**

:   Additional description of the registered website (for documentation purposes only).

**`siteKey`**

:   Key used to associate requests with the registered website.

**`siteSecret`**

:   Secret used to authorise requests for the registered website.

**`validHostNames`**

:   Optional list of valid domain names associated with the website. If specified, only requests originating from these domains will be accepted. When running behind a proxy (e.g. nginx), ensure that the original hostname is correctly forwarded.

## Serilog

The RESTCaptcha server uses the .NET library [Serilog](https://github.com/serilog/serilog) for logging.
RESTCaptcha supports the following output sinks:

* [Console output](https://github.com/serilog/serilog-sinks-console)

  Example:

  ```json
  {
    "Serilog": {
      "Using": [
        "Serilog.Sinks.Console"
      ],
      "WriteTo": [{ "Name": "Console" }]
    }
    ...
  }
  ```

* [File output](https://github.com/serilog/serilog-sinks-file)

  Example:

  ```json
  {
    "Serilog": {
      "Using": [
        "Serilog.Sinks.File"
      ],
      "WriteTo": [
        {
          "Name": "File",
          "Args": {
            "path": "/var/log/restcaptcha/log-.txt",
            "rollingInterval": "Day",
            "retainedFileCountLimit": 7,
            "fileSizeLimitBytes": 10000000,
            "rollOnFileSizeLimit": true
          }
        }
      ]
    }
    ...
  }
  ```

* [OpenTelemetry output](https://github.com/serilog/serilog-sinks-opentelemetry)
