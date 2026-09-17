# Curriculum

## What You Will Learn

Build the technical depth and enterprise mindset required to take AI systems from prototype to production.

### Software & Backend Engineering

Master Python, asynchronous programming, Linux, FastAPI, REST APIs, GraphQL, testing, and production-ready backend development.

### Cloud & DevOps

Deploy secure, scalable applications on AWS using VPC, IAM, RDS, Docker, ECS Fargate, and GitHub Actions CI/CD.

### Advanced RAG

Build high-accuracy RAG systems using Pinecone, Qdrant, hybrid search, semantic chunking, reranking, and retrieval evaluation.

### Knowledge Graphs & Multimodal AI

Work with Neo4j, Cypher, Vision Language Models, ColPali, visual embeddings, and complex enterprise documents including PDFs, tables, and charts.

### Agentic & Multi-Agent AI

Build autonomous AI systems with LangGraph, including supervisor and router patterns, parallel execution, memory, human-in-the-loop workflows, and MCP-based tool integration.

### Enterprise Integrations

Connect AI systems with Slack, Jira, SQL databases, Oracle, SOAP APIs, and legacy enterprise infrastructure.

### AI Security & Governance

Implement OAuth 2.0, SSO, RBAC, prompt-injection protection, PII masking, NeMo Guardrails, and AWS Bedrock Guardrails for secure enterprise AI.

### Production AI & Observability

Learn to operate AI systems in production with LLM gateways, rate limiting, retries, evaluation, tracing, latency monitoring, and token-cost observability.

### AI Consulting & Delivery

Learn the FDE approach to technical discovery, solution architecture, SOW creation, ROI presentations, UAT, and enterprise deployment handoffs.

---

## Module 1: Python & Linux Foundations

1. Python Core data structures
2. Memory management fundamentals
3. Object-oriented programming basics
4. Exception handling
5. File I/O operations
6. Event loop architecture
7. Coroutines and tasks
8. Async context managers
9. Concurrency vs parallelism
10. ThreadPoolExecutor integration
11. Navigating the file system
12. Permission and user management
13. Process monitoring
14. Shell scripting basics
15. Environment variable configurations

## Module 2: Modern API Development

1. Path and query parameters
2. Pydantic data validation
3. Dependency injection
4. CORS middleware
5. Building scalable CRUD endpoints
6. Designing GraphQL schemas and object types using Python frameworks like Strawberry or Graphene
7. Writing efficient data resolvers
8. Handling N+1 query problems
9. Executing queries vs mutations
10. Implementing URL-based vs Header-based API versioning
11. Unit testing fundamentals
12. Fixtures and mocking
13. Test coverage analysis
14. Debugging configurations
15. Essential IDE extensions

## Module 3: Cloud Fundamentals & Networking

1. Provisioning virtual machines
2. Object storage lifecycle policies
3. Managed relational databases setup
4. Event-driven serverless function basics
5. Creating virtual private clouds
6. Public vs private subnet routing
7. Outbound traffic via NAT
8. Configuring strict security groups
9. Principle of least privilege
10. Creating identity policies
11. Assuming cross-account roles
12. Navigating the AWS Billing console
13. Setting automated budget thresholds

## Module 4: Containerization & CI/CD

1. Writing optimized Dockerfiles
2. Managing multi-container environments
3. Volume mounting and networking
4. Containerizing FastAPI backends
5. Serverless container concepts
6. Configuring task definitions
7. Load balancer integration
8. Auto-scaling policy configuration
9. Creating workflow YAML files
10. Triggering automated builds
11. Managing GitHub secrets
12. Continuous deployment to AWS

## Module 5: LLM Fundamentals & Prompting

1. Zero-shot and few-shot prompting
2. Chain of thought reasoning
3. Token calculation
4. Sliding window techniques
5. Context compression
6. Enforcing strict JSON output schemas via APIs
7. Defining complex nested Pydantic models
8. Validating LLM responses natively against type hints
9. Handling and retrying output parsing errors gracefully
10. Using discriminator fields for union types
11. Defining precise function schemas for LLMs
12. Parsing and validating tool arguments
13. Handling multi-tool parallel execution
14. Processing tool results into chat history
15. Managing hallucinated tool calls

## Module 6: Vector Search & Core RAG

1. Fixed-size and semantic chunking
2. Overlap optimization
3. Understanding vector representations
4. Dimensionality trade-offs
5. BM25 sparse matrices
6. Index creation
7. Distance metrics (Cosine, Euclidean)
8. Metadata filtering
9. Cloud vector database provisioning
10. End-to-end basic retrieval
11. Keyword-based search mechanisms
12. Combining dense and sparse signals
13. Implementing RRF algorithms
14. Precision and recall metrics
15. Cross-encoder reranking models
16. API integration for rerankers

## Module 7: Enterprise Graph Architecture

1. Nodes, edges, and properties
2. Ontologies vs taxonomies
3. Transforming relational data tables to graph structures
4. AuraDB cloud provisioning
5. Pattern matching syntax
6. Path traversal queries
7. Optimizing complex graph joins
8. Overview of Amazon Neptune
9. Mapping logistics networks
10. Modeling user interactions
11. Graph-based retrieval logic for AI chatbot

## Module 8: Multimodal RAG & Vision AI

1. Identifying document structures
2. Optical character recognition pipelines
3. OCR-free embedding strategies via ColPali
4. Architecture of vision-language bridges
5. Prompting with images
6. Contrastive language-image pretraining (CLIP) concepts
7. Image-to-image similarity search
8. Extracting nested tabular data natively
9. Reasoning over complex chart visuals and infographs
10. Ingesting unstructured legacy enterprise PDFs
11. Aligning bounding boxes with text chunks
12. Managing multi-page visual context

