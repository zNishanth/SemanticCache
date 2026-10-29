ECOCACHE (SEMANTIC CACHE GATEWAY):

MVP in Brief(GOAL) : 
A differently worded repeat question is answered from cache, for free, and the dashboard shows the savings.(Not just Cache , question with same meaning worded differently)

IN SCOPE PRESENTLY  (MVP):
1)Chat endpoint: POST /api/chat accepts a prompt, returns an answer plus metadata (source: exact | semantic | llm, latency, cost saved).
2)Rate limiting: Bucket4j per client (by IP or API key), returns 429 with Retry-After.
3)Exact-match cache: Redis, keyed by hash of the normalized prompt, with a TTL.
4)Semantic cache: embed the prompt, run a pgvector similarity search, return a match above the threshold.
5)LLM call on miss: one provider, then store prompt, embedding, and answer in both Redis and PostgreSQL.
6)Basic telemetry: hit/miss counts, average latency per layer, estimated money saved.
7)Simple frontend: a chat box that shows the answer with a badge (“Exact hit”, “Semantic hit”, “LLM call”) and a small stats panel.


## Architecture

EchoCache is a proxy layer between a client app and an LLM provider. It answers repeat questions from cache, including differently worded ones, and calls the LLM only on a genuine miss.

```
User ──► Frontend (React)
              │  POST /api/chat
              ▼
        Rate Limiter (Bucket4j, state in Redis)
              │ allowed                └─► 429 Too Many Requests + Retry-After
              ▼
        Spring Boot Backend (ChatService)
              │
              ▼
        1. Exact Cache (Redis) ───────── hit ──► return  (~5 ms, $0)
              │ miss
              ▼
        2. Semantic Cache
           embed prompt → pgvector search ── hit (≥ threshold) ──► return  (~50 ms, $0)
              │ miss
              ▼
        3. LLM Provider (fallback provider on error/timeout)
              │
              ▼
        4. Store new entry: Redis (exact key) + PostgreSQL (prompt, embedding, answer)
              │
              ▼
        5. Record telemetry (every request) ──► return response
```

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 21, Spring Boot 3 |
| Exact cache and rate-limit state | Redis |
| Semantic store | PostgreSQL + pgvector |
| Rate limiting | Bucket4j |
| Embeddings | API-based or local model |
| LLM | Primary provider with fallback provider |
| Frontend | ?? |
| Local environment | Docker Compose |
