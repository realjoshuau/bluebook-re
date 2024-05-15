# Authentication

The authentication is pretty simple.
While the auth on `https://prod.idp.collegeboard.org/` uses Okta, it's believed that Bluebook authentication uses either Amazon Cognito or some internal software, and are interfaced (possibly LDAP?)

> [!NOTE]
> Bluebook bypasses **all** multi-factor authentication, not even notifying the user that they were logged into on Bluebook.
> CollegeBoard knows this (why the tokens are labeled with `CBLoginWeak` everywhere), and Authentication tokens tied to Bluebook are unable to be used elsewhere.

## Authentication

### User Name & Password

Entering your username and password submits a POST request to:

```
POST https://agw.collegeboard.org/api/authn
Accept: application/json, text/plain, _/_
Content-Type: application/json
Origin: https://bluebook-mac.app.collegeboard.org
Content-Length: [VARIABLE]
accept-language: en-US,en;q=0.9
x-api-key: [PULL FROM BLUEBOOK MAC SOURCE!]
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko)
Referer: https://bluebook-mac.app.collegeboard.org/
Accept-Encoding: gzip, deflate, br
```

and a body of

```json
{
  "fingerprint": "<>", // Device Fingerprint
  "password": "<>",
  "username": "<>",
  "wait": false // Unsure what this does. Most likely waits for a result instead of polling /status.
}
```

Collegeboard returns:

> [!NOTE]
> This request has a X-Request-Id attached to it, and is locked down.

```json
{
  "anticipatedProcessingTime": <number-between-500-600>,
  "averageWaitMs": <number-between-500-600>,
  "sessionToken": "<SESSION_TOKEN>" // It's called a Session Token, but it's unused for the rest of the session. Maybe session as in "Authentication Session" or a refresh token?
}
```

averageWaitMs / anticipatedProcessingTime are most likely the time clients should wait in-between polls.

### Polling /api/status for RMT Token

After this, you need to poll `https://agw.collegeboard.org/api/status?sessionToken=<SESSION_TOKEN>`

```
GET https://agw.collegeboard.org/api/status?sessionToken=<SESSION_TOKEN>
Accept: application/json, text/plain, _/_
Origin: https://bluebook-mac.app.collegeboard.org
Accept-Encoding: gzip, deflate, br
x-api-key: <PULL_FROM_MACOS_BLUEBOOK>
Content-Type: application/json
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko)
accept-language: en-US,en;q=0.9
```

On success (failure has not been logged)

```json
{
  "code": "SUCCESS",
  "firstName": "<First Name>",
  "lastName": "<Last name>>",
  "message": "Successfully authenticated",
  "personId": "<See comment>", // This is the same value as "College Board ID" on https://my.collegeboard.org/profile/information
  "preferredName": null,
  "rmt": "<YOU NEED TO USE THIS LATER!>",
  "status": "authenticated"
}
```

### Request a JWT

Using the RMT value from the previous request, send a request to

```
GET https://catapult-api-prod.collegeboard.org/rel/temp-user-aws-creds?cbEnv=<removed>&cbAWSDomains=none&cacheNonce=<current UNIX timestamp>
accept: application/json, text/plain, */*
origin: https://bluebook-mac.app.collegeboard.org
referer: https://bluebook-mac.app.collegeboard.org/
accept-language: en-US,en;q=0.9
user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko)
authorization: CBLoginWeak <RMT VALUE>
accept-encoding: gzip, deflate, br
```

No clue what cbEnv is, nor what cbAWSDomains means.
cacheNonce I assume is a simple [cache busting](https://stackoverflow.com/questions/9692665/cache-busting-via-params) technique.

If the RMT is accepted, you'll get this (redacted):

```json
{
  "catapult": {
    "AssumedRoleUser": {},
    "Credentials": {
      "Expiration": 0 // Unix timestamp
    },
    "SubjectFromWebIdentityToken": "<See cbJwtToken>"
  },
  "cbJwtToken": "", // RS256, JWT. Issued by catapult.collegeboard.org and "sub" is same as "SubjectFromWebIdentityToken" and "catapultId". The JWT also contains some other information under a "cb" key.
  "cbUserProfile": {
    "authStatus": "Success",
    "basicProfile": null,
    "context": {
      "asmtEventId": "",
      "firstName": "<NAME>",
      "lastName": "<NAME>",
      "loginType": "cblogin",
      "preferredName": null
    },
    "sessionInfo": {
      "cbEnv": "<same as request>",
      "expireInSeconds": 0, // 6 hours
      "identityKey": {
        "catapultId": "<See cbJwtToken>",
        "namespace": "<Unknown two-character value>",
        "personId": "<Same as in /api/status>",
        "wans": "<Unknown two-character value>"
      },
      "unifiedSessionId": "<UNUSED>"
    }
  }
}
```

<details>
<summary>Speculation!</summary>

If they're doing what I think they're doing, and based on [`AssumedRoleUser`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumedRoleUser.html) and the URL being `/rel/temp-user-aws-creds`, which is creating a temporary AWS IAM User for every login, I need to know why!

I believe they are using the AWS STS API
https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html, which is supported by mentions of the STS endpoint in the [DTA Content URI CSP](dta-content-uri.md)

</details>
