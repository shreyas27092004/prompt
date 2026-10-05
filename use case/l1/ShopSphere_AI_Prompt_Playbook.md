# ShopSphere: AI Prompt Playbook

Prompts, expected output, received output and correction prompts for five use cases.
Chain: **Requirements → API → UI → Review → Testing**.

## How to read this document

You are the human. Give the prompts to an AI assistant **in order, in one chat**. Every prompt has the same parts:

| Part | Meaning |
| --- | --- |
| Why this prompt | Purpose and prompting technique |
| Prompt | Exact text to paste |
| Expected output | What a correct answer must contain |
| Output received | Real output from the build (logs are quoted as produced) |
| Result / Check | Whether it matched, and the test or tool that proved it |
| If wrong → correction prompt | Always: *explain what was wrong, then give the corrected output* |

**Notes**
- The build used **Node.js 22 + Express 5 + React/Vite** because that could be installed and run to verify results. The case-study text says Java 17 + Spring Boot. To switch, change only the `STACK` line in Prompt 0 and use JUnit 5 + Mockito + JaCoCo in UC5.
- SonarQube and a 10,000-user load test were **not run**. ESLint (complexity, nesting, params) stands in for SonarQube. NFR "10,000 users" is marked NOT TESTED.
- UC1 artifact files (`requirements.json`, `architecture.json`) were produced in the earlier session; their full contents were not in the transcript, so UC1 "received" shows what was verified (validation logs) and the expected shape.

---

## Prompt 0: Shared project context (give once, first)

**Why:** A new chat has no memory of the project. One context message avoids repeating stack and rules. Rule 2 (never invent APIs) is the main defence against hallucinated code. Rule 3 (only JSON/YAML) makes answers machine-checkable. Rule 4 builds the "explain, then correct" habit.

**Prompt**
```text
You are my senior full-stack engineering assistant. Over the next five stages of this one conversation we will design, build, review and test a small Online Shopping System called ShopSphere. Remember this context for every later message.

BUSINESS: A retail company wants an online shop with five modules: Product Catalog, Cart, Order Management, Payment, User Management.
STACK: Node.js 22 + Express 5 (backend), React + Vite (frontend), in-memory storage (demo), node:test and Vitest for tests.
FIXED RULES: REST only. Contract-first with OpenAPI 3.0. API version prefix /api/v1. Money is stored as integer paise. Every error uses this model: {"code": string, "message": string, "details": [string], "traceId": string}. Target scale: 10,000 concurrent users.
WORKING RULES:
1. If information is missing, state your assumption. Do not guess silently.
2. Never invent APIs, endpoints, libraries or functions that are not in the artifacts I give you.
3. When I ask for JSON or YAML, return ONLY that (no text before or after), because I will validate it with a program.
4. If I tell you an output was wrong, first explain in one or two sentences what was wrong, then give the corrected output.

Reply with "Context received" and a three-line summary.
```

**Expected:** "Context received" + 3-line summary naming the five modules, stack, error model, working rules.

**Received**
```text
Context received.
1) ShopSphere: online shop with Catalog, Cart, Order, Payment and User modules, target 10,000 concurrent users.
2) Node.js 22 + Express 5 backend, React + Vite frontend, in-memory data; REST only, contract-first OpenAPI 3.0 under /api/v1, money in integer paise.
3) One error model {code, message, details[], traceId}; I will state assumptions, never invent APIs, return pure JSON/YAML when asked, and explain-then-correct when told an output was wrong.
```
**Result:** Matches. **Check:** compare summary with your context line by line.

**If wrong** (e.g. summary says "Spring Boot" or omits the error model):
```text
Your summary does not match my context. Explain in one or two sentences which items you got wrong or left out, then repeat the corrected three-line summary. Remember: the stack is Node.js + Express + React, and every error uses {code, message, details, traceId}.
```

---

# Use Case 1: Requirements and architecture design

**Covers:** prompting for developers, LLM basics, AI in requirements/design, ADRs, when NOT to use AI.

**Intent vs instruction**

| | Prompt | Problem / strength |
| --- | --- | --- |
| Weak (intent only) | "Design an online shopping system." | No role, limits or format. Cannot be validated by a program. |
| Strong | Role + context + constraints + JSON structure | Limits the answer space; output can be validated automatically. |

## Prompt 1.1: Business → technical requirements

**Why:** Business wants become numbered, testable requirements. A `source` field on each line gives traceability. Assumptions/open questions force the AI to expose what it doesn't know.

**Prompt**
```text
You are a senior business analyst and solution architect. Use the ShopSphere context.

Task: convert these business requirements into technical requirements.
BR-01 Customers browse the product catalogue.
BR-02 Staff can add products.
BR-03 Customers manage a shopping cart.
BR-04 Customers place orders and see their order history.
BR-05 Customers pay online.
BR-06 Customers register, log in and see their profile.

Output ONLY this JSON, nothing else:
{
 "functional": [{"id":"FR-01","module":"","requirement":"","source":"BR-xx"}],
 "non_functional": [{"id":"NFR-01","requirement":"","metric":"a measurable number"}],
 "assumptions": [],
 "open_questions": []
}
Rules: every functional requirement must have a source BR id. Every non-functional requirement must have a measurable metric.
```

**Expected:** valid JSON only; every FR has module + BR source; every NFR has a number; assumptions and open questions listed.
**Received:** `docs/uc1/requirements.json` — 9 functional, 5 non-functional requirements. **Result:** Matches.
**Check:** file parses as JSON; Prompt 1.2's validator checks ids against it.

