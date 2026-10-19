# Generative AI and Spring AI — Complete Learning Roadmap

## Phase 1: Introduction to AI

### 1.1 Introduction to AI
- What is AI?
- AI vs ML vs Deep Learning vs Generative AI
- Evolution of AI
- Rule-Based Systems
- Machine Learning Overview
- Deep Learning Overview
- Generative AI Overview
- Real-world AI Applications

### 1.2 Generative AI Fundamentals
- What is Generative AI?
- Types of Generative Models
- Text Generation
- Image Generation
- Audio Generation
- Video Generation

### 1.3 AI Ecosystem
- Foundation Models
- Pre-trained Models
- Fine-Tuned Models
- Open Source vs Closed Source Models
- Local Models vs Cloud Models

## Phase 2: LLM Architecture

### 2.1 LLM Fundamentals
- What is an LLM?
- Evolution of LLMs
- GPT Family
- Gemini Family
- Claude Family
- Llama Family

### 2.2 Tokenization
- Tokens
- Tokenizers
- Token Counting
- Input Tokens
- Output Tokens
- Context Tokens
- Token Costing

### 2.3 Transformers
- Attention Mechanism
- Self-Attention
- Multi-Head Attention
- Positional Encoding
- Transformer Architecture
- Encoder vs Decoder
- Why Transformers Replaced RNNs and LSTMs

### 2.4 Context Window
- Context Window
- Long Context Models
- Context Management
- Context Truncation

### 2.5 Sampling Parameters
- Temperature
- Top-K
- Top-P
- Frequency Penalty
- Presence Penalty
- Stop Sequences
- Max Tokens

### 2.6 LLM Challenges
- Hallucinations
- Bias
- Knowledge Cutoff
- Latency
- Cost
- Safety Risks

### 2.7 LLM Request Lifecycle
- User Query
- Tokenization
- Embedding Layer
- Attention Layers
- Token Prediction
- Decoding
- Response Generation

## Phase 3: Prompt Engineering

### 3.1 Prompt Fundamentals
- Prompt Structure
- System Prompt
- User Prompt
- Assistant Prompt
- Context Injection

### 3.2 Prompting Techniques
- Zero-Shot Prompting
- One-Shot Prompting
- Few-Shot Prompting
- Chain of Thought
- Self Consistency
- ReAct Prompting
- Tree of Thoughts

### 3.3 Output Structuring
- Structured Output
- JSON Output
- XML Output
- Schema-Based Output

### 3.4 Prompt Optimization
- Prompt Templating
- Prompt Chaining
- Prompt Compression
- Prompt Evaluation

## Phase 4: Spring AI Fundamentals

### 4.1 Spring AI Introduction
- Spring AI Overview
- Supported Providers
- Spring AI Architecture
- Spring AI Request Lifecycle

### 4.2 Chat Models
- ChatModel
- ChatClient
- Prompt Objects
- Prompt Templates
- Streaming Responses

### 4.3 Response Handling
- String Responses
- Structured Responses
- Entity Mapping
- Record Mapping

### 4.4 Advisors
- Advisor Overview
- Logging Advisors
- Memory Advisors
- Retrieval Advisors
- Custom Advisors

### 4.5 Chat Memory
- Message Memory
- InMemory Memory
- JDBC Memory
- Redis Memory
- Memory Strategies

### 4.6 Spring AI Components
- ChatClient
- ChatModel
- EmbeddingModel
- VectorStore
- Tool Calling
- MCP Support

### 4.7 Spring AI Request Flow
- Request Creation
- Advisor Chain
- Model Invocation
- Response Processing

## Phase 5: Embeddings

### 5.1 Embedding Fundamentals
- What are Embeddings?
- Why Embeddings?
- Semantic Meaning
- Numerical Representation

### 5.2 Embedding Models
- OpenAI Embeddings
- Gemini Embeddings
- Ollama Embeddings
- Local Embedding Models

### 5.3 Vector Mathematics
- Vectors
- Dimensions
- Distance Metrics
- Similarity Scores

### 5.4 Similarity Algorithms
- Cosine Similarity
- Euclidean Distance
- Dot Product

### 5.5 Embedding Lifecycle
- Text Processing
- Vector Generation
- Storage
- Retrieval

### 5.6 Spring AI Embeddings
- EmbeddingModel
- Embedding Requests
- Embedding Responses

## Phase 6: Vector Databases

### 6.1 Why Vector Databases
- Traditional Search Limitations
- Keyword Search vs Semantic Search
- Vector Search

### 6.2 Vector Database Fundamentals
- Vector Storage
- Indexing
- Similarity Search
- Approximate Nearest Neighbor (ANN)

### 6.3 Popular Vector Databases
- PGVector
- Pinecone
- Weaviate
- Milvus
- Chroma
- Qdrant
- Elasticsearch

### 6.4 Indexing Strategies
- HNSW
- IVF
- Flat Index

### 6.5 Spring AI Integration
- VectorStore Interface
- Data Ingestion
- Similarity Search

### 6.6 Retrieval Lifecycle
- Query Embedding
- Index Lookup
- Similarity Matching
- Result Ranking

## Phase 7: RAG (Retrieval-Augmented Generation)

### 7.1 Introduction to RAG
- Why RAG?
- Problems Solved by RAG
- What RAG Actually Does
- RAG vs Fine-Tuning
- RAG End-to-End Architecture

### 7.2 Document Ingestion Pipeline
- Data Sources
- File Loading
- Text Extraction
- Data Cleaning
- Chunk Creation
- Chunk Storage

### 7.3 Chunking
- Fixed-Size Chunking
- Recursive Chunking
- Semantic Chunking
- Chunk Size
- Chunk Overlap

