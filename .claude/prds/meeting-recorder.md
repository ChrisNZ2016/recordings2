---
name: meeting-recorder
description: AI-powered meeting recording system with automated transcription via recall.ai and CloudMailin integration
status: backlog
created: 2025-10-16T03:07:52Z
---

# PRD: Meeting Recorder

## Executive Summary

An automated meeting recording and transcription system that coordinates multiple APIs to capture, transcribe, and summarize meetings. The system receives meeting invitations via email (CloudMailin webhook), dispatches AI recording bots through recall.ai, stores transcripts in a Neon database, and provides an administrative interface for viewing and AI-powered summarization of meeting content using Vercel AI SDK.

**Core Value Proposition:** Eliminate manual meeting note-taking and enable efficient meeting review through automated recording, transcription, and AI-powered summarization.

## Problem Statement

### What problem are we solving?
- Manual meeting note-taking is time-consuming and often incomplete
- Important meeting details are frequently lost or forgotten
- Review and analysis of past meetings is difficult without searchable transcripts
- Coordinating multiple recording tools and services is complex and error-prone

### Why is this important now?
- Remote and hybrid work has increased meeting volume significantly
- AI transcription technology has matured to production-ready quality
- Integration APIs (CloudMailin, recall.ai) now enable seamless automation
- Organizations need better meeting documentation for compliance and knowledge management

## User Stories

### Primary User Persona: Meeting Administrator
**Role:** Administrative user responsible for managing meeting recordings and transcripts

**User Journey 1: Automated Recording**
```
As an administrator,
When I forward a meeting invitation to the system email
Then the system should automatically:
- Extract meeting details from the iCal attachment
- Schedule an AI bot to join the meeting via recall.ai
- Use recall.ai to record and transcribe the meeting
- Receive a webhook containing the meeting details from recall.ai and store the transcript in the database
So that I can access meeting content without manual intervention
```

**User Journey 2: Viewing Transcripts**
```
As an administrator,
When I log into the system dashboard
Then I should see a table listing all recorded meetings
And I should be able to click on any meeting to view its full transcript
So that I can review past meeting content
```

**User Journey 3: AI Summarization**
```
As an administrator,
When I select one or more meetings from the transcript list
Then I should be able to generate AI summaries using Vercel AI SDK
And view key points, action items, and meeting highlights
So that I can quickly understand meeting outcomes without reading full transcripts
```

### Pain Points Being Addressed
1. **Manual coordination overhead** - Remembering to start recordings, managing recording tools
2. **Lost context** - Details forgotten after meetings end
3. **Time waste** - Reading through hour-long transcripts to find specific information
4. **Accessibility** - No searchable archive of past meeting discussions
5. **Integration complexity** - Multiple tools required to achieve end-to-end workflow

## Requirements

### Functional Requirements

#### FR1: Email-to-Meeting Pipeline
- **FR1.1** Accept incoming emails via CloudMailin webhook endpoint
- **FR1.2** Parse iCal attachments from emails using next-ical or similar library
- **FR1.3** Extract meeting metadata:
  - Meeting title
  - Start date/time
  - End date/time
  - Meeting link (Zoom/Meet/Teams URL)
  - Organizer information
  - Participant list 
- **FR1.4** Validate meeting data before processing
- **FR1.5** Handle multiple iCal formats and timezone conversions

#### FR2: Recall.ai Integration
- **FR2.1** Create recall.ai bot via API using extracted meeting details
- **FR2.2** Configure bot to join meeting automatically at scheduled time
- **FR2.3** Handle recall.ai API authentication and error responses
- **FR2.4** Store recall.ai bot ID for tracking
- **FR2.5** Support all meeting platforms handled by recall.ai (Zoom, Google Meet, Microsoft Teams, etc.)

#### FR3: Transcript Processing & Storage
- **FR3.1** Receive transcript webhook from recall.ai when meeting completes
- **FR3.2** Validate webhook authenticity (signature verification)
- **FR3.3** Store transcript in Neon database with metadata:
  - Meeting ID (unique identifier)
  - Meeting title
  - Meeting date/time
  - Duration
  - Platform (Zoom/Meet/Teams)
  - Transcript text (full)
  - Recall.ai bot ID
  - Recording status
  - Created/updated timestamps
  - Participants
