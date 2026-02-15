# Requirements Document: Vidyan - AI-Native Student Operating System

## Introduction

Vidyan is a comprehensive AI-native operating system designed specifically for Indian students to address the fragmentation crisis in academic and career management. The platform consolidates event discovery, knowledge management, task organization, and autonomous application submission into a unified, privacy-first system powered by local AI reasoning.

## Glossary

- **System**: The Vidyan platform (web application)
- **Student**: The primary user of the platform (college/university students in India)
- **Event**: Hackathons, workshops, competitions, meetups, or conferences
- **Knowledge_Base**: The RAG-powered document repository with semantic search
- **Agent**: The autonomous Stagehand-powered browser automation system
- **Local_AI**: DeepSeek R1 0528 Qwen3 8B model running via Ollama
- **PGlite**: In-browser PostgreSQL database with pgvector extension
- **RAG_Pipeline**: Retrieval-Augmented Generation with bi-encoder + cross-encoder reranking
- **WhatsApp_Bridge**: Integration layer for scanning WhatsApp group messages
- **Research_Copilot**: AI-powered research assistant with citation-aware output
- **Workspace**: Isolated data container for organizing documents, tasks, and events

## Problem Statement

### The India-Specific Problem

Indian students face a unique set of challenges in managing their academic and career opportunities:

1. **Information Overload**: Critical opportunities (hackathons, internships, workshops) are scattered across 15+ WhatsApp groups, Telegram channels, college portals, and social media
2. **Missed Deadlines**: 67% of students report missing important deadlines due to notification fatigue and poor organization
3. **Repetitive Form Filling**: Students spend 3-5 hours per week filling identical registration forms with the same personal information
4. **Knowledge Fragmentation**: Notes, PDFs, and learning resources are scattered across Google Drive, local storage, and messaging apps
5. **Limited Computing Resources**: Many students lack access to high-end cloud services or paid AI tools
6. **Privacy Concerns**: Reluctance to upload personal data (transcripts, resumes) to foreign cloud services

### Target Audience

- **Primary**: Undergraduate and graduate students in Indian universities (Tier 1, 2, and 3 institutions)
- **Secondary**: Students preparing for competitive exams, hackathons, and placement drives
- **Demographics**: 18-25 years old, tech-savvy, mobile-first users with limited budgets

### Market Size and Impact Potential

- **Total Addressable Market**: 40+ million college students in India
- **Serviceable Market**: 15 million students actively participating in hackathons, competitions, and skill development programs
- **Impact Metrics**:
  - Save 3-5 hours per week per student (time spent on form filling and information search)
  - Increase event participation by 40% through automated discovery and application
  - Reduce missed opportunities by 60% through intelligent deadline tracking

## Solution Overview

### High-Level Description

Vidyan is a full-stack, AI-native operating system that runs entirely in the browser with optional local AI acceleration. It combines:

1. **Automated Event Discovery**: WhatsApp/Telegram message scanning with AI-powered event extraction
2. **Autonomous Application Agent**: Browser automation that fills registration forms with human-in-the-loop review
3. **Agentic Knowledge Base**: RAG-powered document management with semantic search and cross-encoder reranking
4. **Research Copilot**: Citation-aware research assistant for academic work
5. **Local AI Reasoning**: DeepSeek R1 8B model running on user's GPU via Ollama (zero cloud dependency)
6. **In-Browser Database**: PGlite (PostgreSQL WASM) with pgvector for zero-latency data access

### Key Features

1. **WhatsApp Event Intelligence Pipeline**: Automatically scans group chats for event announcements
2. **AI Auto-Apply with Human Review**: Agent fills forms, captures screenshot, waits for user confirmation
3. **Semantic Knowledge Base**: Upload PDFs/documents, search by meaning, get AI summaries
4. **Research Copilot**: Multi-query decomposition, workspace-scoped retrieval, citation-aware briefs
5. **Task Force**: Eisenhower matrix with AI task breakdown
6. **Focus Mode Editor**: Distraction-free writing with AI assistance
7. **Command Center**: Global command palette (⌘+K) for instant access

### Unique Value Proposition (USP)

**What makes Vidyan different:**

1. **100% Privacy-First**: All AI reasoning happens locally on user's GPU - zero data exfiltration
2. **India-Optimized**: WhatsApp-first design (primary communication channel for Indian students)
3. **Zero Backend Latency**: In-browser Postgres eliminates server round-trips
4. **Autonomous Agent**: First student platform with browser automation for form filling
5. **Production-Grade RAG**: Cross-encoder reranking (not basic keyword search)
6. **Mobile-Optimized**: Works on low-bandwidth connections (3G/4G)
7. **Free Forever**: No subscription fees, no API costs (local AI)

## Functional Requirements

