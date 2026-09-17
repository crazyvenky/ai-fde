# Notes Plan — AI Forward Deployed Engineer Course

This file is a **plan**, not the notes themselves. For every topic in every module (matching `CURRICULUM.md` exactly, in the same order), it defines:

- **Focus** — the core idea/mechanism to actually understand, not just the buzzword.
- **Notes to prepare** — what should go into the real notes later: definitions, steps, code/config patterns, comparisons, pitfalls, examples.
- **Connections** — links to other topics/modules this depends on or feeds into, where genuinely relevant.

Once this plan is approved, notes get written topic-by-topic, following this plan exactly.

## How the modules connect (big picture)

- **Modules 1–4 (Foundations):** Python/Linux, API development, cloud basics, containers/CI-CD. This is the engineering substrate everything else runs on.
- **Modules 5–8 (AI Core):** LLM fundamentals, vector RAG, knowledge graphs, multimodal RAG. This is "how the AI actually retrieves and reasons over enterprise data."
- **Modules 9–11 (Agents & Integration):** Agentic frameworks/LangGraph, advanced orchestration, legacy systems integration. This is "how the AI acts autonomously and talks to real enterprise systems."
- **Modules 12–14 (Trust & Ops):** IAM, security/guardrails, observability. This is "how the AI is made safe, compliant, and operable in production."
- **Modules 15–16 (Capstones):** OmniGuard and AuditMesh are where Modules 1–14 get assembled into two real end-to-end systems — nearly every topic here is a direct application of an earlier module's concept.

---

## Module 1: Python & Linux Foundations

Foundation module — the runtime and OS-level mechanics that every later backend, async agent, and deployment topic assumes you already know.

1. **Python Core data structures**
   - Focus: lists/tuples/dicts/sets — mutability, ordering guarantees, when to use which.
   - Notes to prepare: complexity table for common ops (index/append/lookup/membership), comprehensions vs loops, shallow vs deep copy.
   - Connections: underlies Pydantic models (Module 2) and metadata structures used in vector search (Module 6).

2. **Memory management fundamentals**
   - Focus: reference counting + generational GC, mutable default-argument pitfall.
   - Notes to prepare: `is` vs `==`, common leak patterns in long-running services (unclosed connections, growing caches).
   - Connections: relevant to long-running agent/API processes covered in Module 14 observability.

3. **Object-oriented programming basics**
   - Focus: classes vs dataclasses, inheritance vs composition, key dunder methods.
   - Notes to prepare: a small class hierarchy example; when composition is preferable to inheritance.
   - Connections: Pydantic `BaseModel` (Module 2, 5) builds directly on this.

4. **Exception handling**
   - Focus: try/except/else/finally ordering, custom exception classes, exception chaining (`raise ... from`).
   - Notes to prepare: table of common built-in exceptions; a retry-safe exception pattern.
   - Connections: feeds directly into Module 14's exponential backoff/retry design.

5. **File I/O operations**
   - Focus: context managers (`with`), text vs binary mode, encoding gotchas.
   - Notes to prepare: streaming reads for large files; basic file locking.
   - Connections: used later for ingesting legacy PDFs (Module 8) and config files.

6. **Event loop architecture**
   - Focus: asyncio's single-threaded cooperative scheduler; blocking vs non-blocking calls.
   - Notes to prepare: a simple diagram of the event loop and task queue.
   - Connections: this is the mechanism underneath FastAPI async endpoints (Module 2) and async agent execution (Module 9).

7. **Coroutines and tasks**
   - Focus: `async def`/`await`, `asyncio.create_task`, coroutine vs Task vs Future.
   - Notes to prepare: example of `asyncio.gather` running multiple coroutines concurrently.
   - Connections: prerequisite for Module 9's concurrent LangGraph node execution.

8. **Async context managers**
   - Focus: `async with`, `__aenter__`/`__aexit__`.
   - Notes to prepare: use cases — async DB connections, async HTTP clients.
   - Connections: used with async DB drivers in Module 11.

9. **Concurrency vs parallelism**
   - Focus: the GIL's effect on threads; I/O-bound vs CPU-bound workload distinction.
   - Notes to prepare: comparison table — threading vs multiprocessing vs asyncio, with the right tool per workload type.
   - Connections: informs the ThreadPoolExecutor decision in the next topic.

10. **ThreadPoolExecutor integration**
    - Focus: running blocking/CPU-heavy code from async code without freezing the event loop.
    - Notes to prepare: `loop.run_in_executor` pattern example.
    - Connections: relevant when FastAPI endpoints (Module 2) call blocking libraries.

11. **Navigating the file system**
    - Focus: `pathlib` vs `os.path`, absolute vs relative paths.
    - Notes to prepare: common pathlib snippets (glob, resolve, mkdir).

12. **Permission and user management**
    - Focus: Linux rwx permission bits, `chmod`/`chown`, users/groups.
    - Notes to prepare: permission bit table; sudo vs root distinction.
    - Connections: conceptually mirrors least-privilege IAM in Module 3/12.

13. **Process monitoring**
    - Focus: `ps`, `top`/`htop`, signals (`kill -9` vs graceful `SIGTERM`).
    - Notes to prepare: how to identify and safely stop a runaway process.

14. **Shell scripting basics**
    - Focus: bash variables, conditionals, loops, exit codes.
    - Notes to prepare: a minimal deployment script skeleton.
    - Connections: reused inside CI/CD workflow steps in Module 4.

