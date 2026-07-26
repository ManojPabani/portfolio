# MCP SQL Console

> A natural-language-to-SQL console and Model Context Protocol server that lets developers and AI tools (Cursor, Claude Desktop) query Microsoft SQL Server safely, in plain English or raw SQL, without hand-writing joins.

## Overview

MCP SQL Console is a production-ready monorepo that turns a Microsoft SQL Server / LocalDB instance into a queryable resource for both humans and AI agents. It exposes one shared database service layer through three surfaces: a stdio **MCP server** (for Cursor / Claude Desktop tool calling), a **REST HTTP API**, and a **React console UI** — so the same read-only query engine, caching, and safety guarantees back every entry point.

The core problem it solves is making ad-hoc data access safe and fast for non-DBA users and AI assistants: instead of exposing raw database credentials or unrestricted `EXEC` access, the system introspects the live schema, generates parameterized, read-only SQL from a natural-language question, and returns rows plus the SQL that produced them for full transparency. A layered local-first routing strategy (deterministic rule-based SQL generation before any LLM call) keeps common questions fast and free, escalating to an LLM only for open-ended, multi-hop, or ambiguous phrasing.

Impact: eliminates the need for end users to know SQL or the schema to answer questions like "Employees with Departments" or "Attendance where Status is Present," while keeping every execution path — console, HTTP, and MCP — restricted to read-only `SELECT`/`WITH` statements with parameterized inputs, protecting the underlying database from destructive or injection-based misuse.

## My Responsibilities

- Architected the end-to-end system as an npm-workspaces monorepo (`server` + `client`) sharing a single database service layer across three access surfaces (MCP stdio, HTTP API, React UI), avoiding logic duplication.
- Designed and implemented the natural-language-to-SQL pipeline: schema introspection → local rule-based SQL generation → rich multi-table "dossier" query builder → LLM fallback → execution, with explicit escalation heuristics (`shouldEscalateToLlm`, `isComplexQuestion`) to minimize LLM dependency and cost.
- Built a heuristic query planner (`richQuery.ts`) that dynamically infers foreign-key join paths, name columns, and multi-hop joins (e.g. Employees → Departments → Locations, Employees → EmployeeProjects → Projects) purely from live schema metadata, without hardcoded table mappings.
- Owned performance engineering for the NL pipeline: single round-trip schema introspection (eliminating N+1 queries), a 3-tier TTL cache (schema/answer/plan), and in-flight request de-duplication for concurrent identical questions — reducing redundant LLM calls and DB round-trips.
- Implemented a hand-rolled native connection pool (`SqlServerService`) over `msnodesqlv8` for Windows-authenticated LocalDB/SQL Server access, including named-parameter-to-ODBC-placeholder binding, connection acquire/release/waiter queuing, and graceful pool shutdown.
- Enforced a security boundary (`assertReadOnlySql`) applied uniformly across the console, HTTP API, and MCP tools: single-statement enforcement, `SELECT`/`WITH`-only allowlisting, and keyword blocklisting (`INSERT`, `DROP`, `EXEC`, `xp_`, etc.) to guarantee read-only access regardless of entry point.
- Defined and exposed the MCP tool contract (`ask_natural_language`, `execute_sql`, `list_tables`, `list_columns`, `get_table_records`, `sql_health`, predefined-query catalog) plus a mirrored REST API, keeping both interfaces backed by identical business logic in `toolHandlers`.
- Built the React/TypeScript/Tailwind console UI supporting dual query modes (natural language and raw SQL), live schema-driven suggestion chips, client-side search/sort/pagination, and connection-health display.
- Authored setup and operational documentation (README) covering LocalDB/Windows Auth configuration, ODBC driver prerequisites, SQL-authentication fallback, and MCP client (Cursor) configuration for engineers onboarding to the project.

## Tech Stack

`TypeScript` `Node.js 20+` `React 19` `Vite` `Express 5` `Zod`
`Microsoft SQL Server / LocalDB` `msnodesqlv8 (native ODBC driver)` `T-SQL`
`Model Context Protocol (MCP SDK)` `OpenAI API (gpt-4o-mini, pluggable LLM fallback)`
`Tailwind CSS 4` `npm workspaces (monorepo)` `tsx` `Windows Authentication / ODBC 17-18`
`REST API design` `Connection pooling` `TTL caching` `Read-only SQL guardrails`

## GitHub
https://github.com/ManojPabani/mcp-sql-console

____________________________________________________________________________________________________________________________________________________________________________________________________________


#  IntelliRAG
> An enterprise-grade Retrieval-Augmented Generation (RAG) assistant with a ChatGPT-like UX, letting users chat over their own uploaded documents with cited, streaming AI answers.

