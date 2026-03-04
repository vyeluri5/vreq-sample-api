# VReq Sample API

This repository demonstrates how to test APIs using **VReq - API Runner
for VS Code**.

It contains example request files, environment configuration, and
scripts showing how to run API requests, chain calls, and validate
responses using simple text files.

------------------------------------------------------------------------

## What is VReq?

VReq is a lightweight **file-based API testing tool for VS Code**.

Instead of GUI collections like Postman, requests are defined using
simple files that can be version controlled with your project.

### Key Features

-   Run HTTP requests from `.vreq` files
-   Environment variables with `.venv`
-   Request chaining
-   Pre/Post request scripts
-   Built-in API test assertions
-   Response viewer inside VS Code
-   Simple text-based workflow

------------------------------------------------------------------------

## Repository Structure

    vreq-sample-api
    │
    ├── env/
    │   └── dev.venv
    │
    ├── reqs/
    │   └── sample-api.vreq
    │
    ├── scripts/
    │   └── pre-script.vscript
    │
    └── README.md

  Folder    Description
  --------- ---------------------------
  env       Environment configuration
  reqs      API request definitions
  scripts   Optional request scripts

------------------------------------------------------------------------

## Prerequisites

Install the **VReq - API Runner** extension in VS Code.

Search for:

    VReq – API Runner

------------------------------------------------------------------------

## Step 1 --- Select Environment

Open the VS Code command palette.

    VReq: Select Environment

Select:

    env/dev.venv

------------------------------------------------------------------------

## Step 2 --- Run a Request

Open the request file:

    reqs/sample-api.vreq

You will see CodeLens options:

    ▶ Run Request
    ▶ Run File

Click **Run Request** to execute the API call.

------------------------------------------------------------------------

## Example Request

    Collection: Sample API

    ---

    Name: getEcho
    Request:
    GET {{host}}/get?hello=world

    Headers:
    Accept: application/json

    Tests:
    - status is 200
    - header content-type contains json

------------------------------------------------------------------------

## Example Environment

File:

    env/dev.venv

    host: https://postman-echo.com
    tenantId: dev
    timeoutMs: 30000

------------------------------------------------------------------------

## Example Request Chaining

    Collection: Workflow Example

    Chain:
    Order: createUser -> getUser

    ---

    Name: createUser
    Request:
    POST {{host}}/users

    Body:
    {
      "name": "John"
    }

    ---

    Name: getUser
    DependsOn: createUser

    Request:
    GET {{host}}/users/1

------------------------------------------------------------------------

## Tests

VReq supports simple test assertions.

Example:

    Tests:
    - status is 200
    - header content-type contains json
    - json.path id exists
    - json.path name equals John

------------------------------------------------------------------------

## Why Use VReq?

VReq allows API tests to live directly in your repository.

Benefits:

-   version controlled API tests
-   simple text-based requests
-   automation friendly
-   easy to review in pull requests
-   works well with CI/CD pipelines

------------------------------------------------------------------------

## Learn More

VReq Extension on VS Code Marketplace

GitHub: https://github.com/vyeluri5

------------------------------------------------------------------------

## License

MIT License
