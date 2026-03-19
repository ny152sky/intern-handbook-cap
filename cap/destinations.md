
# Using Destinations in CAP (SAP BTP)

This guide explains how to configure and use **Destinations** in SAP CAP to call external services (e.g., APIs like n8n).

---

## Overview

A **Destination** in SAP BTP is a centralized configuration that stores connection details for external services.

Instead of hardcoding URLs and credentials in your code, CAP uses Destinations to:

- manage external endpoints securely
- abstract connectivity details
- integrate with SAP Cloud SDK

---

## Key Concept

When using:

```js
await cds.connect.to('some-service')
````

with a configured destination, CAP internally uses the **SAP Cloud SDK** to resolve the destination and execute HTTP requests.

---

## Architecture

```text
CAP Service (Node.js)
   ↓
cds.connect.to('service')
   ↓
SAP Cloud SDK
   ↓
Destination (BTP)
   ↓
External API (e.g., n8n)
```

---

## 1. Create a Destination (SAP BTP)

Create a destination in your BTP subaccount:

### Configuration Example

* **Name:** `n8n-api`
* **Type:** HTTP
* **Authentication:** NoAuthentication *(or as required)*
* **Proxy Type:** Internet
* **URL:** `https://<your-n8n-domain>.cloud`
* **Trust:** Use default client trust store

---

## 2. Install Required Packages

Install Cloud SDK dependencies:

```bash
npm install @sap-cloud-sdk/connectivity
npm install @sap-cloud-sdk/http-client
npm install @sap-cloud-sdk/resilience
```

These are required for CAP to communicate with destinations.

---

## 3. Configure Destination in CAP (package.json)

Add a remote service configuration:

```json
{
  "cds": {
    "requires": {
      "n8n": {
        "kind": "rest",
        "credentials": {
          "destination": "n8n-api"
        }
      }
    }
  }
}
```

### Explanation

* `"n8n"` → logical service name used in code
* `"kind": "rest"` → indicates REST API
* `"destination": "n8n-api"` → links to BTP destination

---

## 4. Use the Destination in Code (service.js)

### Connect to the remote service

```js
const cds = require("@sap/cds");

const n8n = await cds.connect.to("n8n");
```

---

### Send a request

```js
const result = await n8n.post("/webhook-test/email", {
  email: userEmail
});
```

### Explanation

* `cds.connect.to("n8n")` → resolves destination via SAP Cloud SDK
* `.post(path, data)` → sends HTTP POST request
* Path is relative to the destination URL

Example:

```text
Destination URL: https://example.n8n.cloud
Request path: /webhook-test/email

Final request:
https://example.n8n.cloud/webhook-test/email
```

---

## 5. Local Testing (Hybrid Mode)

When running locally with `cds watch --profile hybrid`, CAP can still resolve destinations via Cloud Foundry bindings.

Make sure:

* You are logged in via `cf login`
* Services are bound (`cds bind`)

---

## 6. Common Errors & Fixes

### ❌ "Invalid URL"

* Missing `https://` in destination URL

---

### ❌ "Destination not found"

* Check destination name in BTP
* Ensure correct subaccount / space

---

### ❌ Missing dependencies

Install required packages:

```bash
npm install @sap-cloud-sdk/connectivity
npm install @sap-cloud-sdk/http-client
npm install @sap-cloud-sdk/resilience
```

---

### ❌ Authentication issues

* Check destination authentication type
* Ensure required credentials are configured

---

## 7. Key Takeaways

* Destinations abstract external service configuration
* CAP uses SAP Cloud SDK under the hood
* `cds.connect.to()` is the entry point for remote services
* Avoid hardcoding URLs — always use destinations
* Paths in requests are relative to the destination URL

---

## References

* CAP Remote Services: [https://cap.cloud.sap/docs/node.js/remote-services](https://cap.cloud.sap/docs/node.js/remote-services)
* SAP Cloud SDK + CAP: [https://community.sap.com/t5/technology-blog-posts-by-sap/sap-cloud-sdk-and-its-relationship-with-sap-cloud-application-programming/ba-p/13570688](https://community.sap.com/t5/technology-blog-posts-by-sap/sap-cloud-sdk-and-its-relationship-with-sap-cloud-application-programming/ba-p/13570688)
* Destinations Guide: [https://community.sap.com/t5/technology-blog-posts-by-sap/surviving-and-thriving-with-cap-destinations-and-the-transparent-proxy-for/ba-p/13555822](https://community.sap.com/t5/technology-blog-posts-by-sap/surviving-and-thriving-with-cap-destinations-and-the-transparent-proxy-for/ba-p/13555822)