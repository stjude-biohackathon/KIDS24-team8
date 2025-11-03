# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **St. Jude BioHackathon 2024 Team 8** project implementing a Retrieval-Augmented Generation (RAG) chatbot. The project is built using the Vercel AI SDK RAG starter template with customizations for a PostgreSQL-based knowledge base system. It allows users to provide personal information that the AI will remember and retrieve in future conversations.

## Development Setup

This project requires the Vercel AI SDK RAG starter template as the foundation. Follow these steps:

1. **Initial Setup** (from README):
   ```bash
   git clone https://github.com/vercel/ai-sdk-rag-starter
   cd ai-sdk-rag-starter
   pnpm install
   ```

2. **Replace Code**: Replace the `lib/` and `app/` folders with the ones from this repository

3. **PostgreSQL Setup**:
   ```bash
   # Install PostgreSQL (Ubuntu example)
   sudo apt update
   sudo apt install postgresql postgresql-server-dev-12

   # Setup database
   sudo -u postgres psql
   ALTER USER postgres PASSWORD 'postgres'
   CREATE EXTENSION IF NOT EXISTS vector;
   \quit
   ```

4. **Install pgvector Extension**:
   ```bash
   cd /tmp
   git clone --branch v0.7.4 https://github.com/pgvector/pgvector.git
   cd pgvector
   make && make install
   ```

5. **Database Migration**:
   ```bash
   pnpm db:migrate
   ```

6. **Start Development**:
   ```bash
   pnpm run dev  # http://localhost:3000
   ```

## Development Commands

```bash
# Development server
pnpm run dev

# Database operations
pnpm db:migrate          # Run migrations
pnpm db:push            # Push schema changes to database
pnpm db:studio          # Open Drizzle Studio UI for database inspection

# Note: package.json exists in the Vercel starter template, not in this repo
```

## Architecture Overview

### Technology Stack
- **Frontend**: Next.js 13+ with App Router, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes with Vercel AI SDK
- **AI**: OpenAI GPT-4o model with text-embedding-ada-002 for embeddings
- **Database**: PostgreSQL with pgvector extension for vector similarity search
- **ORM**: Drizzle ORM with type-safe schema definitions

### Core Application Flow
```
User Input → useChat hook → API Route (/api/chat)
    ↓
OpenAI GPT-4o with Tool Calling
    ↓
Tools: addResource | getInformation
    ↓
Database Operations (PostgreSQL + pgvector)
    ↓
Streaming Response back to UI
```

### Directory Structure
```
Basic-RAG-Vercel/
├── app/                    # Next.js App Router
│   ├── api/chat/route.ts   # Main AI chat endpoint with tool definitions
│   ├── layout.tsx          # Root layout
│   ├── page.tsx           # Chat UI component (client-side)
│   └── globals.css        # Tailwind styles
├── lib/
│   ├── ai/embedding.ts    # Vector embedding operations and similarity search
│   ├── actions/resources.ts  # Server action for knowledge capture
│   ├── db/
│   │   ├── index.ts       # Drizzle database connection
│   │   ├── migrate.ts     # Migration runner
│   │   ├── schema/        # Database schema definitions
│   │   │   ├── resources.ts   # Resources table schema
│   │   │   └── embeddings.ts  # Embeddings table with vector index
│   │   └── migrations/    # Drizzle migration files
│   ├── env.mjs           # Environment variable validation
│   └── utils.ts          # Utility functions
└── .env                  # Environment configuration
```

## Key Architecture Patterns

### 1. Tool-Based Agent Architecture
The main API endpoint (`app/api/chat/route.ts`) implements a tool-calling agent with two tools:
- **addResource**: Captures user knowledge and stores it in the database
- **getInformation**: Retrieves relevant information using vector similarity search

### 2. RAG Implementation
- **Knowledge Storage**: User information is chunked by sentences and stored with vector embeddings
- **Retrieval**: Uses cosine distance search with similarity threshold (>0.5) to find relevant context
- **Generation**: OpenAI model uses retrieved context to answer questions

### 3. Database Schema
- **resources table**: Stores original user knowledge
- **embeddings table**: Stores vector embeddings (1536 dimensions) with HNSW index for efficient similarity search
- Cascading deletes maintain referential integrity

### 4. Type Safety
- Full TypeScript coverage with Zod schema validation
- Drizzle ORM provides type-safe database operations
- Environment variables validated at startup

## Important Implementation Details

### Vector Search Configuration
- **Embedding Model**: text-embedding-ada-002 (1536 dimensions)
- **Chunking Strategy**: Split by sentences (periods)
- **Similarity Metric**: Cosine distance
- **Retrieval Threshold**: 0.5 minimum similarity
- **Results Limit**: Top 4 most relevant chunks

### Server Actions
The `createResource` server action in `lib/actions/resources.ts` handles:
1. Storing user knowledge in the resources table
2. Generating embeddings for text chunks
3. Inserting embeddings into the vector database

### Environment Variables
Required in `.env`:
- `DATABASE_URL`: PostgreSQL connection string
- `OPENAI_API_KEY`: OpenAI API key
- `NODE_ENV`: Environment (development/production)

## Development Guidelines

### Testing the RAG System
1. **Add Knowledge**: Tell the chatbot personal information (e.g., "My favorite food is pizza")
2. **Verify Storage**: Use `pnpm db:studio` to inspect database entries
3. **Test Retrieval**: Ask questions about stored information
4. **Test Boundaries**: Ask about information you haven't provided (should respond "I don't know")

### Database Inspection
Use Drizzle Studio to view stored knowledge:
```bash
pnpm db:studio
# Opens https://local.drizzle.studio
```

### Modifying the Schema
After changing schema files:
```bash
pnpm db:push  # Push changes to database
```

## Key Files and Responsibilities

| File | Purpose |
|------|---------|
| `app/api/chat/route.ts` | Main AI logic, tool definitions, streaming responses |
| `app/page.tsx` | Chat UI with message display and input handling |
| `lib/ai/embedding.ts` | Vector embedding generation and similarity search |
| `lib/actions/resources.ts` | Server action for knowledge storage |
| `lib/db/schema/` | Type-safe database schema definitions |
| `lib/env.mjs` | Environment variable validation with Zod |

## Notes for Development

- This codebase contains only the customized files; the full project requires the Vercel AI SDK starter template
- The system is designed to only answer questions based on stored knowledge (no hallucination)
- Vector search is optimized for conversational, sentence-level knowledge chunks
- All database operations are type-safe through Drizzle ORM
- The UI uses Vercel AI SDK's `useChat` hook for real-time streaming responses