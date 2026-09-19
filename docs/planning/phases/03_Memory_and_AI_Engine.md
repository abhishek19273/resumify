# Phase 3: The Memory Engine & AI Orchestration

## Objective
Implement the semantic search and LLM routing layer. This phase connects the structured JSONB memory of the candidate to Supabase `pgvector` for semantic search, and sets up LiteLLM to handle the AI interactions.

## 1. Vector Database Setup (`pgvector` in Supabase)
We use `pgvector` to semantically chunk and search a candidate's experiences. This is critical for matching specific past projects to the requirements of a new Job Description.

### SQL Migration
```sql
-- Enable the extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create the embeddings table linked to a candidate's memory
CREATE TABLE public.memory_embeddings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
    content_text TEXT NOT NULL, -- The chunk of text (e.g., a specific job responsibility)
    embedding VECTOR(1536), -- 1536 is standard for OpenAI embeddings
    metadata JSONB DEFAULT '{}'::jsonb -- Points back to the specific path in the JSONB memory
);

-- Indexing for fast semantic search
CREATE INDEX ON public.memory_embeddings 
USING hnsw (embedding vector_cosine_ops);
```

## 2. LiteLLM Orchestration (FastAPI)
LiteLLM will act as our proxy to normalize calls between Google Gemini and OpenAI.

### Configuration
1. Install: `uv pip install litellm`
2. Environment: `LITELLM_MASTER_KEY`, `GEMINI_API_KEY`, `OPENAI_API_KEY`.

### AI Routing Strategy
We will strictly route based on the complexity of the task to optimize speed and cost:
- **Fast / Extraction Tasks (Gemini 1.5 Flash)**:
  - Parsing uploaded PDFs via LlamaParse.
  - Generating embedding text chunks.
  - Identifying missing skills (Gap Analysis).
- **Complex / Generation Tasks (GPT-4o or Gemini 1.5 Pro)**:
  - Writing the final, tailored bullet points.
  - The Interactive "Highlight-to-Refactor" editor features.

## 3. The Embedding Pipeline
When a user updates their profile (or uploads a resume):
1. **Trigger**: An update to the `candidate_memories` table triggers a background task in FastAPI.
2. **Chunking**: The JSONB memory is flattened into meaningful chunks (e.g., one chunk per work experience, one chunk for skills).
3. **Embedding**: Call the embedding model (e.g., `text-embedding-3-small`) via LiteLLM.
4. **Storage**: Upsert the new embeddings into `memory_embeddings` and delete any stale embeddings for that user.

## 4. Acceptance Criteria
- [ ] `pgvector` is enabled and the `memory_embeddings` table exists in Supabase.
- [ ] LiteLLM is configured in FastAPI and successfully routes to both Gemini Flash and GPT-4o.
- [ ] Saving data to a user's profile automatically generates and stores vectors in the database.