### 7.4 Embeddings in RAG
- Document Embeddings
- Query Embeddings
- Embedding Models

### 7.5 Retrieval Process
- Similarity Search
- Top-K Retrieval
- Score Threshold
- Ranking
- Re-Ranking

### 7.6 Metadata
- Metadata Storage
- Metadata Filtering
- Source Tracking
- Multi-Tenant Filtering

### 7.7 Prompt Augmentation
- Context Injection
- Retrieved Context
- Grounded Responses

### 7.8 RAG Architectures
- Basic RAG
- Advanced RAG
- Hybrid Search RAG
- Agentic RAG
- Graph RAG

### 7.9 Advanced Retrieval
- Query Transformation
- Query Expansion
- Multi-Query Retrieval
- Parent-Child Retrieval
- Hybrid Search
- Re-Ranking

### 7.10 RAG Flows

**Ingestion flow**
- Load
- Split
- Embed
- Store

**Retrieval flow**
- Query
- Embed
- Search
- Retrieve
- Inject
- Generate

### 7.11 Spring AI RAG
- Document Readers
- Token Splitters
- EmbeddingModel
- VectorStore
- Retrieval Advisors
- Question-Answering Flow

### 7.12 RAG Optimization
- Chunk Tuning
- Top-K Tuning
- Embedding Selection
- Caching
- Cost Optimization

## Phase 8: Tool Calling

### 8.1 Introduction
- Why Tool Calling?
- Function Calling Overview

### 8.2 Tool Fundamentals
- Tools
- Functions
- Tool Schemas
- Tool Discovery

### 8.3 Tool Execution
- Tool Selection
- Parameter Extraction
- Tool Invocation
- Result Processing

### 8.4 Spring AI Tool Calling
- `@Tool`
- Tool Registration
- Tool Execution

### 8.5 Tool Patterns
- REST API Tools
- Database Tools
- Search Tools
- Email Tools
- External Service Tools

### 8.6 Tool Calling Lifecycle
- Request
- Reasoning
- Tool Selection
- Execution
- Final Response

## Phase 9: Agents

### 9.1 Introduction to Agents
- What is an Agent?
- Agent Components
- Agent Lifecycle

### 9.2 Agent Architecture
- LLM
- Memory
- Tools
- Planning
- Reasoning

### 9.3 Agent Patterns
- ReAct
- Plan and Execute
- Reflection
- Self-Correction

### 9.4 Agent Memory
- Short-Term Memory
- Long-Term Memory
- Vector Memory

### 9.5 Agent Workflows
- Single-Agent Systems
- Multi-Agent Systems

### 9.6 Multi-Agent Systems
- Planner Agent
- Research Agent
- Execution Agent
- Reviewer Agent

### 9.7 Agentic RAG
- Planning
- Retrieval
- Tool Usage
- Response Generation

### 9.8 Spring AI Agents
- Agent Workflows
- Tool Integration
- Memory Integration

## Phase 10: MCP (Model Context Protocol)

### 10.1 MCP Fundamentals
- Why MCP?
- Problems Solved by MCP
- MCP vs Traditional Tool Integrations
- MCP Architecture

### 10.2 Core Components
- Host
- Client
- Server
- Tool
- Resource
- Prompt

### 10.3 MCP Communication
- Protocol Overview
- Tool Discovery
- Resource Discovery
- Prompt Discovery

### 10.4 MCP Tools
- Tool Registration
- Tool Exposure
- Tool Invocation

### 10.5 MCP Resources
- Files
- Documents
- Databases
- Dynamic Resources

### 10.6 MCP Prompts
- Prompt Templates
- Dynamic Prompts
- Shared Prompts

### 10.7 MCP Workflows
- Request Flow
- Discovery Flow
- Tool Execution Flow

### 10.8 MCP Security
- Authentication
- Authorization
- Access Control

## Phase 11: Spring AI + MCP

### 11.1 Spring AI MCP Overview
- MCP Client
- MCP Server

### 11.2 MCP Server Development
- Tool Registration
- Resource Registration
- Prompt Registration

### 11.3 MCP Client Development
- Client Configuration
- Remote Discovery
- Tool Consumption

### 11.4 MCP Integration Patterns
- Internal MCP Servers
- Shared Organization MCP Servers
- Third-Party MCP Servers

### 11.5 MCP Request Lifecycle
- Discovery
- Selection
- Execution
- Response

### 11.6 Real-World MCP Use Cases
- HR Assistant
- Knowledge Assistant
- Enterprise Agent
- Developer Assistant

## Phase 12: Production GenAI

### 12.1 Security
- Prompt Injection
- Jailbreaking
- Data Leakage
- PII Protection
- Secure Tool Access
- Secure MCP Servers

### 12.2 Evaluation
- Accuracy
- Relevance
- Faithfulness
- Groundedness
- Hallucination Detection
- Benchmarking

### 12.3 Observability
- Prompt Logging
- Response Logging
- Token Tracking
- Cost Monitoring
- Latency Monitoring
- Tracing

### 12.4 Guardrails
- Input Guardrails
- Output Guardrails
- Safety Filters
- Policy Enforcement
- Compliance Validation

### 12.5 Performance Optimization
- Semantic Caching
- Context Caching
- Response Caching
- Model Selection Strategies

### 12.6 Scalability
- Multi-Model Routing
- Fallback Models
- Rate Limiting
- Load Balancing

### 12.7 Cost Optimization
- Token Optimization
- Prompt Optimization
- Retrieval Optimization
- Caching Strategies

### 12.8 Enterprise AI Architecture
- AI Gateway
- Model Routing
- RAG Layer
- Tool Layer
- MCP Layer
- Monitoring Layer
- Security Layer