**If wrong** (text before JSON, "should be fast" with no number, missing source):
```text
Your last answer cannot be used. Problems: (1) there is text before the JSON, so it does not parse; (2) NFR-01 says "should be fast" and has no measurable metric; (3) FR-04 has no source BR id.
First explain in one or two sentences why each problem matters, then give the corrected JSON only, with a number in every metric (for example "p95 latency under 300 ms").
```

## Prompt 1.2: Monolith vs microservices, boundaries, database model

**Why:** Architecture is the costliest thing to change. Forces comparison before recommendation, one DB per service, and requirement ids on every service. Named keys let a program detect omissions.

**Prompt**
```text
You are a senior solution architect. Use the ShopSphere context and the requirements JSON from Prompt 1.1.

Task: (1) compare Monolith vs Microservices for these requirements; (2) recommend one; (3) propose service boundaries and a database model (one MySQL-style database per service, list the main tables); (4) list asynchronous events; (5) list risks with mitigation.

Output ONLY this JSON:
{
 "comparison": {"monolith": {"pros": [], "cons": []}, "microservices": {"pros": [], "cons": []}},
 "decision": "",
 "services": [{"name":"","responsibilities":[],"api_boundaries":[],"data_model":[],"traced_requirements":["FR-xx"]}],
 "async_events": [{"event":"","producer":"","consumers":[]}],
 "risks": [{"risk":"","impact":"","mitigation":""}]
}
Every FR id must appear in at least one service. Do not use an id that is not in the requirements JSON.
```

**Expected:** balanced comparison, clear decision, five services (Catalog, Cart, Order, Payment, User) with data model/API boundaries/traced ids, risks incl. payment timeout and overselling.
**Received:** `docs/uc1/architecture.json`, validated by schema + id cross-check:

Correct file:
```text
$ node docs/uc1/validate.mjs   (the architecture.json used in this project)
architecture.json schema valid: true
unknown requirement ids: none
functional requirements not covered by any service: none
```
Deliberately broken copy (risks removed, `data_model` removed, invented `FR-99`):
```text
$ node docs/uc1/validate_bad.mjs
architecture.BAD.json schema valid: false
 - #/required: must have required property 'risks'
 - /services/0 must have required property 'data_model'
```
(The invented `FR-99` is reported by the id check once the schema errors are fixed.)

**Result:** Matches after validation.

**If wrong**
```text
The architecture JSON failed validation:
- must have required property 'risks'
- /services/0 must have required property 'data_model'
- service "order-service" traces requirement FR-99, which does not exist in the requirements JSON.
First explain in one or two sentences why each of these makes the output unusable. Then return the complete corrected JSON only.
```

## Prompt 1.3: Logical architecture diagram

**Why:** Non-developers can review a picture. Drawing from the validated JSON stops the diagram showing services that don't exist.

**Prompt**
```text
Using only the validated architecture JSON from Prompt 1.2, produce a logical architecture diagram as a PNG image (you may write a Python matplotlib script for it). Show: the React web app, the API gateway, the five services each with its own database, and the RabbitMQ event OrderPaid. Label each service with its API paths. Add one line saying what is the target architecture and what is delivered.
```

**Expected:** web app, gateway, five services, five DBs, event bus; labels match JSON.
**Received:** `docs/architecture.png`. First version had the dashed event arrow crossing a DB box; fixed by routing it through the gap between boxes. Footer line: *Target: microservices, one DB per service. Delivered: modular monolith with the same boundaries.*
**Check:** visual review against `architecture.json`.

**If wrong**
```text
The diagram contains a service that is not in architecture.json (Notification Service) and one arrow passes through a database box. Explain briefly what was wrong, then give the corrected diagram that shows only the five services in the JSON and has no overlapping arrows.
```

## Prompt 1.4: ADRs

**Why:** An ADR records *why*. ADR-002 covers "when NOT to use AI". ADR-003 records the honest compromise in this build.

**Prompt**
```text
Write three ADRs in this format: Title / Status / Context / Decision / Consequences / Alternatives considered.
ADR-001: Why microservices (use the 10,000 concurrent user target and independent scaling).
ADR-002: Why AI must NOT be used inside payment processing logic (cover determinism, auditability, PCI-DSS scope, hallucination, liability) and say where AI IS appropriate in this project.
ADR-003: Why the first delivery is a modular monolith that keeps the microservice boundaries.
Keep each ADR under 200 words. Do not claim anything about load testing that we have not done.
```

**Expected:** three ADRs with all headings; ADR-002 ≥4 reasons + where AI is appropriate; ADR-003 states load target not tested.
**Received (summary of saved files):**
- **ADR-001 (Accepted):** five services, own DB each, REST + RabbitMQ events. (+) independent scaling of Catalog/Cart vs Order/Payment, Payment outage isolated, independent deploys. (−) ops overhead, no cross-service transactions (consistency via events, retries, idempotency). Alternative: monolith.
- **ADR-002 (Accepted):** no AI/LLM runs inside payment logic. Reasons: determinism, auditability, PCI-DSS scope, hallucinated numbers, liability. Consequence: payment code covered by unit + security tests (declined, timeout, idempotency, price tampering) and human review. AI appropriate for: requirements, API drafting, scaffolding, UI generation, test generation, review assistance.
- **ADR-003 (Accepted):** one Express app, five modules talking through small interfaces (`catalog.getProduct`, `catalog.reserveStock`, `payments.charge`). (+) one process, easy tests, modules extractable later. (−) not independently scalable; 10,000-user target **not load-tested**.

