# Idea Submission — Team Dhruva

---

## Problem Statement

Indian college students face a severe **fragmentation crisis** in managing academic and career opportunities. Critical events — hackathons, internships, workshops, and competitions — are scattered across 15+ WhatsApp groups, Telegram channels, college portals, and social media. Students spend **3–5 hours per week** on repetitive form filling, miss deadlines due to notification fatigue, and struggle with fragmented knowledge bases spread across Google Drive, local storage, and messaging apps. Furthermore, **67% of students report missing important deadlines**, and most lack access to expensive AI-powered productivity tools (ChatGPT Plus at ₹1,650/month, Notion AI at ₹825/month). Existing tools also require uploading sensitive personal data — transcripts, resumes, and academic records — to foreign cloud servers, raising serious **privacy and data sovereignty concerns** for Indian students.

**Who it impacts:** 40+ million college students across India — from IITs and NITs to Tier 2 and Tier 3 institutions — who actively participate in hackathons, skill development, and placement preparation.

---

## Motivation

The motivation behind Vidyan stems from a firsthand understanding of the problem. As students at IIT Bombay, our team of 4 engineers experienced this fragmentation daily — juggling dozens of WhatsApp groups, manually filling the same registration forms repeatedly, and losing track of opportunities buried in chat noise.

**Key gaps we identified:**

1. **No unified platform** exists that combines event discovery, knowledge management, task organization, and autonomous application into a single system.
2. **AI tools are cloud-dependent** — every major AI productivity tool (ChatGPT, Notion AI, Gemini) requires sending personal data to foreign servers. Indian students, governed by the **DPDP Act 2023**, deserve a privacy-first alternative.
3. **WhatsApp is the primary channel** — with 500M+ users in India, WhatsApp groups are where opportunities are shared first, yet no tool taps into this channel for automated event discovery.
4. **Cost is a barrier** — most students cannot afford ₹800–1,650/month for AI tools. A free, local-first alternative can democratize AI-powered productivity for millions.

The opportunity is clear: build an AI-native operating system that is **free, private, offline-capable, and designed for India's unique communication patterns**.

---

## Application

**Real-World Use Case:** Vidyan is an AI-native student operating system that automates the entire lifecycle from **opportunity discovery → knowledge synthesis → application submission**.

**Target Users:**
- Undergraduate and graduate students in Indian universities (Tier 1, 2, and 3)
- Students preparing for hackathons, competitive exams, and placement drives
- Demographics: 18–25 years old, tech-savvy, mobile-first users with limited budgets

**Where it will be applied:**

1. **Automated Event Discovery:** Vidyan scans WhatsApp groups using keyword filters (hackathon, workshop, competition, register) and extracts event details (title, date, URL) using GenAI — surfacing opportunities students would otherwise miss.

2. **Autonomous Form Filling (AI Agent):** When a student selects an event, an AI-powered browser agent navigates to the registration page, analyzes the form structure, maps the student's profile to form fields, fills the form, captures a screenshot for human review, and submits upon confirmation — saving 3–5 hours/week.

3. **Agentic Knowledge Base:** Students upload PDFs, notes, and research papers. The system chunks documents, generates embeddings, and enables semantic search with cross-encoder reranking — finding answers by meaning, not just keywords.

4. **Research Copilot:** A citation-aware research assistant that decomposes queries into sub-questions, performs multi-query vector search across the student's knowledge base, and returns structured research briefs with [1], [2], [3] source citations and confidence scores.

5. **Task Management:** An Eisenhower matrix with AI-powered task breakdown — turning vague goals into actionable subtasks.

---

## Proposed Method

Vidyan is built on **three technical pillars**, each leveraging GenAI at its core:

