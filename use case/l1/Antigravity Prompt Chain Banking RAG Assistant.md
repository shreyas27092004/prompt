# Antigravity Prompt Chain: Banking RAG Assistant

*Six connected prompts for Antigravity or VS Code, one per use case. Each prompt reads the previous prompt's output, builds its deliverables, and writes exactly one documentation file recording Input, Process and Output.*

## 0. One-time setup

**Step 1.** Create an empty folder `banking-rag/`, open it in your IDE, and put the Secure Bank policy PDF at `docs/source/secure-bank-policy.pdf`.

**Step 2.** Start a new agent chat (Planning mode in Antigravity; in VS Code Copilot use Agent mode and add: show me a plan first and wait for approval). Paste the block below once as the first message, or save it as a workspace rule file so every later prompt inherits it.

```text
PROJECT RULES (apply to every prompt in this chain)

Project: AI-Powered Banking Support System for Secure Bank.
Goal: a secure, grounded banking assistant that (a) answers policy questions with Document RAG over the bank policy PDF, with citations, and (b) returns a customer's own account, card and loan data through JWT-protected tool calling. The LLM never generates account data; it only calls tools.

Stack (editable): Java 17, Spring Boot, LangChain4j, Azure OpenAI (or any LLM, configured by env vars), Angular frontend, MySQL 8 installed locally. NO Docker.

Database: local MySQL at localhost:3306, database banking_db. Read DB_USER and DB_PASSWORD from environment variables. Never hardcode credentials or keys. Use Flyway migrations. Use H2 in MySQL mode only for unit tests.
Secrets: LLM endpoint, LLM key and JWT secret come from env vars. Provide .env.example with placeholders only.
Vector store: put it behind a VectorStore interface. Default implementation: LangChain4j embedding store persisted to a file on disk, so it survives restarts and can be swapped later.
Source document: docs/source/secure-bank-policy.pdf. If it is missing, stop and ask me.
Data: synthetic customers only. No real personal data anywhere.

Folder layout (create as needed):
  docs/         -> one documentation file per prompt (docs/01-..., docs/06-...)
  docs/source/  -> source policy PDF
  docs/deliverables/ -> only files a prompt explicitly lists as deliverables
  backend/      -> Spring Boot application
  frontend/     -> Angular application
  evaluation/   -> golden questions, evaluation results
  observability/-> metrics and cost reports
  prompts/      -> prompt-library.md (every prompt used, plus refinements)
  .github/workflows/ -> CI/CD

DOCUMENTATION RULE (mandatory, every prompt):
- Create ONLY ONE documentation file per prompt, at the path I give.
- Do not create any other summary or report .md file, except files explicitly listed as deliverables for that prompt.
- The documentation file must have exactly these top-level sections:
  1. INPUT   - the prompt text used (verbatim), files read, parameters (model, temperature, top_k, chunk size, threshold), assumptions
  2. PROCESS - numbered steps actually performed, decisions and why, tests run, retries or fixes
  3. OUTPUT  - every file created or changed (path + one-line purpose), results, known gaps
  4. HANDOFF - what the next prompt must read and rely on
- Document what really happened. Never invent metrics, scores, token counts, costs or test results. If something could not be run, say so.

GENERAL BEHAVIOUR:
- Restate my request as INTENT (goal) and INSTRUCTIONS (constraints) before working.
- Do not use APIs, libraries or methods you cannot confirm exist. Flag uncertainty.
- Ask me before any assumption that changes scope.

Reply 'Rules loaded' and wait for Prompt 1.
```

**Before you start:** install Java 17, Maven, Node.js, Angular CLI, and MySQL; create the database with `CREATE DATABASE banking_db;`; set `DB_USER`, `DB_PASSWORD`, and your LLM variables in the terminal that runs the app.

## Prompt 1: Foundation and Core Retrieval (Use Case 1)

**Covers:** LLM behaviour, hallucination and stale-knowledge risk, ingestion pipeline, embeddings, vector storage, keyword vs semantic vs hybrid retrieval. **Doc produced:** `docs/01-foundation-retrieval.md`

