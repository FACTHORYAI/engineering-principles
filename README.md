# Facthory Engineering & Architecture Principles

At Facthory, we only build systems that are worth running in production. That means resilient, observable, deterministic, scalable, and brutally cost-efficient. We operate close to real manufacturing floors where noise, latency, and unpredictable environments are the norm. Our architecture must reflect reality—not fantasy.

We aim to build applications that are:

- resilient under load, failure, and bad networks
- maintainable by real engineers, not archaeologists
- deterministic in behavior so debugging is predictable
- designed for scale before scale arrives
- cost-efficient at every layer (storage, GPUs, compute, network)
- compliant by default (GDPR, EU AI Act, auditability)

There is no room for fragile software in an industrial context. We engineer like everything is mission-critical—because in many factories, it is.

## Architecture Philosophy

We prefer loosely coupled, event-driven microservices. Each service owns its data, its lifecycle, and its failure modes. No shared state. No hidden dependencies. No mystery glue code.

This philosophy draws from foundational microservices patterns documented in [Microservices.io](https://microservices.io/) by Martin Fowler and Sam Newman, which establish the core principles of autonomous services, bounded contexts, and domain-driven design. We've learned from real-world implementations at scale, including [Uber's engineering blog](https://eng.uber.com/tag/microservices/) which details lessons from managing 1000+ microservices in production, and [Netflix's tech blog](https://netflixtechblog.com/) which demonstrates resilience patterns, circuit breakers, and async systems that survive at global scale.

We design autonomous services that:

- can deploy independently
- survive dependency failures
- remain responsive under partial degradations
- communicate through async events—not blocking chains
- reflect the business domain (video processing, embedding, retrieval, etc.)

A tightly coupled system is a time bomb. A loosely coupled one is an organism.

## Asynchronous Communication, Always First

Factories are chaotic. Uploads spike. Transcription may take seconds or minutes. Model inference may stall. Synchronous chains collapse under real-world conditions.

We avoid that by defaulting to event-driven workflows. Our event broker is [NATS](https://github.com/nats-io), a cloud-native messaging system that provides always-on messaging with interest-based propagation, persistence with streaming replays, and flexible authentication. NATS is a CNCF incubating project, designed for edge and cloud-native environments—exactly the kind of distributed, resilient infrastructure that industrial systems require.

NATS gives us the foundation for building truly asynchronous systems. Unlike traditional message brokers, NATS provides simple, secure, and performant communications that scale from edge devices to cloud clusters. We leverage NATS for pub/sub messaging, request/reply patterns, and streaming where we need persistence and replay capabilities.

We use asynchronous pipelines because:

- sync calls block threads and create cascading failures
- retries, DLQs, and event logs make failures recoverable
- load can be smoothed out naturally
- workflows become observable end-to-end
- partial failures don't freeze the system

The patterns we follow are informed by the [Google SRE Book's distributed systems patterns](https://sre.google/sre-book/table-of-contents/), which document reliability and failure patterns that microservices must follow. These patterns—retries with exponential backoff, circuit breakers, graceful degradation—are not optional in production systems.

Async is not an optimization. It's survival.

## Service Degradation as a First-Class Behavior

When a dependency fails, we do not break. We degrade gracefully and keep delivering value. This principle is central to building resilient systems, as detailed in the Google SRE Book and demonstrated in Netflix's production systems.

For example:

- If transcription fails → return audio + request reprocess
- If translation fails → keep source language + fallback
- If embedding fails → store metadata and retry later
- If retrieval confidence is low → warn, don't hallucinate
- If a model is slow → serve cached guidance

A degraded output is better than a frozen system. Factories can't wait.

## Low-Tech Coupling

We deliberately choose technologies that age well. This means avoiding proprietary protocols, vendor lock-in, and overly clever abstractions that become technical debt.

Instead of clever coupling, we rely on:

- DNS for service discovery
- HTTP/JSON for universal compatibility
- simple contracts that don't break when teams iterate
- clean, explicit boundaries

Factories don't need fancy. They need reliable.

## RESTful APIs with JSON Payload

We prefer REST-based APIs with JSON payloads. Distributed systems following the REST style have looser coupling between client and server implementations and come with less rigid contracts that don't break when either side makes changes. This makes it easier to build interoperating distributed systems that can be evolved in parallel by different teams while continuing to work.

REST-like APIs with JSON payload are the most widely accepted and used service interfacing style in the internet web service industry. They're simple, debuggable, and work everywhere.

## API First, Human First

APIs are our public contract. They must be designed before code—reviewed by peers, validated against real use cases, and treated as long-lived assets.

Our API culture is informed by industry-leading practices. We follow the [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines), which are used internally across Azure and Microsoft services, providing battle-tested patterns for versioning, error handling, and resource design. We study [Stripe's API Design Principles](https://stripe.com/blog/api-design), widely considered "the best designed APIs on the internet," to understand how to build APIs that developers love to use.

We design APIs using the [OpenAPI (Swagger) Specification](https://swagger.io/specification/) as our standard. API-first means designing here before writing backend code. This approach, detailed in [Red Hat's "API First" guide](https://www.redhat.com/en/topics/api/what-is-api-first), shows how to structure engineering organizations around API contracts. We also reference the [Google API Product Management Guide](https://cloud.google.com/apigee/api-product-management) for defining versioning strategies, SLAs, adoption metrics, and product thinking around APIs.

Our API culture includes:

- OpenAPI specs defined before implementation
- early peer reviews
- stable backward-compatible evolution
- documentation with examples and behavior notes
- predictable patterns across services

APIs must be boring in the best way: obvious, consistent, and trustworthy.

Our APIs should obey [Postel's Law](https://en.wikipedia.org/wiki/Robustness_principle)—a.k.a. "the Robustness Principle": Be conservative in what you send, be liberal in what you accept. APIs must be evolved without breaking any consumers.

## Right-Sized Microservices

We size services around business capabilities, not arbitrary functions. This follows the domain-driven design principles that underpin successful microservice architectures.

Examples:

- Video Processing
- Document Intelligence
- Embedding Engine
- Retrieval Engine
- Learning Path Service
- Issue Management
- Vector Index Service
- Policy Evaluation

A service should be big enough to offer a valid business capability, but small enough to be handled by a team that can be fed by two pizzas (Amazon's Two-Pizza Team rule)—from two to 12 people. In practice, a Two-Pizza Team may be able to own and run a large number of small services, or a smaller number of larger services.

Services must be small enough to understand, large enough to matter.

## Autonomy as Law

A service must be able to stand alone. That means:

- owning its own data
- isolating its failures
- deploying independently
- avoiding shared libraries with hidden logic
- starting up whether dependencies are healthy or not
- never leaking domain concepts into another service's code

A service should run in its own process and be independently deployable. It should start up and be resilient when its dependencies are not available. It should not share its data storage or code repository with any other service, so that changes do not affect other systems. It should not share libraries with other services, unless those libraries are open-source or inner-source and are actively maintained by a community. Shared dependencies may lead to large-scale complexity over time.

A service should not provide a client library containing business logic. The core API and its data model are expressed as REST and JSON.

Autonomy protects velocity. Dependency protects nothing.

## 12-Factor Applications

We build applications following the [12-Factor App methodology](https://12factor.net/), the canonical reference for building modern, scalable applications. The 12 factors provide a methodology for building software-as-a-service apps that are portable, scalable, and maintainable.

We apply these principles with modern containerization in mind, following the [Cloud Native Computing Foundation's guidance on 12-Factor for Containers](https://www.cncf.io/blog/2017/11/30/twelve-factor-app-containerized-microservices/), which shows how to apply 12-factor principles to Kubernetes environments. We also reference [DigitalOcean's guide to modernizing to 12-Factor](https://www.digitalocean.com/community/tutorial_series/the-twelve-factor-app) for practical examples and clear breakdowns.

The 12 factors guide our approach to:

- codebase management (one codebase, many deploys)
- dependencies (explicitly declare and isolate)
- configuration (store in environment, not code)
- backing services (treat as attached resources)
- build, release, run (strictly separate stages)
- processes (execute as stateless processes)
- port binding (export services via port binding)
- concurrency (scale out via the process model)
- disposability (maximize robustness with fast startup and graceful shutdown)
- dev/prod parity (keep development, staging, and production as similar as possible)
- logs (treat logs as event streams)
- admin processes (run admin/management tasks as one-off processes)

These principles ensure our applications are cloud-native, container-ready, and production-hardened from day one.

## Data Architecture Principles

This section is where Facthory is fundamentally different from classic SaaS.

We operate on heavy, multimodal data (video, frames, documents, audio). Handling this data well is not an implementation detail—it's our competitive edge.

### Data Gravity Rules Everything

We minimize data movement:

- direct-to-blob uploads
- no routing video through backend
- processing close to storage
- region-aware compute scheduling

Data that moves slowly creates products that feel slow.

### Metadata Is a Product

We don't treat metadata as decoration. It is the backbone of retrieval quality and explainability.

We maintain metadata for:

- provenance
- timestamps
- machine ID
- operator ID (RBAC-safe)
- validation status
- frame-level evidence

Better metadata = better answers.

### Deterministic Pipelines

We enforce deterministic output for:

- chunking
- speech-to-text
- document parsing
- embedding generation

Same input → same chunks → same retrieval context.

Determinism reduces hallucination risk and makes debugging sane.

### Event-Sourced Knowledge Construction

Nothing is overwritten. Everything is versioned.

We ingest raw knowledge → transform → enrich → index.

The lineage is immutable.

This ensures:

- auditability
- explainability
- compliance
- reproducibility

Factories can't operate on "mystery outputs."

## RAG/KAG Principles

While others play with "chatbots," we treat RAG/KAG as knowledge architecture.

### Chunking Is an Engineering Discipline

Chunk size, boundaries, and modality matter. We engineer chunking based on:

- semantics
- modality
- task type
- retrieval target
- hallucination risk

Bad chunking = bad product.

### Retrieval Is Ranking

Vector similarity alone is not enough. We incorporate:

- metadata filters
- recency bias
- expert validation boosting
- machine context
- source type weighting

RAG is search → ranking → reasoning.

Not "shove text into an LLM."

### KAG (Knowledge-Augmented Generation)

Long-term, we build towards graph-enhanced reasoning:

- relationships between machines
- recurring failure modes
- part sharedness
- shift-level patterns
- domain constraints

This is where Facthory becomes the "brain of the shop floor."

## Agentic Workflows — With Realism

We don't jump on hype. Agentic systems are powerful but dangerous if misapplied.

Agents are useful for:

- orchestrating multi-step workflows
- validating outcomes
- generating structured summaries
- coordinating humans + services

Agents are not permitted to:

- autonomously call production systems
- alter knowledge without human validation
- emit instructions with uncertain accuracy
- rewrite databases
- run unbounded loops

We wrap all agents in:

- strict policy envelopes
- sandboxed toolsets
- determinism guards
- audit logging
- diff validation

AI does not get to "improvise" on real shop floors.

## Security & Privacy Are Non-Negotiable

Industrial environments demand trust.

We enforce:

- TLS everywhere (HTTPS, not HTTP)
- keyless workloads via Managed Identity
- RBAC + ABAC access control
- encrypted blobs + encrypted Postgres
- audit logs for every interaction
- prompt-level redaction
- content validation
- adherence to GDPR and early EU AI Act alignment

Always use TLS and make sure the caller of your service is authenticated and authorized. TLS actually means "HTTPS everywhere, not HTTP."

Security is not a feature. It's a foundation.

## Failure Is Expected. Chaos Is Not.

We engineer predictable failure. This draws from the Google SRE Book's approach to reliability engineering and Netflix's production-tested resilience patterns.

We implement:

- retries with exponential backoff
- idempotent processors
- circuit breakers
- DLQ inspection pipelines
- rollback-first mentality
- blameless (but brutally honest) post-mortems

Whenever possible and reasonable, we make service endpoints [idempotent](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning), so that an operation produces the same result even when it's executed multiple times. This allows clients to safely retry operations in case of timeouts due to service processing or network failures.

If it breaks twice, the engineering team failed.

## General Guidelines

### Stateless

When possible, be stateless. If you can't, persist state outside the address space of the application, for example in a database or through NATS streaming for event-sourced state.

### Immutable

Strive for immutability whenever possible. An object is immutable if its state cannot be modified. Immutable things are automatically thread-safe, without requiring synchronization. Overall, immutability tends to result in fewer bugs and makes it easier to prove a program correct.

## Development Culture

This is where our engineering philosophy comes through strongest.

### We Are Engineers at Heart

We build, test, break, learn, and rebuild.

We don't hide from complexity—we dismantle it.

### Failure Culture

Failure is part of engineering. Avoiding failure is cowardice.

We fail fast, recover decisively, and document so it never happens twice.

### Help Early, Ask Early

Silence kills teams. Collaboration builds momentum.

### Efficiency Over Bureaucracy

No useless meetings. No PowerPoints.

Documentation and execution beat theater.

### Documentation Over Memory

If it matters, it must be written.

No tribal knowledge. No hero bottlenecks.

Document the architecture of your APIs and applications. Make it clear, concise, and current. Use inline documentation for more complex code fragments.

### High Velocity, High Ownership

We move fast, and we clean up after ourselves.

Speed does not justify sloppiness.

### Peer Review

Don't wait until you're done to ask for code review: It's the best way to catch defects early. Create a pull request at the start of your work, not at the end. This pulls people into an ongoing conversation about your code, from Day One.

Code review is expensive in some ways, so get the most out of it. Reviewing code is a great way to learn about style, get help with idioms, and grow as a programmer and reviewer.

Code review can be hard when the culture around it isn't supportive and constructive. It takes practice to learn how to accept code reviews without getting defensive, and to review code without focusing on trivial things. Don't [bike shed](https://en.wikipedia.org/wiki/Law_of_triviality).

Peer review gets easier when you have a good attitude about it. Everybody around you is smart, and you are smart. We're all smart in different ways.

Depending on the team and its codebases, it might be required that at least one person reviews code before it goes live. This is especially true for systems that touch customer or financial data. In general, though, we don't want to focus about when code review is or isn't required: The system works best when people decide on their own that code review is valuable, and seek it out.

Architectural decisions should be made as a team, and the team should ask for help if it's unsure. Embrace open discussions and alternate opinions.

### Quality

Quality is related to mindset, and it's part of engineering. Systems that support industrial manufacturing must be engineered for high quality. Usually this means:

- writing unit tests early on
- mocking external systems so you can test against them while they're not running, and also so that you can simulate various failure scenarios from the service and the network between it
- striving for automation

Automate testing whenever possible. It's not always possible, but life is almost always better if you invest in automated tests of your code. See Martin Fowler's [Testing Strategies in a Microservice Architecture](http://martinfowler.com/articles/microservice-testing/).

We're not going to require you to test your code, but expect your peers to challenge you if you don't. For the most part, a dedicated QA team is a thing of the past. You and your team are responsible for your code's behavior: There's no other safety net.

Years ago, we didn't build systems this way. Now we must. Fortunately, the tooling is pretty amazing.

### Continuous Delivery

Strive for very short release cycles, optimally deploying daily; automating the delivery pipeline makes this possible. Small releases tend to have fewer bugs. Use canary testing for your new deployments to identify problems early.

### Source Code Management

We support GitHub as SCM to check in your code. You might want to use local git hooks for checking references to specifications in commit messages or checks.

## Deployment Principles

We deploy to AKS (Azure Kubernetes Service) with:

- Flux GitOps for declarative infrastructure
- container signing for supply chain security
- progressive rollouts with canary deployments
- isolated namespaces for service boundaries
- region-aware clusters for data locality

We favor containerized application development. Containers provide the isolation, portability, and consistency that 12-Factor applications require. Kubernetes gives us the orchestration layer to manage these containers at scale.

The system must run globally, but operate locally.

## Observability

We require:

- structured logs (treat logs as event streams, per 12-Factor)
- distributed tracing across service boundaries
- metrics with SLOs (Service Level Objectives)
- model drift monitoring for AI/ML systems
- cost visibility at every layer

We use [Prometheus](https://prometheus.io/) for metrics collection and alerting, and [Grafana](https://grafana.com/) for visualization and dashboards. Prometheus provides the time-series database and query language we need to track system health, performance, and business metrics. Grafana gives us the visualization layer to turn those metrics into actionable insights.

If we can't see the system clearly, we can't trust it.

## SaaS-Ready Architecture

Build your services so that it's possible to offer them as a SaaS solution to third parties. In fact, consider any other system a third party with regards to API structure, resilience and service level. This is easier to do than it was a few years ago: cloud platforms push us this way, the Internet model scales, and our security model is geared toward allowing our services to be on the open Internet.

We want to offer services in ways we never imagined or expected. This is part of being a platform. In some cases, this means being multi-tenant from the start.

## The Joy of Engineering

At the end of the day, we do this because engineering is joy.

Solving hard, real industrial problems with elegant systems is meaningful work.

Building the brain of the factory is not hype. It's our craft.

We engineer because we love it.

We ship because factories need it.

We write principles because good teams deserve clarity.

Building software systems can produce substantial existential pleasure. When the conditions are just right, programming is a reliable path to [Flow](https://en.wikipedia.org/wiki/Flow_%28psychology%29): a state almost beyond pleasure. We want to get there, and stay there, and we want you to join us there. We hope these principles help.

## References & Further Reading

### API Design & Standards

- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines) - Used internally across Azure and Microsoft services
- [Stripe API Design Principles](https://stripe.com/blog/api-design) - Widely considered "the best designed APIs on the internet"
- [OpenAPI (Swagger) Specification](https://swagger.io/specification/) - Official specification for API-first design
- ["API First" by Red Hat](https://www.redhat.com/en/topics/api/what-is-api-first) - How to structure engineering orgs around API contracts
- [Google — API Product Management Guide](https://cloud.google.com/apigee/api-product-management) - Defining versioning, SLAs, adoption, monetization

### Microservices & Distributed Systems

- [Microservices.io](https://microservices.io/) - The foundational patterns for modern microservice design by Martin Fowler & Sam Newman
- [Google SRE Book — Distributed Systems Patterns](https://sre.google/sre-book/table-of-contents/) - The reliability and failure patterns microservices must follow
- [Uber Engineering Blog — Microservice Migration & Lessons](https://eng.uber.com/tag/microservices/) - Real cases from 1000+ microservices in production
- [Netflix Tech Blog — Microservices & Resilience Patterns](https://netflixtechblog.com/) - Circuit breakers, service meshes, async systems

### 12-Factor Applications

- [The Original 12-Factor App Manifesto (Heroku)](https://12factor.net/) - The canonical reference
- [DigitalOcean — Modernizing to 12-Factor](https://www.digitalocean.com/community/tutorial_series/the-twelve-factor-app) - Clear breakdown with practical examples
- [Cloud Native Computing Foundation (CNCF): 12-Factor for Containers](https://www.cncf.io/blog/2017/11/30/twelve-factor-app-containerized-microservices/) - How to apply 12-factor to Kubernetes

### Event Broker

- [NATS](https://github.com/nats-io) - The edge & cloud native messaging system we use for asynchronous communication

### Observability & Monitoring

- [Prometheus](https://prometheus.io/) - Time-series database and monitoring system for metrics collection and alerting
- [Grafana](https://grafana.com/) - Open-source analytics and visualization platform for metrics, logs, and traces

### Additional Resources

- [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) - Roy Fielding's dissertation on REST
- [Postel's Law (Robustness Principle)](https://en.wikipedia.org/wiki/Robustness_principle) - Be conservative in what you send, be liberal in what you accept
- [Idempotence](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning) - Making operations safe to retry
- [Testing Strategies in a Microservice Architecture](http://martinfowler.com/articles/microservice-testing/) - Martin Fowler's guide to testing microservices
- [Flow (Psychology)](https://en.wikipedia.org/wiki/Flow_%28psychology%29) - The state of optimal experience

## License

We have published these guidelines under the [Bandpey](LICENSE) license.