## Module 9: Agentic Frameworks & LangGraph

1. Defining agency
2. ReAct framework loops
3. Plan & Execute structures
4. Supervisor and Router patterns
5. Defining graphs and state management
6. Creating nodes and edges
7. Conditional routing logic
8. Compiling graphs
9. Running agent tasks concurrently using async execution
10. Managing nested parent-child graph architectures
11. Aggregating parallel outputs with map-reduce patterns
12. Handling state conflicts during concurrent node execution

## Module 10: Advanced Agent Orchestration

1. Check-pointing graph states
2. Semantic long-term memory retrieval
3. Interrupting graph execution
4. Requesting manual state approval
5. Detecting infinite ReAct loops
6. Self-correction prompting mechanisms
7. Managed agent provisioning
8. Integrating knowledge bases
9. Connecting agents to external tools safely
10. Host/Client/Server architectures
11. Standardizing tool access boundaries

## Module 11: Legacy Systems & Integrations

1. Webhooks and bot tokens for Slack/Teams
2. Interactive message payloads
3. Reading internal Jira pages
4. Automating ticket creation
5. Understanding WSDL document structures and types
6. Constructing valid XML SOAP envelopes using Python libraries like Zeep
7. Parsing complex legacy XML responses safely
8. Handling SOAP faults and legacy error codes
9. Converting XML payloads to modern JSON formats
10. Establishing secure connections using Python-native drivers (pyodbc, oracledb) and SQLAlchemy
11. Constructing safe parameterized queries to prevent SQL injection
12. Implementing read-only database roles
13. Mapping complex database schemas to LLM context
14. Handling Text-to-SQL logic constraints and fallbacks

## Module 12: Identity & Access Management

1. Authentication vs authorization
2. Understanding JWT tokens
3. OAuth grant types
4. SAML assertions
5. Mapping Azure AD groups
6. Implementing Role-Based Access Control
7. Enforcing data-level permissions in retrieval layers

## Module 13: Production AI Security & Guardrails

1. Identifying major LLM vulnerabilities
2. Prompt injection defenses
3. Jailbreak prevention techniques
4. Implementing Microsoft Presidio analyzers and anonymizers
5. Redacting sensitive entities (SSN, credit cards, emails)
6. Customizing regex patterns for domain-specific PII
7. Reversing masks safely post-generation
8. Evaluating false-positive redaction rates
9. Writing programmable conversational rails using Colang
10. Configuring strict input and output filtering pipelines natively in Python
11. Enforcing topical boundaries to prevent off-topic chatter
12. Setting up AWS Bedrock managed guardrail configurations via Boto3
13. Testing rails against jailbreak libraries

## Module 14: AI Observability & Gateway Management

1. Idempotency keys for safe tool execution
2. Exponential backoff strategies
3. Centralizing provider API keys via LiteLLM/Portkey
4. Configuring rate limiting and fallback routing
5. Automating LLM-as-a-judge scoring pipelines
6. Calculating RAGAS faithfulness and answer relevance metrics
7. Measuring context precision and recall
8. Creating synthetic benchmark datasets from source documents
9. Tracking evaluation scores across deployment runs
10. Capturing deep span-level execution traces
11. Monitoring granular token costs and endpoint latency
12. Debugging multi-step agent reasoning and tool inputs
13. Creating custom user-session tracking metrics
14. Exporting trace data for continuous model improvement

## Module 15: OmniGuard — Secure AI Integration

1. Conducting technical discovery and scoping workshops
2. Defining data classifications
3. Drafting architecture SOWs
4. Preparing ROI presentations for CISO/executives
5. Delivering User Acceptance Testing (UAT) runbooks
6. Executing mock OAuth 2.0 / RBAC flows
7. Constructing Hybrid RAG alongside secure Text-to-SQL for MS SQL databases
8. Implementing NeMo & Presidio guardrails
9. Finalizing Dockerized FastAPI cloud deployments

## Module 16: AuditMesh — Multi-Agent Compliance System

1. Mapping 5-step manual compliance workflows
2. Identifying Human-in-the-Loop bottlenecks
3. Drafting latency/cost SLAs
4. Defining Model Context Protocol trust boundaries
5. Leading operations and training handoffs
6. Developing a LangGraph Multi-Agent Supervisor
7. Deploying a custom MCP server for secure Jira ticketing
8. Building comprehensive token cost/trace dashboards
9. Creating Streamlit/Gradio UIs for human approval workflows

---

## Projects You'll Build

In this bootcamp, you'll gain hands-on experience by building end-to-end enterprise AI systems that simulate real-world client engagements. You'll work across secure AI integration, RAG, multi-agent orchestration, enterprise systems, governance, and production deployment.

### Project 1: OmniGuard — Secure AI Integration

Build a secure enterprise AI integration system designed for real-world corporate environments. You'll implement Hybrid RAG, secure Text-to-SQL, OAuth 2.0 and RBAC, SQL database integration, NeMo and Presidio guardrails, and a Dockerized FastAPI deployment on the cloud. You'll also experience the complete FDE consulting lifecycle—from technical discovery and data classification to architecture SOWs, ROI presentations, and UAT runbooks.

### Project 2: AuditMesh — Multi-Agent Compliance System

Build a production-oriented multi-agent compliance auditing system using LangGraph and MCP. You'll design a supervisor-based agent architecture, connect agents securely to Jira through a custom MCP server, implement human-in-the-loop approval workflows, and build token-cost and traceability dashboards. The project also includes defining trust boundaries, SLAs, operational handoffs, and interfaces for human approval.