**If wrong**
```text
ADR-001 lists only benefits and ADR-003 claims the system "handles 10,000 users", which we have not tested. Explain in one or two sentences why both are problems for an ADR, then rewrite both ADRs with at least two real costs each and with the load claim removed.
```

## Prompt 1.5: Traceability table

**Why:** Proves each requirement is designed, built and tested, and exposes gaps.

**Prompt**
```text
Create a traceability table with columns: Requirement id | Requirement | Service | API endpoint | Test that proves it. Use only requirement ids from the requirements JSON and only endpoints from the architecture JSON. If a requirement is not yet proven by a test, write "NOT TESTED" and say why.
```

**Expected:** one row per requirement; no invented ids/endpoints; honest "NOT TESTED".
**Received**

| Id | Requirement | Service / endpoint | Proven by |
| --- | --- | --- | --- |
| FR-01 | List products, paginated, category filter | catalog: GET /products | catalog-users: pagination, category filter; contract: ProductPage schema |
| FR-02 | Get product by id, 404 if unknown | catalog: GET /products/{id} | contract: error schema 404 |
| FR-03 | Admin creates product | catalog: POST /products | only admin; duplicate 409; invalid 400 |
| FR-04 | Add/update/remove cart items | cart: /cart, /cart/items | cart: normal flow; UI Cart tests |
| FR-05 | Reject quantity above stock | cart: POST /cart/items | cart: boundary quantities; concurrent additions |
| FR-06 | Create order with optional coupon | orders: POST /orders | coupon; totals rounding; empty cart |
| FR-07 | Users see only own orders | orders: GET /orders, /orders/{id} | cannot read other user's order |
| FR-08 | Failed payment creates no order | payments: charge() | declined 402; timeout 504; idempotency; unexpected 500 |
| FR-09 | Register, log in, profile | users: /users/register, /auth/login, /users/me | register/login/me; same 401 message |
| NFR-01 | 10,000 concurrent users | design level | **NOT TESTED** (no load test; see ADR-003) |
| NFR-02 | 100% routes in OpenAPI | all | contract: every documented operation has a route |
| NFR-03 | Uniform error model | all | contract error schema; malformed JSON; 500 without leaks |
| NFR-04 | Passwords hashed | users | hash never returned; salted scrypt |
| NFR-05 | WCAG 2.1 AA | frontend | axe tests (automated part only) |

**If wrong**
```text
Row NFR-01 says "tested" but no load test exists, and the test "checkout_stress_test" in row FR-06 does not exist in the repository. Explain in one or two sentences why a traceability table must not claim this, then give the corrected table using only tests that exist and "NOT TESTED" where there is no proof.
```

## Prompt 1.6: Tokens, context window, temperature

**Why:** "LLM basics". Too few output tokens → cut-off YAML; too much randomness → different JSON every run.

**Prompt**
```text
Explain, for this project only: (1) what a token is and roughly how many tokens our OpenAPI file will need (it is about 9,000 characters); (2) what the context window is and what we do when the requirements document is larger than it; (3) the temperature and max_tokens you recommend for each kind of prompt we use (JSON, YAML, ADR text, code, tests, risk brainstorming), in a table with a one-line reason each. Say clearly which settings I can change in a chat window and which only through the API.
```

**Expected:** ~3–4 chars/token for code/YAML + estimate; strategy for large inputs; low temperature for structured output.
**Received**

| Prompt type | Temperature | max_tokens | Reason |
| --- | --- | --- | --- |
| Requirements/architecture JSON (1.1, 1.2) | 0.0–0.2 | 3,000 | Repeatable, parseable; ~1,500–2,500 tokens |
| OpenAPI YAML (2.1, 2.2) | 0.0 | 4,000 | ~9,000 chars ≈ 2,600 tokens; leave headroom |
| ADR text (1.4) | 0.2–0.3 | 1,500 | Prose may vary, decisions must stay consistent |
| Code generation (2.3, 3.x) | 0.0–0.2 | 4,000 | Deterministic code is easier to review |
| Test generation (5.x) | 0.0–0.2 | 4,000 | Stable tests, valid JSON wrapper |
| Risk brainstorming | 0.5–0.7 | 1,500 | Variety useful; human reviews every idea |

- **Context window:** split by module, one part per prompt, carry only short summaries, repeat the short context block instead of full history.
- **Chat vs API:** temperature and max_tokens are API settings. In chat, control output through wording ("return only JSON", "under 200 words") and by splitting prompts.

**Check:** `backend/openapi.yaml` measured at 9,018 characters. The ~2,600-token figure is a rule of thumb, not measured.

**If wrong**
```text
You recommended temperature 1.0 for the OpenAPI YAML, and you said I can set max_tokens in the chat window. Explain in one or two sentences what is wrong with both statements, then give the corrected table: low temperature for structured output, and a clear split between chat and API settings.
```

**UC1 deliverables:** `architecture.json` (+ schema, `validate.mjs`), `requirements.json`, ADR-001/002/003, `architecture.png`, traceability table (1.5), temperature/token note (1.6).

---

# Use Case 2: Backend API scaffolding (contract-first)

**Covers:** AI for backend, code generation patterns, structured output, validation and retries. **Input from UC1:** service boundaries, shared error model.

