# Phase 4: Core Editor & Generation UX

## Objective
Build the split-pane resume editor where candidates can view their generated resumes, manually edit them, and collaborate with the AI via highlight-to-refactor and a chat sidebar. 

## 1. State Management (React Context)
We will avoid heavy libraries like Redux and use **React Context** combined with **TanStack Query**.

### Context Architecture (`ResumeEditorContext.tsx`)
```tsx
interface ResumeState {
  resumeId: string;
  content: any; // The JSON representing the resume layout/text
  activeHighlight: { text: string; path: string } | null;
  isGenerating: boolean;
}

// Actions:
// - updateSection(path, newContent)
// - setHighlight(selection)
// - triggerAIRefactor(prompt)
```
- **TanStack Query** will be used to fetch the initial `resume_draft` from the backend and periodically auto-save changes via a debounced mutation.

## 2. Editor Layout (Split-Pane)
We will use a library like `react-split` or standard Tailwind CSS grid to create a deterministic workspace.

### Left Pane: Structure & Chat
- **Accordion / Form Fields**: Renders the JSON `content` into editable fields (Work Experience, Skills, Education).
- **AI Chat Sidebar**: A toggleable panel where the user can type commands: *"Make the tone of my Google experience more leadership-focused."*
- **Interaction Flow**: Sending a chat message triggers a FastAPI endpoint -> LiteLLM (GPT-4o / Gemini Pro) -> returns a JSON patch -> updates the React Context.

### Right Pane: Live PDF Preview
- **High-Fidelity Rendering**: We will use a deterministic renderer like **React-PDF** (`@react-pdf/renderer`). 
- **Data Binding**: The `<Document>` component from React-PDF will reactively re-render whenever the `ResumeContext` state changes.
- **Highlight to Refactor**: When a user selects text in the preview or the left pane, a floating toolbar appears: `[✨ Improve] [Shorten] [Make Metric-Driven]`. Clicking an option triggers an API call that patches that specific text block.

## 3. Template Engine
- Start with **two** base ATS-friendly templates defined as React-PDF components.
- Templates should map directly to the standardized JSON structure established in Phase 2.

## 4. Acceptance Criteria
- [ ] User can open a draft and see the split-pane view.
- [ ] Editing a field on the left immediately updates the React-PDF preview on the right.
- [ ] Selecting text exposes an AI toolbar.
- [ ] Clicking "Improve" successfully calls the backend, rewrites the text, and updates the local state.
- [ ] The app auto-saves to the database after 2 seconds of inactivity.
