# Phase 2: Database Schema & Data Ingestion

## Objective

Establish the foundational data models in PostgreSQL (via Supabase) utilizing a flexible JSONB approach for candidate memory. Implement the onboarding API pipelines to securely parse uploaded resumes using an AI parsing service.

## 1. Database Schema (Supabase / PostgreSQL)

We are opting for a hybrid approach: strict relational tables for billing and auth, but a flexible `JSONB` schema for the candidate's actual resume data to allow the AI maximum flexibility.

### Tables & SQL Structure

**Table: `users`** (Managed by Supabase Auth, but we maintain a public profile table)

```sql
CREATE TABLE public.profiles (
    id UUID REFERENCES auth.users(id) PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    credits INT DEFAULT 10,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Table: `candidate_memories`** (The core flexible storage)

```sql
CREATE TABLE public.candidate_memories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
    memory_data JSONB DEFAULT '{}'::jsonb, -- Stores flexible schema (work, skills, etc.)
    last_synced TIMESTAMPTZ DEFAULT NOW()
);
```

**Table: `resume_drafts`** (Specific tailored resumes)

```sql
CREATE TABLE public.resume_drafts (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
    target_role TEXT,
    target_jd_text TEXT,
    content JSONB NOT NULL, -- The specific subset/mutated version of memory_data
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

## 2. API: PDF Ingestion Pipeline (FastAPI)

To process resumes (PDF, DOCX) cleanly and without external API costs, we will use Microsoft's open-source **`markitdown`** library to convert documents locally into Markdown before structuring them via LLM.

### Dependencies

- `uv pip install markitdown python-multipart`

### Implementation Details: `POST /api/v1/ingest/upload`

1. **Endpoint Signature**: Accepts an `UploadFile` (the Resume).
2. **Auth**: Protected by `get_current_user` Supabase dependency.
3. **Execution Flow**:
   - Save the file temporarily to `/tmp`.
   - Instantiate the local parser: `md = MarkItDown()` and run `result = md.convert('/tmp/resume.pdf')`.
   - Receive the converted Markdown text.
   - **AI Structuring Step**: Pass the Markdown to LiteLLM (using `Gemini 1.5 Flash(Use the latest flash model this is deprecated search the web for it)` for speed) with a strict system prompt: _"Convert this resume markdown into a structured JSON object encompassing Work Experience, Education, Projects, and Skills."_
   - Merge the resulting JSON into the user's `memory_data` column in the `candidate_memories` table.
4. **Cleanup**: Delete the `/tmp` file.

## 3. PII Sanitization Middleware

Before sending data to external LLMs (if not using enterprise zero-retention agreements), run a regex-based Python scrubber:

```python
import re
def sanitize_pii(text: str) -> dict:
    # Extract emails and phones, replace with [EMAIL_1], [PHONE_1]
    # Return sanitized text and a mapping dictionary for re-insertion
    pass
```

## 4. Acceptance Criteria

- [ ] SQL migrations run successfully in Supabase.
- [ ] Uploading a PDF successfully converts to Markdown via `markitdown`.
- [ ] The resulting text is parsed by Gemini Flash into JSON.
- [ ] The JSON is successfully appended to the `candidate_memories` table for that user.