## Prompt 2.1: OpenAPI contract: Product + Cart

**Why:** Contract first means code is generated *from* an agreed spec, so UI and tests can't drift. "YAML only" lets a validator reject bad output.

**Prompt**
```text
You are a backend engineer. Use the ShopSphere context and the Catalog and Cart service definitions from architecture.json (UC1).

Generate ONE OpenAPI 3.0 YAML file, base path /api/v1.
Product endpoints: GET /products (page, size, optional category), GET /products/{id}, POST /products.
Cart endpoints: GET /cart, POST /cart/items, PATCH /cart/items/{productId}, DELETE /cart/items/{productId}.
Also include: POST /users/register, POST /auth/login, GET /users/me, and POST /orders, GET /orders, GET /orders/{id}.
Constraints:
- Correct status codes (200, 201, 400, 401, 403, 404, 409, 500; 402 and 504 for payment failures)
- components/schemas/Error is the shared error model {code, message, details, traceId}
- Every path with a parameter has a 404; every POST has a 400
- Pagination on GET /products
- Output strictly valid OpenAPI YAML only, no explanation text.
```

**Expected:** valid YAML; shared `Error` schema; every `$ref` resolves; each operation has a 2xx; path-param ops have 404; POSTs have 400.
**Received:** `backend/openapi.yaml` (9,018 chars). Validated by `docs/uc2/validate_openapi.mjs`:
```text
$ node validate_openapi.mjs openapi.yaml
ACCEPT openapi.yaml

$ node validate_openapi.mjs openapi.WRONG.yaml   (draft with no shared Error model and GET /products/{id} missing 404)
REJECT openapi.WRONG.yaml
 - components.schemas.Error (shared error model) missing
 - unresolved $ref #/components/schemas/Error
 - GET /products/{id}: path parameter but no 404 response
```
Later proven against the running server:
```text
$ npm test -- test/contract.test.js
ok 1 - contract: every documented operation has a route (no 404 ROUTE not found)
ok 2 - contract: GET /products response matches ProductPage schema
ok 3 - contract: error responses match the shared Error schema (400, 401, 404)
ok 4 - contract: checkout order response matches Order schema
# tests 4  # pass 4  # fail 0
```

**If wrong**
```text
Your OpenAPI YAML was rejected by the validator:
- components.schemas.Error (shared error model) missing
- unresolved $ref #/components/schemas/Error
- GET /products/{id}: path parameter but no 404 response
First explain in one or two sentences why each makes the contract unusable, then return the complete corrected YAML only.
```

## Prompt 2.2: Retry vs re-prompt, and the max-token experiment

**Why:** Two failure types need two responses. **Retry** = same prompt + error message (fixes small slips). **Re-prompt** = tighter, smaller prompt (when the same failure repeats, e.g. truncation).

**Prompt (first run, deliberately low max_tokens)** — use Prompt 2.1 with `max_tokens` set very low (API). If it fails twice, give:
```text
The output was cut off in the middle of the file twice. Explain in one sentence why a larger prompt with the same token limit keeps failing. Then generate the YAML in two parts: first only the Product and User paths plus components/schemas, then (when I say "part 2") the Cart and Order paths. Return YAML only.
```

**Expected:** truncated YAML rejected; same retry fails the same way; narrower re-prompt (or higher limit) is accepted.
**Received** (`docs/uc2/retry_demo.mjs`, recorded attempts):
```text
[attempt 1] Attempt 1 (max_tokens too low -> output cut off mid-file)
   output size: 2600 chars -> REJECTED
   - components.schemas.Error (shared error model) missing
   - unresolved $ref #/components/schemas/ProductPage
   - unresolved $ref #/components/responses/Error
   - unresolved $ref #/components/schemas/NewProduct
   - unresolved $ref #/components/schemas/Product
   - unresolved $ref #/components/schemas/Cart
   - unresolved $ref #/components/schemas/CartItemInput
   - PATCH /cart/items/{productId}: no responses
[attempt 2] Attempt 2 (RETRY with same prompt, max_tokens still low -> same truncation)
   output size: 2600 chars -> REJECTED
   (same errors)
[attempt 3] Attempt 3 (RE-PROMPT: smaller scope + higher max_tokens -> complete file)
   output size: 9018 chars -> ACCEPTED
```
**Note:** attempts 1–2 are a simulation using a truncated copy of the real file, not live model runs.

## Prompt 2.3: Backend scaffold from the contract

**Why:** Restricting the AI to the YAML ("do not add endpoints not in the spec") prevents extra, undocumented routes. A central error handler makes the error model identical everywhere. (Java/Spring: ask for `@RestController`, DTOs, service interface, `@RestControllerAdvice`.)

**Prompt**
```text
From openapi.yaml, generate the backend: one Express 5 app with modules catalog, cart, orders, payments, users, a central error handler returning {code, message, details, traceId}, and input validation helpers. Rules: implement ONLY endpoints in the spec; money in integer paise; passwords hashed (salted scrypt, never returned); orders priced on the server from the cart (ignore any price sent by the client); on payment failure restore stock and create no order; payments must be idempotent per key; users can read only their own orders (return 404, not 403). Use in-memory storage. Add a mock payment gateway with tokens tok_ok, tok_declined, tok_timeout. Return files only.
```