```text
PROMPT 1 - FOUNDATION AND CORE RETRIEVAL FOR BANKING POLICIES

Role: You are a senior AI engineer building a grounded retrieval layer for a bank.

INTENT: Let users search the official Secure Bank policy manual by meaning, with minimal hallucination risk.

INSTRUCTIONS / CONSTRAINTS:
- Answers must come only from the verified policy document. No unsupported financial advice.
- Use the SAME embedding model for indexing and for queries. Make the model name a config value.
- Chunk size about 800 tokens, overlap about 120 tokens, configurable. Prefer splitting on the manual's section headings (for example 3.1, 4.5) so each chunk keeps its section and page.
- Every chunk stores metadata: document, section, page, category (KYC, Accounts, Loans, Cards, Branch, InfoSec, Grievance, Audit, BCP, FD), version, ingested_at.
- Apply a similarity threshold so weak matches are dropped.

TASKS:
1. Create the Spring Boot backend skeleton (backend/) with config via env vars and a /health endpoint.
2. Build the ingestion pipeline: PDF text extraction (PDFBox), cleaning (page numbers, headers, footers), section-aware chunking, embedding, storing in the persisted vector store. Provide an ingestion command or admin-only endpoint POST /ingest.
3. Write the chunking configuration design (what, why, trade-offs of small vs large chunks).
4. Document the vector store schema (fields, metadata, index/dimension, how persistence and re-indexing work, how to version an index).
5. Retrieval comparison: implement keyword search (MySQL FULLTEXT or equivalent), semantic search, and hybrid. Create evaluation/golden-questions.json with at least 20 questions answerable from the manual (examples: low-risk KYC update frequency, savings minimum balance penalty slabs, days overdue for NPA, FD premature withdrawal penalty, chargeback filing window, grievance escalation levels, dormant account rule, locker break-open conditions) plus 5 questions the manual cannot answer. Each entry: question, expected answer, expected section. Run all three retrieval modes on it and compare hit rate honestly.
6. Choose a similarity threshold using the golden set and explain the choice.
7. Write the hallucination risk analysis: probabilistic LLM behaviour, stale policy knowledge, wrong-chunk retrieval, rate and fee numbers that change, and how this design reduces each.
8. Save this prompt and its refinements in prompts/prompt-library.md.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/01-foundation-retrieval.md. Put the retrieval comparison summary and hallucination risk analysis inside it. The golden questions file, config and code are separate deliverables.

Do not build chat generation yet. Stop when done and list the files created.
```

## Prompt 2: End-to-End RAG Assistant (Use Case 2)

**Covers:** prompt templates, citations, guardrails, fallback, LangSmith tracing and evaluation. **Reads:** `docs/01-...`, golden questions, vector store **Doc produced:** `docs/02-rag-assistant.md`

```text
PROMPT 2 - END-TO-END RAG BANKING ASSISTANT

Role: You are a senior AI engineer building a safe, citation-based RAG assistant.

INPUT TO READ FIRST: docs/01-foundation-retrieval.md, evaluation/golden-questions.json, the retrieval code and chosen threshold from Prompt 1. Reuse them; if you must change a decision, tell me why.

INTENT: Generate grounded policy answers with citations, and reject unsafe or unsupported questions.

INSTRUCTIONS / CONSTRAINTS:
- Retrieve top_k chunks (configurable), pass context plus metadata to the LLM, answer ONLY from that context.
- Response shape for policy questions: answer, citations (document, section, page), confidence (high, medium, low), query_type = policy.
- Fallback: if no chunk passes the threshold, answer that there is insufficient context in the policy documents and do not guess.
- Query classifier with these classes: POLICY, ACCOUNT, CREDIT_CARD, LOAN, UNAUTHORIZED, REFUSE. Only POLICY is answered in this prompt; the data classes are routed in Prompt 3.
- Refuse operational requests (for example increase my credit limit now), unrelated questions, and anything asking the model to ignore its rules.
- Prompt-injection defence: strip or neutralise instructions found inside retrieved text, never reveal the system prompt, never output secrets.
- Endpoint: POST /chat/ask (open for development only, behind a config flag; Prompt 3 locks it with JWT).

TASKS:
1. Build prompt templates (system prompt, context block, citation format) and the generation flow.
2. Build the classifier and the guardrails with tests (at least 10 injection or unsafe examples).
3. Add LangSmith tracing so each request records retrieval, prompt and output. LangSmith has no first-party Java SDK, so use its REST API or OpenTelemetry integration; if that cannot be configured here, say so, implement the same trace schema in structured local logs, and give me exact steps to connect LangSmith. Do not claim LangSmith results you did not get.
4. Evaluate on the golden set: faithfulness (is every claim supported by the cited chunks), answer relevance, hallucination rate, latency. Save raw results to evaluation/results-prompt2.json and compare against the expected answers. Report real numbers only.
5. Show the abstain behaviour working on the 5 unanswerable questions.
6. Add the refined prompt versions to prompts/prompt-library.md.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/02-rag-assistant.md. Include the prompt and retriever configuration, guardrail list, trace evidence (or the honest LangSmith status), and the short evaluation summary inside it.

Stop when done and list the files created.
```