### Pillar 1: Local-First AI Reasoning
- **Primary Model:** DeepSeek R1 0528 Qwen3 8B (8.2B parameters, 4-bit quantized) running locally on the student's GPU via **Ollama** (localhost:11434).
- **Why this model:** Distilled from DeepSeek R1 (671B params), it retains chain-of-thought reasoning while fitting in 5.7GB VRAM on consumer GPUs (RTX 3060+). Zero API cost, zero data exfiltration.
- **Fallback:** OpenRouter API (deepseek/deepseek-r1-0528) proxied through server-side routes when Ollama is unavailable.
- **Inference pipeline:** User Query → Next.js Server Action → Ollama HTTP API → DeepSeek R1 → Strip `<think>` reasoning tokens → Return response.

### Pillar 2: Production-Grade RAG Pipeline (Agentic Retrieval-Augmented Generation)
- **Stage 1 — Bi-Encoder Retrieval:** Documents are chunked (512 tokens, 50-token overlap), embedded using **Xenova/all-MiniLM-L6-v2** (384-dim, WebGPU-accelerated), and stored in **pgvector** (PostgreSQL vector extension running in-browser via PGlite WASM). Queries retrieve top-20 nearest neighbors via cosine similarity.
- **Stage 2 — Cross-Encoder Reranking:** Candidates are reranked using **Xenova/ms-marco-MiniLM-L-6-v2** (pairwise [query, passage] scoring), yielding 15–20% precision improvement over bi-encoder alone.
- **Stage 3 — Contextual Compression:** Top-5 results are compressed to 3 most relevant sentences per chunk.
- **Stage 4 — LLM Synthesis:** Compressed context is sent to DeepSeek R1 with strict JSON prompts, producing citation-aware answers.

### Pillar 3: Autonomous Browser Agent
- **Stagehand** (AI-powered Playwright wrapper) navigates to event registration URLs.
- **Google Gemini** analyzes form structure and detects field types.
- **AI Reasoning** maps the student's profile (stored in PGlite) to form fields.
- **Human-in-the-Loop:** Full-page screenshot is captured before submission for user review and confirmation.

### Architecture Stack
- **Frontend:** Next.js 16, React 19, Shadcn UI, Tailwind CSS 4, Framer Motion
- **Database:** PGlite (PostgreSQL compiled to WASM) with pgvector, persisted in IndexedDB
- **ORM:** Drizzle ORM with type-safe schemas
- **ML Runtime:** @huggingface/transformers (ONNX Runtime Web, WebGPU backend)
- **Parallelism:** Web Workers for embedding and reranking (non-blocking UI)

---

## Datasets / Data Source

Vidyan uses **pre-trained models** (no custom training required) and **user-generated data** (fully local):

| Data Source | Type | Availability | Purpose |
|------------|------|-------------|---------|
| **User's WhatsApp Groups** | Real-time messages | Live (via whatsapp-web.js) | Event discovery and extraction |
| **User-Uploaded PDFs/Documents** | Text (PDF, MD, TXT) | User-provided | Knowledge base for RAG pipeline |
| **Student Profile Data** | Structured (name, education, projects, experience) | User-entered | Form filling by autonomous agent |
| **MS MARCO Passage Ranking** | 8.8M passages (pre-trained) | Public dataset | Embedding model (all-MiniLM-L6-v2) pre-training |
| **MS MARCO with Hard Negatives** | Pairwise relevance labels (pre-trained) | Public dataset | Cross-encoder (ms-marco-MiniLM-L-6-v2) pre-training |
| **DeepSeek R1 Training Corpus** | 14.8T tokens (pre-trained) | Open-weight model | LLM reasoning (multilingual, code, reasoning) |

**Key point:** All runtime data stays on the user's machine. No data is sent to external servers (except optional OpenRouter fallback, which is opt-in).

---

## Experiments

We will validate Vidyan across **four dimensions**:

### 1. RAG Pipeline Quality
| Metric | Target | Method |
|--------|--------|--------|
| Retrieval Precision@5 | > 85% | MS MARCO benchmark on bi-encoder |
| Reranking MRR@10 | > 0.38 | MS MARCO benchmark on cross-encoder |
| Answer Relevance | > 85% | Human evaluation (20 students, 50 queries each) |
| Citation Accuracy | > 90% | Manual verification: do citations match the answer? |
| Hallucination Rate | < 5% | Grounding check: is every claim traceable to retrieved context? |

