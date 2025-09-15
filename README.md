# Chat With Your Files (RAG Pipeline)

Upload Markdown → ask questions → get grounded answers sourced only from your content.

## Snapshot (Why It Matters)
Demonstrates end‑to‑end Retrieval Augmented Generation with:
- Automatic ingestion & processing (triggers + Edge Functions)
- Vector similarity search (pgvector, HNSW)
- Hybrid embeddings (server for documents, client for queries)
- Streaming LLM responses with constrained context
- Strong data isolation via Row Level Security

## Architecture Flow
```
Upload → Storage trigger → process fn (parse + chunk) → sections → embed fn (batch embeddings)
→ Postgres (pgvector index)
→ Query: local embedding (Web Worker) → match_document_sections → OpenAI (stream) → UI
```

## Tech Stack
Next.js (App Router) · Supabase (Auth, Storage, Postgres, Edge Functions) · pgvector (HNSW) · OpenAI API · Transformers.js (in-browser) · TypeScript · Tailwind

## Core Data & Retrieval
| Object | Purpose |
|--------|---------|
| `documents` | One row per uploaded file |
| `document_sections` | Chunked text + embedding (vector 384) |
| RPC `match_document_sections` | Inner product similarity lookup |

Retrieval: normalized embeddings + thresholded inner product for fast filtering.

## Key Decisions
| Choice | Benefit |
|--------|---------|
| DB triggers over external queue | Fewer moving parts early on |
| Client-side query embedding | Lower latency & cost control |
| Heading-based chunking | Keeps semantic cohesion |
| Streaming responses | Better UX under longer generations |

## Security
- RLS + storage policies enforce per-user isolation
- Edge Functions require bearer token
- Only selected chunk text goes to OpenAI

## Selected Source Files
| File | Role |
|------|------|
| `supabase/migrations/*.sql` | Schema, triggers, RLS, vector index |
| `supabase/functions/process/index.ts` | Parse + chunk markdown |
| `supabase/functions/embed/index.ts` | Batch embedding writes |
| `supabase/functions/chat/index.ts` | Retrieval + streaming answer |
| `supabase/functions/_lib/markdown-parser.ts` | Heading-based chunk logic |
| `lib/hooks/use-pipeline.ts` | Web Worker embedding hook |
| `app/chat/page.tsx` | Chat UI + streaming integration |

## Local Setup
```
npm install
supabase start
npm run dev
```
Env:
```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
OPENAI_API_KEY=...
```
Regenerate DB types: `npm run gen:types`

## Possible Extensions
Citizens / citations, metadata filtering, observability, rate limiting, tests for chunker & retrieval.

## Focus Demonstrated
Applied RAG design, secure multi-tenant data handling, vector retrieval wiring, and pragmatic trade-offs for latency & cost.


---