## Prompt 3: Secure Banking Data Integration (Use Case 3)

**Covers:** JWT authentication, tool calling, user scoping, masking, unauthorized access. **Reads:** `docs/02-...`, classifier and `/chat/ask` **Doc produced:** `docs/03-secure-data-integration.md`

```text
PROMPT 3 - SECURE BANKING DATA INTEGRATION

Role: You are a backend security engineer for a bank.

INPUT TO READ FIRST: docs/02-rag-assistant.md, the classifier, guardrails and /chat/ask from Prompt 2.

INTENT: Customers can ask about their own balance, cards, loans and transactions. This data must be fetched securely from the database and never generated by the AI.

INSTRUCTIONS / CONSTRAINTS:
- MySQL banking_db with Flyway migrations. Tables: users, accounts, cards, loans, transactions (include user_id foreign keys, DECIMAL money, timestamps). Seed 3 synthetic customers (for example CU45, CU46, CU105) with realistic fake data.
- JWT (HS256, 24h expiry), BCrypt password hashing, stateless sessions. Endpoints: POST /auth/register, POST /auth/login, GET /api/me/balance, /api/me/cards, /api/me/loans, /api/me/transactions.
- Tool calling: the LLM may choose a tool (get_balance, get_cards, get_loans, get_transactions) but tools take NO customer id from the LLM. The customer id comes only from the validated JWT. Use fixed, parameterised queries. Do not use free-form text-to-SQL; record this as a decision with reasons.
- Mask sensitive data in every response and log: account numbers show last 4 only, card numbers show last 4 only.
- Structured JSON responses: for accounts, customer id, accounts list (masked number, type, currency, balance), message, query_type. For cards, customer id, cards list (type, masked number, credit limit, available credit, status), message, query_type.
- Unauthorized access: if the user asks for another customer's data (for example Show loans of customer 105), block it, return a structured error (error, message, requested customer, authenticated customer, timestamp, query_type = unauthorized), and write an audit log entry with trace id.
- Lock /chat/ask with JWT and remove the open-development flag from Prompt 2.

TASKS:
1. Implement schema, seed data, auth, user-scoped APIs, tool-calling module, masking utility, audit log.
2. Route classified queries: POLICY goes to the RAG flow, ACCOUNT/CREDIT_CARD/LOAN goes to tools, UNAUTHORIZED is blocked, REFUSE is refused.
3. Write automated tests: JWT valid, expired and tampered; cross-customer access attempt; masking; SQL injection strings in the message; prompt-injection attempting to change the customer id.
4. Create the security validation checklist and mark each item pass or fail from real test runs.
5. Save example request and response pairs for all four test scenarios (policy query, balance, credit cards, unauthorized).
6. Update prompts/prompt-library.md.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/03-secure-data-integration.md. Include the JWT validation workflow, tool list, decision on tool calling vs text-to-SQL, structured response examples, and the security checklist inside it.

Stop when done and list the files created.
```

## Prompt 4: Optimization and Observability (Use Case 4)

**Covers:** token tracking, latency, caching, context size, cost, agent reasoning. **Reads:** `docs/03-...`, the working chat flow **Doc produced:** `docs/04-optimization-observability.md`