### 2. Agent Performance
| Metric | Target | Method |
|--------|--------|--------|
| Form Fill Success Rate | > 80% | Test on 50 real hackathon registration forms (Devfolio, Lu.ma, Google Forms, Unstop) |
| Field Mapping Accuracy | > 90% | Compare agent-filled values to ground truth (student profile) |
| End-to-End Time | < 30s | Measure from "Auto-Apply" click to screenshot capture |

### 3. System Performance
| Metric | Target | Method |
|--------|--------|--------|
| Vector Search Latency | < 100ms | Benchmark on 100K chunks in PGlite pgvector |
| Cross-Encoder Reranking | < 500ms | 20 candidates on WebGPU |
| LLM Inference (100 tokens) | < 2s | RTX 4070 8GB VRAM, 4-bit quantized |
| Document Chunking (50-page PDF) | < 3s | Web Worker parallelization |

### 4. User Impact Study
| Metric | Target | Method |
|--------|--------|--------|
| Time Saved per Week | > 3 hours | Before/after survey with 50 students |
| Missed Deadlines Reduction | > 60% | Track event participation before/after Vidyan adoption |
| Event Participation Increase | > 40% | Compare hackathon registrations over 4 weeks |

---

## Novelty and Scope to Scale

### What Makes Vidyan Unique

1. **First student platform with local AI reasoning.** No existing student tool runs a full LLM (8.2B params) on the user's GPU. Vidyan delivers GPT-class reasoning with zero cloud dependency and zero cost.

2. **Production-grade RAG in the browser.** While most RAG implementations use basic vector search, Vidyan implements a **4-stage pipeline** (bi-encoder → cross-encoder reranking → contextual compression → LLM synthesis) entirely in-browser using WebGPU-accelerated inference.

3. **Autonomous browser agent with human-in-the-loop.** No student platform offers AI-powered form filling with screenshot review. This is a novel combination of Stagehand browser automation + Gemini form analysis + profile-to-field AI reasoning.

4. **In-browser PostgreSQL with vector search.** PGlite (Postgres compiled to WASM) with pgvector running inside the browser is a cutting-edge approach that eliminates server round-trips and enables true offline-first architecture.

5. **WhatsApp-first event discovery for India.** No existing platform scans WhatsApp groups for event extraction — yet WhatsApp is the #1 channel where Indian students discover opportunities.

### Scope to Scale

| Dimension | Current (MVP) | Scale Target |
|-----------|--------------|-------------|
| **Users** | Single-user local app | 40M+ Indian college students |
| **Knowledge Base** | 10,000 docs/workspace | 100K+ documents with IVFFlat indexing |
| **Languages** | English | Hindi, Tamil, Telugu, Bengali, Marathi |
| **Platform** | Web (browser) | PWA + React Native mobile app |
| **Deployment** | Fully client-side | AWS (S3 + CloudFront + Lambda@Edge + RDS pgvector) |
| **Authentication** | None (local) | Clerk/Auth.js with role-based access |
| **Collaboration** | Solo | Team workspaces with real-time sync |
| **Pricing** | Free forever (local) | Free tier + ₹99/month cloud + ₹299/month premium |
| **Partnerships** | None | University placement cells, corporate sponsors |

**Scaling path:** Vidyan's architecture is designed for progressive scaling — from a fully client-side MVP (zero server costs) to a multi-region AWS deployment (Mumbai + Hyderabad) with CloudFront CDN, RDS PostgreSQL, and SageMaker model hosting. The local-first architecture means the free tier will always remain free, while cloud features unlock collaboration and advanced analytics for premium users.

**Impact at scale:** If even 1% of India's 40M college students adopt Vidyan, that's **400,000 students saving 3+ hours per week** — equivalent to **62M+ hours of productivity recovered annually**, at zero cost to the student.

---

*Team Dhruva — IIT Bombay | AI for Bharat 2026*