### Requirement 1: WhatsApp Event Ingestion

**User Story:** As a student, I want to automatically discover hackathons and events from my WhatsApp groups, so that I don't miss opportunities buried in chat history.

#### Acceptance Criteria

1. WHEN the WhatsApp Bridge is initialized, THE System SHALL authenticate using QR code and establish a persistent session
2. WHEN a message is received in a monitored group, THE System SHALL scan the message content for event-related keywords (hackathon, workshop, competition, register, apply, lu.ma, conf)
3. WHEN an event-like message is detected, THE System SHALL extract the URL, timestamp, group name, and author metadata
4. WHEN a date range is specified, THE System SHALL fetch historical messages within that range (default: 30 days)
5. WHEN duplicate events are detected (same URL), THE System SHALL deduplicate and merge metadata
6. WHEN events are ingested, THE System SHALL store them in the events table with status "Detected"

### Requirement 2: AI Event Parsing and Enrichment

**User Story:** As a student, I want the system to automatically extract event details (title, date, description) from raw messages, so that I can quickly evaluate opportunities.

#### Acceptance Criteria

1. WHEN a raw event message is processed, THE System SHALL use the Local_AI to extract structured fields (title, date, description, event_type)
2. WHEN the event URL is available, THE System SHALL attempt to fetch additional metadata from the webpage
3. WHEN event details are extracted, THE System SHALL assign a relevance score (0-100) based on user profile and interests
4. WHEN parsing fails, THE System SHALL mark the event as "Failed" and log the error for manual review
5. WHEN events are enriched, THE System SHALL update the events table with parsed data

### Requirement 3: Autonomous Form Filling with Human-in-the-Loop

**User Story:** As a student, I want an AI agent to fill out event registration forms on my behalf, so that I can save time and avoid repetitive data entry.

#### Acceptance Criteria

1. WHEN a user selects an event and clicks "Auto-Apply", THE System SHALL fetch the student profile from the database
2. WHEN the Agent navigates to the event URL, THE System SHALL analyze the form structure and identify required fields
3. WHEN form fields are identified, THE System SHALL map student profile data to form fields using AI reasoning
4. WHEN the form is filled, THE System SHALL capture a full-page screenshot for user review
5. WHEN the user reviews the screenshot, THE System SHALL provide "Confirm & Submit" and "Cancel" options
6. WHEN the user confirms, THE Agent SHALL click the submit button and update event status to "Applied"
7. WHEN the user cancels, THE System SHALL discard the session and keep event status as "Queued"
8. IF the Agent encounters an error, THEN THE System SHALL mark the event as "Failed" and return an error message

### Requirement 4: Student Profile Management

**User Story:** As a student, I want to maintain a comprehensive profile with my education, projects, and experience, so that the Agent can accurately fill forms on my behalf.

#### Acceptance Criteria

1. WHEN a student creates a profile, THE System SHALL collect identity fields (name, email, phone, links)
2. WHEN education details are entered, THE System SHALL store university, major, GPA, student ID, and transcript
3. WHEN projects are added, THE System SHALL generate embeddings for semantic search
4. WHEN experience is added, THE System SHALL generate embeddings for semantic search
5. WHEN the profile is updated, THE System SHALL regenerate embeddings for changed fields
6. WHEN the Agent requests profile data, THE System SHALL return the complete profile with projects and experience

### Requirement 5: Knowledge Base with Semantic Search

**User Story:** As a student, I want to upload PDFs and documents to a knowledge base, so that I can search them by meaning and get AI-generated summaries.

#### Acceptance Criteria

1. WHEN a document is uploaded, THE System SHALL extract text content and chunk it into 512-token segments
2. WHEN chunks are created, THE System SHALL generate 384-dimensional embeddings using Xenova/all-MiniLM-L6-v2
3. WHEN embeddings are generated, THE System SHALL store them in the knowledge_chunks table with pgvector
4. WHEN a user searches, THE System SHALL generate a query embedding and retrieve top-20 nearest neighbors
5. WHEN candidates are retrieved, THE System SHALL rerank them using Xenova/ms-marco-MiniLM-L-6-v2 cross-encoder
6. WHEN reranking is complete, THE System SHALL return top-5 results with similarity scores
7. WHEN a user requests a summary, THE System SHALL send the document to Local_AI and return a concise summary

### Requirement 6: Research Copilot with Citation Awareness

**User Story:** As a student, I want an AI research assistant that provides citation-aware answers from my knowledge base, so that I can trust the information and verify sources.

#### Acceptance Criteria

