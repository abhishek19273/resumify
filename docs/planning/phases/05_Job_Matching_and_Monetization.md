# Phase 5: Matchmaking, Gap Analysis & Monetization

## Objective
Implement the core value-proposition of the SaaS: dynamically matching a candidate's memory against a specific Job Description (JD), prompting the user for missing info, and integrating Stripe for the credit-based billing system.

## 1. Job Description (JD) Ingestion & Gap Analysis
When a candidate wants to generate a new resume for a job, they provide a JD.

### Implementation Flow:
1. **JD Input**: User pastes text or a URL in the frontend. (If URL, FastAPI scrapes it).
2. **Semantic Matching**: 
   - Embed the JD requirements.
   - Perform a vector similarity search using `pgvector` against the user's `memory_embeddings`.
3. **Gap Analysis (Gemini Flash)**:
   - Prompt: *"Here is the JD. Here are the candidate's matching experiences. Identify any explicitly required skills in the JD that are NOT in the candidate's experiences."*
4. **Intrusive Clarification (Pop-up Modal)**:
   - If gaps are found, return them to the frontend.
   - Frontend displays a modal: *"The JD asks for Docker. Do you have experience with this?"*
   - If the user answers yes and provides context, send it back to the Memory Engine to permanently update their JSONB profile.

## 2. Auto-Curation (The Final Output)
1. Once gaps are resolved, gather all relevant chunks.
2. Send to **Gemini 1.5 Pro / GPT-4o**.
3. System Prompt: *"Construct a highly targeted resume in the strict JSON format matching our templates. Emphasize the matching skills. Do not invent experience."*
4. Save the result to the `resume_drafts` table.
5. Redirect the user to the Split-pane Editor (Phase 4) to review and export.

## 3. Monetization (Stripe)
We operate on a freemium, credit-based model.

### Database Updates
- `users` table already has a `credits` integer column.

### Stripe Integration
1. **Setup**: Install `stripe-node` (or Python Stripe SDK in backend).
2. **Products**: Create Stripe products for credit bundles (e.g., "$5 for 10 Generation Credits").
3. **Checkout Sessions**: 
   - Create a `POST /api/v1/billing/checkout` endpoint.
   - Redirect user to Stripe Checkout.
4. **Webhooks (`POST /api/v1/billing/webhook`)**:
   - Listen for `checkout.session.completed`.
   - Update the `credits` column for the `user_id` in Supabase.

### Credit Deduction Logic
- **Full Resume Generation**: -5 credits.
- **Inline Highlight-to-Refactor**: -1 credit.
- FastAPI dependency checks `if user.credits < cost: raise 402 Payment Required`.

## 4. Acceptance Criteria
- [ ] User can paste a JD and receive a pop-up asking about missing skills.
- [ ] Answering the pop-up updates the global memory.
- [ ] AI generates a tailored JSON resume draft.
- [ ] User can purchase credits via Stripe test mode, and the database updates accurately via webhooks.
- [ ] API endpoints reject generation requests if credits are insufficient.