## Overview
IntelliRAG is a full-stack, production-oriented RAG platform that lets users upload documents (PDF, DOCX, TXT, Markdown) and converse with an AI assistant that answers strictly from their own indexed content, complete with source citations. It solves the common enterprise problem of trustworthy, grounded LLM answers over private knowledge bases rather than relying on a model's raw training data.

The system pairs a FastAPI backend implementing a hybrid retrieval pipeline (dense Voyage embeddings + BM25 lexical search, fused via Reciprocal Rank Fusion, diversified with MMR, and refined with an optional reranker) with Claude-powered streaming chat over Server-Sent Events. A React/TypeScript frontend delivers a polished, ChatGPT-style conversational experience with authenticated, multi-tenant document scoping.

Built solo end-to-end across 7 completed phases (setup → auth/data layer → document ingestion → vector retrieval → LLM streaming chat → frontend UX → containerization/testing/docs), the project also includes a documented before/after upgrade of the retrieval architecture, demonstrating iterative, measurable improvement to answer relevance and tenant isolation.

## My Responsibilities
- Designed and built the entire system solo, end-to-end, across both backend and frontend, following a phased roadmap from initial scaffolding through Dockerized production deployment.
- Architected the backend using Clean Architecture and SOLID principles (API → services → repositories → infrastructure), with a dependency-injection composition root and swappable providers for embeddings, vector store, and database, exposed entirely through configuration.
- Owned the RAG subsystem end-to-end: document loaders/chunking, hybrid dense+BM25 retrieval, RRF fusion, MMR diversity, Voyage reranking, query rewriting for follow-ups, and citation-aware prompt construction.
- Led a retrieval architecture upgrade (documented in a formal before/after comparison) migrating from a local MiniLM/FAISS-only pipeline to a hybrid Voyage-embeddings + BM25 + rerank pipeline.
- Implemented secure, user-scoped JWT authentication (access/refresh tokens, bcrypt hashing) and enforced per-user document ownership to prevent cross-tenant data leakage.
- Built real-time streaming chat via Server-Sent Events, including fixes for CRLF frame parsing and non-blocking stream initialization.
- Designed the ChatGPT-style frontend UX: collapsible sidebar, conversation history, drag-and-drop document management, and a custom dark/light design system.
- Drove performance/reliability improvements: lazy-loaded/warm-started embedding models, SQLite WAL mode, correct UTC datetime handling.
- Set up testing (pytest/Vitest), Alembic migrations, and multi-stage Docker builds with Nginx reverse proxy and health checks, orchestrated via Docker Compose.
- Authored architecture diagrams, API references, and an old-vs-new retrieval design comparison document.

## Tech Stack
Python TypeScript FastAPI React 19 Vite LangChain SQLAlchemy (async) Alembic SQLite/PostgreSQL-ready FAISS Anthropic Claude API Voyage AI sentence-transformers BM25 MUI TanStack React Query Docker & Docker Compose Nginx JWT/bcrypt dependency-injector pytest Vitest SSE

## Github
https://github.com/ManojPabani/IntelliRAG
__________________________________________________________________________________________________________________________________________



# AI-Assisted Workflow Automation & Case Management Platform

> An enterprise-grade AI-assisted workflow automation and case management platform that governs complex, multi-party business review processes for a global professional services firm.

## Overview

The client's case review and approval processes historically relied on email chains, manual handoffs, and disconnected tools, creating delays, audit gaps, and inconsistent outcomes. I contributed to building a unified, intelligent platform that automates and governs these processes end-to-end, combining structured workflow automation, human-in-the-loop approval gates, and AI agent execution into a single governed system.

The platform operates as an event-driven workflow engine: administrators define versioned workflow templates, each submitted case becomes a tracked workflow instance, and AI-generated outputs are always routed to human reviewers for approval, rejection, or feedback before progressing — ensuring AI augments rather than replaces professional judgment. The result is measurable gains in operational efficiency, auditability, and regulatory compliance across the client's service delivery lifecycle.

## My Responsibilities

- Owned development of REST APIs exposing case, workflow, and AI-unit data through a secure, OData-enabled API gateway for front office and back office clients
- Contributed to the AI Agent Orchestrator, enabling any AI service (LLMs, RPA bots, custom ML models) to be plugged into workflow steps without re-engineering core platform logic
- Designed and implemented human-in-the-loop approval workflows, capturing reviewer feedback and routing it back to AI units for iterative refinement
- Built real-time notification features using SignalR to keep participants informed of workflow status changes without delay
- Implemented document upload handling integrated with automated security threat scanning prior to workflow entry
- Enforced role-based access control and row-level security to ensure users could only access authorized cases and capabilities
- Built audit-trail logging for AI reasoning traces, tool invocations, guardrail events, and human decisions, supporting full compliance and traceability
- Collaborated with administrators and stakeholders on workflow versioning and publishing capabilities used to define and update governed process templates
- Supported deployment and operational monitoring using cloud-native telemetry and logging practices

