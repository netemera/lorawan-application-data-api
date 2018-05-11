# Netemera LoRaWAN Application Data API V3

## Table of Contents

- [Introduction](#introduction)
- [Authorization](#authorization)
  - [Retrieve access token](#retrieve-access-token)
- [Uplink Packets](#uplink-packets)
  - [Uplink Packet object](#uplink-packet-object)
    - [Gateway Information object](#gateway-information-object)
  - [Receive live uplink packets from end-device](#receive-live-uplink-packets-from-end-device)
  - [Receive live uplink packets from application](#receive-live-uplink-packets-from-application)
  - [Retrieve historical uplink packets by end-device and time range](#retrieve-historical-uplink-packets-by-end-device-and-time-range)
  - [Retrieve historical uplink packets by application and time range](#retrieve-historical-uplink-packets-by-application-and-time-range)
- [Downlink Packets](#downlink-packets)
  - [Downlink packet object](#downlink-packet-object)
  - [Send downlink packets to end-device](#send-downlink-packets-to-end-device)

## Introduction

Netemera LoRaWAN Application Data API V3 is an HTTPS-based interface that enables programmatic communication with commissioned end-devices organized into applications.

A client can subscribe to receive live uplink packets via SSE ([Server Sent Events](https://www.w3.org/TR/eventsource/)) and send downlink packets via a [RESTful](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) endpoint. Additional RESTful endpoints allow for retrieval of historical communications.

The API is available at the following base URL: https://APPLICATION_SERVER_HOST/api/v3. Requests and response bodies are formatted in [JSON](https://www.json.org/) and the [OAuth 2.0](https://tools.ietf.org/html/rfc6749) protocol is used for authorization.

> **Important:** You cannot run any of the sample requests in this guide as-is. Replace call-specific parameters such as host names, tokens and IDs with your own values.

## Authorization

All API calls must include an OAuth 2.0 access token that authorizes a client.

To get a token, the client must [send a request](#retrieve-access-token) to Authorization Server and authenticate itself with a client ID and secret. Successful response contains an access token valid for 24h. A new request can be made at any time.

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
`audience`|string|false|The target API's URL the access token should be generated for. Must equal "https://APPLICATION_SERVER_HOST/api/v3"

#### Request headers

Header|Optional|Description
---|---|---
`Authorization: Basic {token}`|false|The `token` parameter is a base64-encoding of the following string "CLIENT_ID:CLIENT_SECRET" where client ID and secret are user name and password assigned to the client

#### Response body

An object with the following attributes:

Field|Type|Description
---|---|---
`access_token`|string|The access token
`token_type`|string|The type of the token
`expires_in`|string|The validity time of the token in seconds

#### Sample request

HTTP:

```http
POST /api/v1/oauth2/token?grant_type=client_credentials&audience=https://APPLICATION_SERVER_HOST/api/v3" HTTP/1.1
Host: AUTHORIZATION_SERVER_HOST
Authorization: Basic TOKEN
```

Shell:

```shell
curl \
  --request POST \
  --url 'https://AUTHORIZATION_SERVER_HOST/api/v1/oauth2/token?grant_type=client_credentials&audience=https://APPLICATION_SERVER_HOST/api/v3' \
  --user 'CLIENT_ID:CLIENT_SECRET'
```

#### Sample response

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

## Uplink Packets

### Uplink Packet object

An Uplink Packet object has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`dev_eui`|string|false|The EUI-64 identifier of the end-device in hex
`recv_time`|string|false|The timestamp of packet reception in ISO 8601 format (UTC)
`f_port`|integer|true|The port in the range from 1 to 223
`f_cnt_up`|integer|false|The uplink frame counter
`ack`|boolean|false|The acknowledgment of the last downlink packet
`adr`|boolean|false|The ADR-enabled flag
`data_rate`|integer|false|The data rate
`ul_freq`|number|false|The frequency in Mhz
`gw_info`|[[Gateway Information](#gateway-information-object)]|true|An optional array of Gateway Information objects (in descending order by RSSI)
`frm_payload`|string|true|An optional payload in hex

#### Gateway Information object

A Gateway Information object has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`id`|string|false|The EUI-64 identifier of the gateway
`rssi`|number|false|The RSSI
`snr`|number|false|The SNR

### Receive live uplink packets from end-device

GET /api/v3/uplink-packets/end-devices/{dev_eui}

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`dev_eui`|string|false|The EUI-64 identifier of the end-device in hex

#### Request headers

Header|Optional
---|---
`Accept: text/event-stream`|false
`Last-Event-ID: {id}`|true
`Cache-Control: no-cache`|false

#### Response

Status|Body|Description
---|---|---
`200 OK`|A SSE stream of [Uplink Packet objects](#uplink-packet-object) with intermittent empty keep-alive messages|
`401 Unauthorized`|Empty|Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`|Empty|Missing permissions

#### Sample request

HTTP:

```http
GET /api/v3/uplink-packets/end-devices/DEV_EUI HTTP/1.1
Host: APPLICATION_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
Accept: text/event-stream
Cache-Control: no-cache
```

Shell:

```shell
curl \
  --request GET \
  --url 'https://APPLICATION_SERVER_HOST/api/v3/uplink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: text/event-stream' \
  --header 'Cache-Control: no-cache'
```

#### Sample response

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream
Last-Event-ID: AAABWBywFbU=

data:{
  "dev_eui": "DEV_EUI",
  "recv_time": "2016-10-31T21:41:51.765598114Z",
  "f_port": 1,
  "f_cnt_up": 1,
  "ack": false,
  "adr": false,
  "data_rate": 1,
  "ul_freq": 868.3,
  "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
  }],
  "frm_payload":"05F0"
}
id:AAABWByxABU=

data:

data:

data:{
  "dev_eui": "DEV_EUI",
  "recv_time": "2016-10-31T21:41:53.765598114Z",
  "f_port": 11,
  "f_cnt_up": 123,
  "ack": false,
  "adr": false,
  "data_rate": 0,
  "ul_freq": 868.2,
  "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -80.0,
      "snr": -11.5
  }],
  "frm_payload":"AAF0"
}
id:AAABWByxB+U=
```

### Receive live uplink packets from application

GET /api/v3/uplink-packets/applications/{app_id}

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`app_id`|string|false|The identifier of an application

#### Request headers

Header|Optional
---|---
`Accept: text/event-stream`|false
`Last-Event-ID: {id}`|true
`Cache-Control: no-cache`|false

#### Response

Status|Body|Description
---|---|---
`200 OK`|A SSE stream of [Uplink Packet objects](#uplink-packet-object) with intermittent empty keep-alive messages|
`401 Unauthorized`|Empty|Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`|Empty|Missing permissions

#### Sample request

HTTP:

```http
GET /api/v3/uplink-packets/applications/APP_ID HTTP/1.1
Host: APPLICATION_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
Accept: text/event-stream
Cache-Control: no-cache
```

Shell:

```shell
curl \
  --request GET \
  --url 'https://APPLICATION_SERVER_HOST/api/v3/uplink-packets/applications/APP_ID' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: text/event-stream' \
  --header 'Cache-Control: no-cache'
```

#### Sample response

```http
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/event-stream
Last-Event-ID: AAABWBywFbU=

data:{
  "dev_eui": "70b3d5ca40000031",
  "recv_time": "2016-10-31T21:41:51.765598114Z",
  "f_port": 1,
  "f_cnt_up": 1,
  "ack": false,
  "adr": false,
  "data_rate": 0,
  "ul_freq": 868.3,
  "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
  }],
  "frm_payload":"200A"
}
id:AAABWByxABU=

data:

data:

data:{
  "dev_eui": "70b3d5ca40000032",
  "recv_time": "2016-10-31T21:41:53.765598114Z",
  "f_port": 11,
  "f_cnt_up": 123,
  "ack": false,
  "adr": false,
  "data_rate": 1,
  "ul_freq": 868.2,
  "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -80.0,
      "snr": -11.5
  }],
  "frm_payload":"001F"
}
id:AAABWByxB+U=
```

### Retrieve historical uplink packets by end-device and time range

GET /api/v3/uplink-packets/end-devices/{dev_eui}?from_time={from_time}&until_time={until_time}

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`dev_eui`|string|false|The EUI-64 identifier of the end-device in hex
`from_time`|string|false|The beginning of the timestamp range (inclusive) in ISO 8601 format (UTC)
`until_time`|string|true|The end of the timestamp range (exclusive) in ISO 8601 format (UTC). Defaults to current timestamp

#### Request headers

Header|Optional
---|---
`Accept: application/json`|true

#### Response

Status|Body|Description
---|---|---
`200 OK`|An array of [Uplink Packet](#uplink-packet) objects|
`401 Unauthorized`|Empty|Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`|Empty|Missing permissions

#### Sample request

HTTP:

```http
GET /api/v3/uplink-packets/end-devices/DEV_EUI?from_time=2016-10-31T21:41:51.765598114Z&until_time=2016-10-31T22:32:09.000000000Z HTTP/1.1
Host: APPLICATION_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
Accept: application/json
```

Shell:

```shell
curl
  --request GET \
  --url 'https://APPLICATION_SERVER_HOST/api/v3/uplink-packets/end-devices/DEV_EUI?from_time=2016-10-31T21:41:51.765598114Z&until_time=2016-10-31T22:32:09.000000000Z' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample response

```http
HTTP/1.1 200 OK
Content-Type: application/json

[{
   "dev_eui": "DEV_EUI",
   "recv_time": "2016-10-31T21:41:51.765598114Z",
   "f_port": 1,
   "f_cnt_up": 1,
   "ack": false,
   "adr": false,
   "data_rate": 0,
   "ul_freq": 868.3,
   "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
   }],
  "frm_payload":"0000"
 }]
```

### Retrieve historical uplink packets by application and time range

GET /api/v3/uplink-packets/applications/{app_id}?from_time={from_time}&until_time={until_time}

#### Request parameters

Parameter|Type|Optional|Description
---|---|---|---
`app_id`|string|false|The identifier of the application
`from_time`|string|false|The beginning of the timestamp range (inclusive) in ISO 8601 format (UTC)
`until_time`|string|true|The end of the timestamp range (exclusive) in ISO 8601 format (UTC). Defaults to current timestamp

#### Request headers

Header|Optional
---|---
`Accept: application/json`|true

#### Response

Status|Body|Description
---|---|---
`200 OK`|An array of [Uplink Packet](#uplink-packet) objects|
`401 Unauthorized`|Empty|Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`|Empty|Missing permissions

#### Sample request

HTTP:

```http
GET /api/v3/uplink-packets/applications/APP_ID?from_time=2016-10-31T21:41:51.765598114Z&until_time=2016-10-31T22:32:09.000000000Z HTTP/1.1
Host: APPLICATION_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
Accept: application/json
```

Shell:

```shell
curl
  --request GET \
  --url 'https://APPLICATION_SERVER_HOST/api/v3/uplink-packets/applications/APP_ID?from_time=2016-10-31T21:41:51.765598114Z&until_time=2016-10-31T22:32:09.000000000Z' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Accept: application/json'
```

#### Sample response

```http
HTTP/1.1 200 OK
Content-Type: application/json

[{
   "dev_eui": "7276ff002e046600",
   "recv_time": "2016-10-31T21:41:51.765598114Z",
   "f_port": 1,
   "f_cnt_up": 1,
   "ack": false,
   "adr": false,
   "data_rate": 0,
   "ul_freq": 868.3,
   "gw_info": [{
      "id": "7276ff002e0400e8",
      "rssi": -116.0,
      "snr": -15.2
   }],
  "frm_payload":"0000"
 }]
```

## Downlink Packets

### Downlink Packet object

A Downlink Packet object has the following attributes:

Attribute|Type|Optional|Description
---|---|---|---
`dev_eui`|string|false|The EUI-64 identifier of the end-device in hex
`f_port`|integer|true|The port in range from 1 to 223
`confirmed`|boolean|false|Require confirmation from the end-device
`frm_payload`|string|true|The payload in hex

### Send downlink packets to end-device

POST /api/v3/downlink-packets/end-devices/{dev_eui}

#### Request headers

Header|Optional
---|---
`Accept: application/json`|false

#### Request body

A [Downlink Packet](#downlink-packet-object) object.

#### Response

Status|Body|Description
---|---|---
`202 Accepted`|Empty|The packet has been submitted
`401 Unauthorized`|Empty|Missing or invalid access token. Please [retrieve a new access token](#retrieve-access-token)
`405 Forbidden`|Empty|Missing permissions

#### Sample request

HTTP:

```http
POST /api/v3/downlink-packets/end-devices/DEV_EUI HTTP/1.1
Host: APPLICATION_SERVER_HOST
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "dev_eui": "DEV_EUI",
  "f_port": 2,
  "confirmed": false,
  "frm_payload": "0000"
}
```

Shell:

```shell
curl \
  --request POST \
  --url 'https://APPLICATION_SERVER_HOST/api/v3/downlink-packets/end-devices/DEV_EUI' \
  --header 'Authorization: Bearer ACCESS_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{"DEV_EUI","f_port":2,"confirmed":false,"frm_payload":"0000"}'
```

#### Sample response

```
HTTP/1.1 202 Accepted
```