```text
PROMPT 4 - INTELLIGENCE MATURITY AND OPTIMIZATION

Role: You are a performance and cost engineer for an AI system.

INPUT TO READ FIRST: docs/03-secure-data-integration.md, docs/02-rag-assistant.md, evaluation/golden-questions.json and the full chat flow.

INTENT: Keep answer quality while controlling latency and cost, and make the system observable.

INSTRUCTIONS / CONSTRAINTS:
- Never cache user-specific data across customers. Cache policy answers only (key = normalised question + index version), or key any other cache by customer id with a short TTL.
- Do not invent model prices. Put price per 1K input and output tokens in config; use placeholder values clearly marked as placeholders and tell me where to put the real ones.
- Quality must not drop: re-run the golden set after every optimisation.

TASKS:
1. Measure the baseline on the golden set: tokens in and out per request, latency per stage (classify, retrieve, generate, tool call), cache hit rate (zero at start).
2. Add logging and metrics: token usage and latency per request (Micrometer plus /actuator endpoints), and a structured JSON audit line per chat request with type, trace id, customer id, intent, prompt tokens, duration, success.
3. Optimise: context size (fewer or shorter chunks, token budgeting), top_k and threshold tuning, policy-answer cache, skipping the LLM call when classification or guardrails already decide the outcome.
4. Re-run the golden set. Produce a before and after table for tokens, latency, cost estimate and quality (faithfulness, relevance).
5. Cost estimation: cost per query and an estimate for 1,000 and 100,000 queries per month, with the assumptions listed.
6. Write a short agent reasoning concept note: how a tool-calling agent decides between tools, where this system limits that freedom on purpose, and why.
7. Update prompts/prompt-library.md.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/04-optimization-observability.md. Put the metrics report, token usage summary, cost estimation, optimisation summary and the agent reasoning note inside it. Raw numbers go in observability/.

Stop when done and list the files created.
```

## Prompt 5: Frontend and Final Integration (Use Case 5)

**Covers:** UI, end-to-end validation, HLD and LLD, architecture diagrams, RAG evaluation summary, presentation. **Reads:** all earlier docs **Doc produced:** `docs/05-final-integration.md`

```text
PROMPT 5 - FINAL INTEGRATED BANKING RAG SYSTEM

Role: You are a solution architect and full-stack engineer finishing the system for an architecture review.

INPUT TO READ FIRST: docs/01 to docs/04, the backend code, evaluation/ and observability/ results.

INTENT: Deliver one cohesive, validated system with a user interface and review-ready documentation.

INSTRUCTIONS / CONSTRAINTS:
- Frontend: Angular. Use only endpoints that exist in the backend; list any API gap instead of inventing endpoints.
- Screens: login and token handling, dashboard (balance, cards, loans), policy query chat with citations and confidence, clear error messages for guardrail and unauthorized responses, loading and error states.
- Accessibility basics: labelled form fields, keyboard navigation, visible focus, errors announced to screen readers.
- Never store card numbers or passwords in the browser beyond the JWT; keep the token in memory or a secure choice you justify.

TASKS:
1. Build the Angular app and wire it to the backend.
2. Validate the full workflow with the four expected scenarios (policy query via RAG, authenticated balance, credit cards, unauthorized access) and record real request and response evidence.
3. Write deliverables under docs/deliverables/: HLD.md, LLD.md, architecture diagrams as Mermaid (system context, component, RAG sequence, JWT and tool-calling sequence, deployment view), RAG evaluation summary, performance and observability summary, and a presentation outline with one slide per use case.
4. Create a requirement-to-component traceability table (requirement, component, API, test).
5. List known limitations and risks honestly, including anything not verified.
6. Update prompts/prompt-library.md.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/05-final-integration.md. Put the scenario validation evidence, traceability table and limitations inside it. HLD, LLD, diagrams, evaluation summary, performance summary and presentation outline are the separate deliverables listed above.

Stop when done and list the files created.
```

## Prompt 6: CI/CD and Release Quality Gates (Use Case 6)

**Covers:** automated tests, evaluation gate, security and PII scans, deployment, rollback. **Reads:** all earlier docs and tests **Doc produced:** `docs/06-cicd-release.md`