**Expected:** all spec routes present, no extras; 400/401/403/404/409/402/504/500 mapped to the error model; stock safe on failure.
**Received:** `backend/src/{app,server,errors,validate}.js`, `src/modules/{users,catalog,cart,payments,orders}.js`. First smoke run:
```text
200 4
```
First full test run revealed real problems (these are the "wrong output" cases):
```text
# tests 25  # pass 24  # fail 1
SyntaxError: The requested module 'js-yaml' does not provide an export named 'default'
```
then, after fixing the import:
```text
not ok 1 - contract: every documented operation has a route
  error: 'Request with GET/HEAD method cannot have body.'
```
After fixing the contract test (no body on GET) and adding missing cases:
```text
# tests 27  # pass 27  # fail 0
```
Live end-to-end run of the finished API:
```text
$ node src/server.js   (PORT=3111)
ShopSphere API listening on http://localhost:3111/api/v1

$ curl GET /products?size=2
{"total": 4, "first": "Wireless Mouse"}
$ curl POST /users/register
{"id":"708d7cf9-...","email":"demo@shop.test","role":"USER"}
$ curl POST /cart/items (2 x Wireless Mouse)
{"items":[{"productId":"e1d9eef2-...","name":"Wireless Mouse","unitPricePaise":79900,"quantity":2}],"totalPaise":159800}
$ curl POST /orders (coupon SAVE10, token tok_ok)
{"subtotalPaise": 159800, "discountPaise": 15980, "totalPaise": 143820, "status": "PAID", "coupon": "SAVE10"}
$ curl POST /orders again (cart is now empty)
{"code": "BAD_REQUEST", "message": "Cart is empty", "details": [], "traceId": "<uuid>"}
```

**If wrong** (use the real error text)
```text
Running the tests gives: "SyntaxError: The requested module 'js-yaml' does not provide an export named 'default'". Explain in one or two sentences what is wrong with the import, then give the corrected file in full. Do not change any other behaviour.
```

## Prompt 2.4: Backward compatibility

**Why:** Teaches what is safe in v1 and what needs /v2.

**Prompt**
```text
Add an optional field "category" to the Product schema without breaking v1 clients. List what is backward compatible and what would require /v2.
```

**Expected:** optional field and optional `category` query filter = compatible. Removing/renaming fields, making `category` required, changing types, or changing error codes = needs /v2.
**Received:** `category` is present on products and works as an optional `GET /products?category=` filter; the pagination/category test and the ProductPage contract test pass (`catalog: category filter and invalid size are handled`).

**If wrong**
```text
You listed "make category required" as backward compatible. Explain in one or two sentences why that would break existing v1 clients, then give the corrected two lists.
```

**UC2 deliverables:** `backend/openapi.yaml`, backend scaffold, validation logs (`docs/evidence/uc2_validation.txt`, `uc2_retry.txt`, `uc2_contract_tests.txt`), this prompt refinement record.

---

# Use Case 3: Frontend generation and UI risk control

**Covers:** AI for frontend, accessibility, UI pitfalls, state bugs, regression risk. **Input from UC2:** only endpoints in `openapi.yaml` (prevents hallucinated APIs).

## Prompt 3.1: Product listing

**Why:** Accessibility, loading/empty/error states and race-condition handling must be **in the prompt**; a first draft without them fails axe (shown below).

**Prompt**
```text
Generate a React product listing component. Use the ShopSphere context.
Input: openapi.yaml (UC2). Use ONLY GET /api/v1/products and the response schema in it.
Requirements:
- Accessible (WCAG 2.1 AA): ARIA labels, keyboard navigation, focus management, live region for status
- Loading, empty and error states (error text from the shared error model), with a retry button
- No direct state mutation; cancel stale requests with AbortController
- Add-to-cart button disabled when logged out or out of stock
- Output: component code, then a "UI risks" section.
```

**Expected:** `ProductList.jsx` calling only `/api/v1/products`; `role="status"` live region; `role="alert"` on error; AbortController cleanup.
**Received:** `frontend/src/components/ProductList.jsx` + 8 tests:
```text
 ✓ ProductList > calls only the documented endpoint /api/v1/products
 ✓ ProductList > shows a loading status, then the products
 ✓ ProductList > shows an empty state
 ✓ ProductList > shows an accessible error alert and retries
 ✓ ProductList > disables add-to-cart when logged out or out of stock, and exposes descriptive button names
 ✓ ProductList > calls onAdd with the product
 ✓ ProductList > aborts the in-flight request on unmount (race-condition guard)
 ✓ ProductList > has no axe accessibility violations (ready and error states)
```
**Why the constraint matters:** a first draft written *without* the accessibility rule:
```text
AXE VIOLATIONS IN FIRST DRAFT:
 - button-name (critical): Buttons must have discernible text
 - image-alt (critical): Images must have alternative text
```

**If wrong**
```text
axe reports: "button-name (critical): Buttons must have discernible text" and "image-alt (critical): Images must have alternative text". Explain in one or two sentences why each fails WCAG 2.1 AA, then return the corrected component in full with descriptive aria-labels and alt text.
```

## Prompt 3.2: Cart page and checkout form

**Prompt**
```text
Using openapi.yaml (UC2), generate the Cart component, a CheckoutForm and an App that wires login, product list, cart and checkout. Same rules as 3.1. Form validation with accessible error messages (label, aria-invalid, aria-describedby, role="alert"). Immutable state updates only. Block double submission of the checkout form. Call only endpoints that exist in the spec through one api.js file.
```

