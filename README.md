# VReq Sample API

<a href="https://peerpush.com/p/vreq-api-runner"
  target="_blank"
  rel="noopener">
  <img
    src="https://peerpush.com/p/vreq-api-runner/badge.png"
    alt="VReq API Runner on PeerPush"
    style="width: 230px;"
  />
</a>

This repository demonstrates the current **VReq - API Runner for VS Code**
workflow using request files, environments, scripts, and scenarios.

The examples are intentionally file-based so API checks can live with the
application code, be reviewed in pull requests, and be reused across local,
test, and CI runs.

## What is VReq?

VReq is a lightweight API runner for VS Code. Requests are written as plain
text `.vreq` files instead of GUI collections.

Use it for:

- Running HTTP requests from `.vreq` files
- Switching environments with `.venv` files
- Injecting variables with `{{name}}` syntax
- Defining request-level variables and environment defaults
- Running pre-request scripts and reusable scenario scripts
- Chaining requests with `DependsOn` or scenario flows
- Validating responses with built-in assertions
- Persisting scenario outputs as JSON artifacts

## Repository Structure

```text
vreq-sample-api/
├── env/
│   ├── dev.venv
│   └── test.venv
├── integrations/
│   ├── Jira.vreq
│   └── Salesforce.vreq
├── reqs/
│   ├── chain-demo.vreq
│   ├── echo.vreq
│   ├── ForwardPrices.vreq
│   ├── HistoricalPrices.vreq
│   └── test.vreq
├── scenarios/
│   ├── price-validation.vscenario
│   └── artifacts/
├── scripts/
│   ├── CompareSalesforce.vscript
│   ├── FormulaeCalculation.vscript
│   ├── pre-script.vscript
│   └── set-vars.vscript
└── README.md
```

| Folder | Purpose |
| --- | --- |
| `env/` | Environment files used by requests and scenarios |
| `reqs/` | Core API request definitions |
| `integrations/` | Integration request definitions used by scenarios |
| `scripts/` | Pre-request scripts and scenario calculation scripts |
| `scenarios/` | Multi-step `.vscenario` workflows |
| `scenarios/artifacts/` | Persisted scenario outputs |

## Prerequisites

Install the **[VReq - API Runner](https://marketplace.visualstudio.com/items?itemName=VenkatY.vreq)** extension in VS Code.

Open the command palette and search for:

```text
VReq: Select Environment
```

## Environment Usage

Environment files use the `.venv` extension and contain shared values for
requests and scenarios.

Example from `env/dev.venv`:

```text
host: https://postman-echo.com
quoteStartDate: 2026-01-01
quoteEndDate: 2027-12-31
debug: true

headers.Accept: application/json
headers.User-Agent: vreq/1.0

timeoutMs: 30000
tenantId: dev
```

Select an environment before running requests:

```text
VReq: Select Environment
```

Then choose one of:

```text
env/dev.venv
env/test.venv
```

Variables are referenced with double braces:

```text
GET {{host}}/get
X-Tenant: {{tenantId}}
```

### Variable Sources

This sample uses variables from several places:

- Environment variables in `env/*.venv`
- Request-level `Vars:` blocks inside `.vreq` files
- Scenario `Inputs:` blocks inside `.vscenario` files
- Scenario step `Vars:` overrides
- Script context values set with `ctx.set(...)`

For example, `reqs/ForwardPrices.vreq` defines a default `Code`:

```text
Vars:
    Code: 1234
```

The scenario overrides it for one flow step:

```text
- Run: ForwardPrices
  Vars:
    Code: {{code}}
```

## Running Individual Requests

Open a `.vreq` file in VS Code and use the CodeLens actions shown above each
request:

```text
Run Request
Run File
```

Good starting points:

- `reqs/echo.vreq` for a basic GET request
- `reqs/chain-demo.vreq` for request chaining and pre-request scripts
- `reqs/HistoricalPrices.vreq` and `reqs/ForwardPrices.vreq` for pricing calls
- `integrations/Salesforce.vreq` and `integrations/Jira.vreq` for integration calls

## Request File Format

A `.vreq` file starts with collection metadata and one or more request blocks.

```text
Collection: Pricing

---
Name: ForwardPrices

Vars:
    Code: 1234

Request:
GET {{host}}/get

Query:
type: forward
code: {{Code}}
startDate: 2026-06-25
endDate: {{quoteEndDate}}

Headers:
Accept: application/json
X-Tenant: {{tenantId}}

Tests:
- status is 200
- header content-type contains json
```

## Scripts

VReq scripts use JavaScript and can read or update request/scenario context.

Pre-request scripts can mutate the outgoing request. For example,
`scripts/pre-script.vscript` adds a trace header when one is not already set:

```javascript
if (!req.headers["X-Trace-Id"]) {
  req.headers["X-Trace-Id"] = "trace-" + (crypto?.randomUUID?.() ?? Date.now());
}

ctx.set("lastTraceId", req.headers["X-Trace-Id"]);
```

Scenario scripts export a `run(inputs, ctx)` function and return structured
data for later steps:

```javascript
export default function run(inputs, ctx) {
  const calculated = Number(inputs.calculated.finalValue);
  const salesforceValue = Number(inputs.salesforce.args.expectedValue);

  return {
    match: calculated === salesforceValue,
    finalValue: calculated,
    salesforceValue
  };
}
```

## Running the Price Validation Scenario

The main end-to-end example is:

```text
scenarios/price-validation.vscenario
```

It uses the `dev` environment:

```text
Name: Price Validation Scenario
Env: dev
```

The scenario registers request definitions from both `reqs/` and
`integrations/`:

```text
Use Requests:
- Collection: Pricing
  Request: HistoricalPrices

- Collection: Pricing
  Request: ForwardPrices

- Collection: Integrations
  Request: GetSalesforceData

- Collection: Integrations
  Request: CreateJiraIssue
```

It then:

1. Runs historical price lookup
2. Runs forward price lookup
3. Re-runs forward prices with a scenario variable override
4. Calculates a margin difference in `FormulaeCalculation.vscript`
5. Retrieves Salesforce data
6. Compares calculated data with Salesforce data
7. Creates a Jira issue only when the comparison fails
8. Asserts that `comparison.match` is `true`

Open the scenario file in VS Code and run it with the VReq scenario CodeLens
action.

## Scenario Outputs

The price validation scenario persists intermediate results under
`scenarios/artifacts/`:

```text
scenarios/artifacts/historical.json
scenarios/artifacts/forward.json
scenarios/artifacts/calcResult.json
scenarios/artifacts/comparison.json
scenarios/artifacts/jiraResult.json
```

Scenario steps control what is stored:

```text
- Run: HistoricalPrices
  SaveAs: historical
  Persist: artifacts/historical.json
  PersistAs: json
```

Use `SaveAs` to name data for later steps, and `Persist` when you also want a
file artifact for inspection or CI output.

## Assertions

Request assertions live in `.vreq` files:

```text
Tests:
- status is 200
- header content-type contains json
- json.path args.hello exists
```

Scenario assertions live at the end of `.vscenario` files:

```text
Assertions:
- expr comparison.match equals true
```

## Notes

- `env/dev.venv` uses `https://postman-echo.com`, so most examples run without
  authentication.
- `env/test.venv` points at `https://jsonplaceholder.typicode.com` for simple
  test data experiments.
- `reqs/test.vreq` targets `http://localhost:8000`, so it requires a local API
  server before running.
- Files are plain text and are safe to version with the project.

## Learn More

- [VReq Extension on the VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=VenkatY.vreq)
- GitHub: https://github.com/vyeluri5

## License

MIT License