## Tech Stack

`C#` `.NET` `ASP.NET Core` `Event-Driven Architecture` `OData` `SignalR` `Azure SQL Database` `Azure Service Bus` `Azure Event Grid` `Azure Cache for Redis` `Azure Blob Storage` `Azure Active Directory` `Microsoft Defender for Cloud` `Azure Key Vault` `Azure Virtual Network / Private Endpoints` `Azure API Management` `Azure Application Insights`

____________________________________________________________________________________________________________________________________________________________________________________________________________

# AI-Powered Internal Knowledge & Productivity Platform

> An enterprise-grade AI-powered conversational platform that accelerates report drafting and knowledge discovery for staff at a global professional services firm.

## Overview

Knowledge workers at the client spent significant time drafting reports, researching precedents, and navigating large, fragmented document archives — work that was slow, inconsistent, and diverted time from higher-value tasks. I contributed to building a single, governed AI workspace that lets staff generate brand-compliant written content instantly, query the firm's internal document archive in natural language, analyze uploaded client files with AI, and share reusable AI configurations across teams.

The platform is a React single-page application backed by a secure ASP.NET Core API, with all AI inference routed through a load-balanced Azure OpenAI gateway to remain performant during high-demand periods. Its defining capability is a persona system — curated, shareable AI configurations tailored to specific professional tasks — combined with Retrieval-Augmented Generation that grounds responses in the firm's internal knowledge base with source citation, delivering measurable productivity gains while maintaining firm-wide governance and brand consistency.

## My Responsibilities

- Contributed to the ASP.NET Core REST API backing the React SPA, supporting real-time, token-streamed AI chat responses
- Owned development of the persona system, including creation, customization, categorization, and peer-to-peer sharing workflows with accept/reject handling
- Implemented a Retrieval-Augmented Generation (RAG) integration to ground AI responses in indexed internal document collections, including source-cited answers
- Built document upload and analysis features, including OCR-based text extraction and AI-driven question-answering over processed content
- Developed persistent chat history functionality, enabling users to resume prior conversations with full context restored
- Implemented PDF export functionality using templated layouts for brandable, client-ready deliverables
- Built scheduled background jobs enforcing configurable data-retention and automated purge policies, including proactive user notifications
- Implemented load-distribution logic across multiple AI model endpoints to sustain performance during organization-wide demand spikes
- Operated within strict enterprise security constraints, including SSO authentication, centralized secrets management, and private network isolation

## Tech Stack

`C#` `.NET 8` `ASP.NET Core` `React` `Azure OpenAI (GPT models)` `Azure API Management` `Azure AI Search (RAG)` `Azure Document Intelligence (OCR)` `Azure Cosmos DB` `Azure Blob Storage` `Azure Cache for Redis` `Azure Functions` `Microsoft Entra ID (Azure AD)` `Microsoft Graph API` `Azure Key Vault` `Azure Virtual Network / Private Endpoints` `Azure Application Insights + Log Analytics` `Liquid Templating`

____________________________________________________________________________________________________________________________________________________________________________________________________________


# Multi-Vendor Network Security Policy Orchestration Platform

> An air-gapped enterprise platform that unifies multi-vendor firewall management into one console with automated rule cleanup, compliance reporting, and migration analysis.

## Overview

Enterprises often manage firewalls across multiple incompatible vendors (Cisco, VMware NSX-T, Check Point, Cilium/Isovalent), making it hard to see what's actually enforced, clean up outdated rules, or migrate between systems. This platform brings all of that into a single console with a normalized data model, plus built-in tools for rule optimization, compliance reporting, and migration simulation — fully air-gapped for regulated industries like defense, government, and finance.

## My Responsibilities

- Built vendor integration modules (login, sync, CRUD, publish) using a shared, extensible contract
- Developed rule-analysis logic to detect shadowed, redundant, and unused firewall rules at scale
- Built compliance-scoring features mapped to SOC 2, ISO 27001, NIST 800-53, PCI-DSS, and HIPAA
- Implemented migration/simulation logic for moving policies between vendors and to Kubernetes
- Built interactive UI features in Angular, including a topology/flow visualization canvas
- Implemented RBAC, audit logging, and multi-tenant data isolation
- Designed for strict air-gap requirements: no telemetry, no external calls, signed update packages
- Replaced cloud-native services with self-hosted equivalents (background jobs, real-time updates, caching)
- Packaged the app as a self-contained OVA appliance for on-prem deployment