**Expected:** table with `scope` headers, labelled +/− buttons, disabled while busy; checkout validates token, blocks double submit.
**Received:** `Cart.jsx`, `CheckoutForm.jsx`, `App.jsx`, `api.js`, `styles.css`. Build and tests:
```text
dist/assets/index-B5baQr0m.js   226.56 kB │ gzip: 70.79 kB
✓ built in 148ms

 ✓ Cart > renders items, total and descriptive controls
 ✓ Cart > shows the empty state
 ✓ Cart > sends the new quantity without mutating the cart prop
 ✓ Cart > disables decrease at quantity 1 and everything while busy
 ✓ Cart > has no axe violations
 ✓ CheckoutForm > shows an accessible validation error and does not submit when the token is empty
 ✓ CheckoutForm > submits trimmed values; empty coupon becomes undefined
 ✓ CheckoutForm > prevents double submission while the first is in flight
 ✓ CheckoutForm > has no axe violations (clean and with error shown)
 Test Files  3 passed (3)   Tests  18 passed (18)
```

**If wrong**
```text
The checkout form submitted twice when I pressed Enter during the first request, which could charge the customer twice. Explain in one or two sentences why this happens, then give the corrected CheckoutForm in full with a guard that ignores a second submit while the first is running.
```

## Prompt 3.3: Audit and regression checklist

**Prompt**
```text
Audit the components above for WCAG 2.1 AA issues, state mutation, race conditions and any API call not in openapi.yaml. List each finding with file and fix. Then give a regression checklist: what to re-test when the API or state changes.
```

**Expected:** findings list (or "none found" with evidence) + checklist.
**Received / evidence:** axe passes on product list (ready + error), cart, checkout (clean + error); frozen-object test proves no state mutation; `api.js` only calls `/products`, `/auth/login`, `/users/register`, `/cart`, `/cart/items`, `/orders`, all in the contract.
**Regression checklist (re-test when API or state changes):**
- Product list: loading → ready → empty → error → retry; stale response never overwrites a newer one
- Cart: add/update/remove, totals, quantity > stock message from the API
- Checkout: empty token error, double-submit blocked, tok_declined / tok_timeout messages shown
- Any change to `openapi.yaml`: re-run contract tests and the `calls only the documented endpoint` test
- Re-run axe tests after any markup change
- Limit: axe covers only the automatic part of WCAG; keyboard and screen-reader testing are manual.

**If wrong**
```text
Your audit says "no issues" but you did not check keyboard focus after a failed validation. Explain in one or two sentences what you missed, then give an updated audit that covers focus management and a manual keyboard test script.
```

**UC3 deliverables:** UI code, axe test results (`docs/evidence/uc3_vitest.txt`), UI risk notes, regression checklist, prompt library entry (prompt, model, settings, result, fixes).

---

# Use Case 4: Code review and quality governance

**Covers:** AI code review mindset, static analysis (SonarQube stand-in), complexity, hallucinated APIs, licensing.

## Prompt 4.1: Review before deployment

**Why:** AI-written code can look right and be wrong. A checklist and a JSON output make the review comparable with a tool's findings. Draft under review (`docs/uc4/OrderServiceDraft.js`) was written *without* the checklist and has seeded flaws: hardcoded `sk_live_` key, `crypto.randomUUIDv7()` (does not exist), no null checks, 5-level nesting, unawaited `gateway.charge`, floating-point money.

**Prompt**
```text
Review the following Order Service code before deployment. Checklist:
- Missing null checks
- Cyclomatic complexity (flag methods above 10) and nesting depth
- Hardcoded secrets
- Hallucinated APIs (calls to classes or methods that do not exist in Node's standard library, our dependencies or the OpenAPI contract)
- Money handling and unawaited asynchronous calls
- Test coverage gaps
- Licensing concerns in dependencies (GPL/AGPL in a proprietary product)
Output ONLY this JSON:
{ "security_issues": [{"file":"","line":0,"issue":"","fix":""}],
  "complexity_score": "",
  "hallucinated_apis": [],
  "license_risks": [],
  "refactor_suggestions": [],
  "risk_level": "LOW|MEDIUM|HIGH" }
```

**Expected:** secret at line 4; `randomUUIDv7` as hallucinated; null-check, floating-point, unawaited call; risk HIGH.
**Received** — what the tools found on the same draft:
```text
### ESLint (complexity>8, max-depth>4, max-params>3) on OrderServiceDraft.js
  6:8   warning  Function 'placeOrder' has too many parameters (4). Maximum allowed is 3  max-params
  13:11  warning  Blocks are nested too deeply (5). Maximum allowed is 4                   max-depth
✖ 2 problems (0 errors, 2 warnings)

Running the draft -> TypeError: crypto.randomUUIDv7 is not a function

../docs/uc4/OrderServiceDraft.js:4:const PAYMENT_API_KEY = 'sk_live_51HqXyZabcdef123456'; // hardcoded secret
```
Comparison (AI review vs static tools):

| Finding | AI review (checklist) | ESLint | Run / grep |
| --- | --- | --- | --- |
| Hardcoded secret | ✔ | ✘ (needs a secrets plugin) | ✔ grep |
| Hallucinated `randomUUIDv7` | ✔ | ✘ | ✔ runtime TypeError |
| Deep nesting / many params | ✔ | ✔ | n/a |
| Floating-point money, unawaited charge | ✔ | ✘ | n/a |
| Missing null checks | ✔ | ✘ | n/a |

SonarQube was **not** run; `sonar-project.properties` is provided for you to run it.