1. WHEN a research query is submitted, THE System SHALL decompose it into 3 sub-questions using Local_AI
2. WHEN sub-questions are generated, THE System SHALL expand each into 2 additional query variations
3. WHEN queries are expanded, THE System SHALL perform workspace-scoped vector search for each query
4. WHEN results are retrieved, THE System SHALL deduplicate chunks and merge by similarity score
5. WHEN deduplication is complete, THE System SHALL build a citation block with source indices [1], [2], [3]
6. WHEN the citation block is ready, THE System SHALL send it to Local_AI with a strict JSON prompt
7. WHEN the LLM responds, THE System SHALL parse the JSON and extract title, summary, key_findings, evidence, tasks, and confidence_score
8. WHEN parsing succeeds, THE System SHALL return the research brief with citations
9. IF parsing fails, THEN THE System SHALL return a fallback brief with raw sources and low confidence

### Requirement 7: Local AI Reasoning Engine

**User Story:** As a student, I want to run AI models locally on my GPU, so that my data remains private and I don't incur cloud API costs.

#### Acceptance Criteria

1. WHEN the System initializes, THE Local_AI SHALL check if Ollama is running at localhost:11434
2. WHEN Ollama is detected, THE System SHALL warm up the DeepSeek R1 0528 Qwen3 8B model
3. WHEN the model is loaded, THE System SHALL report readiness status to the UI
4. WHEN a chat request is made, THE System SHALL send messages to Ollama's /v1/chat/completions endpoint
5. WHEN the LLM responds, THE System SHALL strip `<think>...</think>` reasoning tokens from the output
6. WHEN Ollama is unavailable, THE System SHALL fall back to OpenRouter API (if configured)
7. WHEN the API fallback is used, THE System SHALL proxy requests through /api/llm/chat to keep API keys server-side

### Requirement 8: In-Browser Database with Vector Search

**User Story:** As a student, I want my data to be stored locally in the browser, so that I can access it instantly without internet connectivity.

#### Acceptance Criteria

1. WHEN the System initializes, THE Database SHALL load PGlite with IndexedDB persistence
2. WHEN the pgvector extension is required, THE System SHALL enable it for vector similarity search
3. WHEN vector queries are executed, THE System SHALL use the `<=>` operator for cosine distance
4. WHEN data is inserted, THE System SHALL persist it to IndexedDB for offline access
5. WHEN the browser is closed, THE System SHALL preserve all data for the next session
6. WHEN schema migrations are needed, THE System SHALL apply Drizzle migrations automatically

### Requirement 9: Task Management with AI Breakdown

**User Story:** As a student, I want to organize tasks in an Eisenhower matrix, so that I can prioritize urgent and important work.

#### Acceptance Criteria

1. WHEN a task is created, THE System SHALL assign it a priority (low, medium, high) and status (todo, in-progress, done)
2. WHEN a vague task is entered, THE System SHALL use Local_AI to break it down into actionable subtasks
3. WHEN tasks are displayed, THE System SHALL render them in a drag-and-drop Eisenhower matrix
4. WHEN a task is moved, THE System SHALL update its priority in the database
5. WHEN a task is completed, THE System SHALL update its status to "done"

### Requirement 10: Focus Mode Editor with AI Assistance

**User Story:** As a student, I want a distraction-free editor with AI writing assistance, so that I can write essays and reports efficiently.

#### Acceptance Criteria

1. WHEN the editor is opened, THE System SHALL render a TipTap rich text editor
2. WHEN text is highlighted, THE System SHALL show an AI menu with options (rewrite, expand, summarize, translate)
3. WHEN an AI action is selected, THE System SHALL send the text to Local_AI and replace it with the result
4. WHEN Zen Mode is activated, THE System SHALL hide all UI elements except the editor
5. WHEN related context is requested, THE System SHALL search the knowledge base and suggest relevant notes

## Non-Functional Requirements

### Performance Metrics

1. **Vector Search Latency**: < 100ms for top-20 retrieval (in-browser pgvector)
2. **Cross-Encoder Reranking**: < 500ms for 20 candidates (WebGPU-accelerated)
3. **Local AI Inference**: < 2s for 100-token response (RTX 4070 8GB VRAM)
4. **Form Analysis**: < 5s for complex multi-page forms (Stagehand + Gemini)
5. **Document Chunking**: < 3s for 50-page PDF (Web Worker parallelization)
6. **Database Queries**: < 50ms for complex joins (PGlite in-memory)

### Scalability

1. **Knowledge Base**: Support up to 10,000 documents per workspace
2. **Vector Index**: Handle 100,000+ chunks with sub-second search
3. **Concurrent Searches**: Support 10+ simultaneous RAG queries (Web Worker pool)
4. **Event Ingestion**: Process 1,000+ WhatsApp messages in < 30s

### Security and Privacy

