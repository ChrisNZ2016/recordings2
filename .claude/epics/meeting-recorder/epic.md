---
name: meeting-recorder
status: backlog
created: 2025-10-16T03:42:31Z
progress: 0%
prd: .claude/prds/meeting-recorder.md
github: https://github.com/ChrisNZ2016/recordings2/issues/1
---

# Epic: meeting-recorder

## Overview
Build an AI-powered meeting recording system that automates the entire meeting lifecycle: email reception → bot scheduling → recording/transcription → storage → AI summarization. The system leverages Next.js 15 serverless architecture with webhook-driven automation, eliminating manual intervention while providing a clean admin interface for transcript viewing and AI-powered analysis.

## Architecture Decisions

### Framework & Deployment
- **Next.js 15 App Router**: Leverages server components, server actions, and integrated API routes for a unified codebase
- **Vercel Serverless**: Natural hosting choice for Next.js with automatic scaling and zero-config deployments
- **TypeScript**: Type safety across frontend, backend, and API integrations

### Database & ORM
- **Neon Serverless Postgres**: Auto-scaling, serverless-optimized database with excellent Vercel integration
- **Drizzle ORM**: Lightweight, type-safe ORM with minimal overhead (prefer over Prisma for this scale)
- **Schema design**: Normalized structure with `meetings`, `summaries`, and `admin_users` tables

### Authentication
- **NextAuth.js v5**: Industry-standard auth with credentials provider for single-admin use case
- **Session-based auth**: Simple email/password login with secure session cookies

### UI Framework
- **Tailwind CSS 4**: Already in the project stack (see CLAUDE.md)
- **shadcn/ui**: Provides pre-built, accessible components (data tables, forms, dialogs)
- **Responsive design**: Desktop-first with tablet support

### API Integrations Strategy
- **CloudMailin webhook**: Inbound email processing with iCal parsing (use `ical.js` library)
- **recall.ai API client**: Custom HTTP client with retry logic and error handling
- **Vercel AI SDK**: Built-in streaming support for summary generation (use OpenAI GPT-4)

## Technical Approach

### Frontend Components
1. **Authentication Layer**
   - Login page (`/login`)
   - Auth middleware protecting dashboard routes
   - Session management with NextAuth

2. **Dashboard UI** (`/dashboard`)
   - Data table with sortable columns, pagination, search
   - Multi-select rows for batch AI summarization
   - Meeting detail modal/page for transcript viewing
   - Summary display panel

### Backend Services

**API Routes Structure:**
```
/api/auth/[...nextauth]     # NextAuth endpoints
/api/webhooks/cloudmailin   # Receive meeting invites
/api/webhooks/recall        # Receive transcripts
/api/meetings               # CRUD operations
/api/summaries              # AI summary generation
```

**Core Business Logic:**
1. **Email Processing Pipeline** (`/api/webhooks/cloudmailin`)
   - Validate CloudMailin signature
   - Parse iCal attachment using `ical.js`
   - Extract meeting metadata (title, time, URL, participants)
   - Store meeting record (status: "scheduled")
   - Call recall.ai to create bot
   - Update meeting with bot_id (status: "planned")

2. **Transcript Ingestion** (`/api/webhooks/recall`)
   - Validate recall.ai webhook signature
   - Extract transcript and metadata
   - Update meeting record (status: "completed")
   - Store full transcript text

3. **AI Summarization** (`/api/summaries`)
   - Accept meeting IDs array
   - Fetch transcripts from database
   - Stream summaries using Vercel AI SDK
   - Store generated summaries with metadata
   - Return formatted summary to UI

### Infrastructure

**Database Schema:**
```sql
-- meetings table
CREATE TABLE meetings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  meeting_date TIMESTAMPTZ NOT NULL,
  duration_minutes INTEGER,
  platform VARCHAR(50),
  meeting_url TEXT,
  organizer_email VARCHAR(255),
  recall_bot_id VARCHAR(255),
  transcript TEXT,
  participants JSONB,
  status VARCHAR(50) NOT NULL DEFAULT 'scheduled',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- summaries table
CREATE TABLE summaries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  meeting_id UUID REFERENCES meetings(id) ON DELETE CASCADE,
  summary_text TEXT NOT NULL,
  key_points JSONB,
  action_items JSONB,
  generated_at TIMESTAMPTZ DEFAULT NOW(),
  model_used VARCHAR(50)
);

-- admin_users table
CREATE TABLE admin_users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_meetings_date ON meetings(meeting_date DESC);
CREATE INDEX idx_meetings_status ON meetings(status);
CREATE INDEX idx_summaries_meeting_id ON summaries(meeting_id);
```

**Environment Variables:**
```
DATABASE_URL=<Neon connection string>
NEXTAUTH_SECRET=<random secret>
NEXTAUTH_URL=<deployment URL>
CLOUDMAILIN_SECRET=<webhook signature key>
RECALL_AI_API_KEY=<recall.ai key>
OPENAI_API_KEY=<OpenAI key>
```

**Monitoring & Logging:**
- Vercel Analytics for performance
- Console logging for webhook events (structured JSON)
- Error tracking via Vercel runtime logs

## Implementation Strategy

### Development Approach
1. **Bottom-up**: Build database schema → API routes → UI components
2. **Test with mocks**: Use webhook testing tools (webhook.site) for CloudMailin/recall.ai
3. **Incremental deployment**: Deploy to Vercel preview branches throughout development

### Risk Mitigation
- **Webhook reliability**: Implement idempotency keys to handle duplicate deliveries
- **API failures**: Retry logic with exponential backoff (3 attempts)
- **Cost control**: Rate limiting on webhook endpoints, cap AI summary requests
- **Data validation**: Strict TypeScript types + runtime validation (Zod)

