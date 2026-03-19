# Implementing Actions in CAP (SAP Cloud Application Programming Model)

This guide explains how to define and implement a custom action in CAP and how it is triggered from the frontend (SAP UI5).

---

## Overview

An action in CAP allows the frontend to trigger custom backend logic (e.g., registration, evaluation, processing workflows).

The flow consists of three main parts:

1. Action definition in `service.cds`
2. Action handler in `service.js`
3. Action call from the frontend (UI5)

---

## 1. Define the Action (service.cds)

Actions are defined in the service definition file.

```cds
action register(data : RegisterInput) returns RegisterResult;
````

### Explanation

* `register` → name of the action
* `data` → input parameter (custom type)
* `RegisterInput` → structure of incoming data
* `RegisterResult` → structure of response

This defines the API contract exposed via OData.

---

## 2. Implement the Action Handler (service.js)

Actions are implemented in the service handler using `this.on`.

```js
this.on("register", async (req) => {
```

### 2.1 Accessing Request Data

```js
const { firstName, lastName, email, password } = req.data?.data || {};
```

### Explanation

* `req` → request object (contains everything sent from frontend)
* `req.data` → payload sent by frontend
* `req.data?.data` → safely access nested data (avoids errors if undefined)
* `|| {}` → fallback to empty object to prevent destructuring errors

---

### 2.2 Querying the Database (CQN)

Example query:

```js
SELECT
  .one
  .from(Users)
  .where({ email });
```

### Explanation

This uses the **CDS Query Notation (CQN)**:

* `.one` → fetch a single record
* `.from(Users)` → target entity
* `.where({ email })` → filter condition

Equivalent to SQL:

```sql
SELECT * FROM Users WHERE email = ? LIMIT 1;
```

---

## 3. Calling the Action from the Frontend (UI5)

### 3.1 Get the OData Model

```js
const oModel = this.getOwnerComponent().getModel();
```

* `oModel` is an **OData V4 model** connected to the CAP backend.

---

### 3.2 Bind the Action

```js
const oBindingCtx = oModel.bindContext("/TesaService.register(...)");
```

### Explanation

* `/TesaService.register(...)` → OData action path
* `TesaService` → name of CAP service
* `register` → action name
* `(…)` → indicates this is an action call (not a read)

---

### 3.3 Execute the Action

```js
await oBindingCtx.execute();
```

---

## 4. End-to-End Flow

```text
Frontend (UI5)
   ↓
bindContext("/TesaService.register(...)")
   ↓
Backend (CAP service.js)
   ↓
this.on("register", async (req) => { ... })
   ↓
Database / External APIs (e.g., n8n)
   ↓
Response returned to frontend
```

---

## 5. Minimal Working Example

### service.cds

```cds
action register(data : RegisterInput) returns RegisterResult;
```

---

### service.js

```js
this.on("register", async (req) => {
  const { email } = req.data?.data || {};

  console.log("Register action triggered for:", email);

  return {
    message: "User registration triggered"
  };
});
```

---

### Frontend (UI5)

```js
const oModel = this.getOwnerComponent().getModel();

const oBindingCtx = oModel.bindContext("/TesaService.register(...)");

await oBindingCtx.execute();
```

---

## 6. Key Concepts

### Actions vs Reads

* **Reads** → retrieve data (`GET`)
* **Actions** → execute logic (`POST`-like behavior)

---

### `this.on()` vs `this.before()/this.after()`

* `this.on("action")` → handles custom actions
* `this.before/after` → hooks into CRUD events

---

### Safe Data Access

Always use:

```js
req.data?.data || {}
```

to avoid runtime errors when payload is missing.

---

## 7. Common Pitfalls

* Forgetting `(…)` in `bindContext` → action won’t execute
* Mismatch between action name in CDS and handler
* Not calling `.execute()` in UI5
* Assuming `req.data` structure without checking
* Returning data that does not match the defined return type

---

## 8. Notes

* Actions can call external services (e.g., n8n workflows)
* Keep handlers lightweight and move complex logic into helper functions if needed
* Always validate input before processing