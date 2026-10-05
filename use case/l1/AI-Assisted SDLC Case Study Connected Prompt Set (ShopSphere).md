# AI-Assisted SDLC Case Study: Connected Prompt Set

One project, five stages. Each use case consumes the artifacts of the previous one, so the whole set reads as a single build: **Requirements → API → UI → Review → Testing**.

## Shared Project Context (paste at the top of every prompt)

```
Project: ShopSphere, an Online Shopping System for a retail company.
Modules: Product Catalog, Cart, Order Management, Payment, User Management.
Stack: Java 17 + Spring Boot (backend), React (frontend), MySQL, RabbitMQ, Docker.
Style: microservices, REST only, contract-first (OpenAPI 3.0), API version v1.
Non-functional target: 10,000 concurrent users.
Error model (used everywhere): { "code": string, "message": string, "details": [string], "traceId": string }
Rule: if information is missing, state your assumption explicitly. Never invent APIs, libraries or endpoints that are not in the provided artifacts.
```

**Artifact chain**

| Stage | Produces | Used by |
| --- | --- | --- |
| UC1 | architecture.json, ADRs, requirements, traceability matrix | UC2 (service boundaries, error model) |
| UC2 | openapi-product.yaml, openapi-cart.yaml, Spring scaffold | UC3 (endpoints), UC4 (code), UC5 (CartService) |
| UC3 | React components, a11y audit, UI risk report | UC4 (frontend review), UC5 (UI regression points) |
| UC4 | AI review, SonarQube comparison, refactored code | UC5 (stable code to test) |
| UC5 | Test suite, coverage, edge-case document | Final quality gate |

---

## Use Case 1: AI-Assisted Requirements and Architecture Design

**Covers:** prompting for developers, LLM basics, AI in requirements and design, ADR-style decisions, when NOT to use AI.

**Intent vs instruction:** "Design an online shopping system" states only an intent. A good prompt adds role, context, constraints and an output format.

**Prompt 1A: Requirements breakdown**

```
You are a senior business analyst and solution architect.
[Paste Shared Project Context]

Task: Convert the business requirements below into technical requirements.
Business requirements: customers browse products, manage a cart, place orders, pay online, and manage their accounts.

Output strictly as JSON:
{ "functional": [{"id":"FR-01","module":"","requirement":"","source":"BR-xx"}],
  "non_functional": [{"id":"NFR-01","requirement":"","metric":""}],
  "assumptions": [], "open_questions": [] }
Every technical requirement must trace to a business requirement ID.
```

**Prompt 1B: Architecture (uses 1A output)**

```
You are a senior solution architect.
[Paste Shared Project Context]
Input: the requirements JSON from Prompt 1A.

Task: Compare Monolith vs Microservices for these requirements, then recommend one.
For the recommended design, propose microservice boundaries and a database model
(database per service, MySQL; list the main tables of each service).

Output strictly as JSON:
{ "comparison": {"monolith": {"pros":[],"cons":[]}, "microservices": {"pros":[],"cons":[]}},
  "decision": "",
  "services": [{"name":"","responsibilities":[],"api_boundaries":[],"data_model":[],"traced_requirements":[]}],
  "async_events": [{"event":"","producer":"","consumers":[]}],
  "risks": [{"risk":"","impact":"","mitigation":""}] }
```

**Prompt 1C: ADRs (uses 1B output)**

```
Write two ADRs in the format Title / Status / Context / Decision / Consequences / Alternatives.
ADR-001: Why microservices (justify with the 10k concurrent user target and independent scaling).
ADR-002: Why AI must NOT be used inside payment processing logic
(determinism, auditability, PCI-DSS scope, non-reproducible output, liability).
Also state where AI IS appropriate in this project (design, scaffolding, review, testing).
```

**Prompt 1D: LLM controls explanation**

```
Explain, with concrete values, the settings used for UC1 prompts: temperature (suggest 0.0 to 0.2 for
JSON and ADR output, and why), max_tokens, and how context-window limits affect a large requirements
document. Describe how you would split the input when it exceeds the context window.
```

**Build:** logical architecture diagram, JSON schema validation of 1B output, requirement traceability matrix (business requirement → FR/NFR → service). **Deliverables:** architecture.json, ADR-001 and ADR-002, requirements breakdown, temperature and token-control note.

---

## Use Case 2: AI for Backend API Scaffolding (Contract-First)

**Covers:** AI for backend, code generation patterns, API scaffolding prompts, structured outputs, validation and retries. **Input from UC1:** service boundaries for Product and Cart, and the shared error model.

**Prompt 2A: OpenAPI contract**

```
You are a backend engineer.
[Paste Shared Project Context]
Input: the Product Service definition from architecture.json (UC1).

Generate an OpenAPI 3.0 spec for Product Service, base path /api/v1.
Endpoints: GET /products, GET /products/{id}, POST /products.
Constraints:
- Correct HTTP status codes (200, 201, 400, 404, 409, 500)
- Reuse the shared error model as a components/schemas/Error
- Pagination on GET /products (page, size)
- Output strictly valid OpenAPI YAML only, with no explanation text.
```

Repeat for Cart Service (add, update, remove item, get cart) using the same structure.

**Prompt 2B: Controller scaffold (uses 2A output)**

