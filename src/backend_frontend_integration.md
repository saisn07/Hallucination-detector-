# Backend and Frontend Integration Guide

This document summarizes the current FastAPI backend and shows how a frontend can connect to it.

## Backend Entry Point

Main backend file:

```text
main.py
```

Run the API server on port `5000`:

```bash
uvicorn main:app --host 0.0.0.0 --port 5000 --reload
```

You can also run:

```bash
python main.py
```

Base API URL for local frontend integration:

```text
http://localhost:5000
```

## CORS Configuration

The backend allows frontend requests from common local development ports:

```text
http://localhost:3000
http://localhost:5173
http://127.0.0.1:3000
http://127.0.0.1:5173
```

This supports React, Vite, Next.js, or similar frontend development servers.

## API Endpoints

### GET /api/health

Checks whether the backend, model layer, and database layer are available.

Request:

```http
GET http://localhost:5000/api/health
```

Successful response:

```json
{
  "status": "ok",
  "model": "loaded",
  "database": "connected"
}
```

Status code:

```text
200 OK
```

### POST /api/chat

Sends a user prompt to the backend. The backend currently uses a modular placeholder pipeline for hallucination detection.

Pipeline structure:

```text
User prompt
  -> LLM generation placeholder
  -> NLI verification placeholder
  -> RAG verification placeholder
  -> token uncertainty scoring
  -> weighted hallucination score
  -> grounded / hallucinated verdict
```

Request:

```http
POST http://localhost:5000/api/chat
Content-Type: application/json
```

Request body:

```json
{
  "prompt": "Who invented the telephone?"
}
```

Successful response:

```json
{
  "response": "Alexander Graham Bell is commonly credited with inventing the telephone.",
  "hallucination_score": 0.062,
  "verdict": "grounded"
}
```

Response fields:

```text
response: string
hallucination_score: number
verdict: "grounded" or "hallucinated"
```

Status code:

```text
200 OK
```

### GET /api/history

Returns mock conversation history.

Request:

```http
GET http://localhost:5000/api/history
```

Successful response:

```json
{
  "conversations": [
    {
      "prompt": "Who invented the telephone?",
      "response": "Alexander Graham Bell is commonly credited with inventing the telephone.",
      "score": 0.08
    },
    {
      "prompt": "Summarize hallucination detection.",
      "response": "Hallucination detection estimates whether generated claims are grounded in reliable context.",
      "score": 0.21
    }
  ]
}
```

Response fields:

```text
conversations: array of conversation objects
conversations[].prompt: string
conversations[].response: string
conversations[].score: number
```

Status codes:

```text
200 OK
400 Bad Request
```

### GET /api/users

Returns mock user information.

Request:

```http
GET http://localhost:5000/api/users
```

Successful response:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Status codes:

```text
200 OK
400 Bad Request
```

## Frontend Integration Examples

Set a reusable API base URL:

```javascript
const API_BASE_URL = "http://localhost:5000";
```

### Health Check

```javascript
async function checkHealth() {
  const response = await fetch(`${API_BASE_URL}/api/health`);

  if (!response.ok) {
    throw new Error("Backend health check failed");
  }

  return response.json();
}
```

### Send Chat Prompt

```javascript
async function sendChatPrompt(prompt) {
  const response = await fetch(`${API_BASE_URL}/api/chat`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ prompt }),
  });

  if (!response.ok) {
    throw new Error("Chat request failed");
  }

  return response.json();
}
```

Example usage:

```javascript
const result = await sendChatPrompt("Who invented the telephone?");

console.log(result.response);
console.log(result.hallucination_score);
console.log(result.verdict);
```

### Load Conversation History

```javascript
async function getHistory() {
  const response = await fetch(`${API_BASE_URL}/api/history`);

  if (!response.ok) {
    throw new Error("Could not load conversation history");
  }

  return response.json();
}
```

Example usage:

```javascript
const history = await getHistory();

console.log(history.conversations);
```

### Load User Info

```javascript
async function getUser() {
  const response = await fetch(`${API_BASE_URL}/api/users`);

  if (!response.ok) {
    throw new Error("Could not load user information");
  }

  return response.json();
}
```

## Suggested Frontend State Shape

```javascript
const chatState = {
  prompt: "",
  response: "",
  hallucinationScore: null,
  verdict: null,
  loading: false,
  error: null,
};
```

## Notes for Future Backend Wiring

The current `/api/chat` endpoint already has placeholder functions for the real hallucination pipeline:

```text
run_llm_generation()
run_nli_module()
run_rag_verifier()
calculate_uncertainty_score()
aggregate_hallucination_score()
```

When the actual engine is ready, replace the placeholder logic inside these functions without changing the frontend API contract.

The frontend can continue sending:

```json
{
  "prompt": "user message"
}
```

and receiving:

```json
{
  "response": "model response",
  "hallucination_score": 0.25,
  "verdict": "grounded"
}
```