15. **Environment variable configurations**
    - Focus: managing secrets/config via env vars, `.env` files.
    - Notes to prepare: `python-dotenv` usage; dev/staging/prod config separation.
    - Connections: directly feeds Module 3 (AWS config) and Module 4 (GitHub secrets).

---

## Module 2: Modern API Development

Builds the HTTP/API layer on top of Module 1's runtime — this is the interface every AI system in the course will be served through.

1. **Path and query parameters**
   - Focus: FastAPI routing with type-annotated params; required vs optional.
   - Notes to prepare: side-by-side path-param vs query-param example.

2. **Pydantic data validation**
   - Focus: `BaseModel`, field validators, automatic request validation.
   - Notes to prepare: example of validation error response shape; nested model example.
   - Connections: reused heavily for structured LLM output parsing in Module 5.

3. **Dependency injection**
   - Focus: FastAPI `Depends()` for reusable logic (auth, DB sessions).
   - Notes to prepare: example of a dependency chain and overriding a dependency in tests.
   - Connections: this is where auth checks (Module 12) plug into endpoints.

4. **CORS middleware**
   - Focus: what CORS prevents, preflight OPTIONS requests, allowed origins/methods/headers.
   - Notes to prepare: `CORSMiddleware` config example.

5. **Building scalable CRUD endpoints**
   - Focus: REST conventions, correct status codes, pagination patterns.
   - Notes to prepare: endpoint design template; idempotency considerations.
   - Connections: idempotency ties into Module 14's idempotency-key topic.

6. **Designing GraphQL schemas and object types (Strawberry/Graphene)**
   - Focus: schema-first vs code-first GraphQL; types, queries, mutations.
   - Notes to prepare: REST vs GraphQL comparison table; a minimal schema snippet.

7. **Writing efficient data resolvers**
   - Focus: how resolver functions execute per field.
   - Notes to prepare: resolver execution order and performance notes.

8. **Handling N+1 query problems**
   - Focus: why naive nested resolvers cause repeated DB hits.
   - Notes to prepare: the DataLoader/batching pattern that fixes it.

9. **Executing queries vs mutations**
   - Focus: read vs write semantics in GraphQL.
   - Notes to prepare: mutation response design (returning updated object + errors).

10. **URL-based vs Header-based API versioning**
    - Focus: tradeoffs of each versioning approach.
    - Notes to prepare: comparison table + migration strategy for breaking changes.

11. **Unit testing fundamentals**
    - Focus: arrange-act-assert structure, pytest basics.
    - Notes to prepare: example test file skeleton.

12. **Fixtures and mocking**
    - Focus: pytest fixtures; mocking external calls (DB, APIs, LLM providers).
    - Notes to prepare: mock vs monkeypatch distinction; fixture scope levels.
    - Connections: needed later for testing/evaluation pipelines in Module 14.

13. **Test coverage analysis**
    - Focus: what coverage % actually measures (and its limits).
    - Notes to prepare: how to read a coverage report and spot untested branches.

14. **Debugging configurations**
    - Focus: IDE breakpoints and launch configs for a FastAPI app.
    - Notes to prepare: a sample debugger launch config.

15. **Essential IDE extensions**
    - Focus: productivity tooling — linting, formatting, Docker/AWS extensions.
    - Notes to prepare: short list of extensions and what each one checks/automates.

---

## Module 3: Cloud Fundamentals & Networking

Moves the app onto AWS — networking, compute, storage, and identity primitives needed before anything can be deployed for real.

1. **Provisioning virtual machines**
   - Focus: EC2 basics — instance types, AMIs, launch process.
   - Notes to prepare: a launch checklist and instance-sizing considerations.

2. **Object storage lifecycle policies**
   - Focus: S3 storage classes and lifecycle rules (transition/expiration).
   - Notes to prepare: example lifecycle rule for cost optimization.

3. **Managed relational databases setup**
   - Focus: RDS basics — engine choice, backups, connectivity.
   - Notes to prepare: RDS setup checklist.
   - Connections: this is the DB layer used later for SQL integration (Module 11).

4. **Event-driven serverless function basics**
   - Focus: Lambda triggers, execution model, cold starts.
   - Notes to prepare: Lambda vs container tradeoffs table.

5. **Creating virtual private clouds**
   - Focus: VPC building blocks — subnets, route tables, internet gateway.
   - Notes to prepare: a simple VPC architecture diagram.

6. **Public vs private subnet routing**
   - Focus: what differentiates a public subnet's route table from a private one's.
   - Notes to prepare: diagram showing traffic flow for both subnet types.

7. **Outbound traffic via NAT**
   - Focus: why private subnets need a NAT gateway for outbound internet access.
   - Notes to prepare: NAT gateway vs NAT instance comparison.

8. **Configuring strict security groups**
   - Focus: security groups as stateful firewalls; least-privilege rule design.
   - Notes to prepare: example inbound/outbound SG rule table.
   - Connections: same least-privilege principle as Module 12's IAM/RBAC work.

9. **Principle of least privilege**
   - Focus: the security principle applied consistently across network, IAM, and app layers.
   - Notes to prepare: side-by-side example of an over-permissioned vs a properly scoped policy.
   - Connections: reappears explicitly in Module 12 and Module 13.

10. **Creating identity policies**
    - Focus: IAM policy JSON structure — actions, resources, conditions.
    - Notes to prepare: a sample policy snippet with an explanation of evaluation logic.

