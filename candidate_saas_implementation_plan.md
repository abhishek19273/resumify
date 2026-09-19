# Candidate-Side SaaS: Implementation & Architecture Document

This document outlines the technical architecture, design decisions, and a phase-by-phase implementation plan for the Candidate-Side Resume Generation SaaS. This document is structured to be isolated and contained so that it can be picked up by another agent or developer for immediate execution.

## 1. Architecture & Tech Stack

**Frontend**
* **Framework:** Next.js / React (utilizing a TanStack-based starter kit).
* **Optimization:** Server-Side Rendering (SSR) / Static Site Generation (SSG) for high SEO and GEO performance.

**Backend**
* **Framework:** Python FastAPI (decoupled from frontend).
* **AI Orchestration:** LiteLLM / LangChain to support multiple providers (Gemini, OpenAI).
* **PII Middleware:** Custom sanitizer to replace PII (Email, Phone) with placeholders (e.g., `[Phone Number]`) before LLM processing, and auto-populating post-processing.

**Database & Auth**
* **Relational DB:** PostgreSQL.
* **Vector DB:** Qdrant (or Pinecone) for semantic memory search.
* **Authentication:** Supabase Auth.

**Deployment & Monetization**
* **Cloud Provider:** Google Cloud Platform (GCP). (e.g., Cloud Run for FastAPI, Vercel/Cloud Run for Next.js).
* **Payments:** Stripe (Freemium model with a credit-based system tied to LLM usage).

**Export Engine**
* **PDF Generation:** [Typst](https://typst.app/) or `React-PDF`. These provide deterministic, LaTeX-like exact preview-to-PDF generation without the heavy syntax overhead of traditional LaTeX.

---

## 2. Phase-by-Phase Implementation Plan

### Phase 1: Foundation, Auth & Data Ingestion (Weeks 1-2)
**Goal:** Setup the skeleton, user authentication, and the initial data ingestion pipelines for the candidate's "Memory."

1. **Infrastructure Setup**
   * Initialize Next.js project with TanStack Query/Router.
   * Initialize FastAPI backend with CORS, logging, and Supabase Auth middleware.
   * Provision GCP resources (Cloud SQL for PostgreSQL, Cloud Run).
2. **Authentication & User Profiles**
   * Integrate Supabase Auth for Signup/Login.
   * Create base PostgreSQL schemas for Users, Credits, and Resumes.
3. **Data Ingestion (Onboarding Flow)**
   * **Manual Entry:** Standard forms for Work Experience, Education, Skills.
   * **File Upload:** PDF/Docx parser (using libraries like `PyMuPDF` or `python-docx`) to extract raw text and structure it via an LLM.
   * **LinkedIn Connection:** OAuth integration or manual PDF export parsing from LinkedIn.
4. **PII Anonymization Service**
   * Implement a regex/NLP-based scrubber in FastAPI to swap sensitive PII with tokens (`[PHONE]`, `[EMAIL]`) before saving to the LLM context.

### Phase 2: The "Memory" Engine & AI Core (Weeks 3-4)
**Goal:** Establish how candidate information is stored, retrieved, and queried against Job Descriptions.

1. **R&D / Agentic Task: Memory Structuring**
   * *Assigned Task:* Launch an AI research agent to evaluate the best memory structure (e.g., Knowledge Graphs vs. chunked Vector Embeddings) based on state-of-the-art RAG practices for personal profiles.
2. **Vector DB Integration**
   * Deploy Qdrant/Pinecone.
   * Implement embedding pipelines (e.g., OpenAI `text-embedding-3-small` or Gemini Embeddings) to map candidate experiences.
3. **LLM Orchestration**
   * Integrate LiteLLM to handle routing between Gemini and OpenAI.
   * Build prompts for standardizing raw uploaded data into the decided memory structure.

### Phase 3: High-Fidelity Editor & UX (Weeks 5-6)
**Goal:** Build the core interaction loop where candidates edit and refine their resumes.

1. **Split-Pane Editor Interface**
   * Left Pane: Data blocks, styling controls, and structural elements.
   * Right Pane: Live, deterministic PDF preview (using `React-PDF` or a WASM-compiled Typst engine).
2. **AI Collaboration Features**
   * **Highlight & Refactor:** User highlights text -> right-clicks "Improve" -> AI rewrites inline.
   * **Chat Sidebar:** Context-aware chat where the user can type "Make the tone of my last job more leadership-focused."
3. **Template Engine**
   * Create 2-3 highly customizable, ATS-friendly base templates.
   * Ensure layout and styling configurations (margins, fonts, colors) immediately reflect in the live preview.

### Phase 4: Job Matchmaking & Gap Analysis (Weeks 7-8)
**Goal:** The core "curation" loop based on a specific Job Description.

1. **Job Description (JD) Ingestion**
   * Input field for pasting a JD or JD URL.
2. **Gap Analysis Engine**
   * AI compares the JD against the candidate's Vector/Graph Memory.
   * Identifies missing skills or underrepresented experiences.
3. **Intrusive Clarification (Pop-up Modals)**
   * If the JD requires "Docker" and the memory lacks it, trigger a UI Pop-up Modal: *"The JD requires Docker. Have you worked with this? If yes, briefly explain how."*
   * Feed the user's response back into the Memory Engine and auto-update the targeted resume.
4. **Auto-Curation**
   * Generate a targeted draft utilizing the closest matching experiences from the candidate's memory.

### Phase 5: Monetization & Launch Prep (Week 9)
**Goal:** Secure the platform and enable billing.

1. **Stripe Integration**
   * Setup Stripe Webhooks.
   * Implement the Credit System database logic (e.g., deduct 5 credits for a targeted resume generation, 1 credit for an inline edit).
2. **Freemium Logic**
   * Grant `X` free credits upon Supabase account creation.
3. **SEO & GEO Optimization**
   * Setup Next.js sitemaps, meta tags, and landing page content optimized for search engines.

---

## 3. Recommended Tech Spike / R&D for Implementation Agent
Before writing code for Phase 2, the assigned implementation agent must run a spike on **Memory Structuring**. 
* **Directive:** Compare a purely semantic vector approach (chunking work history) against a Knowledge Graph approach (Nodes: Candidate, Skills, Companies. Edges: Used_At, Worked_At). Report back with the architecture that yields the best context-retrieval for resume tailoring before provisioning the database.