**If wrong**
```text
Your review missed that crypto.randomUUIDv7 does not exist: running the code gives "TypeError: crypto.randomUUIDv7 is not a function". Explain in one or two sentences why you did not catch it and how a hallucinated API can be detected, then return the corrected review JSON including it under hallucinated_apis.
```

## Prompt 4.2: Fix the real code, test-first

**Why:** Asking for a failing test *before* the fix proves the problem was real and the fix works.

**Prompt**
```text
Review the real backend (payments.js, orders.js, catalog.js) for security problems. For each problem: first write a test that fails, show me the failure, then fix the code and show the test passing. Keep behaviour and the API contract unchanged.
```

**Expected:** at least the idempotency-key and coupon-lookup issues found, each with a failing test, then a passing one.
**Received**
Before fix:
```text
$ npm test   (new UC4 security tests written BEFORE the fix)
not ok 14 - security (UC4 finding): idempotency keys are scoped per user, one user cannot replay another's payment
    Expected "actual" to be strictly unequal to:
  expected: '5f0f2daf-b972-4c78-9d9f-e397a61fc580'
  actual: '5f0f2daf-b972-4c78-9d9f-e397a61fc580'
ok 15 - security (UC4 finding): coupon lookup ignores prototype keys like "constructor"
# pass 14  # fail 1
```
Fixes: idempotency key scoped as `userId:key`; coupon lookup uses `Object.hasOwn`; paging helper split to cut complexity.
After fix:
```text
$ npm test   (after the fix)
# tests 30  # pass 30  # fail 0
$ npm run lint
  cart.js     5:8  Function 'createCartModule' has too many lines (56). Maximum allowed is 40
  catalog.js 19:8  Function 'createCatalogModule' has too many lines (59). Maximum allowed is 40
  orders.js  21:8  Function 'createOrderModule' has too many lines (47). Maximum allowed is 40
  users.js   20:8  Function 'createUserModule' has too many lines (49). Maximum allowed is 40
✖ 4 problems (0 errors, 4 warnings)
```
Remaining 4 warnings are function-length only and were accepted, not fixed. Secret scan of `src/`: `no secrets found in src/`.

**If wrong**
```text
After your fix, Bob still received Alice's cached payment result. Explain in one or two sentences why scoping by key alone is not enough, then return the corrected payments.js in full, with the idempotency key scoped by user id.
```

## Prompt 4.3: Licence check

**Prompt**
```text
Write a script that lists the licence of every installed production dependency (direct and transitive) and flags anything that is not MIT, ISC, BSD, Apache-2.0 or similar permissive.
```
**Expected:** per-package licence counts; flag for copyleft/unknown.
**Received** (after moving dev tools out of production dependencies):
```text
$ node docs/uc4/license_check.mjs backend
backend: 65 production packages (direct: express)
licenses: {"MIT":60,"ISC":4,"BSD-3-Clause":1}
No copyleft or unknown licenses found.

$ node docs/uc4/license_check.mjs frontend
frontend: 3 production packages (direct: react, react-dom)
licenses: {"MIT":3}
No copyleft or unknown licenses found.
```
(An earlier run counted 140 packages because `ajv`, `js-yaml`, `eslint` were wrongly listed as production dependencies; fixing `package.json` corrected the count.)

**If wrong**
```text
Your licence report counts eslint and ajv as production dependencies. Explain in one sentence why that inflates the report and may flag the wrong packages, then correct package.json and rerun the check.
```

**UC4 deliverables:** AI review (4.1), static-analysis output, refactored code + before/after logs (4.2), licence report (4.3), quality summary: **tests 28→30 passing, 1 real security bug fixed, secret scan clean, lint 0 errors**.

---

# Use Case 5: Testing and edge-case simulation

**Covers:** AI in testing, edge cases, output validation, retry vs re-prompt. **Input from UC4:** the reviewed backend.

## Prompt 5.1: Unit and boundary tests

**Prompt**
```text
Generate node:test tests for the Cart module (Java equivalent: JUnit 5 + Mockito for CartService). Cover: normal flow, empty cart, large/boundary quantity, concurrent modification, invalid product id.
Output ONLY this JSON array:
[{"test_name":"","scenario":"","expected_result":"","code_snippet":""}]
```
**Expected:** five scenarios incl. quantities 0, −1, 1.5, "2", null, exactly stock, stock+1, MAX_SAFE_INTEGER; 10 parallel adds never exceed stock.
**Received** (`backend/test/cart.test.js`, `docs/uc5/test-cases.json`):
- Normal flow: 2 + 1 merge to 3; total = unit price × quantity; remove → `{items: [], totalPaise: 0}`
- Empty cart: 200, `items=[]`, `totalPaise=0`
- Boundary: invalid numbers → 400; exactly stock (3) → 200; stock+1 and `MAX_SAFE_INTEGER` → 409 "Only 3 unit(s)"
- Invalid product → 404; absent item → 404; no token → 401
- Concurrent: 10 parallel adds, stock 3 → at least one 409, cart quantity ≤ 3

A first generated "out of stock" test was confused (a user could not even add the item) and was replaced with: *"once the last units are bought, further adds are rejected with 409"*.

**If wrong**
```text
This test passes for the wrong reason: it asserts the second user "cannot add" before any order exists, so it never tests checkout. Explain in one or two sentences what scenario it should cover, then replace it with a test where two users hold the last units and both try to check out.
```

## Prompt 5.2: Failure simulation and security