### Testing Approach
- **Manual testing**: Use CloudMailin test interface and recall.ai sandbox
- **Webhook simulation**: Create test scripts to POST to local endpoints
- **Database queries**: Test with Drizzle Studio (visual DB browser)
- **No unit tests initially**: Focus on integration testing for MVP

## Task Breakdown Preview

High-level implementation tasks (≤10 total):

1. **Database Setup & Schema**: Create Neon database, define schema with Drizzle, run migrations, seed admin user
2. **Authentication System**: Implement NextAuth.js with credentials provider, create login page, add auth middleware
3. **CloudMailin Webhook & iCal Parsing**: Build `/api/webhooks/cloudmailin` endpoint, parse iCal attachments, store meetings in DB
4. **recall.ai Integration**: Create API client, implement bot creation workflow, handle errors/retries
5. **Transcript Webhook Handler**: Build `/api/webhooks/recall` endpoint, validate signatures, update meetings with transcripts
6. **Dashboard UI & Meeting List**: Create meeting list page with shadcn/ui data table, add sorting/pagination/search
7. **Transcript Detail View**: Build meeting detail page/modal, display full transcript with metadata
8. **AI Summarization Feature**: Integrate Vercel AI SDK, create `/api/summaries` endpoint, add multi-select and summary UI
9. **Error Handling & Logging**: Add comprehensive error handling, structured logging for all webhooks
10. **Deployment & Configuration**: Configure environment variables, deploy to Vercel, test end-to-end flow

## Dependencies

### External Services
- **CloudMailin**: Email-to-webhook service (configure forwarding address)
- **recall.ai**: Meeting bot and transcription service (API key required)
- **Neon**: Serverless Postgres database (connection string)
- **Vercel**: Hosting and serverless functions (deployment target)
- **OpenAI**: AI model for summary generation (API key via Vercel AI SDK)

### Internal Dependencies
- Existing Next.js 15 project structure (already setup per CLAUDE.md)
- Tailwind CSS 4 configuration (already in place)
- TypeScript configuration (already configured)

### NPM Packages (New)
```json
{
  "dependencies": {
    "next-auth": "^5.0.0-beta",
    "drizzle-orm": "latest",
    "@neondatabase/serverless": "latest",
    "ical.js": "latest",
    "ai": "latest",
    "@ai-sdk/openai": "latest",
    "zod": "latest",
    "bcryptjs": "latest"
  },
  "devDependencies": {
    "drizzle-kit": "latest",
    "@types/bcryptjs": "latest"
  }
}
```

## Success Criteria (Technical)

### Performance Benchmarks
- Webhook response time: <2s (CloudMailin, recall.ai)
- Dashboard load time: <3s (initial page load)
- Transcript rendering: <2s (up to 50k words)
- AI summary generation: <30s per meeting

### Quality Gates
- [ ] All TypeScript types compile without errors
- [ ] No hardcoded secrets (all in environment variables)
- [ ] Database queries use parameterized statements
- [ ] Webhook endpoints validate signatures
- [ ] Error responses include appropriate HTTP status codes
- [ ] UI is responsive on desktop and tablet viewports

### Acceptance Criteria
- [ ] Email with iCal attachment triggers bot creation
- [ ] recall.ai bot successfully scheduled and tracked
- [ ] Completed meetings populate database with transcripts
- [ ] Admin can login and view meeting list
- [ ] Admin can view full transcript for any meeting
- [ ] Admin can generate AI summaries for selected meetings
- [ ] Summaries are stored and displayed correctly

## Estimated Effort

### Overall Timeline
**4-6 weeks** (single developer, part-time work)

### Task-Level Estimates
1. Database Setup & Schema: **2-3 days**
2. Authentication System: **2-3 days**
3. CloudMailin Webhook & iCal Parsing: **3-4 days**
4. recall.ai Integration: **3-4 days**
5. Transcript Webhook Handler: **2-3 days**
6. Dashboard UI & Meeting List: **3-4 days**
7. Transcript Detail View: **2-3 days**
8. AI Summarization Feature: **3-4 days**
9. Error Handling & Logging: **2-3 days**
10. Deployment & Configuration: **2-3 days**

### Critical Path Items
1. **Database schema** (blocks all backend work)
2. **CloudMailin webhook** (blocks recall.ai integration)
3. **recall.ai bot creation** (blocks transcript testing)
4. **Authentication** (blocks dashboard access)

### Resource Requirements
- 1 full-stack developer (Next.js, TypeScript, React)
- Access to all service accounts (CloudMailin, recall.ai, Neon, Vercel)
- Budget for API costs (~$50-100/month for initial testing)

## Tasks Created

- [ ] #4 - Database Setup & Schema (parallel: false)
- [ ] #7 - Authentication System (parallel: true)
- [ ] #8 - CloudMailin Webhook & iCal Parsing (parallel: false)
- [ ] #9 - recall.ai Integration (parallel: false)
- [ ] #2 - Transcript Webhook Handler (parallel: false)
- [ ] #3 - Dashboard UI & Meeting List (parallel: false)
- [ ] #5 - Transcript Detail View (parallel: false)
- [ ] #6 - AI Summarization Feature (parallel: false)
- [ ] #10 - Error Handling & Logging (parallel: true)
- [ ] #11 - Deployment & Configuration (parallel: false)

**Total tasks**: 10
**Parallel tasks**: 2 (7, 10)
**Sequential tasks**: 8
**Estimated total effort**: 176-256 hours (4-6 weeks at 40 hours/week)
