# Implementation Blueprint: Chef Digital
*Action plan and technical design for the Chef Digital recipe assistant.*

---

## 1. Task Analysis and Implementation Phases

**Goal:**  
Build a web app to manage recipes — add, search, update, and discover — using Groq AI to structure and transcribe content from text, images, or videos.  
Only the video link is stored (never the file itself).

**Phases:**

### Phase 1 — Base Setup
- Initialize **Next.js (App Router)** project.  
- Integrate **TailwindCSS** and **shadcn/ui**.  
- Configure **Prisma** with **PostgreSQL**.  
- Create initial schema for `recipes` table.  
- Implement **Auth.js** (Google OAuth restricted to your email).

---

### Phase 2 — Add Recipe Module (/add)
- Local OCR using **Tesseract.js** (no image stored).  
- Extracted text sent to API for Groq normalization.  
- If “Source” contains a URL, fetch the page and merge missing data using Groq.  
- Persist formatted recipe to PostgreSQL.

---

### Phase 3 — Update Recipe Module (/update)
- Display the current recipe in an editable form.  
- Apply changes, validate format, and update in database.

---

### Phase 4 — Search Recipe Module (/search)
- Search by exact title.  
- If no match, perform ingredient full-text search.  
- If empty, list all recipes.

---

### Phase 5 — Discover Recipes (/discover)
- Backend calls **Serper API** for relevant results.  
- Fetch top result(s), extract readable content.  
- Send text to **Groq AI** for structured formatting.  
- Display result, add to database only upon user confirmation.

---

### Phase 6 — UI & Interactions
- Navigation bar with `/add`, `/update`, `/search`, `/discover`.  
- Add Form tabs:
  - **Image** (local OCR)
  - **Video** (URL only)
  - **Manual Text Entry**
- Recipe list with cards.  
- Detailed recipe view page.

---

### Phase 7 — Testing and Deployment
- Unit and integration tests for all API routes.  
- Mock Groq responses for deterministic validation.  
- Deploy to **Render** with secure environment variables.

---

## 2. Sample Code per Phase

### Prisma Schema
```prisma
model Recipe {
  id           String   @id @default(uuid())
  title        String   @unique
  sourceUrl    String?
  ingredients  Json
  instructions Json
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}
```

### API Route `/api/recipes/ingest`

```typescript
import { groqFormatRecipe } from "@/lib/groq";
import { z } from "zod";
import prisma from "@/lib/prisma";

const RecipeInput = z.object({
  title: z.string().optional(),
  sourceUrl: z.string().url().optional(),
  extractedText: z.string(),
});

export default async function handler(req, res) {
  const parsed = RecipeInput.parse(req.body);
  const formatted = await groqFormatRecipe(parsed.extractedText, parsed.sourceUrl);
  const saved = await prisma.recipe.create({ data: formatted });
  res.json(saved);
}
```

### Example — Groq AI Integration

```typescript
export async function groqFormatRecipe(text, sourceUrl) {
  const prompt = `
You are Chef Digital. Convert the text below into a structured recipe format:
Text: """${text}"""
Source: ${sourceUrl || "Not provided"}
Return the result in this exact format:
- Title:
- Source:
- Ingredients:
    - [Category 1]
        - [Ingredient 1]
    - [Category 2]
        - [Ingredient 2]
- Instructions:
    1. [Step 1]
    2. [Step 2]
  `;
  const response = await groq.chat.completions.create({
    model: "llama3-70b-8192",
    messages: [{ role: "system", content: prompt }],
  });
  return parseRecipeResponse(response.choices[0].message.content);
}
```## 2. Sample Code per Phase

### Prisma Schema
```prisma
model Recipe {
  id           String   @id @default(uuid())
  title        String   @unique
  sourceUrl    String?
  ingredients  Json
  instructions Json
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}
```

### API Route `/api/recipes/ingest`

```typescript
import { groqFormatRecipe } from "@/lib/groq";
import { z } from "zod";
import prisma from "@/lib/prisma";

const RecipeInput = z.object({
  title: z.string().optional(),
  sourceUrl: z.string().url().optional(),
  extractedText: z.string(),
});

export default async function handler(req, res) {
  const parsed = RecipeInput.parse(req.body);
  const formatted = await groqFormatRecipe(parsed.extractedText, parsed.sourceUrl);
  const saved = await prisma.recipe.create({ data: formatted });
  res.json(saved);
}
```

### Example — Groq AI Integration

```typescript
export async function groqFormatRecipe(text, sourceUrl) {
  const prompt = `
You are Chef Digital. Convert the text below into a structured recipe format:
Text: """${text}"""
Source: ${sourceUrl || "Not provided"}
Return the result in this exact format:
- Title:
- Source:
- Ingredients:
    - [Category 1]
        - [Ingredient 1]
    - [Category 2]
        - [Ingredient 2]
- Instructions:
    1. [Step 1]
    2. [Step 2]
  `;
  const response = await groq.chat.completions.create({
    model: "llama3-70b-8192",
    messages: [{ role: "system", content: prompt }],
  });
  return parseRecipeResponse(response.choices[0].message.content);
}
```

---

## 3. Testing Strategy

| Module    | Type        | Key Tests                           |
| --------- | ----------- | ----------------------------------- |
| OCR       | Unit        | Validate extraction from mock image |
| Groq AI   | Integration | Output follows Recipe Format        |
| /add      | E2E         | Image → Text → Groq → DB            |
| /update   | Unit        | Field-level update consistency      |
| /search   | Unit        | Search by title or ingredient       |
| /discover | Integration | Web scraping + LLM normalization    |

---

## 4. Consolidated Execution Checklist

* [ ] Create Next.js project with Tailwind and shadcn/ui
* [ ] Configure Prisma + PostgreSQL
* [ ] Set up Auth.js (Google OAuth)
* [ ] Implement `/api/recipes/ingest` route
* [ ] Add local OCR with Tesseract.js
* [ ] Integrate Groq AI (formatting + transcription)
* [ ] Integrate Serper API (web discovery)
* [ ] Implement `/update`, `/search`, `/discover`
* [ ] Create responsive UI with tabs (Image, Video, Text)
* [ ] Write unit and integration tests
* [ ] Deploy on Render
