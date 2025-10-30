In this chapter, we describe the configuration options available for the RESTCaptcha server.

All configuration settings are stored in the JSON file `appsettings.Production.json`, which you created during installation.

## RestCaptcha

All RESTCaptcha-specific configuration data is found under the `RestCaptcha` section.

The following snippet shows the default values for `RestCaptcha`:

``` json
{
  ...
  "RestCaptcha": {
    "ChallengeType": {
      ...
    },
    "HMACKey": "<Generated value>",
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

The properties have the following meanings:

**`ChallengeType`**

:   Configuration data for the challenge type returned when a form page requests one (see below).
The challenge type defines what the user or the form page itself must do to generate a valid solution.

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

**`HealthCheck`**

:   Configuration data for the HealthCheck endpoint (see below).

**`Sites`**

:   Configuration data for the registered websites (see below).

---

## RestCaptcha.ChallengeType

RESTCaptcha is designed to support multiple challenge types (currently, only one is implemented 😊).

**`type`**

:   The challenge type returned when a form page requests one:

    Value         | Description
    -------------- | -----------
    `proofOfWork`  | Proof of Work challenge

### ProofOfWork

The following snippet shows the default values for the `proofOfWork` challenge type:

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

---

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