## Tech Stack

`C#` `.NET 10` `ASP.NET Core` `Angular 19` `PostgreSQL` `Hangfire` `SignalR` `RBAC/JWT` `Multi-Tenant Architecture` `Ubuntu/nginx/systemd` `Jenkins CI/CD` `Cisco` `VMware NSX-T` `Check Point` `Cilium/Kubernetes`

____________________________________________________________________________________________________________________________________________________________________________________________________________

# Field Data Reporting Solution for Tracking Deployed Assets
 
> A customized ServiceNow portal for real-time monitoring, location tracking, and activity visualization of deployed field assets.
 
## Overview
 
Field teams needed a way to monitor and track deployed assets in real time, including location, direction, and activity status, along with support for round counting and OTA firmware updates. I helped build a customized ServiceNow portal to deliver this reporting and visualization capability, tailored to client-specific workflows and integrated with mobile applications via REST APIs.
 
## My Responsibilities
 
- Configured ACLs to manage role-based security across the platform
- Developed workflows, flows, business rules, UI pages, layouts, record producers, catalog items, and widgets per client requirements
- Built custom UI actions to override default behavior, and applied validations/customizations via business and assignment rules
- Wrote scripts and client scripts, and developed custom fields, pages, and widgets
- Created database views and nested queries to power data visualization
- Integrated Highcharts for dynamic, real-time charting
- Developed and exposed REST APIs for the mobile team to enable cross-platform integration
- Performed deployments and conducted smoke testing in production environments
## Tech Stack
 
`ServiceNow` `JavaScript` `Client Scripts` `Business Rules` `ACLs` `REST APIs` `Highcharts` `Database Views` `Workflows/Flows` `Widgets & Catalog Items`


____________________________________________________________________________________________________________________________________________________________________________________________________________

# Opportunity Tracking & Bid Awarding Platform

> A cross-platform system that helps companies increase their chances of winning opportunities through real-time tracking and notifications.

## Overview

The platform was built to help companies track and act on business opportunities more effectively, improving their chances of being awarded. It included a .NET Core backend, a Node.js-based web client, and native Android/iOS mobile apps, with Firebase powering real-time notifications whenever opportunity data changed.

## My Responsibilities

- Analyzed business requirements to ensure accurate feature implementation
- Identified and fixed issues in existing features to improve system stability
- Built REST APIs to expose data to the web client
- Collaborated with QA to automate API testing and improve reliability
- Coordinated with mobile and web teams to deliver required APIs
- Optimized and merged database collections to reduce Azure costs and improve processing efficiency

## Tech Stack

`.NET Core` `Angular` `Node.js` `Microsoft Azure` `Cosmos DB` `Firebase` `REST APIs` `Android` `iOS`

____________________________________________________________________________________________________________________________________________________________________________________________________________


# Networking Appliances Management System – Web Application

> An enterprise web application for creating and managing network objects like routers, switches, load balancers, and firewalls on the fly.

## Overview

The application allowed customers to dynamically create and manage network objects — including routers, switches, load balancers, virtual machines, security groups, and distributed firewalls — through a centralized web interface.

## My Responsibilities

- Built REST APIs to expose data to web clients
- Developed front-end features using Razor, jQuery, and JavaScript
- Performed functional testing to ensure bug-free delivery to the client
- Worked on a migration tool to move data between virtual machines
- Participated in weekly client sync-up calls to discuss status and identify new tasks

## Tech Stack

`ASP.NET MVC` `.NET Core` `Razor` `Entity Framework` `JavaScript` `jQuery` `PostgreSQL`

____________________________________________________________________________________________________________________________________________________________________________________________________________

# Hospital ERP Solution – HRMS Module

> A web-based ERP system for a well-renowned hospital, with a focus on the HRMS module.

## Overview

The ERP solution supported multiple operational modules for a large hospital, streamlining internal workflows and data management. My work focused on the HRMS module, involving close collaboration with the client and an offshore team to design and deliver the solution.

## My Responsibilities

- Attended daily client meetings to understand the existing system and identify new tasks
- Collaborated with an offshore team to develop the ERP solution
- Built REST APIs to expose data to the web client
- Created a custom front-end framework using jQuery and JQWidgets to improve the UI
- Coordinated with QA to automate API testing and improve stability
- Worked with the business analyst to review existing workflows and suggest improvements
- Migrated data from the legacy system to the new ERP platform

## Tech Stack

`ASP.NET Web API` `HTML` `JavaScript` `jQuery` `JQWidgets` `Entity Framework` `SQL Server`