11. **Assuming cross-account roles**
    - Focus: STS `AssumeRole` and trust policies.
    - Notes to prepare: when/why enterprises need cross-account access.

12. **Navigating the AWS Billing console**
    - Focus: Cost Explorer and the billing dashboard.
    - Notes to prepare: where to check spend broken down by service.

13. **Setting automated budget thresholds**
    - Focus: AWS Budgets and alerting.
    - Notes to prepare: example budget-alert setup.
    - Connections: ties into Module 14's token-cost observability at the application layer.

---

## Module 4: Containerization & CI/CD

Packages the app (Module 2) and infrastructure knowledge (Module 3) into a repeatable, automated deployment pipeline.

1. **Writing optimized Dockerfiles**
   - Focus: layer caching, multi-stage builds, minimal base images.
   - Notes to prepare: a before/after Dockerfile optimization example.

2. **Managing multi-container environments**
   - Focus: docker-compose services, networks, volumes.
   - Notes to prepare: sample `docker-compose.yml` for an app + database.

3. **Volume mounting and networking**
   - Focus: persistent data vs ephemeral containers; container networking modes.
   - Notes to prepare: bind mount vs named volume comparison.

4. **Containerizing FastAPI backends**
   - Focus: a production-ready Dockerfile (uvicorn/gunicorn workers, non-root user).
   - Notes to prepare: full Dockerfile example for a FastAPI service.
   - Connections: this exact pattern is reused to ship OmniGuard in Module 15.

5. **Serverless container concepts**
   - Focus: Fargate vs EC2-backed ECS.
   - Notes to prepare: when Fargate is preferable (no server management) vs when it isn't (cost at scale).

6. **Configuring task definitions**
   - Focus: ECS task definition structure — CPU/memory, container definitions.
   - Notes to prepare: a sample task definition JSON.

7. **Load balancer integration**
   - Focus: ALB target groups and health checks.
   - Notes to prepare: diagram of ALB → target group → ECS service wiring.

8. **Auto-scaling policy configuration**
   - Focus: scaling triggers (CPU/memory/custom metrics).
   - Notes to prepare: an example scaling policy with thresholds.

9. **Creating workflow YAML files**
   - Focus: GitHub Actions syntax — jobs, steps, triggers.
   - Notes to prepare: a sample CI workflow YAML.

10. **Triggering automated builds**
    - Focus: build triggers on push/PR; producing build artifacts.
    - Notes to prepare: a build-stage checklist.

11. **Managing GitHub secrets**
    - Focus: secrets storage and referencing inside workflows.
    - Notes to prepare: secrets vs plain env vars; rotation practice.
    - Connections: same secret-handling discipline as Module 1's env var topic and Module 12's auth secrets.

12. **Continuous deployment to AWS**
    - Focus: deploy step — push image to ECR, update the ECS service.
    - Notes to prepare: an end-to-end diagram from `git push` to a live production update.

---

## Module 5: LLM Fundamentals & Prompting

The first AI-specific module — how to talk to an LLM reliably and get structured, trustworthy output back.

1. **Zero-shot and few-shot prompting**
   - Focus: the difference, and when adding examples actually helps vs adds noise.
   - Notes to prepare: one zero-shot and one few-shot example on the same task, compared.

2. **Chain of thought reasoning**
   - Focus: eliciting step-by-step reasoning; when it improves accuracy vs wastes tokens.
   - Notes to prepare: a CoT prompt template.

3. **Token calculation**
   - Focus: how tokenization works; estimating cost and context usage.
   - Notes to prepare: rough token-to-word heuristics; a worked cost calculation.
   - Connections: this is the basis for Module 14's token-cost observability.

4. **Sliding window techniques**
   - Focus: handling conversations/documents that exceed the context window.
   - Notes to prepare: sliding-window diagram; tradeoffs vs summarization.

5. **Context compression**
   - Focus: summarizing/pruning history to fit budget.
   - Notes to prepare: comparison of compression strategies (truncation vs summarization vs retrieval-based).

6. **Enforcing strict JSON output schemas via APIs**
   - Focus: structured-output / `response_format` style enforcement.
   - Notes to prepare: example API call enforcing a JSON schema.

7. **Defining complex nested Pydantic models**
   - Focus: nested/recursive models for structured LLM output.
   - Notes to prepare: a nested schema example (e.g., an object containing a list of sub-objects).
   - Connections: direct continuation of Module 2's Pydantic validation topic.

8. **Validating LLM responses natively against type hints**
   - Focus: using Pydantic to parse and validate raw model output.
   - Notes to prepare: the validation error → handling flow.

9. **Handling and retrying output parsing errors gracefully**
   - Focus: the retry-with-feedback pattern when parsing fails.
   - Notes to prepare: pseudocode for a bounded retry loop that feeds the error back to the model.
   - Connections: same backoff discipline as Module 14.

10. **Using discriminator fields for union types**
    - Focus: tagged/discriminated unions for polymorphic LLM output.
    - Notes to prepare: a discriminator-field example (e.g., different "action type" schemas).

11. **Defining precise function schemas for LLMs**
    - Focus: designing tool/function schemas — name, description, parameters.
    - Notes to prepare: a well-written vs poorly-written tool schema, side by side.