```
From the OpenAPI YAML above, generate a Spring Boot controller, DTOs, service interface and a global
@RestControllerAdvice that returns the shared error model. Do not add endpoints that are not in the spec.
```

**Prompt 2C: Backward compatibility**

```
Add an optional field "category" to the Product schema without breaking v1 clients.
List what is backward compatible and what would require /v2.
```

**Validation and retry:** validate the YAML with an OpenAPI linter (for example Spotless/Spectral). If validation fails, **retry** with the error message appended; if the same failure repeats, **re-prompt** with a clearer, more constrained instruction. Log every attempt. **Max token experiment:** run 2A with a low max\_tokens and observe truncated YAML, then raise the limit and record the difference.

**Deliverables:** openapi-product.yaml, openapi-cart.yaml, backend scaffold, validation logs, prompt refinement documentation.

---

## Use Case 3: AI for Frontend Generation and UI Risk Control

**Covers:** AI for frontend, accessibility checks, UI pitfalls, state bugs, regression risk. **Input from UC2:** only endpoints that exist in the OpenAPI files, such as GET /api/v1/products. This is what prevents hallucinated APIs.

**Prompt 3A: Product listing**

```
Generate a React product listing component.
[Paste Shared Project Context]
Input: the Product OpenAPI YAML (UC2). Use ONLY GET /api/v1/products and the response schema in it.
Requirements:
- Accessible (WCAG 2.1 AA): ARIA labels, keyboard navigation, focus management, live region for status
- Loading, empty and error states (error text from the shared error model)
- No direct state mutation; handle race conditions (cancel stale requests with AbortController)
- Output: component code, then a "UI risks" section.
```

**Prompt 3B: Cart page and checkout form**

```
Using the Cart OpenAPI YAML (UC2), generate the Cart page and a Checkout form with the same rules as 3A.
Add form validation with accessible error messages. Use Redux Toolkit with immutable updates.
```

**Prompt 3C: Audit and regression**

```
Audit the components above for WCAG 2.1 AA issues, state mutation, race conditions and any API call not in
the OpenAPI files. Then give a regression checklist (what to re-test when the API or state changes).
```

**Deliverables:** UI code, accessibility compliance proof (axe or Lighthouse report), UI risk report, regression checklist, prompt library entry (prompt, model, settings, result, fixes).

---

## Use Case 4: AI-Assisted Code Review and Quality Governance

**Covers:** AI code review mindset, SonarQube, readability, complexity, hallucinated APIs, licensing risk. **Input from UC2 and UC3:** the generated Product/Cart services, the Order Service, and the React code.

**Prompt 4A: Review**

```
Review the following Order Service and Cart Service code before deployment.
Checklist:
- Missing null checks
- Cyclomatic complexity (flag methods above 10)
- Hardcoded secrets
- Hallucinated APIs (calls to classes or methods that do not exist in the dependencies or the OpenAPI contract)
- Test coverage gaps
- Licensing concerns in dependencies (e.g. GPL/AGPL in a proprietary product)
Output strictly as JSON:
{ "security_issues": [{"file":"","line":0,"issue":"","fix":""}],
  "complexity_score": "",
  "hallucinated_apis": [],
  "license_risks": [],
  "refactor_suggestions": [],
  "risk_level": "LOW|MEDIUM|HIGH" }
```

**Prompt 4B: Refactor**

```
Refactor the code for every item in the JSON above. Keep behavior and API contract unchanged.
Show a before/after diff for each change.
```

**Build:** run SonarQube on the same code, compare findings with the AI review (what each found or missed, and any false positives), fix the issues, rescan. **Deliverables:** AI review report, SonarQube report, refactored code, quality improvement summary.

---

## Use Case 5: AI-Generated Testing and Edge Case Simulation

**Covers:** AI in testing, test generation, edge cases, output validation, retry vs re-prompt. **Input from UC4:** the refactored, reviewed CartService and Payment code.

**Prompt 5A: Unit and boundary tests**

```
Generate JUnit 5 + Mockito test cases for CartService (refactored version from UC4).
Cover: normal flow, empty cart, large quantity (boundary values), concurrent modification, invalid product ID.
Output as JSON: [{"test_name":"","scenario":"","expected_result":"","code_snippet":""}]
```

**Prompt 5B: Failure simulation**

```
Generate tests for: out-of-stock product, payment timeout (with retry and no double charge),
invalid coupon, and one security scenario (tampered price or unauthorized cart access).
Use the same JSON structure as 5A.
```

**Validation and retry:** validate the JSON against the schema. If invalid, **retry** with the validation error; if it fails again, **re-prompt** with a tighter format. Compile and run the generated tests, and fix any that reference non-existent methods. **Build:** run the suite with JaCoCo and record coverage. **Deliverables:** test code, coverage report, edge case document, security scenario test.

---

## Topic Coverage

| Topic | Covered in |
| --- | --- |
| Prompting | UC 1 to 5 |
| LLM basics | UC 1 |
| AI in requirements and design | UC 1 |
| Backend | UC 2 |
| Frontend | UC 3 |
| Code review | UC 4 |
| Testing | UC 5 |
| Output control (structured output, validation, retry vs re-prompt) | UC 2, 4, 5 |
