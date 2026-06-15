# Laytime Calculator Pro — API & MCP Server

AI-powered **laytime, demurrage and despatch** calculation for dry-bulk and tanker
chartering. Send a Charter Party, Fixture Recap, Notice of Readiness (NOR) or
Statement of Facts (SoF) — get back a full, auditable laytime statement with the
demurrage / despatch settlement.

Built for software teams in shipping: integrate it into your own systems over a
plain **REST API**, or plug it into an AI assistant via the **Model Context
Protocol (MCP)**. Same key, same quota, same engine.

- 🌐 Product: https://navitt.com/web
- 🔑 API access & pricing: https://navitt-mcp.marine-356.workers.dev/authorize
- 📇 MCP Registry: `io.github.shipping-ship-it/laytime-calculator-pro`
- ✉️ Contact: shipping@agrix.world

---

## What it does

Laytime calculation requires holding every charter-party term in mind at once —
laycan, NOR validity, turn time, SHINC/SHEX, weather working days, exceptions,
half-rate periods, reversible vs non-reversible time. Miss one clause and the
settlement is wrong. This service reads the documents, applies the terms and
returns a day-by-day calculation you can audit, with the final demurrage or
despatch figure.

- Accepts **Charter Party, Fixture Recap, NOR, Statement of Facts, Bills of Lading**
- Input as **plain text** and/or **files** (PDF, image, Excel)
- Returns a **full, line-by-line laytime statement** plus the settlement
- Works with **any language or platform** over REST — no AI required on your side

---

## Quick start (REST)

```bash
curl -X POST https://navitt-mcp.marine-356.workers.dev/api/calculate \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "NOR tendered 01 Jun 08:00. Loading 01 Jun 12:00 to 03 Jun 18:00. Laytime allowed 3 days SHINC. Demurrage USD 10000/day. Despatch half demurrage."
  }'
```

Response:

```json
{
  "result": "… full day-by-day laytime statement and demurrage/despatch settlement …",
  "remaining": 92
}
```

`remaining` is the number of calculations left on your plan.

---

## Authentication

Every request must include your API key in the **`x-api-key`** header.
Keys are issued after sign-up on the [access & pricing page](https://navitt-mcp.marine-356.workers.dev/authorize).

```
x-api-key: YOUR_API_KEY
```

---

## REST API reference

### `POST /api/calculate`

**Base URL:** `https://navitt-mcp.marine-356.workers.dev`

**Headers**

| Header          | Required | Value              |
|-----------------|----------|--------------------|
| `x-api-key`     | yes      | Your API key       |
| `Content-Type`  | yes      | `application/json`  |

**Body** (JSON) — provide `text`, `documents`, or both:

| Field          | Type     | Required | Description                                                        |
|----------------|----------|----------|--------------------------------------------------------------------|
| `text`         | string   | no\*     | Plain-text input, e.g. a pasted fixture recap or SoF               |
| `documents`    | array    | no\*     | Document files, base64-encoded (see below)                         |
| `instructions` | string   | no       | Extra instructions, e.g. `"SHEX applies. Demurrage USD 3,500/day"` |

\* At least one of `text` or `documents` must be supplied.

**`documents[]` item**

| Field      | Type   | Description                              |
|------------|--------|------------------------------------------|
| `mimeType` | string | e.g. `application/pdf`, `image/png`      |
| `data`     | string | base64-encoded file content              |

**Response** `200 OK`

| Field       | Type    | Description                                   |
|-------------|---------|-----------------------------------------------|
| `result`    | string  | The full laytime statement and settlement     |
| `remaining` | integer | Calculations left on your plan                |

**Errors**

| Status | Meaning                                                      |
|--------|-------------------------------------------------------------|
| `401`  | Missing or invalid `x-api-key`                              |
| `400`  | Body is not valid JSON, or neither `text` nor `documents`   |
| `402`  | Calculation quota exhausted (`NO_CREDITS`) — top up         |
| `502`  | Calculation engine error — retry later                     |

Each successful request consumes one calculation from your plan.

---

## Example: sending a document (Node.js)

```javascript
import { readFileSync } from "node:fs";

const data = readFileSync("charter_party.pdf").toString("base64");

const res = await fetch("https://navitt-mcp.marine-356.workers.dev/api/calculate", {
  method: "POST",
  headers: {
    "x-api-key": process.env.LAYTIME_API_KEY,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    documents: [{ mimeType: "application/pdf", data }],
    instructions: "SHINC. Demurrage USD 12,000/day, despatch half demurrage.",
  }),
});

const { result, remaining } = await res.json();
console.log(result);
console.log("Calculations left:", remaining);
```

## Example: Python

```python
import base64, requests

with open("statement_of_facts.pdf", "rb") as f:
    data = base64.b64encode(f.read()).decode()

r = requests.post(
    "https://navitt-mcp.marine-356.workers.dev/api/calculate",
    headers={"x-api-key": "YOUR_API_KEY"},
    json={"documents": [{"mimeType": "application/pdf", "data": data}]},
)
print(r.json()["result"])
```

---

## MCP server

The same calculator is available as a **Model Context Protocol** server, so an AI
assistant can call it directly as a tool.

- **Endpoint:** `https://navitt-mcp.marine-356.workers.dev/mcp` (streamable HTTP, OAuth-protected)
- **Registry name:** `io.github.shipping-ship-it/laytime-calculator-pro`
- **Tool:** `calculate_laytime`

**Tool input**

| Field          | Type   | Description                                              |
|----------------|--------|----------------------------------------------------------|
| `text`         | string | Plain-text input (recap, SoF, etc.)                      |
| `documents`    | array  | `{ mimeType, data }` — base64-encoded files              |
| `instructions` | string | Optional extra instructions                              |

Authorize and obtain access on the
[access & pricing page](https://navitt-mcp.marine-356.workers.dev/authorize).

---

## Pricing

Pay-as-you-go: **USD 1,000 for 500 calculations** (USD 2 per calculation),
by card or by invoice to a corporate account. Corporate rates available for teams.
See the [access & pricing page](https://navitt-mcp.marine-356.workers.dev/authorize)
or contact **shipping@agrix.world**.

---

## About

Built by **AGRIX / Navitt** — modern tools for shipping. BIMCO member 181628.

- https://agrix.world
- https://navitt.com