1. **Local-First Architecture**: All AI reasoning happens on user's GPU (zero cloud dependency)
2. **Data Encryption**: IndexedDB data encrypted at rest (browser-level)
3. **API Key Protection**: OpenRouter API key stored server-side (never exposed to client)
4. **Session Isolation**: WhatsApp sessions stored in `.wwebjs_auth` with LocalAuth
5. **CORS Headers**: Cross-Origin-Embedder-Policy and Cross-Origin-Opener-Policy for WebGPU

### Reliability and Availability

1. **Offline Mode**: Full functionality without internet (except WhatsApp sync and API fallback)
2. **Error Recovery**: Graceful degradation when Ollama is unavailable (API fallback)
3. **Data Persistence**: Automatic IndexedDB backup every 5 minutes
4. **Agent Retry Logic**: 3 retry attempts for form filling failures

### Compliance Requirements

1. **Data Residency**: All data stored locally (no cross-border data transfer)
2. **GDPR Compliance**: User data deletion via IndexedDB clear
3. **Accessibility**: WCAG 2.1 AA compliance for UI components (Radix UI primitives)

### Accessibility and Localization

1. **Keyboard Navigation**: Full keyboard support for command palette and forms
2. **Screen Reader Support**: ARIA labels for all interactive elements
3. **Language Support**: English (primary), Hindi (planned)
4. **Mobile Responsiveness**: Optimized for 360px+ screen widths
5. **Low-Bandwidth Mode**: Lazy loading for images and heavy components

## User Stories

### User Story 1: Automated Event Discovery

**As a** computer science student at IIT Delhi,  
**I want to** automatically discover hackathons from my 12 WhatsApp groups,  
**So that** I don't miss opportunities while focusing on exams.

**Acceptance**: System scans groups, extracts events, and shows them in a calendar view with relevance scores.

### User Story 2: One-Click Event Application

**As a** final-year student applying to 20+ hackathons,  
**I want to** auto-fill registration forms with my profile,  
**So that** I can save 3-4 hours per week on repetitive data entry.

**Acceptance**: Agent fills form, shows screenshot, I confirm, and it submits.

### User Story 3: Research Paper Summarization

**As a** graduate student working on a thesis,  
**I want to** upload 50 research papers and ask questions,  
**So that** I can quickly find relevant information with citations.

**Acceptance**: Upload PDFs, search "What are the main approaches to neural architecture search?", get answer with [1], [2], [3] citations.

### User Story 4: Privacy-First AI

**As a** student concerned about data privacy,  
**I want to** run AI models locally on my laptop,  
**So that** my transcripts and personal data never leave my machine.

**Acceptance**: Ollama runs locally, all inference happens on my GPU, no cloud API calls.

### User Story 5: Offline Knowledge Base

**As a** student with unreliable internet,  
**I want to** access my notes and documents offline,  
**So that** I can study during power cuts or while traveling.

**Acceptance**: Open browser, access all documents, search works without internet.

### User Story 6: Task Prioritization

**As a** student juggling assignments, projects, and hackathons,  
**I want to** organize tasks in an Eisenhower matrix,  
**So that** I can focus on urgent and important work first.

**Acceptance**: Drag tasks into quadrants, AI breaks down vague tasks into subtasks.

### User Story 7: AI Writing Assistant

**As a** student writing a lab report,  
**I want to** highlight text and ask AI to improve it,  
**So that** I can write faster and with better clarity.

**Acceptance**: Highlight paragraph, click "Rewrite", AI improves it, I accept or reject.

## Success Metrics

### User Adoption Metrics

1. **Active Users**: 10,000+ students in first 6 months
2. **Retention Rate**: 60% monthly active users (MAU)
3. **Event Applications**: 50,000+ auto-filled forms in first year
4. **Knowledge Base Usage**: 100,000+ documents uploaded

### Performance Benchmarks

1. **Time Saved**: Average 3.5 hours per week per student
2. **Event Participation**: 40% increase in hackathon applications
3. **Missed Deadlines**: 60% reduction in missed opportunities
4. **Search Accuracy**: 85%+ relevance for top-5 RAG results

### Technical KPIs

1. **Local AI Adoption**: 70% of users run Ollama locally
2. **Vector Search Performance**: 95th percentile < 200ms
3. **Agent Success Rate**: 80% successful form submissions
4. **Uptime**: 99.5% availability (client-side)

### Competition Metrics (AI for Bharat)

1. **Innovation Score**: Novel combination of local AI + browser automation + in-browser DB
2. **India Relevance**: WhatsApp-first design, privacy-focused, low-bandwidth optimized
3. **Technical Depth**: Production-grade RAG, cross-encoder reranking, WebGPU acceleration
4. **Impact Potential**: 40M+ students, 3-5 hours saved per week, free forever
5. **Scalability**: Fully client-side (zero server costs), works offline
