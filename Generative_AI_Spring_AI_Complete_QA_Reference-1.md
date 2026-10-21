# Generative AI & Spring AI — Complete Questions and Answers

This single Markdown reference includes explanations and answers for the learning roadmap, all 12 phases, an end-to-end architecture, a recommended learning order, and a glossary. Framework APIs change over time; verify code and configuration against the exact Spring AI version used in your project.

## Table of Contents
1. [AI and Generative AI](#phase-1--ai-and-generative-ai)
2. [LLM Architecture](#phase-2--llm-architecture)
3. [Prompt Engineering](#phase-3--prompt-engineering)
4. [Spring AI Fundamentals](#phase-4--spring-ai-fundamentals)
5. [Embeddings](#phase-5--embeddings)
6. [Vector Databases](#phase-6--vector-databases)
7. [RAG](#phase-7--retrieval-augmented-generation-rag)
8. [Tool Calling](#phase-8--tool-calling)
9. [Agents](#phase-9--ai-agents)
10. [MCP](#phase-10--model-context-protocol-mcp)
11. [Spring AI + MCP](#phase-11--spring-ai--mcp)
12. [Production GenAI](#phase-12--production-genai)
13. [Integration Architecture](#end-to-end-integration-architecture)
14. [Learning Order](#recommended-learning-order)
15. [Glossary](#glossary)

---

# Phase 1 — AI and Generative AI

### Q1. What is Artificial Intelligence (AI)?
**Answer:** AI is the broad field of building systems that perform tasks associated with intelligence, such as understanding language, recognizing patterns, making predictions, planning, and solving problems. It includes rule-based systems, machine learning, deep learning, and generative models.

### Q2. What is the difference between AI, ML, deep learning, and Generative AI?
**Answer:** AI is the umbrella field. Machine Learning (ML) learns patterns from data. Deep Learning uses multi-layer neural networks. Generative AI creates new content such as text, images, audio, video, or code. A common hierarchy is AI ⊃ ML ⊃ deep learning; Generative AI is a capability that often uses deep learning.

### Q3. How has AI evolved?
**Answer:** Broad stages include explicit rule-based systems, statistical machine learning, deep neural networks, pretrained foundation models, multimodal generative models, and systems that use tools to complete workflows. These approaches coexist; new techniques do not make traditional rules or ML obsolete.

### Q4. What is a rule-based system?
**Answer:** It applies explicit human-written conditions, such as `IF balance < 0 THEN flag account`. Rules are transparent and useful when policy is precise. ML learns patterns from examples and is useful when rules are difficult to write manually. Production systems often combine both.

### Q5. What is Generative AI?
**Answer:** Generative AI produces new outputs based on patterns learned during training. An LLM generates text token by token, a diffusion model can generate images by denoising, and an audio model can synthesize speech. Generated content is not automatically true or safe.

### Q6. What are the main types of generative models?
**Answer:** Autoregressive models generate sequences step by step; diffusion models iteratively denoise; GANs train a generator against a discriminator; VAEs learn latent representations and generate samples; specialized audio/video models generate or transform sound, speech, music, frames, or video.

### Q7. What can Generative AI create?
**Answer:** Text, summaries, translations, code, images, speech, music, transcripts, video, synthetic data, structured JSON, and multimodal outputs. Capabilities depend on the selected model.

### Q8. What is a foundation model?
**Answer:** A foundation model is trained on broad data and can be adapted to many tasks. It may be used with prompts, connected to external knowledge with RAG, fine-tuned, or integrated with tools.

### Q9. What is the difference between pretrained and fine-tuned models?
**Answer:** A pretrained model has learned general patterns during its original training. A fine-tuned model receives additional training to adjust behavior for a narrower task or domain. RAG usually supplies external information at inference time without changing model parameters.

### Q10. Open-weight versus closed models?
**Answer:** Open-weight models make weights available under a license that may permit local deployment or modification. Closed hosted models expose capabilities through a provider API without publishing weights. Check licensing, redistribution restrictions, data terms, and hardware needs; “open source” is not identical to “weights downloadable.”

### Q11. Local versus cloud models?
**Answer:** Local models run on hardware you control and can improve data locality or offline use, but require compute and maintenance. Cloud models are easier to scale and operate but depend on network/provider availability and introduce cost and data-handling considerations.

### Q12. Where is Generative AI used?
**Answer:** Customer support, enterprise knowledge search, document processing, report drafting, code assistance, tutoring, semantic search, meeting summaries, data extraction, and controlled workflow automation. Human review is important for consequential decisions.

---

# Phase 2 — LLM Architecture

### Q13. What is a Large Language Model (LLM)?
**Answer:** An LLM is a neural network trained on large amounts of data to model token sequences. During generation it predicts a likely next token from the context, selects a token, and repeats until a stop condition. It can summarize, translate, answer questions, extract data, and generate code.

### Q14. What are examples of LLM families?
**Answer:** GPT, Gemini, Claude, and Llama are examples. Models differ in quality, supported modalities, context limits, tool interfaces, licensing, cost, latency, and deployment options. Compare specific models using current documentation and your own evaluations.

### Q15. Does an LLM search the internet by default?
**Answer:** No. A model normally uses learned parameters and the supplied context. Live web access requires a browsing or retrieval integration. Without evidence, a model may generate a plausible but incorrect answer.

### Q16. What is a token?
**Answer:** A token is a unit used by a tokenizer. It may be a whole word, part of a word, punctuation, or another text fragment. Tokens are not the same as words; counts depend on tokenizer and language.

### Q17. What is tokenization?
**Answer:** Tokenization converts text into token IDs that a model can process. The model processes IDs and its generated IDs are decoded back into readable text.

### Q18. What are input tokens, output tokens, and context tokens?
**Answer:** Input tokens are prompt/context content sent to the model; output tokens are generated tokens; the context window limits the amount of context and generation a model can process. Pricing often distinguishes input and output usage.

### Q19. How do tokens affect cost?
**Answer:** Hosted APIs often charge based on input/output token counts and model tier. Long history, large retrieved passages, and verbose output increase cost. Track usage, remove irrelevant context, retrieve only useful evidence, and select a model that meets quality needs.

### Q20. What is a context window?
**Answer:** It is the model's token limit for the request context and generation, according to its API rules. Context may include instructions, history, user input, retrieved documents, and tool results. A larger context does not guarantee perfect recall of every detail.

### Q21. What is context truncation?
**Answer:** It happens when context exceeds a limit or an application intentionally removes older content. Important details may be lost. Manage history deliberately, summarize old turns when useful, and retrieve relevant passages rather than sending an entire corpus.

### Q22. What is a Transformer?
**Answer:** A Transformer is a neural-network architecture built around attention. It learns relationships between tokens and supports parallel training more effectively than recurrent sequence models, making it central to many modern LLMs.

### Q23. What is self-attention?
**Answer:** Self-attention lets each token representation weigh information from other tokens. In simplified terms, query, key, and value projections are used to calculate compatibility scores and combine information. This helps the model use surrounding context.

### Q24. What is multi-head attention?
**Answer:** It runs several attention operations in parallel. Different heads can learn different relationships; their outputs are combined to produce a richer representation.

### Q25. What is positional encoding?
**Answer:** Attention does not inherently encode sequence order, so Transformers incorporate positional information through encodings, embeddings, or architecture-specific methods. This helps distinguish sequences in which token order changes meaning.

### Q26. Encoder versus decoder architecture?
**Answer:** Encoders typically build input representations with access to context on both sides. Decoders generate autoregressively and commonly use causal attention so a token cannot see future tokens. Encoder-only models are useful for representation tasks, decoder-only models for generation, and encoder-decoder models for input-to-output transformations.

### Q27. Why are Transformers often preferred over RNNs/LSTMs?
**Answer:** RNNs process sequences step by step and can struggle with long-range dependencies and parallel training. Transformers allow parallel training and connect distant tokens through attention. RNNs/LSTMs still have suitable use cases.

### Q28. What is temperature?
**Answer:** Temperature changes the sharpness of the next-token probability distribution. Lower values generally produce more predictable text; higher values increase variety and may increase inconsistency. Temperature does not add knowledge or guarantee accuracy.

### Q29. What are Top-K and Top-P?
**Answer:** Top-K restricts sampling to the K most likely tokens. Top-P selects from the smallest set of likely tokens whose cumulative probability reaches a threshold. They control diversity differently; API support and behavior vary.

### Q30. What are frequency and presence penalties?
**Answer:** Where supported, frequency penalties discourage tokens according to how often they appeared; presence penalties discourage tokens that appeared at least once. They can reduce repetition but may make text unnatural if poorly tuned.

### Q31. What are stop sequences and max output tokens?
**Answer:** A stop sequence ends generation when a configured sequence appears. A maximum output-token limit caps response length. These can control cost and format, but low limits may cut off useful or structured output.

### Q32. What is an LLM hallucination?
**Answer:** It is plausible-sounding content that is unsupported, false, or fabricated. Mitigations include RAG, tool verification, structured output, evaluation, refusal when evidence is missing, and human review. No single technique eliminates all hallucinations.

### Q33. What are other LLM limitations?
**Answer:** Bias, stale knowledge, privacy risks, prompt injection, inconsistent outputs, latency, cost, context limits, and sensitivity to wording. Treat outputs as unverified until checked.

### Q34. What happens in a typical LLM request?
**Answer:** The app constructs the request; text is tokenized; token representations pass through model layers; the model predicts the next-token distribution; decoding selects tokens repeatedly; generation stops at a limit or stop condition; and the API returns the result. The internal token embedding layer is distinct from a separate text-embedding model used for vector search.

---

# Phase 3 — Prompt Engineering

### Q35. What is prompt engineering?
**Answer:** It is the practice of designing instructions and context so a model produces useful, safe, correctly formatted results. It includes task definition, constraints, examples, context selection, schemas, and evaluation.

### Q36. What are system, user, and assistant messages?
**Answer:** System/developer instructions define role and constraints; user messages contain requests and supplied information; assistant messages contain responses or tool-call requests. Keep trusted instructions separate from untrusted user/document content.

### Q37. What is context injection?
**Answer:** It is adding relevant material such as retrieved passages, database results, or conversation summaries to a prompt. Delimit this material clearly and treat untrusted content as data, not trusted instructions.

### Q38. Zero-shot, one-shot, and few-shot prompting?
**Answer:** Zero-shot provides instructions without examples; one-shot includes one example; few-shot includes several. Examples help when the desired format or classification boundary is difficult to explain.

### Q39. What is chain-of-thought prompting?
**Answer:** It is a technique intended to encourage multi-step problem solving. In applications, request concise rationales or verifiable intermediate results when useful, but do not treat a model's explanation as proof. Verify important conclusions independently.

### Q40. What is self-consistency?
**Answer:** It generates multiple candidate solutions and compares them, for example by selecting a common answer or using a verifier. It can improve some tasks but adds cost and does not guarantee correctness.

### Q41. What is ReAct prompting?
**Answer:** ReAct combines reasoning-oriented planning with actions such as searching or calculating, then uses observations to continue. Implement actions through explicit, permissioned tool interfaces rather than unrestricted access.

### Q42. What is Tree of Thoughts?
**Answer:** It explores multiple candidate paths instead of a single linear path. It may help with planning/search tasks but costs more and requires a way to score or verify candidates.

### Q43. What is structured output?
**Answer:** Structured output constrains results to formats such as JSON or typed objects. Validate required fields, types, ranges, and business rules: valid JSON does not mean correct content.

### Q44. Why use a schema?
**Answer:** A schema defines field names, data types, required values, and constraints. It reduces parsing ambiguity and supports validation. Use provider constrained-output features where available, plus server-side checks.

### Q45. What is a prompt template?
**Answer:** A reusable prompt pattern with variable values, such as `Summarize {document} in {max_points} bullets`. Templates improve consistency and testability. Never let untrusted data impersonate trusted system instructions.

### Q46. What is prompt chaining?
**Answer:** It breaks a task into stages, such as extracting facts, checking them, then drafting a summary. Each stage should have a clear input/output contract. Chaining improves debuggability but adds latency and can propagate errors.

### Q47. What is prompt compression?
**Answer:** It removes redundant wording, irrelevant history, and unnecessary context while preserving key information. Use summaries and selective retrieval, but never remove essential evidence or safety constraints merely to save tokens.

### Q48. How should prompts be evaluated?
**Answer:** Create representative tests, define success criteria, include edge/adversarial cases, measure relevance, factuality, format validity, safety, latency, and cost, compare versions, and run regression tests after changes. Use human review for subjective or high-impact outcomes.

---

# Phase 4 — Spring AI Fundamentals

### Q49. What is Spring AI?
**Answer:** Spring AI is a Spring ecosystem project for building AI-enabled Java applications. It offers abstractions and integrations for chat models, embedding models, vector stores, retrieval, tool calling, and MCP features. Use documentation matching your actual library release.

### Q50. Why use Spring AI instead of direct provider HTTP calls?
**Answer:** Spring AI provides reusable abstractions, Spring Boot configuration, prompt/response types, and provider integrations. Direct HTTP may be better for a provider-specific feature not exposed by the library or when full control is needed.

### Q51. What are ChatModel and ChatClient?
**Answer:** `ChatModel` is a lower-level abstraction for calling a chat model. `ChatClient` is a fluent API for composing prompts, configuring options, invoking a model, and extracting responses. Exact APIs vary by release.

### Q52. What are Prompt and PromptTemplate?
**Answer:** A `Prompt` represents the messages and options sent to a model. A prompt template is reusable content with variables, improving consistency and testability.

### Q53. What is streaming?
**Answer:** Streaming sends partial output as it is generated instead of waiting for the full answer. It reduces perceived latency, but clients must handle partial chunks, cancellation, errors, timeouts, and safe rendering.

### Q54. How can a response be consumed?
**Answer:** It can be read as text, mapped to a Java class/record, or processed as a stream. Validate structured data before using it, especially when it influences business logic.

### Q55. What are Spring AI Advisors?
**Answer:** Advisors participate in or wrap model-call workflows to add reusable behavior, such as retrieval, memory, or logging, depending on the implementation. Understand advisor ordering and side effects and test them.

### Q56. What is chat memory?
**Answer:** Chat memory stores selected previous messages or summaries so later turns can use context. Stored memory must be loaded into the request; it is not automatically the same as a model's context window.

### Q57. What are common chat-memory storage options?
**Answer:** In-memory storage is simple but usually not durable or shared across instances. JDBC/database storage provides persistence. Redis can provide low-latency shared state. Choose based on durability, privacy, scale, retention, and consistency needs.

### Q58. What are the main Spring AI components?
**Answer:** `ChatClient`/`ChatModel` for chat; `EmbeddingModel` for vectors; `VectorStore` for vector retrieval; tool calling for controlled functions; advisors and memory abstractions for reusable context; and MCP integrations where supported.

### Q59. What is a typical Spring AI request flow?
**Answer:** Validate the incoming request, construct a prompt, apply advisors/retrieval, invoke the model, process and validate the response, and return it. A tool-enabled flow may include tool selection, backend execution, and another model call.

### Q60. How should Spring AI be configured?
**Answer:** Add the starter for the chosen provider/features, configure credentials through environment variables or a secret manager, externalize model settings, and inject the required abstraction. Never commit API keys. Align Spring AI and Spring Boot dependencies and use version-specific docs.

---

# Phase 5 — Embeddings

### Q61. What is an embedding?
**Answer:** An embedding is a numerical vector representing content such as text. Semantically related texts may be close under a selected similarity metric. Embeddings support semantic search, clustering, recommendations, and similarity comparison.

### Q62. Why use embeddings instead of only keywords?
**Answer:** Keyword search is strong for exact terms but may miss synonyms or paraphrases. Embeddings can find conceptually related content with different wording. Hybrid search combines lexical and semantic methods.

### Q63. How does text become an embedding?
**Answer:** An embedding model processes text and returns a fixed-length vector. Dimensions are learned features and usually do not map directly to human concepts. Use a compatible model and preprocessing for document and query vectors.

### Q64. What is embedding dimension?
**Answer:** It is the number of numeric values in a vector, for example 768 or another model-specific length. Bigger is not automatically better. The vector index dimension must match the model output.

### Q65. What is cosine similarity?
**Answer:** Cosine similarity compares vector directions: `cos(A,B) = (A · B) / (||A|| × ||B||)`. It is common for text retrieval. Interpret scores according to the model and implementation.

### Q66. What is Euclidean distance?
**Answer:** It is the straight-line distance between vectors: `sqrt(sum((A_i - B_i)^2))`. Smaller distances indicate closer vectors for this metric. It can rank items differently from cosine similarity.

### Q67. What is dot-product similarity?
**Answer:** It is `sum(A_i * B_i)`. Its meaning depends on vector magnitude and model training. Use the metric recommended for the embedding model.

### Q68. Which similarity metric should be selected?
**Answer:** Use the metric recommended by the embedding model and supported by the database index, then evaluate retrieval quality on real queries. Do not assume that a higher score is always better without understanding whether the system returns similarity or distance.

### Q69. What is the embedding lifecycle?
**Answer:** Clean content, split long documents, generate a vector per chunk, store vectors with text and metadata, embed incoming queries with a compatible model, search the index, rank matches, and pass relevant passages and source identifiers to the application.

### Q70. What is Spring AI's EmbeddingModel?
**Answer:** It abstracts embedding generation behind a Spring-managed component. Provider-specific dimensions, batching limits, request types, and configuration depend on the implementation and library version.

### Q71. Can document and query embeddings use different models?
**Answer:** Only when the models are explicitly designed to share a compatible embedding space, such as paired query/document encoders. Arbitrarily mixing unrelated embeddings makes similarity results unreliable.

---

# Phase 6 — Vector Databases

### Q72. What is a vector database?
**Answer:** It stores vectors and supports similarity search, often with text and metadata. It may offer approximate-nearest-neighbor indexing, filtering, persistence, and scaling. Vector search can also be supported by relational and search databases.

### Q73. Keyword search versus semantic search?
**Answer:** Keyword search matches lexical terms and is often good for exact identifiers. Semantic search compares embedding meaning and can handle paraphrases. Hybrid search uses both to improve coverage.

### Q74. What is approximate nearest-neighbor (ANN) search?
**Answer:** ANN retrieves vectors likely to be among the nearest without exhaustively comparing every stored vector. It trades some exactness for speed and scale. Tune it using recall, latency, and memory measurements.

### Q75. What are examples of vector-search technologies?
**Answer:** pgvector for PostgreSQL, Pinecone, Weaviate, Milvus, Chroma, Qdrant, and Elasticsearch vector-search features are examples. Compare operations, filtering, scaling, deployment, security, and cost.

### Q76. What is HNSW?
**Answer:** Hierarchical Navigable Small World is a graph-based ANN index. It often provides fast retrieval but trades off memory, index-build time, and search settings.

### Q77. What are IVF and Flat indexes?
**Answer:** Flat/exact search compares against all or most vectors and is simple but expensive at scale. IVF partitions vectors into clusters and searches selected partitions. HNSW navigates a graph to find nearby vectors. Availability and tuning depend on the database.

### Q78. What is Spring AI's VectorStore?
**Answer:** It is an abstraction for storing documents with embeddings and metadata and performing similarity search. It helps reduce dependence on one vendor, although provider-specific features may still require custom configuration.

### Q79. What metadata should be stored?
**Answer:** Source ID, title, page/section, chunk number, timestamp, version, tenant, access labels, and content type can be useful. Metadata supports citations, filtering, freshness, and authorization. Avoid unnecessary sensitive data.

### Q80. What is a retrieval lifecycle?
**Answer:** Embed the query, search the index, apply authorization and metadata filters, retrieve candidates, rerank or filter them, and return a bounded set with source identifiers.

---

# Phase 7 — Retrieval-Augmented Generation (RAG)

### Q81. What is RAG?
**Answer:** Retrieval-Augmented Generation finds relevant external content and supplies it to an LLM as context. It can ground answers in current, private, or domain-specific sources without retraining the model. It can still fail if retrieval is poor or the model misinterprets the evidence.

### Q82. Why use RAG?
**Answer:** RAG is useful for frequently changing, private, large, or specialized knowledge. It can provide source citations and access-aware retrieval. It does not guarantee truth: sources must be relevant, current, authorized, and interpreted correctly.

### Q83. RAG versus fine-tuning?
**Answer:** RAG supplies information at inference time and can be updated by re-indexing data. Fine-tuning changes model parameters to improve behavior or task performance. They can be combined, but frequently changing facts are often better supplied through retrieval.

### Q84. What are the main parts of RAG?
**Answer:** Ingestion, extraction, chunking, embedding, indexing, query processing, retrieval, optional reranking, prompt augmentation, generation, and citation/answer validation. Production RAG also needs access control, evaluation, monitoring, and refresh workflows.

### Q85. What is a document ingestion pipeline?
**Answer:** Import files/records, extract and clean text, split into chunks, attach metadata, generate embeddings, and store records. Make ingestion repeatable, observable, and safe to rerun without uncontrolled duplicates.

### Q86. Why is chunking necessary?
**Answer:** Documents may exceed embedding or prompt limits. Chunks let the system retrieve relevant sections. Poor boundaries can separate facts from their context or split a question from its answer.

### Q87. Fixed-size, recursive, and semantic chunking?
**Answer:** Fixed-size splitting uses a target size; recursive splitting prefers boundaries such as headings, paragraphs, and sentences; semantic splitting tries to detect topic changes. Choose based on document structure and evaluation results.

### Q88. What are chunk size and overlap?
**Answer:** Chunk size controls the amount of text in each unit. Overlap repeats some content across neighboring chunks to preserve boundary context. Small chunks may lack context; large chunks may dilute relevance and consume tokens. Tune with real queries.

### Q89. What is Top-K retrieval?
**Answer:** Top-K returns the K highest-ranked candidates. Higher K may improve coverage but also add noise and cost. Tune it alongside thresholds, chunking, reranking, and model context limits.

### Q90. What is a score threshold?
**Answer:** It excludes results that do not meet a relevance criterion. Direction and scale depend on whether the system returns similarity or distance. Calibrate it using representative data; overly strict thresholds may hide useful evidence.

### Q91. What is reranking?
**Answer:** Reranking scores an initial candidate set more carefully, often using a cross-encoder or another model, and reorders results. It may improve precision but adds latency and cost and cannot recover documents missed during initial retrieval.

### Q92. Why is metadata filtering important?
**Answer:** It limits results by date, version, category, tenant, or access rights. Enforce access control in trusted backend logic before content reaches the model; do not rely on the prompt to enforce permissions.

### Q93. What is prompt augmentation in RAG?
**Answer:** It adds retrieved passages and instructions to the generation request. Clearly separate the question from the source text, request evidence-based answers, and define what to do when evidence is insufficient. Retrieved documents are untrusted and may contain prompt injection.

### Q94. What should the system do when evidence is missing?
**Answer:** State that available sources do not establish the answer, ask a clarifying question, or use an approved fallback. Do not invent missing details. Use retrieval quality and source agreement as signals, not absolute proof.

### Q95. What is basic RAG?
**Answer:** Embed the question, retrieve nearest chunks, add them to a prompt, and generate an answer. It is simple but can struggle with exact keywords, multi-hop questions, and long documents.

### Q96. What is advanced RAG?
**Answer:** It may add query rewriting, hybrid search, metadata filtering, parent-child retrieval, reranking, contextual compression, and citation validation. Add complexity to solve measured problems, not merely because a technique exists.

### Q97. What is hybrid search?
**Answer:** It combines lexical retrieval, such as BM25, with vector similarity. Results can be fused or reranked. It is useful when queries mix meaning with exact product IDs, names, or error codes.

### Q98. What are query transformation and expansion?
**Answer:** Transformation rewrites a query for search; expansion adds related terms or alternatives. Multi-query retrieval searches several interpretations and combines results. These can improve recall but may introduce drift, noise, and cost.

### Q99. What is parent-child retrieval?
**Answer:** Small child chunks are indexed for precise matching, while a larger parent section is returned for generation. This balances precise retrieval with sufficient context.

### Q100. What is Graph RAG?
**Answer:** Graph RAG uses entities and relationships represented as a graph to answer questions that require connecting information across sources. Graph creation, entity resolution, freshness, and query complexity add operational overhead.

### Q101. What is Agentic RAG?
**Answer:** It lets a model-driven workflow decide when to search, rewrite a query, use tools, or retrieve again. It helps with multi-step questions but is harder to control than fixed RAG. Bound iterations, tools, time, and cost.

### Q102. Which Spring AI components support RAG?
**Answer:** Depending on release, Spring AI provides document readers/abstractions, text splitters, `EmbeddingModel`, `VectorStore`, and retrieval Advisors or utilities. The application defines data sources, security, prompt policy, and answer behavior.

### Q103. What is the RAG ingestion flow?
**Answer:** Load documents, extract/normalize text, split into chunks, attach source/access metadata, generate embeddings, store vectors and content, then re-index changed/deleted documents using a versioning strategy.

### Q104. What is the RAG query flow?
**Answer:** Authenticate; validate/rewrite the question; apply authorization filters; embed and retrieve candidates; rerank/filter; construct a prompt with passages and source IDs; generate; validate answer and citations; return answer/sources; record metrics.

### Q105. How is RAG optimized?
**Answer:** Measure retrieval recall/precision, groundedness, relevance, citation correctness, latency, and cost. Tune chunking, embedding model, hybrid weights, Top-K, filters, reranking, and prompts. Keep a representative evaluation set and compare changes to a baseline.

---

# Phase 8 — Tool Calling

### Q106. What is tool calling?
**Answer:** Tool calling lets a model request that the application execute a defined function, such as searching a knowledge base, checking an order, or calculating a value. The model proposes the tool and arguments; application code validates and executes it.

### Q107. Why is tool calling needed?
**Answer:** A model cannot reliably know live private system state or perform external operations merely by generating text. Tools provide controlled access to APIs, databases, search, and calculations. The backend remains responsible for authorization and validation.

### Q108. What is a tool schema?
**Answer:** It describes a tool's name, purpose, input fields, types, and constraints. Clear schemas help tool selection, but the backend must validate every argument because model-generated calls are untrusted input.

### Q109. What is tool discovery?
**Answer:** The application exposes a set of permitted tools to the model/orchestrator. The model selects from descriptions. In dynamic systems, filter available tools by identity, task, environment, and risk.

### Q110. What is the tool-calling lifecycle?
**Answer:** The app sends a prompt and tool definitions; the model may request a tool; the backend validates its name and arguments; authorization and business rules are checked; the function runs; the result is returned to the model or rendered; and the final response is validated.

### Q111. What tools can an application expose?
**Answer:** Read-only search, calculators, approved REST APIs, safe database-backed service methods, order status, ticket lookup, and other business operations. Avoid unrestricted shell, arbitrary SQL, or broad filesystem access unless an isolated and authorized design explicitly requires it.

### Q112. What is the difference between a tool call and a final answer?
**Answer:** A tool call is a request to execute an operation, not proof it succeeded. The final answer must reflect the actual tool result. If execution fails, the application must handle the error rather than claim success.

### Q113. How does Spring AI support tool calling?
**Answer:** Supported Spring AI versions can expose application methods as tools, including annotation-based patterns such as `@Tool`. The model requests a tool, application infrastructure invokes the registered method, and the result can be sent back to the model. Confirm exact imports, registration, and configuration in your chosen release.

**Conceptual example (registration/configuration omitted; verify version-specific APIs):**
```java
@Component
public class CalculatorTools {
    // In supported versions, register this method using the appropriate @Tool annotation.
    public double add(double a, double b) {
        return a + b;
    }
}
```

### Q114. How should database tools be designed?
**Answer:** Expose narrow methods such as `getOrderStatus(orderId)` rather than unrestricted SQL. Validate identifiers, enforce tenant authorization, use parameterized queries, limit returned fields, and log sensitive operations.

### Q115. How should side-effecting tools be protected?
**Answer:** Require authorization, validate arguments, apply business rules, use idempotency keys where appropriate, set timeouts, and audit operations. For consequential actions such as deleting data or sending money, require explicit confirmation or approval when appropriate.

### Q116. How should tool errors be handled?
**Answer:** Return structured, bounded errors without secrets or stack traces. Distinguish validation, authorization, timeout, and dependency failures. Bound retries and ensure retrying a non-idempotent action cannot duplicate the side effect.

---

# Phase 9 — AI Agents

### Q117. What is an AI agent?
**Answer:** An agent is a system that uses a model and surrounding software to pursue a goal through multiple steps, often choosing tools, observing results, and adjusting its plan. The term is broad; not every multi-step workflow needs an autonomous agent.

### Q118. What are the main agent components?
**Answer:** Model, instructions, state/memory, tools, planning/orchestration, observations, and stop conditions. Production agents also need authorization, budgets, timeouts, tracing, evaluation, and safe failure behavior.

### Q119. What is an agent lifecycle?
**Answer:** Receive a goal, inspect context, choose a step, call a tool or generate an intermediate result, observe the outcome, update state, and continue or finish. Explicit iteration limits and termination criteria prevent runaway loops.

### Q120. What is ReAct as an agent pattern?
**Answer:** It alternates decisions with tool actions and incorporates tool observations. It is useful when the next step depends on results. Only approved actions should be available, and every call should be validated.

### Q121. What is Plan-and-Execute?
**Answer:** The system creates a plan, executes steps, and may revise the plan after observations. The plan can be wrong or stale, so validate prerequisites and re-plan when actual results differ.

### Q122. What are reflection and self-correction?
**Answer:** The system reviews an earlier result against criteria and attempts to fix defects. This can improve quality but can also repeat errors or consume excessive tokens. Use objective checks and bounded retries.

### Q123. Short-term versus long-term agent memory?
**Answer:** Short-term memory holds current task/conversation state. Long-term memory persists across sessions in a database or vector store. Persistent memory needs retention, access, correction/deletion, and protection against storing untrusted instructions as facts.

### Q124. What is a multi-agent system?
**Answer:** Multiple model-driven components have roles such as planner, researcher, executor, and reviewer. This can parallelize work but adds communication, failure modes, latency, and cost. A single workflow is often simpler.

### Q125. What is Agentic RAG?
**Answer:** It combines retrieval with a decision loop that may rewrite a query, retrieve evidence, inspect gaps, use tools, or search again. Use it for genuinely multi-step retrieval; fixed RAG is usually preferable for straightforward questions.

### Q126. How can Spring AI support agent workflows?
**Answer:** Spring AI provides model interactions, tool calling, memory-related abstractions, and integration points that can be orchestrated into workflows. The application defines state, permissions, iteration limits, and failure handling; features vary by version.

### Q127. When should you avoid autonomous agents?
**Answer:** Prefer deterministic workflows when steps are known, outcomes must be reproducible, actions are high-risk, or strict cost/latency limits apply. Agents are useful when choices depend on uncertain intermediate results and the action space can be safely bounded.

---

# Phase 10 — Model Context Protocol (MCP)

### Q128. What is MCP?
**Answer:** The Model Context Protocol is a protocol for connecting AI applications to external tools and context providers through a standardized interface. It helps separate the AI host from integrations exposing tools, resources, and prompts. MCP itself does not make a capability safe or trustworthy.

### Q129. Why use MCP?
**Answer:** Without a common protocol, each AI application may need a custom integration for every external system. MCP provides reusable patterns for discovery and communication. Authentication, authorization, and secure deployment still have to be designed.

### Q130. MCP versus direct tool integration?
**Answer:** A direct integration is custom to an application. MCP can expose a capability through a standard interface for multiple compatible clients. Direct calls may be simpler for one application; MCP is attractive for reusable integrations and interoperability.

### Q131. What is an MCP host?
**Answer:** The host is the AI application coordinating model interactions and MCP connections. It determines which servers and capabilities are available to a session.

### Q132. What is an MCP client?
**Answer:** It is the component inside a host that communicates with an MCP server and performs protocol operations such as capability discovery and tool/resource requests.

### Q133. What is an MCP server?
**Answer:** A service that exposes tools, resources, and/or prompts to clients. It must enforce its own validation and access controls.

### Q134. What is an MCP tool?
**Answer:** An executable operation with a name, description, and input schema. The model may select it, but the server validates input and enforces authorization and business rules.

### Q135. What is an MCP resource?
**Answer:** A resource exposes information such as document or application content. Resource access must respect authorization; not all resources are public or safe to place in prompts.

### Q136. What is an MCP prompt?
**Answer:** A reusable prompt template exposed by a server, potentially with arguments. The host decides how to combine it with its own policy.

### Q137. What is capability discovery?
**Answer:** A client connects and discovers which tools, resources, or prompts a server offers. Discovery is not authorization: a capability must still be permitted for the current user and task.

### Q138. What is the MCP tool-execution flow?
**Answer:** The host connects to a configured server, discovers capabilities, selects allowed tools, requests a tool, sends structured input, and receives a result/error. The host then decides how to use or present that result.

### Q139. What transports does MCP use?
**Answer:** MCP defines protocol messages and supports transports including standard input/output for local processes and HTTP-based patterns for remote servers, depending on protocol version. Consult the current specification because transport details evolve.

### Q140. What security controls should MCP servers implement?
**Answer:** Authentication where required, per-tool/resource authorization, least privilege, validation, output minimization, secret management, secure transport, timeouts, rate limits, audit logs, and prompt-injection defenses. Never trust a caller solely because it uses MCP.

### Q141. Can MCP tools be dangerous?
**Answer:** Yes. Tools may read confidential data, change records, send messages, or invoke external systems. Expose narrow operations, scope credentials, require approval for high-impact actions, and audit side effects.

### Q142. Tool versus resource versus prompt in MCP?
**Answer:** A tool performs an operation; a resource provides information; a prompt provides a reusable model-interaction template.

---

# Phase 11 — Spring AI + MCP

### Q143. How do Spring AI and MCP work together?
**Answer:** Spring AI can be used to build Java applications that consume MCP servers or expose MCP capabilities where supported by the chosen release. A Spring app can connect to external tools or publish selected service operations through an MCP server. Exact starters and properties are version-dependent.

### Q144. What is an MCP client in Spring AI?
**Answer:** It connects the Spring application to configured MCP servers, discovers capabilities, and invokes permitted operations. The application should not blindly expose every discovered tool to every user.

### Q145. What is an MCP server in Spring?
**Answer:** It is a Spring application configured to expose approved tools, resources, and/or prompts. It should still use Spring Security, service-layer authorization, validation, and observability.

### Q146. How should a Spring application expose a tool?
**Answer:** Define a narrow method with clear description and structured input, register it using the version-supported mechanism, and delegate to an existing service-layer method. Validate input and enforce authorization on the server.

### Q147. How should resources be exposed?
**Answer:** Register resources through the applicable MCP server APIs and return only authorized content. Use stable identifiers, control content size, avoid exposing secrets or filesystem paths, and maintain freshness.

### Q148. How should a client consume remote tools?
**Answer:** Configure trusted endpoints and credentials, connect, discover capabilities, apply allowlists/policies, and expose only authorized tools. Handle timeouts, connection failures, malformed responses, and version changes.

### Q149. What are common MCP integration patterns?
**Answer:** Internal service servers expose company capabilities; shared organizational servers offer integrations to multiple applications; third-party servers connect to external services. Each needs ownership, monitoring, version management, and access controls.

### Q150. How could MCP support an HR assistant?
**Answer:** It might expose policy search and an authorized employee-specific leave-balance lookup. Backend identity checks must prevent one employee accessing another's records. Sensitive actions should follow company policy and require approval where appropriate.

### Q151. How could MCP support a knowledge assistant?
**Answer:** Expose search over approved documents and source metadata for RAG. Honor document permissions, show citations, and report missing or conflicting evidence.

### Q152. How could MCP support a developer assistant?
**Answer:** Provide approved documentation search, repository information, build status, or issue lookup. Actions such as deploying or changing production settings need strict authorization and approval.

### Q153. What is a complete Spring AI + MCP lifecycle?
**Answer:** Authenticate the user, select permitted servers/capabilities, build the model request, handle a requested tool through the MCP client, receive the result, return it to the model or application, validate the final answer, and respond with source/action details. Log safely without secrets or unnecessary personal data.

---

# Phase 12 — Production GenAI

### Q154. What is prompt injection?
**Answer:** It is an attempt to manipulate an AI system through user input or retrieved content that conflicts with intended policy—for example, a document saying to ignore previous instructions and reveal secrets. Defenses include treating content as untrusted, least-privilege tools, allowlists, policy checks, output validation, and adversarial tests. Prompt wording alone is not a security boundary.

### Q155. What is jailbreaking?
**Answer:** An attempt to bypass model safety or application restrictions through crafted inputs. Use layered defenses: provider safety features, input/output checks, restricted tools, access controls, monitoring, and red-team testing.

### Q156. How can data leakage occur?
**Answer:** Sensitive information can leak through prompts, retrieved results, tool outputs, logs, responses, or error traces. Minimize data, enforce authorization before retrieval, redact logs, control retention, and review provider data-handling terms.

### Q157. How should PII be protected?
**Answer:** Collect only necessary data, restrict access, redact/tokenize where practical, define retention/deletion policies, encrypt data, and prevent logs or evaluation sets from retaining sensitive information unnecessarily. Follow applicable law and organizational policy.

### Q158. How should tool access be secured?
**Answer:** Use least privilege, authenticated service identities, backend authorization, input validation, allowlists, scoped credentials, rate limits, idempotency, and audit logs. Never rely on the LLM to enforce permissions.

### Q159. What should be measured in GenAI evaluation?
**Answer:** Task success, factuality, relevance, groundedness, citation correctness, output-schema validity, refusal behavior, safety, latency, and cost. RAG also needs retrieval metrics. Use human judgment where automated criteria are insufficient.

### Q160. What are groundedness and faithfulness?
**Answer:** They describe how well an answer is supported by supplied evidence. Definitions vary across evaluation frameworks. A response can be fluent and relevant but still unsupported by its sources.

### Q161. How can hallucinations be evaluated?
**Answer:** Create known-answer tests with source documents; check claims against evidence; verify citations; include cases with missing or contradictory evidence; and count unsupported claims. LLM-as-judge methods can help but should be calibrated against human-reviewed examples.

### Q162. What is benchmarking?
**Answer:** Benchmarking evaluates a model/application on a consistent task set and metrics. Domain-specific tests are often more useful than generic leaderboards. Repeat benchmarks after model, prompt, retrieval, or dependency changes.

### Q163. What is observability?
**Answer:** It uses logs, metrics, and traces to understand behavior. Track request count, model/provider, tokens, latency, retries, tool calls, retrieval results, errors, safety interventions, and cost estimates.

### Q164. What should be logged?
**Answer:** Log correlation IDs, model, timing, token counts, retrieval identifiers, tool names, status, and validation outcomes. Avoid full sensitive prompts and secrets by default. Apply redaction, retention limits, and access controls.

### Q165. Why are traces useful?
**Answer:** Traces connect a request to retrieval, model calls, tools, databases, and downstream services, helping identify the source of latency or failure.

### Q166. What are guardrails?
**Answer:** Controls that validate or restrict inputs, outputs, and actions: topic policy, PII detection, schema validation, source requirements, tool permissions, and human approval. Use layers; no single filter catches every failure.

### Q167. Input versus output guardrails?
**Answer:** Input guardrails validate requests before processing. Output guardrails inspect generated content or proposed actions before returning or executing them. Authorization and business rules must also be enforced independently.

### Q168. How should compliance be addressed?
**Answer:** Map data flows, identify applicable requirements, control access and retention, document provider/model use, maintain audit trails, test safety/bias, and involve security/privacy/legal teams as appropriate.

### Q169. What is semantic caching?
**Answer:** It reuses a result when a new query is sufficiently similar to a cached query. It can reduce cost and latency but risks stale or context-inappropriate answers. Cache identity, permissions, prompt/model version, and freshness must be considered.

### Q170. What is response caching?
**Answer:** It stores prior responses for repeated requests. It works for deterministic or slowly changing content but requires careful invalidation and access control. Do not reuse private user responses for other users unless explicitly safe and authorized.

### Q171. How can token usage be reduced?
**Answer:** Remove redundant instructions/history, retrieve fewer but more relevant chunks, summarize long conversations, limit output length, use structured fields, and choose a smaller model when evaluation shows it is sufficient. Preserve necessary evidence and safety checks.

### Q172. How should models be selected?
**Answer:** Evaluate quality, latency, cost, context, modalities, tool support, data terms, availability, and reliability on your own test set. Different models may be used for classification, complex reasoning, embeddings, and high-risk tasks if routing is monitored.

### Q173. What are fallback models?
**Answer:** An alternate model used when the primary provider fails or is unavailable. Fallbacks need compatible inputs, timeout/retry rules, expected quality, and safe behavior. Avoid indefinite retries or silent quality downgrades on consequential tasks.

### Q174. What is multi-model routing?
**Answer:** It selects a model based on task, quality, latency, cost, or availability. Routing should be measured and versioned, with regression tests to catch quality changes.

### Q175. Why are rate limiting and load balancing important?
**Answer:** Rate limiting protects systems and budgets from excess traffic; load balancing distributes requests. AI systems also need concurrency controls, queueing, timeouts, circuit breakers, and provider quota awareness.

### Q176. What is an AI gateway?
**Answer:** A centralized layer for model routing, authentication, rate limits, policy enforcement, observability, and usage accounting. Provide redundancy and clear operational ownership to avoid a single point of failure.

### Q177. What are the layers of an enterprise GenAI architecture?
**Answer:** Experience/UI; application/API; model/provider; knowledge ingestion and RAG; tools/business actions; MCP integrations; security and guardrails; evaluation/observability; and operations for caching, rate limits, retries, deployment, and cost management.

### Q178. What does a production-ready GenAI service need beyond a demo?
**Answer:** Authentication, authorization, validation, data governance, secret management, prompt-injection defenses, bounded tools, timeouts/retries, rate limits, evaluation, monitoring, cost controls, deployment automation, and plans for data/model/prompt changes. A demo proves feasibility; production readiness requires safety, reliability, and maintainability.

---

# End-to-End Integration Architecture

### Q179. How do React, Spring Boot, Spring AI, RAG, tools, and MCP fit together?
**Answer:** A typical arrangement is:

```text
User
  |
React Web UI
  | HTTPS REST / streaming
Spring Boot API
  |-- Authentication, authorization, validation, rate limiting
  |-- Conversation/session management
  |
Spring AI orchestration
  |-- ChatClient / ChatModel
  |-- Prompt templates and policies
  |-- Chat memory, if required
  |-- RAG: EmbeddingModel + VectorStore + document ingestion
  |-- Tool calling: approved application services and APIs
  |-- MCP client: trusted MCP servers and allowed capabilities

Cross-cutting: security, guardrails, evaluation, observability,
caching, cost tracking, timeouts, retries, audit logs
```

The React UI should not call privileged tools directly. Spring Boot is the trusted boundary for identity, authorization, validation, and business rules.

### Q180. What happens when a user asks about company documents?
**Answer:** React sends the question to Spring Boot; the API authenticates the user; retrieval embeds the question and searches only authorized documents; relevant chunks and source metadata are added to the prompt; the model generates an evidence-based answer; the backend validates the answer/citations; React displays the response and sources; monitoring records useful metrics without unnecessarily retaining sensitive content.

### Q181. What happens when the user requests live business information?
**Answer:** The application exposes a narrow tool such as `getOrderStatus(orderId)`. The model may request it, but backend code validates the ID, checks authorization, calls the business service, and returns the actual result. The final answer must reflect the tool result, not a guess.

### Q182. When should MCP be used?
**Answer:** Use MCP when capabilities should be exposed through a standardized interface to compatible AI clients or reused across applications. A direct Spring service call may be simpler for a private operation used by only one application. Decide based on interoperability, reuse, ownership, security, and operations.

### Q183. Where should security checks live?
**Answer:** In trusted backend and service boundaries. UI checks and prompts improve usability but are not security controls. Enforce identity, tenant boundaries, document permissions, tool permissions, and business rules in application code or the system that owns the data/action.

### Q184. How should errors propagate?
**Answer:** Use structured error categories and correlation IDs. Distinguish validation, authorization, retrieval misses, model-provider errors, tool failures, timeouts, and output-validation failures. Return safe messages to users, log diagnostic details under access control, and never claim an action succeeded without downstream confirmation.

---

# Recommended Learning Order

1. AI and Generative AI fundamentals.
2. LLMs: tokens, context windows, Transformers, sampling, and limitations.
3. Prompt engineering and structured output.
4. Spring AI basics: build a small Spring Boot chat endpoint.
5. Embeddings and vector mathematics.
6. Vector stores and semantic retrieval.
7. RAG with source references and permission filters.
8. Tool calling with narrow, validated functions.
9. Agent workflows, only where multi-step decisions are needed.
10. MCP concepts, discovery, transports, and security.
11. Spring AI + MCP client/server integration.
12. Production hardening: evaluation, monitoring, guardrails, caching, cost controls, and deployment.

## Suggested Practical Projects

1. Basic chat endpoint using Spring Boot and Spring AI.
2. Structured JSON response mapped to a Java record and validated.
3. Chat history with persistence and retention rules.
4. Document ingestion and vector search.
5. RAG question answering with citations and access filters.
6. Tool calling for a safe read-only business lookup.
7. Bounded multi-step workflow combining retrieval and tools.
8. MCP client consuming a trusted server, or an MCP server exposing an approved tool.
9. Production hardening with evaluation tests, traces, token/cost metrics, security tests, rate limits, and timeouts.

---

# Glossary

| Term | Meaning |
|---|---|
| AI | Broad field of systems performing intelligent tasks |
| ML | Methods that learn patterns from data |
| Deep Learning | ML using multi-layer neural networks |
| Generative AI | Models that generate new content |
| LLM | Large Language Model |
| Token | Unit used by a model tokenizer |
| Context Window | Limit on the context and output a model can process |
| Transformer | Neural architecture built around attention |
| Prompt | Instructions and input sent to a model |
| Embedding | Numerical vector representation of content |
| Vector Store | System for storing vectors and similarity-searching them |
| RAG | Retrieval-Augmented Generation |
| Chunk | Segment of a document used in embedding and retrieval |
| Top-K | Number of highest-ranked results returned |
| Reranking | Reordering retrieved candidates with a relevance scorer |
| Tool Calling | Model request for the application to execute a defined function |
| Agent | Model-enabled system taking steps toward a goal |
| MCP | Model Context Protocol for connecting AI applications to external capabilities |
| MCP Host | AI application coordinating model and MCP interactions |
| MCP Client | Component connecting to an MCP server |
| MCP Server | Service exposing MCP tools, resources, and/or prompts |
| Guardrail | Control validating inputs, outputs, or actions |
| Groundedness | Degree to which an answer is supported by evidence |
| Observability | Logs, metrics, and traces for understanding system behavior |
| Semantic Cache | Cache that reuses results for sufficiently similar requests |
| AI Gateway | Central layer for model routing, policies, rate limits, and usage tracking |

---

# Production Readiness Checklist

- [ ] API credentials are stored securely and never committed to source control.
- [ ] Inputs and structured outputs are validated.
- [ ] User identity, tenant boundaries, and tool/document permissions are enforced in backend code.
- [ ] Retrieved documents are treated as untrusted data.
- [ ] RAG answers can cite sources and handle missing evidence.
- [ ] Tools are narrow, permissioned, logged, and bounded by timeouts.
- [ ] MCP servers expose only necessary capabilities.
- [ ] High-impact actions require suitable confirmation or approval.
- [ ] Model, prompt, retrieval, and tool changes are evaluated with regression tests.
- [ ] Latency, errors, token use, and cost are monitored.
- [ ] Sensitive data is minimized in logs and retained only as required.
- [ ] Rate limits, retries, fallbacks, and failure messages are tested.
- [ ] Dependency versions and APIs match the documentation used by the project.