12. **Parsing and validating tool arguments**
    - Focus: validating LLM-generated function-call arguments before executing anything.
    - Notes to prepare: a validation guardrail example placed before tool execution.
    - Connections: this guardrail idea reappears in Module 13.

13. **Handling multi-tool parallel execution**
    - Focus: an LLM requesting multiple tool calls in a single turn.
    - Notes to prepare: pattern for executing and collecting multiple tool calls.
    - Connections: mirrors the parallel-node execution in Module 9.

14. **Processing tool results into chat history**
    - Focus: correctly appending tool outputs back into the conversation.
    - Notes to prepare: message-role structure example (the "tool" role slot).

15. **Managing hallucinated tool calls**
    - Focus: detecting calls to non-existent tools or malformed arguments.
    - Notes to prepare: defensive checks to run before any tool executes.
    - Connections: feeds directly into Module 13's guardrail design.

---

## Module 6: Vector Search & Core RAG

The core retrieval mechanism most of the course's AI systems rely on — turning unstructured enterprise text into searchable, ranked context.

1. **Fixed-size and semantic chunking**
   - Focus: the two chunking philosophies and their tradeoffs.
   - Notes to prepare: comparison table (pros/cons/use case) for each.

2. **Overlap optimization**
   - Focus: why overlap prevents losing meaning at chunk boundaries.
   - Notes to prepare: rule-of-thumb overlap percentages and when to increase them.

3. **Understanding vector representations**
   - Focus: embeddings as dense vectors encoding semantic meaning.
   - Notes to prepare: a simple diagram of nearby vs distant points in embedding space.

4. **Dimensionality trade-offs**
   - Focus: higher dimensions capture more nuance but cost more to store/search.
   - Notes to prepare: a dimension-vs-cost/performance table.

5. **BM25 sparse matrices**
   - Focus: term-frequency-based sparse retrieval (TF-IDF lineage).
   - Notes to prepare: the intuition behind BM25 scoring (not full derivation) and when sparse beats dense.

6. **Index creation**
   - Focus: vector index types (e.g., HNSW, IVF) and what they trade off.
   - Notes to prepare: an index-type comparison table (speed vs recall vs memory).

7. **Distance metrics (Cosine, Euclidean)**
   - Focus: when to use cosine vs Euclidean vs dot product.
   - Notes to prepare: formula + one-line intuition for each metric.

8. **Metadata filtering**
   - Focus: combining vector similarity with structured filters (e.g., date, department).
   - Notes to prepare: an example filtered-search query.
   - Connections: this is exactly the mechanism behind permission-filtered retrieval in Module 12.

9. **Cloud vector database provisioning**
   - Focus: setting up a Pinecone/Qdrant index — dimension and metric configuration.
   - Notes to prepare: a provisioning checklist.

10. **End-to-end basic retrieval**
    - Focus: the full pipeline — embed query → search index → return chunks.
    - Notes to prepare: a simple pipeline diagram tying steps together.

11. **Keyword-based search mechanisms**
    - Focus: traditional lexical search mechanics.
    - Notes to prepare: how it differs from semantic (embedding-based) search.

12. **Combining dense and sparse signals**
    - Focus: the rationale for hybrid search.
    - Notes to prepare: a hybrid-search architecture diagram.

13. **Implementing RRF algorithms**
    - Focus: Reciprocal Rank Fusion for merging two ranked lists.
    - Notes to prepare: the RRF formula plus one fully worked example.

14. **Precision and recall metrics**
    - Focus: standard retrieval evaluation metrics.
    - Notes to prepare: formulas plus a worked example calculation.
    - Connections: this is the foundation for Module 14's RAGAS metrics.

15. **Cross-encoder reranking models**
    - Focus: why rerankers improve precision after initial retrieval.
    - Notes to prepare: cross-encoder vs bi-encoder comparison.

16. **API integration for rerankers**
    - Focus: calling a reranker service as a pipeline stage.
    - Notes to prepare: the retrieve-top-K → rerank → keep-top-N pipeline pattern.

---

## Module 7: Enterprise Graph Architecture

An alternative/complementary retrieval structure to Module 6 — for data where relationships matter as much as content.

1. **Nodes, edges, and properties**
   - Focus: the basic graph data model.
   - Notes to prepare: a small labeled diagram (nodes/edges/properties).

2. **Ontologies vs taxonomies**
   - Focus: hierarchical taxonomy vs richer, relationship-based ontology.
   - Notes to prepare: a side-by-side example of the same domain modeled both ways.

3. **Transforming relational data tables to graph structures**
   - Focus: mapping rows and foreign keys onto nodes and edges.
   - Notes to prepare: a before (table) / after (graph) transformation example.

4. **AuraDB cloud provisioning**
   - Focus: setting up a managed Neo4j AuraDB instance.
   - Notes to prepare: a provisioning steps checklist.

5. **Pattern matching syntax**
   - Focus: Cypher's `MATCH` clause basics.
   - Notes to prepare: a few sample Cypher queries with explanations.

6. **Path traversal queries**
   - Focus: multi-hop traversal syntax in Cypher.
   - Notes to prepare: a variable-length path query example.

7. **Optimizing complex graph joins**
   - Focus: indexing nodes and query-plan awareness for performance.
   - Notes to prepare: indexing strategy notes for frequently traversed properties.

8. **Overview of Amazon Neptune**
   - Focus: managed graph DB alternative to Neo4j (Gremlin/openCypher support).
   - Notes to prepare: a Neptune vs Neo4j comparison table.

