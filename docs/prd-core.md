# AI Operational Guide (PRD) - Core
## Central operational and technical standards for the "Chef Digital" project.

---

## 1. Guideline: The Knowledge Base

### 1.1. Project Overview and Architecture
**Chef Digital** is a responsive web application that helps manage a personal recipe collection.  
Users can add recipes from text, image, or video (using only the link as "Source").  
The system integrates **Groq AI** for transcription and normalization, and **Serper API** for discovering new recipes on the web.

**Core Architecture:**
- **Frontend:** Next.js (App Router) + TailwindCSS + shadcn/ui  
- **Backend:** Next.js API Routes  
- **Database:** PostgreSQL (Neon or Supabase)  
- **ORM:** Prisma  
- **LLM:** Groq API (for transcription and recipe formatting)  
- **Web Search:** Serper API  
- **Authentication:** Auth.js (Google OAuth, restricted to owner email)  
- **Hosting:** Render  
- **OCR:** Tesseract.js (local, browser-based)  
- **Video Transcription:** Groq ASR / YouTube Transcript API  
- **Security:** Environment variables, rate limiting, Zod validation

---

### 1.2. Code Standards and Tech Stack
- **Frontend:**  
  - Functional React components with hooks.  
  - TailwindCSS utility classes only (no inline CSS).  
  - UI built with modular **shadcn/ui** components.

- **Backend:**  
  - RESTful API routes under `/api` (`/add`, `/update`, `/search`, `/discover`).  
  - Input validation via **Zod**.  
  - Structured logging using **pino**.

- **Database:**  
  - Prisma schema versioned with migrations.  
  - GIN indexes for full-text search.  

- **LLM Integration:**  
  - All Groq prompts and completions handled server-side.  
  - Responses normalized to the **standard recipe format**.

---

### 1.3. Principle of Clarity
- Prioritize simplicity and transparency.  
- All user-facing messages are friendly and written in Portuguese (as the culinary assistant persona).  
- All AI-generated outputs must strictly follow the **Recipe Format Standard**.

---

### 1.4. Implementation and Quality Rules
- **Dependencies:** Always use latest stable versions.  
- **Tests:**  
  - Unit tests (Jest) for API routes and recipe normalization.  
  - Mocked OCR and Groq responses.  
- **Accessibility:** ARIA labels, semantic HTML, and color contrast compliance.

---

## 2. Workflow: Operational Flow

### 2.1. Task Lifecycle
Blueprint → Human Approval → Execution via `instruction.md` → Human Review → Iterative Learning

### 2.2. Document Update Protocol
When `prd-core.md` or `blueprint-chef-digital.md` changes:
1. Announce the change clearly.  
2. Present the full updated document.  
3. Ensure no internal comments or annotations remain — clean and ready to save.

---

## 3. Output Documentation Standard

### 3.1. Implementation Report Template
1. Task title and context  
2. Technical decisions summary  
3. Functional code and tests  
4. Results and observations
