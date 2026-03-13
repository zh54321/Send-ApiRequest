# Send-ApiRequest - PowerShell Module

## Introduction

The `Send-ApiRequest` PowerShell module allows you to send single requests to any HTTP API using `Invoke-RestMethod` as the transport layer.

Key features:

- Generic API wrapper for GET, POST, PATCH, PUT, and DELETE requests
- Automatic pagination support for APIs returning `@odata.nextLink`, `nextLink`, or `odata.nextLink`
- Retry logic with exponential backoff for transient errors (for example `429`, `500`, `502`, `503`, `504`)
- Structured error parsing for cleaner troubleshooting
- Custom headers and query parameters
- Optional raw JSON output
- Simple HTTP proxy support for debugging
- Optional certificate validation bypass with `-SkipCertificateCheck`
- Verbose logging option
- User-Agent customization

Note:

- This module is API-agnostic and can be used with Microsoft Graph, Azure REST APIs, internal web services, or any other REST endpoint.
- Cleartext access tokens can be obtained, for example, using [EntraTokenAid](https://github.com/zh54321/EntraTokenAid).

## Parameters

| Parameter | Description |
| ------------------------------ | ------------------------------------------------------------------------------------------- |
| `-Method` *(Mandatory)* | HTTP method to use (`GET`, `POST`, `PATCH`, `PUT`, `DELETE`) |
| `-Uri` *(Mandatory)* | Absolute or relative request URI |
| `-AccessToken` | OAuth bearer token. If specified, the module adds `Authorization: Bearer <token>` |
| `-Body` | Request body as string, hashtable, or object. Non-string values are converted to JSON |
| `-MaxRetries` *(Default: 6)* | Maximum retry attempts for transient API failures |
| `-UserAgent` | Custom User-Agent header |
| `-Proxy` | Use a proxy (for example `http://127.0.0.1:8080`) |
| `-SkipCertificateCheck` | Skips TLS certificate validation if supported by the current PowerShell runtime |
| `-DisablePagination` | Prevents the function from automatically following pagination links |
| `-VerboseMode` | Enables request and pagination logging |
| `-Silent` | Suppresses informational host output for retries and request failures |
| `-RawJson` | Returns the response as a raw JSON string instead of a PowerShell object |
| `-QueryParameters` | Hashtable of query parameters appended to the initial URI |
| `-AdditionalHeaders` | Additional HTTP headers merged into the request |
| `-JsonDepthRequest` *(Default: 10)* | JSON conversion depth for request body serialization |
| `-JsonDepthResponse` *(Default: 10)* | JSON conversion depth for response serialization when using `-RawJson` |

## Examples

### Example 1: **Call Microsoft Graph**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"

$Response = Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/users' -AccessToken $AccessToken

# Show data
$Response
```

### Example 2: **Call Azure Resource Manager**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"

$Response = Send-ApiRequest -Method GET -Uri 'https://management.azure.com/subscriptions?api-version=2022-12-01' -AccessToken $AccessToken

# Show data
$Response
```

### Example 3: **Use query parameters**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"
$QueryParameters = @{
    '$top'    = 50
    '$select' = 'id,displayName'
}

Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/users' -AccessToken $AccessToken -QueryParameters $QueryParameters
```

### Example 4: **Create a new object with a JSON body**

```powershell
$Body = @{
    name    = 'item01'
    enabled = $true
}

$Response = Send-ApiRequest -Method POST -Uri 'https://api.contoso.local/v1/items' -Body $Body

# Show response
$Response
```

### Example 5: **Use additional headers**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"
$Headers = @{
    'ConsistencyLevel' = 'eventual'
}

Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/users' -AccessToken $AccessToken -AdditionalHeaders $Headers
```

### Example 6: **Use a proxy and disable certificate validation**

```powershell
Send-ApiRequest -Method GET -Uri 'https://api.contoso.local/v1/status' -Proxy 'http://127.0.0.1:8080' -SkipCertificateCheck -VerboseMode
```

### Example 7: **Return raw JSON**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"

$Json = Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/me' -AccessToken $AccessToken -RawJson

$Json
```

### Example 8: **Get only the first page**

```powershell
$AccessToken = "YOUR_ACCESS_TOKEN"
$QueryParameters = @{
    '$top' = 1
}

Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/users' -AccessToken $AccessToken -QueryParameters $QueryParameters -DisablePagination
```

### Example 9: **Catch errors**

```powershell
try {
    Send-ApiRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/doesnotexist' -AccessToken 'YOUR_ACCESS_TOKEN' -ErrorAction Stop
}
catch {
    $err = $_

    Write-Host "[!] API error occurred:"
    Write-Host " Message              : $($err.Exception.Message)"
    Write-Host " FullyQualifiedErrorId: $($err.FullyQualifiedErrorId)"
    Write-Host " TargetURL            : $($err.TargetObject)"
    Write-Host " Category             : $($err.CategoryInfo.Category)"
    Write-Host " Script Line          : $($err.InvocationInfo.Line)"
}
```