9. **Mapping logistics networks**
   - Focus: applied example — modeling a supply chain as a graph.
   - Notes to prepare: a sample logistics graph model.

10. **Modeling user interactions**
    - Focus: applied example — a social/interaction graph.
    - Notes to prepare: a sample interaction graph model.

11. **Graph-based retrieval logic for AI chatbot**
    - Focus: GraphRAG — retrieving context via graph traversal for LLM grounding.
    - Notes to prepare: a GraphRAG pipeline diagram.
    - Connections: directly comparable/combinable with Module 6's vector RAG (hybrid graph+vector retrieval); consumed by agent tools in Module 9–10.

---

## Module 8: Multimodal RAG & Vision AI

Extends RAG (Module 6) beyond plain text — to the tables, scans, and charts real enterprise documents actually contain.

1. **Identifying document structures**
   - Focus: layout analysis — headers, tables, columns.
   - Notes to prepare: a checklist for detecting structural elements in a document.

2. **Optical character recognition pipelines**
   - Focus: the traditional OCR flow.
   - Notes to prepare: OCR pipeline steps and common failure modes.

3. **OCR-free embedding strategies via ColPali**
   - Focus: embedding page images directly, skipping OCR entirely.
   - Notes to prepare: ColPali's approach summarized against the OCR pipeline from the previous topic (direct contrast).

4. **Architecture of vision-language bridges**
   - Focus: how VLMs connect a vision encoder to a language model.
   - Notes to prepare: a high-level architecture diagram.

5. **Prompting with images**
   - Focus: constructing multimodal prompts (text + image).
   - Notes to prepare: an example multimodal prompt structure.

6. **Contrastive language-image pretraining (CLIP) concepts**
   - Focus: how contrastive pretraining aligns image and text embeddings into one space.
   - Notes to prepare: a diagram giving the training intuition.

7. **Image-to-image similarity search**
   - Focus: using image embeddings for visual search.
   - Notes to prepare: pipeline — embed images → vector search (ties back to Module 6).

8. **Extracting nested tabular data natively**
   - Focus: table extraction from documents via VLMs.
   - Notes to prepare: a before/after table-extraction example.

9. **Reasoning over complex chart visuals and infographs**
   - Focus: VLM chart-QA capability and its limits.
   - Notes to prepare: an example chart-reasoning prompt with expected output.

10. **Ingesting unstructured legacy enterprise PDFs**
    - Focus: end-to-end ingestion pipeline for messy real-world PDFs.
    - Notes to prepare: an ingestion pipeline checklist.
    - Connections: this exact pipeline feeds OmniGuard's document ingestion in Module 15.

11. **Aligning bounding boxes with text chunks**
    - Focus: linking spatial layout metadata to extracted text chunks.
    - Notes to prepare: a bounding-box-to-chunk mapping example.

