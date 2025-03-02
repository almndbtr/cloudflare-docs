---
order:
pcx-content-type: reference
---

# Using JSON

Cloudflare's DNS over HTTPS endpoint also supports JSON format for querying DNS data. For lack of an agreed upon JSON schema for DNS over HTTPS in the Internet Engineering Task Force (IETF), Cloudflare has chosen to follow the same schema as Google's DNS over HTTPS resolver.

JSON formatted queries are sent using a `GET` request. When making requests using `GET`, the DNS query is encoded into the URL. The client should include an HTTP `Accept` request header field with a MIME type of `application/dns-json` to indicate that the client is able to accept a JSON response from the DNS over HTTPS resolver.

Supported parameters:

<TableWrap>

| Field | Required? | Description | Example |
| ---|---|---|--- |
| `name` | Yes | Query Name. | `example.com` |
| `type` | Yes | Query type (either a numeric value or text). | `AAAA` |
| `do` | No | DO bit - set if client wants DNSSEC data (either boolean or numeric value).| `true` |
| `cd` | No | CD bit - set to disable validation (either boolean or numeric value). | `false` |

</TableWrap>

Example request:

```sh
$ curl -H 'accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=AAAA'
```

Example response:

```json
{
  "Status": 0,
  "TC": false,
  "RD": true,
  "RA": true,
  "AD": true,
  "CD": false,
  "Question": [
    {
      "name": "example.com.",
      "type": 28
    }
  ],
  "Answer": [
    {
      "name": "example.com.",
      "type": 28,
      "TTL": 1726,
      "data": "2606:2800:220:1:248:1893:25c8:1946"
    }
  ]
}
```

The following table has more information on each response field:

<TableWrap>

| Field | Description |
| --- |--- |
| `Status` | The Response Code of the DNS Query. These are defined by [IANA](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-6). |
| `TC` | If true, it means the truncated bit was set. This happens when the DNS answer is larger than a single UDP or TCP packet. `TC` will almost always be false with Cloudflare DNS over HTTPS because Cloudflare supports the maximum response size. |
| `RD` | If true, it means the Recursive Desired bit was set. This is always set to true for Cloudflare DNS over HTTPS. |
| `RA` | If true, it means the Recursion Available bit was set. This is always set to true for Cloudflare DNS over HTTPS. |
| `AD` | If true, it means that every record in the answer was verified with DNSSEC. |
| `CD` | If true, the client asked to disable DNSSEC validation. In this case, Cloudflare will still fetch the DNSSEC-related records, but it will not attempt to validate the records. |
| `Question: name` | The record name requested. |
| `Question: type` | The type of DNS record requested. These are defined by [IANA](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-4). |
| `Answer: name` | The record owner. |
| `Answer: type` | The type of DNS record. These are defined by [IANA](https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-4). |
| `Answer: TTL` | The number of seconds the answer can be stored in cache before it is considered stale. |
| `Answer: data` | The value of the DNS record for the given name and type. The data will be in text for standardized record types and in hex for unknown types. |

</TableWrap>