```text
PROMPT 6 - FINAL SYSTEM WITH CI/CD

Role: You are a DevOps engineer for a regulated environment.

INPUT TO READ FIRST: docs/01 to docs/05, all test suites, evaluation/golden-questions.json, the build files.

INTENT: Releases happen only when tests, evaluation and security checks pass, with a safe way back.

INSTRUCTIONS / CONSTRAINTS:
- GitHub Actions (or Azure DevOps if I say so). No Docker or container images; build and deploy the Spring Boot jar and the Angular build output instead. Mention containerisation only as a future option.
- CI must not need real banking data or paid LLM calls by default: run RAG tests with a stub LLM and fixed retrieval fixtures, and run the live-LLM evaluation as a separate, secret-gated job.
- Secrets only in repository secrets, never in files. MySQL for CI: a service container is not allowed, so use H2 in MySQL mode for tests, or install MySQL on the runner.

TASKS:
1. Create .github/workflows/ci.yml with stages in this order: lint and compile, unit tests, API and JWT tests, retrieval tests, masking and unauthorized-access tests, evaluation gate, security scans, build, deploy.
2. Evaluation gate: run the golden set and fail the pipeline if faithfulness, relevance or hit rate fall below thresholds you store in evaluation/thresholds.json (justify each threshold from the Prompt 2 and Prompt 4 results).
3. Security scans: secret scanning (gitleaks or similar), dependency vulnerability check (OWASP dependency-check), and a PII masking test that fails if an unmasked account or card number appears in any response or log fixture.
4. Deployment: deploy only from main and only when every earlier stage passed. Provide a deploy script or workflow for a target I name (default: package artifacts and a documented manual deploy; optionally Azure App Service). Add a smoke test after deploy hitting /health, /auth/login and a policy question.
5. Rollback strategy: keep the previous artifact versions, document the exact rollback steps, and include index versioning so the vector index can be rolled back with the code.
6. Run the pipeline steps locally where possible and report which parts ran for real and which are untested. Do not claim a green pipeline you did not run.
7. Write a README.md at the repo root: what it is, how to run locally (env vars, MySQL, ingest, start backend and frontend), folder layout, links to docs/01 to docs/06.
8. Update prompts/prompt-library.md with the final refined version of all six prompts.

DOCUMENTATION: Follow the Documentation Rule. Write only docs/06-cicd-release.md. Include the pipeline stage table, thresholds and reasons, rollback steps, and final project summary (files per prompt and open risks) inside it. The workflow files, thresholds file and root README are the separate deliverables.

Stop when done and list the files created.
```

## How the chain connects

| Prompt | Reads | Produces | Hands off |
| --- | --- | --- | --- |
| 1 | Policy PDF | Ingestion, vector store, golden questions, `docs/01` | Retrieval layer, threshold, golden set |
| 2 | Retrieval, golden set | RAG chat, classifier, guardrails, evaluation, `docs/02` | Safe policy answers, routing classes |
| 3 | Classifier, chat | MySQL schema, JWT, tool calling, masking, tests, `docs/03` | Secure data access, locked endpoints |
| 4 | Full chat flow | Metrics, cache, cost report, `docs/04` | Optimised, observable system |
| 5 | Everything | Angular UI, HLD, LLD, diagrams, `docs/05` | Validated system, test suites |
| 6 | Everything | CI/CD, gates, rollback, README, `docs/06` | Release process |

## Final deliverables checklist

- **Use Case 1:** ingestion pipeline, chunking design, embedding module, vector schema, retrieval comparison, hallucination risk analysis (`docs/01`)
- **Use Case 2:** RAG pipeline with tracing, prompt and retriever config, citations, guardrails and fallback, evaluation summary (`docs/02`)
- **Use Case 3:** secure API, JWT workflow, tool-calling module, response examples, security checklist (`docs/03`)
- **Use Case 4:** metrics report, token summary, cost estimate, optimisation summary, agent reasoning note (`docs/04`)
- **Use Case 5:** working system, HLD and LLD, diagrams, evaluation and performance summaries, presentation outline (`docs/05`)
- **Use Case 6:** CI/CD pipeline with gates, security and PII scans, rollback plan, README (`docs/06`)

## Tips for running

- Run one prompt per chat turn and review the plan before approving edits. Commit to git after each prompt.
- If the agent creates extra summary files, remind it of the one-doc rule and ask it to delete them.
- If a chat gets long, start a new one and tell the agent to read the `docs/` folder first.
- Keep screenshots of test runs, evaluation results, and the pipeline: they are your proof for evaluation.
- Be ready to explain one thing the AI got wrong in each use case and how you caught it.