12. **Managing multi-page visual context**
    - Focus: handling documents spanning many pages within VLM context limits.
    - Notes to prepare: a pagination/context strategy note (ties back to Module 5's sliding window/compression topics).

---

## Module 9: Agentic Frameworks & LangGraph

Where the system stops just answering and starts acting — planning, routing, and executing multi-step work autonomously.

1. **Defining agency**
   - Focus: what makes a system "agentic" (autonomy, planning, tool use) vs a single LLM call.
   - Notes to prepare: a one-paragraph definition plus a contrasting example.

2. **ReAct framework loops**
   - Focus: the Reason → Act → Observe loop.
   - Notes to prepare: a ReAct loop diagram.

3. **Plan & Execute structures**
   - Focus: upfront planning then execution, vs ReAct's step-by-step improvisation.
   - Notes to prepare: a comparison table (ReAct vs Plan & Execute) with when to use each.

4. **Supervisor and Router patterns**
   - Focus: an orchestrator agent delegating to sub-agents/tools.
   - Notes to prepare: a supervisor architecture diagram.
   - Connections: this exact pattern becomes AuditMesh's supervisor in Module 16.

5. **Defining graphs and state management**
   - Focus: LangGraph's `StateGraph` and a shared state schema.
   - Notes to prepare: an example state schema.

6. **Creating nodes and edges**
   - Focus: node functions and edge definitions.
   - Notes to prepare: a minimal graph code skeleton.

7. **Conditional routing logic**
   - Focus: conditional edges based on current state.
   - Notes to prepare: an example routing function.

8. **Compiling graphs**
   - Focus: the `.compile()` step and what it validates.
   - Notes to prepare: a short list of compile-time checks to be aware of.

9. **Running agent tasks concurrently using async execution**
   - Focus: async nodes enabling parallel branches.
   - Notes to prepare: how this builds directly on Module 1's asyncio topics.

10. **Managing nested parent-child graph architectures**
    - Focus: subgraphs used as nodes within a larger graph.
    - Notes to prepare: a nested-graph diagram.

11. **Aggregating parallel outputs with map-reduce patterns**
    - Focus: fan-out/fan-in patterns in LangGraph.
    - Notes to prepare: a map-reduce node example.

12. **Handling state conflicts during concurrent node execution**
    - Focus: reducers for merging concurrent state updates safely.
    - Notes to prepare: an example reducer function.

---

## Module 10: Advanced Agent Orchestration

Adds the production-grade controls agents need once they run for real: memory, pausing, correction, and standardized tool access.

1. **Check-pointing graph states**
   - Focus: persisting graph state so execution can resume later.
   - Notes to prepare: a checkpointer setup example.
   - Connections: builds directly on Module 9's state management.

2. **Semantic long-term memory retrieval**
   - Focus: storing/retrieving memories via embeddings across sessions.
   - Notes to prepare: a memory-store architecture sketch (ties back to Module 6's vector search).

3. **Interrupting graph execution**
   - Focus: pausing execution at defined points.
   - Notes to prepare: an interrupt configuration example.

4. **Requesting manual state approval**
   - Focus: human-in-the-loop approval gates.
   - Notes to prepare: an approval-flow diagram.
   - Connections: this exact concept becomes AuditMesh's HITL step in Module 16.

5. **Detecting infinite ReAct loops**
   - Focus: loop detection and max-iteration safeguards.
   - Notes to prepare: a guard-condition example.

6. **Self-correction prompting mechanisms**
   - Focus: reflection/self-critique loops.
   - Notes to prepare: a self-correction prompt template.

7. **Managed agent provisioning**
   - Focus: using managed agent-hosting services vs self-hosting.
   - Notes to prepare: a tradeoffs table.

8. **Integrating knowledge bases**
   - Focus: wiring RAG retrieval (Modules 6–8) into agent tool calls.
   - Notes to prepare: how retrieval becomes just another callable tool for the agent.

9. **Connecting agents to external tools safely**
   - Focus: sandboxing/validating tool execution.
   - Notes to prepare: a pre-execution safety checklist.
   - Connections: same guardrail discipline as Module 13.

10. **Host/Client/Server architectures**
    - Focus: MCP's host–client–server model.
    - Notes to prepare: an MCP architecture diagram.

11. **Standardizing tool access boundaries**
    - Focus: MCP as a standard protocol for scoping what tools an agent can reach.
    - Notes to prepare: an example of scoping an MCP server's exposed tools.
    - Connections: this is exactly what AuditMesh's custom Jira MCP server (Module 16) implements.

---

## Module 11: Legacy Systems & Integrations

The unglamorous but critical FDE skill: wiring modern AI into the SOAP/XML/SQL/ticketing systems enterprises actually run on.

1. **Webhooks and bot tokens for Slack/Teams**
   - Focus: bot authentication and webhook endpoint setup.
   - Notes to prepare: an example webhook payload.

2. **Interactive message payloads**
   - Focus: Slack Block Kit / interactive components.
   - Notes to prepare: a sample interactive message JSON.

3. **Reading internal Jira pages**
   - Focus: Jira REST API basics for reading issues.
   - Notes to prepare: a sample GET request/response pair.

4. **Automating ticket creation**
   - Focus: Jira API POST to create issues programmatically.
   - Notes to prepare: a sample create-issue payload.
   - Connections: reused directly by AuditMesh's Jira ticketing in Module 16.

5. **Understanding WSDL document structures and types**
   - Focus: WSDL as a SOAP service's contract.
   - Notes to prepare: a breakdown of WSDL's parts (types, messages, ports, bindings).

6. **Constructing valid XML SOAP envelopes using Python libraries like Zeep**
   - Focus: building SOAP requests programmatically.
   - Notes to prepare: a Zeep client usage example.

7. **Parsing complex legacy XML responses safely**
   - Focus: safe XML parsing that avoids XXE-style risks.
   - Notes to prepare: safe-parser configuration notes.
   - Connections: same security mindset as Module 13.

8. **Handling SOAP faults and legacy error codes**
   - Focus: SOAP fault structure and error handling.
   - Notes to prepare: a fault-handling try/except pattern.

9. **Converting XML payloads to modern JSON formats**
   - Focus: transforming XML into JSON for downstream use.
   - Notes to prepare: a conversion example.

10. **Establishing secure connections using Python-native drivers (pyodbc, oracledb) and SQLAlchemy**
    - Focus: DB driver setup, connection strings, and secret handling.
    - Notes to prepare: a connection-setup checklist.
    - Connections: ties to Module 3's RDS setup and Module 12's secret/credential handling.

11. **Constructing safe parameterized queries to prevent SQL injection**
    - Focus: parameterized queries vs unsafe string concatenation.
    - Notes to prepare: a vulnerable-query vs safe-query side-by-side example.
    - Connections: directly a Module 13 security concern.

12. **Implementing read-only database roles**
    - Focus: least-privilege DB roles specifically for AI agents.
    - Notes to prepare: a role-creation example.

13. **Mapping complex database schemas to LLM context**
    - Focus: representing a DB schema as text the LLM can reason over.
    - Notes to prepare: a schema-summarization strategy.

14. **Handling Text-to-SQL logic constraints and fallbacks**
    - Focus: constraining generated SQL and falling back safely on failure.
    - Notes to prepare: a validate-then-fallback flow diagram.
    - Connections: this is exactly OmniGuard's "secure Text-to-SQL" in Module 15.

---

## Module 12: Identity & Access Management

The trust layer: who is allowed to do what, and how that gets enforced down to the retrieval level.

1. **Authentication vs authorization**
   - Focus: the core distinction — who you are vs what you're allowed to do.
   - Notes to prepare: definitions plus one concrete example of each failing independently.

2. **Understanding JWT tokens**
   - Focus: JWT structure (header/payload/signature) and claims.
   - Notes to prepare: a decoded JWT example with claims annotated.

3. **OAuth grant types**
   - Focus: authorization code, client credentials, and other grant types.
   - Notes to prepare: a grant-type comparison table with use cases.

4. **SAML assertions**
   - Focus: the basic SAML SSO flow.
   - Notes to prepare: a SAML vs OAuth/OIDC comparison.

5. **Mapping Azure AD groups**
   - Focus: group-based role mapping from an identity provider.
   - Notes to prepare: a group-to-role mapping example.

6. **Implementing Role-Based Access Control**
   - Focus: designing a roles/permissions model.
   - Notes to prepare: a sample RBAC schema.
   - Connections: applies the least-privilege principle from Module 3; used directly in OmniGuard's auth flow (Module 15).

7. **Enforcing data-level permissions in retrieval layers**
   - Focus: filtering RAG results by the requesting user's permissions.
   - Notes to prepare: an example of permission-filtered retrieval.
   - Connections: implemented via Module 6's metadata-filtering mechanism.

---

## Module 13: Production AI Security & Guardrails

Hardens everything built so far against the specific ways LLM-based systems get attacked or misused.

1. **Identifying major LLM vulnerabilities**
   - Focus: an overview of the major LLM-specific vulnerability categories.
   - Notes to prepare: a short list of each category with a one-line description.

2. **Prompt injection defenses**
   - Focus: direct vs indirect injection, and mitigation strategies for each.
   - Notes to prepare: a defense checklist.

3. **Jailbreak prevention techniques**
   - Focus: common jailbreak patterns and countermeasures.
   - Notes to prepare: one example jailbreak attempt paired with its mitigation.

4. **Implementing Microsoft Presidio analyzers and anonymizers**
   - Focus: Presidio's detect-then-anonymize pipeline.
   - Notes to prepare: a code snippet showing analyzer + anonymizer usage together.

5. **Redacting sensitive entities (SSN, credit cards, emails)**
   - Focus: built-in PII entity types and recognizers.
   - Notes to prepare: an entity-recognizer configuration example.

6. **Customizing regex patterns for domain-specific PII**
   - Focus: extending Presidio with custom recognizers.
   - Notes to prepare: a custom regex recognizer example.

7. **Reversing masks safely post-generation**
   - Focus: controlled de-anonymization for authorized use cases.
   - Notes to prepare: a reversible-anonymization pattern.

8. **Evaluating false-positive redaction rates**
   - Focus: measuring the impact of over-redaction.
   - Notes to prepare: an evaluation approach outline.

9. **Writing programmable conversational rails using Colang**
   - Focus: NeMo Guardrails' Colang DSL basics.
   - Notes to prepare: a sample Colang rail definition.

10. **Configuring strict input and output filtering pipelines natively in Python**
    - Focus: writing custom guardrail functions outside of NeMo.
    - Notes to prepare: an input/output filter function example.

11. **Enforcing topical boundaries to prevent off-topic chatter**
    - Focus: topical-rail configuration.
    - Notes to prepare: a topical rail example.

12. **Setting up AWS Bedrock managed guardrail configurations via Boto3**
    - Focus: creating Bedrock Guardrails through the API.
    - Notes to prepare: a Boto3 guardrail-creation snippet.

13. **Testing rails against jailbreak libraries**
    - Focus: red-teaming guardrails against known attack sets.
    - Notes to prepare: a testing checklist.
    - Connections: this whole module's output is what gets implemented in OmniGuard (Module 15).

---

## Module 14: AI Observability & Gateway Management

Closes the loop: once deployed, how do you know the system is working, what it costs, and how to keep it reliable.

1. **Idempotency keys for safe tool execution**
   - Focus: preventing duplicate side effects when a call is retried.
   - Notes to prepare: an idempotency-key pattern example.
   - Connections: pairs with Module 5's retry-on-parse-failure topic.

2. **Exponential backoff strategies**
   - Focus: retry timing for transient failures.
   - Notes to prepare: the backoff formula plus a note on jitter.

3. **Centralizing provider API keys via LiteLLM/Portkey**
   - Focus: the LLM-gateway pattern for abstracting multiple providers.
   - Notes to prepare: a gateway configuration example.

4. **Configuring rate limiting and fallback routing**
   - Focus: handling provider rate limits by falling back to a secondary provider/model.
   - Notes to prepare: a fallback-routing configuration example.

5. **Automating LLM-as-a-judge scoring pipelines**
   - Focus: using an LLM to grade other LLM outputs at scale.
   - Notes to prepare: a judge-prompt template.

6. **Calculating RAGAS faithfulness and answer relevance metrics**
   - Focus: what these two RAGAS metrics actually measure.
   - Notes to prepare: formula/intuition for each metric.
   - Connections: builds directly on Module 6's precision/recall groundwork.

7. **Measuring context precision and recall**
   - Focus: the retrieval-specific half of RAGAS metrics.
   - Notes to prepare: definitions plus a worked example calculation.

8. **Creating synthetic benchmark datasets from source documents**
   - Focus: generating QA pairs from source docs for evaluation.
   - Notes to prepare: a synthetic-dataset generation approach.

9. **Tracking evaluation scores across deployment runs**
   - Focus: catching evaluation regressions over time.
   - Notes to prepare: a tracking/dashboard approach outline.

10. **Capturing deep span-level execution traces**
    - Focus: distributed-tracing concepts (spans, trace IDs) applied to LLM apps.
    - Notes to prepare: an example trace structure.

11. **Monitoring granular token costs and endpoint latency**
    - Focus: per-request cost/latency tracking.
    - Notes to prepare: a list of the metrics worth logging per call.
    - Connections: builds directly on Module 5's token-calculation topic.

12. **Debugging multi-step agent reasoning and tool inputs**
    - Focus: using traces to debug an agent's decision chain.
    - Notes to prepare: a debugging workflow using a trace/observability tool.

13. **Creating custom user-session tracking metrics**
    - Focus: designing session-level metrics.
    - Notes to prepare: example custom metric definitions.

14. **Exporting trace data for continuous model improvement**
    - Focus: feeding production traces back into evaluation/fine-tuning loops.
    - Notes to prepare: an export/feedback-loop diagram.
    - Connections: this whole module's tooling is what AuditMesh's dashboards (Module 16) are built from.

---

## Module 15: OmniGuard — Secure AI Integration (Capstone Project 1)

This module doesn't introduce new theory — it's Modules 4, 6, 11, 12, and 13 assembled into one deployed system, plus the FDE consulting process wrapped around it.

1. **Conducting technical discovery and scoping workshops**
   - Focus: the FDE discovery process for understanding a client's real problem.
   - Notes to prepare: a discovery-workshop question checklist.

2. **Defining data classifications**
   - Focus: sensitivity tiers for enterprise data.
   - Notes to prepare: a classification scheme (e.g., public/internal/confidential/restricted).

3. **Drafting architecture SOWs**
   - Focus: the structure of a Statement of Work for a technical proposal.
   - Notes to prepare: an SOW section template.

4. **Preparing ROI presentations for CISO/executives**
   - Focus: framing technical value in business/risk terms for a non-technical executive audience.
   - Notes to prepare: a ROI-presentation outline.

5. **Delivering User Acceptance Testing (UAT) runbooks**
   - Focus: structuring a UAT test plan and sign-off process.
   - Notes to prepare: a UAT runbook template.

6. **Executing mock OAuth 2.0 / RBAC flows**
   - Focus: applying Module 12's auth concepts inside this specific system.
   - Notes to prepare: an end-to-end auth-flow diagram for OmniGuard.
   - Connections: direct application of Module 12.

7. **Constructing Hybrid RAG alongside secure Text-to-SQL for MS SQL databases**
   - Focus: combining Module 6's hybrid search with Module 11's constrained Text-to-SQL in one architecture.
   - Notes to prepare: a combined architecture diagram showing both retrieval paths feeding the same agent.
   - Connections: direct application of Module 6 and Module 11.

8. **Implementing NeMo & Presidio guardrails**
   - Focus: applying Module 13's guardrail stack inside this system.
   - Notes to prepare: a diagram marking where each guardrail sits in the request/response pipeline.
   - Connections: direct application of Module 13.

9. **Finalizing Dockerized FastAPI cloud deployments**
   - Focus: applying Module 4's containerized deployment pipeline to ship OmniGuard.
   - Notes to prepare: a deployment checklist specific to this project.
   - Connections: direct application of Module 4.

---

## Module 16: AuditMesh — Multi-Agent Compliance System (Capstone Project 2)

The second capstone — Modules 9, 10, 11, and 14 assembled into a production-style multi-agent system, plus the operational handoff work an FDE owns after go-live.

1. **Mapping 5-step manual compliance workflows**
   - Focus: understanding the manual process being automated before automating it.
   - Notes to prepare: a workflow diagram of the 5 manual steps.

2. **Identifying Human-in-the-Loop bottlenecks**
   - Focus: pinpointing where automation still needs a human checkpoint.
   - Notes to prepare: notes on which steps in the mapped workflow need HITL and why.
   - Connections: direct application of Module 10's HITL approval topic.

3. **Drafting latency/cost SLAs**
   - Focus: defining measurable service commitments for the system.
   - Notes to prepare: an SLA definition template.

4. **Defining Model Context Protocol trust boundaries**
   - Focus: applying Module 10's MCP security boundaries in this specific system.
   - Notes to prepare: a trust-boundary diagram for AuditMesh's MCP usage.
   - Connections: direct application of Module 10.

5. **Leading operations and training handoffs**
   - Focus: the FDE's responsibility to hand the system off to the client's operations team.
   - Notes to prepare: a handoff documentation checklist.

6. **Developing a LangGraph Multi-Agent Supervisor**
   - Focus: applying Module 9's supervisor pattern concretely to compliance auditing.
   - Notes to prepare: the AuditMesh supervisor graph design (which sub-agents it routes to and why).
   - Connections: direct application of Module 9.

7. **Deploying a custom MCP server for secure Jira ticketing**
   - Focus: combining Module 11's Jira integration with Module 10's MCP server standard.
   - Notes to prepare: a design sketch of the custom MCP server's exposed tools and scoping.
   - Connections: direct application of Module 10 and Module 11.

8. **Building comprehensive token cost/trace dashboards**
   - Focus: applying Module 14's observability stack to this project.
   - Notes to prepare: a dashboard design outline (what's tracked, at what granularity).
   - Connections: direct application of Module 14.

9. **Creating Streamlit/Gradio UIs for human approval workflows**
   - Focus: building the UI layer for the HITL approvals identified earlier in this module.
   - Notes to prepare: a UI component checklist (approve/reject actions, audit trail display).
   - Connections: direct application of Module 10's approval-gate topic.
