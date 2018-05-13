# Netemera LoRaWAN Network Data API V1

## Table of Contents

- [Introduction](#introduction)
- [Authorization](#authorization)
  - [Retrieve access token](#retrieve-access-token)
- [Queued Downlink Packets](#queued-downlink-packets)
  - [Delete queued downlink packets for end-device](#delete-queued-downlink-packets-for-end-device)

## Introduction

Netemera LoRaWAN Network Data API V1 is an HTTPS-based interface that enables programmatic management of queued downlink packets.

The API is available at the following base URL: https://NETWORK_SERVER_HOST/api/v1. Requests and response bodies are formatted in [JSON](https://www.json.org/) and the [OAuth 2.0](https://tools.ietf.org/html/rfc6749) protocol is used for authorization.

> **IMPORTANT:** You cannot run any of the sample requests in this guide as-is. Replace call-specific parameters such as host names, tokens, IDs, and secrets with your own values.

## Authorization

All API calls must include an OAuth 2.0 access token that authorizes a client.

To get a token, the client must [send a request](#retrieve-access-token) to Authorization Server and authenticate itself with its own client ID and secret. Successful response contains an access token valid for 24h. A new token request can be made at any time.

The client must include the received access token in each API call, either as the `Authorization` HTTP header (recommended):

`Authorization: Bearer ACCESS_TOKEN`

or as the `access_token` URL query parameter:

`access_token=ACCESS_TOKEN`

### Retrieve access token

POST https://AUTHORIZATION_SERVER_HOST/api/v1/oauth2/token?grant_type={grant_type}&audience={audience}

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`grant_type`|string|false|The grant type. Must equal "client_credentials"
`audience`|string|false|The target API's URL the access token will be generated for. Must equal "https://NETWORK_SERVER_HOST/api/v3"

#### Request headers

Header|Optional|Description
---|---|---
`Authorization: Basic {token}`|false|The `token` parameter is a base64-encoding of the following string "CLIENT_ID:CLIENT_SECRET" where client ID and secret are user name and password given to the client

#### Response body

An object with the following attributes:

Field|Type|Optional|Description
---|---|---
`access_token`|string|false|The access token
`token_type`|string|false|The type of the token
`expires_in`|string|false|The validity time of the token in seconds

#### Sample request

HTTP:

```http
POST /api/v1/oauth2/token?grant_type=client_credentials&audience=https://NETWORK_SERVER_HOST/api/v1" HTTP/1.1
Host: AUTHORIZATION_SERVER_HOST
Authorization: Basic TOKEN
```

Shell:

```shell
curl \
  --request POST \
  --url 'https://AUTHORIZATION_SERVER_HOST/api/v1/oauth2/token?grant_type=client_credentials&audience=https://NETWORK_SERVER_HOST/api/v1' \
  --user 'CLIENT_ID:CLIENT_SECRET'
```

#### Sample response

HTTP:

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "eyJz93a...k4laUWw",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

## Queued Downlink Packets

### Delete queued downlink packets

DELETE /api/v1/queued-downlink-packets/end-devices/{dev_eui}

Deletes all queued packets for the given end-device.

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`dev_eui`|string|false|The EUI-64 identifier of the end-device

#### Response

Status|Body|Description
---|---|---
`202 Accepted`||The packet has been submitted
`401 Unauthorized`||Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`||Missing permissions

#### Sample request

HTTP:

```http
DELETE /api/v1/queued-downlink-packets/end-devices/DEV_EUI HTTP/1.1
Host: NETWORK_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
```

Shell:

```shell
curl \
  --request DELETE \
  --url 'https://NETWORK_SERVER_HOST/api/v1/queued-downlink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN'
```

#### Sample response

HTTP:

```http
HTTP/1.1 204
```