- **FR3.4** Handle transcript format variations
- **FR3.5** Support large transcripts (paginated storage if needed)

#### FR4: Admin Dashboard
- **FR4.1** Single admin user authentication (email/password)
- **FR4.2** Protected routes - redirect to login if not authenticated
- **FR4.3** Meeting list view:
  - Display meetings in paginated data table
  - Sortable columns (date, title, duration, status)
  - Search/filter functionality
  - Show meeting metadata at a glance
- **FR4.4** Meeting detail view:
  - Full transcript display
  - Meeting metadata panel
  - Timestamp navigation
  - Readable formatting
- **FR4.5** Responsive design (desktop and tablet support)

#### FR5: AI Summarization
- **FR5.1** Multi-select capability in meeting list
- **FR5.2** "Generate Summary" action for selected meetings
- **FR5.3** Integration with Vercel AI SDK
- **FR5.4** AI-generated output includes:
  - Executive summary
  - Key discussion points
  - Action items
  - Decisions made
  - Important timestamps/quotes
- **FR5.5** Display summaries in the UI
- **FR5.6** Store generated summaries in database
- **FR5.7** Re-generate summaries on demand

### Non-Functional Requirements

#### NFR1: Performance
- **NFR1.1** Webhook endpoints respond within 2 seconds
- **NFR1.2** Dashboard page loads within 3 seconds
- **NFR1.3** Transcript view renders within 2 seconds for transcripts up to 50,000 words
- **NFR1.4** AI summary generation completes within 30 seconds per meeting
- **NFR1.5** Support up to 100 meetings per month without performance degradation

#### NFR2: Reliability
- **NFR2.1** 99% uptime for webhook endpoints
- **NFR2.2** Retry logic for failed recall.ai API calls (3 retries with exponential backoff)
- **NFR2.3** Graceful handling of malformed iCal attachments
- **NFR2.4** Database connection pooling and error recovery
- **NFR2.5** Logging of all webhook events for debugging

#### NFR3: Security
- **NFR3.1** HTTPS-only endpoints
- **NFR3.2** Webhook signature verification (CloudMailin and recall.ai)
- **NFR3.3** Admin authentication with secure session management
- **NFR3.4** Environment variable-based secrets (no hardcoded credentials)
- **NFR3.5** Rate limiting on webhook endpoints (prevent abuse)
- **NFR3.6** SQL injection protection via parameterized queries

#### NFR4: Scalability
- **NFR4.1** Serverless architecture (Vercel Functions)
- **NFR4.2** Database schema supports >1000 meetings without query slowdown
- **NFR4.3** Efficient database indexes on frequently queried fields
- **NFR4.4** Pagination for large meeting lists

#### NFR5: Maintainability
- **NFR5.1** TypeScript throughout (type safety)
- **NFR5.2** Modular architecture (separation of concerns)
- **NFR5.3** API route structure:
  - `/api/webhooks/cloudmailin` - Receive meeting invites
  - `/api/webhooks/recall` - Receive transcripts
  - `/api/meetings` - CRUD operations
  - `/api/summaries` - AI summary generation
- **NFR5.4** Environment-based configuration (dev/staging/production)
- **NFR5.5** Comprehensive error logging

## Success Criteria

### Measurable Outcomes

1. **Automation Success Rate**
   - Target: 95% of received meeting invites successfully trigger recall.ai bots
   - Metric: (Successful bot creations / Total emails received) × 100

2. **Transcript Capture Rate**
   - Target: 90% of scheduled meetings produce stored transcripts
   - Metric: (Transcripts stored / Bots created) × 100

3. **System Reliability**
   - Target: 99% webhook uptime
   - Metric: Monthly uptime percentage

4. **User Efficiency**
   - Target: Admin can access any meeting transcript within 3 clicks
   - Metric: Click-through navigation depth

5. **AI Summary Quality**
   - Target: Summaries generated within 30 seconds
   - Metric: Average generation time per meeting