**Prompt**
```text
Generate tests for: stock sold out between add-to-cart and checkout (no payment attempt for the second buyer), payment timeout (no order, stock restored), declined card, invalid coupon, idempotency (no double charge), unexpected gateway failure (500 with no leaked internals), and security scenarios: tampered price in the request body and reading another user's order. Same JSON structure as 5.1.
```
**Expected:** each failure leaves stock and cart consistent; no double charge; no data leak.
**Received** (`backend/test/orders-payments.test.js`):

| Scenario | Expected result (passing) |
| --- | --- |
| Sold out between add and checkout | first order 201; second 409; gateway called **once**; second cart kept |
| Payment timeout (`tok_timeout`, 200 ms vs 50 ms limit) | 504 `PAYMENT_TIMEOUT`; stock restored (25); no order |
| Declined (`tok_declined`) | 402 `PAYMENT_DECLINED`; stock restored; no order; cart kept |
| Invalid coupon `FAKE99`, then `save10` | 400 naming coupon, stock untouched; valid → discount 7,990 paise, stored as `SAVE10` |
| Same idempotency key twice | gateway charged once; same payment id |
| Gateway throws `db password is hunter2` | 500 `INTERNAL_ERROR`; body does not contain `hunter2`; stock restored |
| Tampered price (`totalPaise: 1`) | ignored; total is real 349,900 |
| Other user's order id | 404 (not 403) |

**If wrong**
```text
The timeout test passes but the stock afterwards is 23 instead of 25, so a failed payment still removed stock. Explain in one or two sentences why this is a defect, then fix orders.js so stock is released on timeout and extend the test to assert the stock value.
```

## Prompt 5.3: Output validation (schema, retry, re-prompt)

**Why:** The AI's JSON test list must itself be validated before use. Invalid → **retry** with the validator's message; invalid again → **re-prompt** with a tighter format.

**Prompt (retry)**
```text
Your test-case JSON failed schema validation:
- /0 must have required property 'expected_result'
- /1 must NOT have additional properties: notes
First explain in one or two sentences what is wrong, then return the complete corrected JSON array only, with exactly the four keys test_name, scenario, expected_result, code_snippet on every item.
```
**Prompt (re-prompt, if it fails again)**
```text
Return ONLY a JSON array. Each item has exactly these four string keys and no others: test_name, scenario, expected_result, code_snippet. code_snippet must be at least 20 characters. No text, no markdown fences, no comments.
```
**Expected:** invalid file rejected with exact paths; corrected file valid.
**Received**
```text
$ node validate_cases.mjs test-cases.BAD.json   (attempt 1: a field missing, an extra field added)
./test-cases.BAD.json: INVALID (2 test cases)
 - /0 must have required property 'expected_result': expected_result
 - /1 must NOT have additional properties: notes

$ node validate_cases.mjs test-cases.json   (attempt 2 after RETRY with the validation errors appended to the prompt)
test-cases.json: VALID (13 test cases)
```

## Prompt 5.4: Run the suite and record coverage

**Prompt**
```text
Run the whole backend suite with coverage and report the numbers. List any file below 100% and say why.
```
**Expected:** all tests pass; coverage table; honest gaps.
**Received**
```text
$ npm run coverage
# tests 30  # pass 30  # fail 0
#  app.js          | 100.00 | 100.00 | 100.00
#  errors.js       | 100.00 | 100.00 | 100.00
#   cart.js        | 100.00 | 100.00 | 100.00
#   catalog.js     | 100.00 | 100.00 | 100.00
#   orders.js      | 100.00 | 100.00 | 100.00
#   payments.js    | 100.00 | 100.00 | 100.00
#   users.js       | 100.00 | 100.00 | 100.00
#  validate.js     | 100.00 | 100.00 | 100.00
#  helpers.js      | 100.00 |  93.75 | 100.00
# all files        | 100.00 |  99.54 | 100.00
```
(columns: line % | branch % | funcs %.) Note: 100% line coverage proves lines ran, not that behaviour is right; the failure and security tests above are what give confidence. Frontend: 18 tests. Load testing: not done.

**If wrong**
```text
You reported "100% coverage, fully tested" but NFR-01 (10,000 users) has no test. Explain in one or two sentences why coverage does not prove this, then rewrite the summary to separate "covered by tests" from "not tested".
```

**UC5 deliverables:** test code (`backend/test/*.test.js`, `frontend/src/**/*.test.jsx`), coverage report, edge-case document (`docs/uc5/test-cases.json`), security scenario tests (price tampering, other user's order, idempotency scoping).

---

## Topic coverage

| Topic | Covered in |
| --- | --- |
| Prompting | UC1–UC5 |
| LLM basics | 1.6 |
| AI in requirements/design | UC1 |
| Backend | UC2 |
| Frontend | UC3 |
| Code review | UC4 |
| Testing | UC5 |
| Output control (structured output, validation, retry vs re-prompt) | 2.1–2.2, 4.1, 5.3 |

## Honest limits

- SonarQube not run (ESLint used instead). Load test not run (NFR-01 NOT TESTED).
- Retry demo in 2.2 uses recorded/simulated attempts, not live model calls.
- Bugs shown as "wrong output" (js-yaml import, GET with body, idempotency scoping) are real errors from this build; others in "If wrong" blocks are templates for your own runs.
- Project source and evidence logs lived in the earlier Claude session's folder (`/home/claude/shopsphere`); this file was written from the transcript.
