# DevDocs — Scope Document

**Project**: DevDocs — AI-Powered API Debugging & Documentation Platform  
**Status**: Locked (Phase 0 Complete)  
**Last Updated**: September 2026

---

## 1. Core Purpose

DevDocs is a focused API debugging tool.

The single core loop that must work:

**Upload OpenAPI → Pick endpoint → Send real request → See response + automatic problem detection + clear explanation**

Documentation and AI are supporting features. Debugging is the primary goal.

---

## 2. Final Locked MVP

The finished product must do exactly the following:

1. User uploads an OpenAPI file (`.json` or `.yaml`).
2. Tool parses it and shows the list of endpoints.
3. User selects one endpoint.
4. Tool shows a form with the required fields (path parameters, query parameters, headers, JSON body).
5. User fills the form and clicks **Send**.
6. Tool actually calls the real API.
7. Tool shows the full response (status code, body, latency).
8. Tool automatically checks whether the response matches the schema defined in the OpenAPI (deterministic validation).
9. If the request failed (non-2xx) **or** a schema mismatch is found → Tool calls AI and shows a clear diagnosis (problem, likely cause, explanation, suggested fix, confidence).
10. Every request is saved in history so the user can revisit it.
11. Basic documentation view is generated from the OpenAPI (no AI required).

This is the complete MVP. Nothing else is in scope for the first shippable version.

---

## 3. Explicitly Out of Scope

The following will **not** be built in the MVP:

- User accounts / authentication / multi-user support
- Teams or collaboration features
- Environments, variables, or collections (Postman-style)
- GraphQL support
- File uploads / multipart requests
- Complex authentication flows (OAuth2, etc.) — only simple Bearer token and API key in headers
- Automatic test generation
- Mock servers
- RAG / knowledge base
- Advanced analytics or dashboards

---

## 4. How Each Major Feature Works

### 4.1 OpenAPI Upload & Parsing
- User uploads `.json` or `.yaml` file.
- Backend uses a library to parse and validate the OpenAPI document.
- Extracts: title, version, servers, paths, methods, parameters, request body schemas, response schemas.
- Saves the structured data in MongoDB.
- Returns the list of endpoints to the frontend.

### 4.2 API Explorer
- Frontend displays the list of endpoints.
- When an endpoint is selected, a form is automatically generated from its parameters and request body definition.
- User fills the form and submits.

### 4.3 Send Request
- Frontend sends the filled data to the backend.
- Backend constructs and executes the real HTTP request to the target API.
- Returns status code, headers, body, and latency to the frontend.

### 4.4 Schema Validation (Deterministic)
- After receiving the response, the backend validates the response body against the corresponding response schema in the OpenAPI using a JSON Schema validator.
- Detects type mismatches, missing required fields, etc.
- This step does **not** use AI.

### 4.5 AI Diagnosis
- Runs only when status is non-2xx **or** a schema mismatch is detected.
- Backend sends the exact request, response, schema mismatch details, and relevant OpenAPI parts to the LLM.
- Expects structured output: problem, likely cause, explanation, suggested fix, confidence.
- Frontend displays the diagnosis in a clear card.

### 4.6 Request History
- Every executed request (including response, validation result, and diagnosis) is saved in MongoDB.
- User can view and re-open past requests.

### 4.7 Documentation View
- Renders the information already present in the parsed OpenAPI in a clean, readable format.
- No AI is used.

---

## 5. Complete User Flow
```
Upload OpenAPI file
↓
Parse & show list of endpoints
↓
User selects one endpoint
↓
Form is generated automatically
↓
User fills form → Clicks Send
↓
Backend calls the real API
↓
Show response + run schema validation
↓
If error or mismatch → Call AI for diagnosis
↓
Save everything in History
```
---

## 6. Locked Tech Stack

| Layer              | Technology                                      |
|--------------------|-------------------------------------------------|
| Frontend           | Vite + React + TypeScript + Tailwind CSS + TanStack Query |
| Backend            | Node.js + Express + TypeScript                  |
| Database           | MongoDB + Mongoose (MongoDB Atlas)              |
| Caching            | Redis (only later, for AI diagnosis caching)    |
| Package Manager    | npm                                             |
| Repository         | Single repo with `client/`, `server/`, `shared/` |
| AI                 | Provider abstraction (start with one provider, design for multiple) |

---

## 7. Repository Structure
```
devdocs/
├── client/                 # Vite + React + TypeScript frontend
├── server/                 # Express + TypeScript backend
├── shared/                 # Shared TypeScript types
├── package.json            # Root (workspaces)
├── .gitignore
├── docker-compose.yml
├── .env.example
└── README.md
```
---

## 8. Guiding Principles

- Deterministic logic first, AI second.
- Schema validation must work without AI.
- AI must be grounded on real request/response + schema mismatch data.
- Keep the product focused on the core debugging loop.
- Prefer simple, clear code over premature abstraction.

---

**This document is the single source of truth.**  
Any new idea must be evaluated against this scope before being added.