### Key Performance Indicators (KPIs)

- **Monthly meetings processed:** Track volume growth
- **Database storage utilization:** Monitor Neon usage
- **API call volume:** recall.ai and Vercel AI SDK usage
- **Failed webhook events:** Error rate monitoring
- **Time saved:** Estimated hours saved vs. manual note-taking

## Technical Architecture

### Tech Stack
- **Framework:** Next.js 15 (App Router)
- **Language:** TypeScript
- **Deployment:** Vercel
- **Database:** Neon (Serverless Postgres)
- **ORM:** Drizzle ORM or Prisma
- **Authentication:** NextAuth.js
- **UI Components:** Tailwind CSS + shadcn/ui
- **AI SDK:** Vercel AI SDK
- **Email Processing:** CloudMailin webhook
- **iCal Parsing:** next-ical or ical.js
- **Recording Service:** recall.ai API

### Data Schema (Conceptual)

```typescript
// meetings table
{
  id: uuid (PK)
  title: string
  meeting_date: timestamp
  duration_minutes: integer
  platform: enum (zoom, google_meet, teams, other)
  meeting_url: string
  organizer_email: string
  recall_bot_id: string (nullable)
  transcript: text (nullable, large)
  status: enum (scheduled, recording, completed, failed)
  created_at: timestamp
  updated_at: timestamp
}

// summaries table
{
  id: uuid (PK)
  meeting_id: uuid (FK -> meetings.id)
  summary_text: text
  key_points: jsonb
  action_items: jsonb
  generated_at: timestamp
  model_used: string (e.g., "gpt-4")
}

// admin_users table
{
  id: uuid (PK)
  email: string (unique)
  password_hash: string
  created_at: timestamp
}
```

### Integration Flow

```
1. Email Received by CloudMailin
   └─> CloudMailin webhook → /api/webhooks/meeting-booker
       └─> Parse iCal attachment
           └─> Store meeting in database (status: scheduled)
               └─> Call recall.ai API to create bot
                   └─> Update meeting with bot_id (status: planned)

2. Meeting Occurs
   └─> recall.ai bot joins automatically
       └─> Records and transcribes

3. Meeting Completes
   └─> recall.ai webhook → /api/webhooks/transcript-inwards
       └─> Store transcript in database
           └─> Update meeting status (status: completed)

4. Admin Views
   └─> Login → Dashboard
       └─> View meeting list
           └─> Click meeting → View transcript
               └─> Select meeting(s) → Generate AI summary
                   └─> Vercel AI SDK → Store summary
```

## Constraints & Assumptions

### Technical Limitations
- CloudMailin free tier limits (may require paid plan for volume)
- recall.ai API rate limits and pricing
- Vercel AI SDK token usage costs
- Neon database storage limits (5GB free tier, may need paid plan)
- Vercel function execution time limits (10s for Hobby, 300s for Pro)

### Timeline Constraints
- MVP target: 4-6 weeks for core functionality
- Assumes APIs (CloudMailin, recall.ai) are stable and documented

### Resource Limitations
- Single developer implementation
- No dedicated QA environment initially
- Limited budget for API costs (optimize for <100 meetings/month)

### Assumptions
1. Meeting invitations will consistently include iCal attachments
2. recall.ai supports all required meeting platforms
3. Transcripts will be in consistent format from recall.ai
4. Admin user can manage system without multi-user collaboration features
5. 100 meetings/month is sufficient for initial use case
6. English language transcripts (no i18n required initially)

## Out of Scope

### Explicitly NOT Building (v1)
1. **Multi-user access** - Only single admin user
2. **Participant-level permissions** - No per-meeting access control
3. **Real-time transcription** - Only post-meeting transcripts
4. **Recording playback** - Text transcripts only, no audio/video
5. **Calendar integration** - No bidirectional sync with Google Calendar/Outlook
6. **Mobile app** - Web UI only (responsive design for tablets)
7. **Transcript editing** - Read-only transcripts
8. **Advanced analytics** - No sentiment analysis, speaker analytics, or trends
9. **Export functionality** - No PDF/Word export (copy-paste acceptable)
10. **Notification system** - No email/Slack alerts for completed recordings
11. **Meeting scheduling** - Only processes received invitations
12. **Custom AI prompts** - Fixed summary format only
13. **API for third-party access** - Internal use only

