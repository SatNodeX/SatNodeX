
# SatNodeX Satellite Communication API

This document describes the SatNodeX API for interacting with its global network of satellites. These satellites operate via Software-Defined Radio (SDR) links, enabling secure transmission and reception of transactions and telemetry data. The API provides structured endpoints to manage transmissions, query satellite states, and retrieve details about processed transactions.

All requests must be made over HTTPS, using standard HTTP verbs (GET, POST, etc.) and must include proper authentication credentials. The API returns JSON-encoded responses.

## Authentication

Requests require a valid API key or access token. Include it as a Bearer token in the `Authorization` header:

```
Authorization: Bearer <your_access_token>
```

Without authentication, requests are limited to public endpoints and may be rate-limited.

## Rate Limiting

Each authenticated client has a defined rate limit. If the limit is reached, subsequent requests will return HTTP 429 until the reset time. Rate limit details are provided via `X-RateLimit-*` headers in the response.

## Error Handling

The API uses standard HTTP status codes. On error, a JSON response includes a `message` field. For example:

```json
{
  "message": "Invalid frequency parameter.",
  "documentation_url": "[(https://github.com/SatNodeX/SatNodeX/edit/main/apidocs.md)](https://github.com/SatNodeX/SatNodeX/](https://github.com/SatNodeX/SatNodeX/edit/main/apidocs.md))"
}
```

## Core Concepts

**Satellites:** SatNodeX satellites orbit at various altitudes and provide SDR-based communication channels for sending and receiving transactions. Each satellite has a unique ID, coverage zones, and telemetry data accessible via the API.

**SDR Channels:** Communication is established by tuning to specific frequencies and bandwidths. Transactions, telemetry, and acknowledgments are exchanged through these channels.

## Endpoints

### Satellite Information

**GET /satellites**

Retrieve a list of available satellites currently online and their operational parameters.

**Response:**
```json
[
  {
    "id": "GXY18",
    "name": "Galaxy 18 (Americas)",
    "orbit": "GEO",
    "coverage_area": "North & South America",
    "downlink_freq_mhz": 10750.0,
    "uplink_freq_mhz": 14250.0
  },
  {
    "id": "TS11N",
    "name": "TelStar 11N (Africa and Europe)",
    "orbit": "GEO",
    "coverage_area": "Africa & Europe",
    "downlink_freq_mhz": 10950.0,
    "uplink_freq_mhz": 14450.0
  }
]
```

### Satellite Status

**GET /satellites/{id}/status**

Retrieve real-time status, including SDR channel conditions and telemetry for a specific satellite.

**Parameters:**
- `id` (string): Satellite ID (e.g., `GXY18`).

**Response:**
```json
{
  "id": "GXY18",
  "healthy": true,
  "downlink_snr_db": 13.5,
  "uplink_power_dbm": -10.2,
  "last_telemetry_update": "2023-10-05T12:22:49Z"
}
```

### Transmitting a Transaction

**POST /transactions**

Send a transaction payload through the SDR uplink channel of a chosen satellite. The API will queue the transaction for uplink transmission at the next available window.

**Request Body:**
```json
{
  "satellite_id": "GXY18",
  "frequency_mhz": 14250.0,
  "payload_hex": "a1b2c3d4...",
  "priority": "normal"
}
```

**Response:**
```json
{
  "transaction_id": "tx_7845ABC",
  "status": "queued",
  "estimated_uplink_time": "2023-10-05T12:24:10Z"
}
```

**Notes:**
- `payload_hex` should be a hex-encoded string representing the transaction data.
- `frequency_mhz` must match a valid uplink channel supported by the satellite.

### Receiving Downlink Transactions

**GET /transactions/downlink**

Query downlink transactions recently received by a specific satellite’s SDR channel.

**Parameters:**
- `satellite_id` (string, required): ID of the satellite.
- `since` (string, optional): ISO8601 timestamp to filter transactions received after a certain time.
- `limit` (integer, optional): Number of records to return (default 50).

**Response:**
```json
[
  {
    "transaction_id": "tx_9F12BCD",
    "satellite_id": "GXY18",
    "downlink_frequency_mhz": 10750.0,
    "received_at": "2023-10-05T12:25:05Z",
    "payload_hex": "deadbeef..."
  }
]
```

### Adjusting SDR Parameters

**POST /satellites/{id}/sdr/config**

Adjust SDR parameters for uplink or downlink transmissions.

**Parameters:**
- `id` (string): Satellite ID.
- Request Body (example):
```json
{
  "downlink_freq_mhz": 10752.5,
  "uplink_freq_mhz": 14252.5,
  "bandwidth_khz": 500,
  "modulation": "QPSK"
}
```

**Response:**
```json
{
  "id": "GXY18",
  "downlink_freq_mhz": 10752.5,
  "uplink_freq_mhz": 14252.5,
  "bandwidth_khz": 500,
  "modulation": "QPSK",
  "updated_at": "2023-10-05T12:30:10Z"
}
```

### Telemetry and Diagnostics

**GET /satellites/{id}/telemetry**

Retrieve extended telemetry data for diagnostics, including power levels, signal quality, and recent error counts.

**Response:**
```json
{
  "id": "TS11N",
  "last_update": "2023-10-05T12:31:55Z",
  "signal_quality": {
    "snr_db": 14.2,
    "rss_dbm": -95.0
  },
  "channel_errors": {
    "crc_errors": 2,
    "symbol_errors": 10
  }
}
```

## Webhooks

Set up webhooks to be notified of certain events, such as new downlink transactions or changes in satellite health. Webhook payloads are POSTed in JSON format to your registered endpoint.

## Support and Documentation

For more examples, advanced configurations, and additional endpoints, please refer to the [official SatNodeX documentation]([[https://github.com/SatNodeX/SatNodeX/](https://github.com/SatNodeX/SatNodeX/)](https://github.com/SatNodeX/SatNodeX/)) or contact support if you have any questions.
```