### Future Considerations
- Multi-tenant support (multiple organizations)
- Advanced search (semantic search, keywords)
- Integration with Slack/Teams for summary distribution
- Custom branding/white-labeling
- GDPR compliance features (data retention policies, deletion)

## Dependencies

### External Dependencies
1. **CloudMailin**
   - Service availability
   - Webhook reliability
   - Email parsing accuracy
   - Pricing: Free tier → paid if >100 emails/month

2. **recall.ai**
   - API stability
   - Bot reliability (joining meetings)
   - Transcription quality
   - Platform support (Zoom, Meet, Teams)
   - Pricing: Per-meeting cost

3. **Vercel AI SDK**
   - API availability
   - Model access (OpenAI/Anthropic)
   - Token pricing

4. **Neon**
   - Database uptime
   - Storage limits
   - Connection reliability

### Internal Team Dependencies
1. **Design** - UI/UX for dashboard and transcript views
2. **DevOps** - Environment setup, secrets management
3. **Testing** - Manual QA for webhook flows
4. **Documentation** - User guide for admin, setup instructions

### Third-Party Integrations
- NextAuth.js documentation and examples
- shadcn/ui component library
- Tailwind CSS ecosystem
- TypeScript tooling

## Risk Analysis

### High Risk
1. **recall.ai API reliability** - Core dependency for recording
   - Mitigation: Implement comprehensive error logging, retry logic, fallback notifications

2. **Webhook delivery failures** - Could miss meetings
   - Mitigation: Webhook retry mechanisms, monitoring/alerts

### Medium Risk
1. **Cost overruns** - API usage exceeds budget
   - Mitigation: Usage monitoring dashboard, rate limiting, monthly caps

2. **Transcript quality** - AI transcription errors
   - Mitigation: No automated actions on transcripts, human review

### Low Risk
1. **Database performance** - Query slowdown at scale
   - Mitigation: Proper indexing, pagination, caching

2. **Authentication security** - Admin account compromise
   - Mitigation: Strong password requirements, session management, HTTPS only

## Implementation Phases

### Phase 1: Foundation (Week 1-2)
- Next.js project setup with TypeScript
- Neon database setup and schema
- Basic authentication (NextAuth.js)
- Dashboard layout and routing

### Phase 2: Email Processing (Week 2-3)
- CloudMailin webhook endpoint
- iCal parsing implementation
- Database integration for meetings
- Error handling and validation

### Phase 3: Recording Integration (Week 3-4)
- recall.ai API client
- Bot creation workflow
- Transcript webhook endpoint
- Status tracking

### Phase 4: UI & Viewing (Week 4-5)
- Meeting list data table
- Transcript detail view
- Search and filtering
- Responsive design

### Phase 5: AI Summaries (Week 5-6)
- Vercel AI SDK integration
- Summary generation workflow
- Summary storage and display
- Multi-select functionality

### Phase 6: Polish & Deploy (Week 6)
- Error handling improvements
- Logging and monitoring
- Production deployment
- Documentation

## Acceptance Criteria

### Definition of Done
- [ ] CloudMailin webhook successfully receives and parses meeting invitations
- [ ] recall.ai bots are created automatically for extracted meetings
- [ ] Transcripts are stored in Neon database when meetings complete
- [ ] Admin can log in and view paginated list of all meetings
- [ ] Admin can click any meeting to view full transcript
- [ ] Admin can select meetings and generate AI summaries
- [ ] All API endpoints have error handling and logging
- [ ] Application is deployed to Vercel production
- [ ] Environment variables are configured for all services
- [ ] Basic user documentation is complete

### Quality Gates
- TypeScript compiles without errors
- All webhook endpoints return appropriate HTTP status codes
- Database queries use parameterized statements (no SQL injection)
- Sensitive credentials are in environment variables
- UI is responsive on desktop and tablet
- No console errors in production build
