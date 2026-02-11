# Utho AI Automation — Implementation Plan: Phase 1 & Phase 2

**Document Version:** 1.0  
**Date:** 11 February 2026  
**Prepared By:** AI Automation Architect  
**Purpose:** Detailed implementation blueprint for Phase 1 (Marketing & Lead Gen) and Phase 2 (Sales Automation)  
**Prerequisite:** Read `Workflow_Ecosystem_Audit.md` for current-state analysis  
**Total Workflows:** 16 (4 shared + 6 Phase 1 + 6 Phase 2)

---

## Table of Contents

1. [Foundation Architecture Decisions](#1-foundation-architecture-decisions)
2. [Shared Infrastructure Workflows](#2-shared-infrastructure-workflows)
3. [Phase 1: Cold Outreach Workflows](#3-phase-1-cold-outreach-workflows)
4. [Phase 1: Inbound Signup Workflows](#4-phase-1-inbound-signup-workflows)
5. [Phase 2: AI Lead Scoring](#5-phase-2-ai-lead-scoring)
6. [Phase 2: Lead Routing & Assignment](#6-phase-2-lead-routing--assignment)
7. [Phase 2: Sales Enablement](#7-phase-2-sales-enablement)
8. [CRM Schema Requirements](#8-crm-schema-requirements)
9. [Build Order & Timeline](#9-build-order--timeline)
10. [Complete System View](#10-complete-system-view)

---

## 1. Foundation Architecture Decisions

These 5 decisions govern EVERY workflow. They were made before designing any individual automation.

### Decision 1: Single Source of Truth → Zoho CRM

**Rule:** Every workflow reads from and writes to Zoho CRM. No Google Sheets as data stores.

- Zoho CRM exists at Utho (confirmed via `added_to_ZOHO` column in Cold Enriched Data sheet)
- All lead states, engagement history, cadence tracking, and scoring live in CRM
- Google Sheets may be used ONLY for reporting/exports, never as operational data source

### Decision 2: Event-Driven Architecture (Not Polling)

**Rule:** Use webhooks wherever possible. Schedule triggers only for time-based cadence checks.

| Instead Of | Use |
|------------|-----|
| Poll sheet every 5 min for new signups | Zoho CRM webhook: "New Lead Created" |
| Poll sheet every 1 min for assignments | Zoho CRM webhook: "Assigned To changed" |
| Poll for email replies | Brevo webhook: "Reply Received" |
| Daily cadence check (which leads need follow-up today?) | ✅ Keep as scheduled trigger |
| Daily SLA check | ✅ Keep as scheduled trigger |

### Decision 3: Separate Cadence Workflows + Shared Reply Detection

**Rule:** Keep email cadence as separate daily workflows (like current WF4/5/6 pattern) BUT add a shared Reply Detection workflow that updates CRM status. Each cadence workflow checks `lead_replied = false` before sending.

```
Why not 1 mega-workflow?
- n8n Wait nodes hold executions for days (resource waste)
- Hard to debug 8-day-long executions
- If n8n restarts, in-progress waits may be lost

Why separate workflows + reply detection?
- Each workflow is simple, testable, debuggable
- Reply detection is instant (webhook-driven)
- CRM is the coordination layer (not shared variables)
```

### Decision 4: AI Integration — Where It Adds Real Value

| AI Use Case | Value | Priority | Model | Temperature |
|-------------|-------|----------|-------|-------------|
| Reply Intent Classification | 🔴 CRITICAL — stops wrong follow-ups | Must Have | gpt-4o-mini | 0.1 (deterministic) |
| Lead Scoring | 🔴 CRITICAL — determines routing | Must Have | Rule-based + AI assist | N/A |
| Email Personalization (hook paragraph) | 🟠 HIGH — improves response rates | Should Have | gpt-4o-mini | 0.6 (creative) |
| Sales Brief Generation | 🟠 HIGH — helps reps convert | Should Have | gpt-4o-mini | 0.4 |
| Inbound Lead Qualification | 🟡 MEDIUM — segment classification | Nice to Have | gpt-4o-mini | 0.2 |

**Key principle:** AI generates ONLY the personalized hook (2-3 sentences). The rest of the email (Utho story, USPs, social proof, CTA, signature) uses a fixed template. This ensures consistency, control, and low cost.

### Decision 5: Centralized Error Handling

**Rule:** Every workflow routes errors to a shared Error Handler sub-workflow. Failed sends NEVER mark leads as "Sent."

```
Error Flow:
  Any node fails →
    ├── DO NOT update lead status
    ├── Call Error Handler sub-workflow:
    │     CRITICAL → Slack #automation-alerts channel
    │     HIGH     → Slack DM to vishal.m
    │     LOW      → Log only
    └── CRM: lead.automation_status = "ERROR"
```

---

## 2. Shared Infrastructure Workflows

These are sub-workflows called by other workflows. They are NOT triggered independently.

### WF-0A: Error Handler (Sub-Workflow)

**Called by:** Every workflow on failure  
**Purpose:** Centralized error logging, notification, and CRM status update

```
Input: { workflow_name, node_name, error_message, lead_email, severity }

Flow:
  → Switch on severity:
      CRITICAL → Post to Slack #automation-alerts channel + log
      HIGH     → Slack DM to vishal.m + log
      LOW      → Log only (silent retry expected)
  → Update Zoho CRM:
      lead.automation_error = "[workflow_name] > [node_name]: [error_message]"
      lead.automation_status = "ERROR"
```

### WF-0B: Compliance Check (Sub-Workflow)

**Called by:** Every outbound workflow before sending any message  
**Purpose:** Verify lead has not opted out before sending email/WhatsApp

```
Input: { email, channel ("email" | "whatsapp") }

Flow:
  → Query Zoho CRM by email
  → If channel = "email" AND contact.opt_out_email = true → Return { send: false }
  → If channel = "whatsapp" AND contact.opt_out_whatsapp = true → Return { send: false }
  → Else → Return { send: true }
```

### WF-0C: AI Email Generation (Sub-Workflow)

**Called by:** WF-1A (Mail 1) and optionally WF-1B/1C for personalized follow-ups  
**Purpose:** Generate personalized hook paragraph using lead data

```
Input: { first_name, organization_name, industry, title, company_size, 
         email_position ("mail_1" | "followup_1" | "followup_2") }

Flow:
  → Build system prompt:
      "You are a sales copywriter for Utho, India's own cloud platform.
       Target audience: Indian tech leaders and decision-makers.
       
       Generate a 2-3 sentence personalized opening for a cold email.
       The lead is {title} at {organization_name} in {industry} with {company_size} employees.
       This is {email_position} in the outreach sequence.
       
       Rules:
       - Write in simple, conversational English
       - Reference their industry or role specifically
       - Do NOT use buzzwords like 'synergy' or 'leverage'
       - Do NOT make claims about what their company does (you might be wrong)
       - Be genuine, curious, and respectful
       - India-first context (reference Indian market if relevant)"

  → OpenAI API (gpt-4o-mini, temperature 0.6, max_tokens 150)
  → Return { personalized_hook }
```

### WF-0D: AI Intent Classification (Sub-Workflow)

**Called by:** WF-1E (Reply Detection)  
**Purpose:** Classify email replies into actionable intents

```
Input: { reply_text, lead_context }

Flow:
  → Build system prompt:
      "Classify this email reply into exactly ONE category:
       
       INTERESTED - Wants to learn more, book a call, continue conversation
       NOT_NOW - Not interested right now but may be open later
       NOT_INTERESTED - Does not want the service
       UNSUBSCRIBE - Explicitly wants to stop receiving emails
       OUT_OF_OFFICE - Automatic out-of-office reply
       AUTO_REPLY - Automated system response (not human)
       QUESTION - Has a specific question about Utho
       
       Respond in JSON: { \"intent\": \"CATEGORY\", \"reason\": \"one sentence\" }"

  → OpenAI API (gpt-4o-mini, temperature 0.1, max_tokens 50)
  → Parse JSON response
  → Return { intent, reason }
```

---

## 3. Phase 1: Cold Outreach Workflows

### WF-1A: Cold Email — Mail 1 (Initial Outreach)

**Trigger:** Schedule — Daily at 10:02 AM  
**Rebuilds:** Current WF4  
**Sender:** Lalit Mohan (lalitmohan@utho.io)

```
Flow:
  Schedule Trigger (10:02 AM)
    ↓
  Query Zoho CRM:
    lead_type = "cold" AND lead_market = "india"
    AND cadence_status = "not_started"
    AND opt_out_email ≠ true AND email IS NOT NULL
    LIMIT 50
    ↓
  Loop (batch of 5):
    ↓
    CALL WF-0B: Compliance Check → if opted out, SKIP
    ↓
    CALL WF-0C: AI Email Generation → get personalized_hook
    ↓
    Build email:
      Subject: A/B test from 3-4 pre-approved variants
      Body:
        [AI personalized hook — 2-3 sentences]
        [Fixed Utho pitch — value props, social proof, 22K+ users]
        [CTA: Book discovery call / Reply]
        [Unsubscribe link] ← COMPLIANCE
      Headers: standard (no In-Reply-To for first email)
    ↓
    Send via Brevo (n8n credential management — NO raw API key)
    ↓
    SUCCESS → Update Zoho CRM:
      cadence_status = "mail_1_sent"
      mail_1_sent_at = now
      mail_1_message_id = brevo.messageId
      next_followup_date = now + 3 days
      next_followup_type = "email_followup_1"
    ↓
    FAILURE → CALL WF-0A: Error Handler (HIGH)
              Do NOT update CRM status (lead stays in queue)
    ↓
    Wait 1 minute → Next batch
```

**Key improvements over current WF4:**
- Zoho CRM instead of Google Sheets
- AI-generated personalized hook
- Compliance check + unsubscribe link
- Brevo via credential management (no exposed API key)
- Failed sends do NOT mark as "Sent"

---

### WF-1B: Cold Email — Follow-up 1 (Day 3)

**Trigger:** Schedule — Daily at 10:10 AM  
**Rebuilds:** Current WF5

```
Flow:
  Schedule Trigger (10:10 AM)
    ↓
  Query Zoho CRM:
    cadence_status = "mail_1_sent"
    AND next_followup_date <= today
    AND next_followup_type = "email_followup_1"
    AND lead_replied = false          ← CRITICAL: Skip if replied
    AND opt_out_email ≠ true
    LIMIT 50
    ↓
  Loop (batch of 5):
    ↓
    CALL WF-0B: Compliance Check
    ↓
    Build follow-up email:
      Subject: Re: [original subject] (thread continuation)
      Headers:
        In-Reply-To: mail_1_message_id  ← KEEP threading pattern
        References: mail_1_message_id
      Body:
        [Personalized nudge using org name + industry — NOT just first_name]
        [Soft CTA: Book slot / Reply]
        [Unsubscribe link]
    ↓
    Send via Brevo → SUCCESS:
      cadence_status = "followup_1_sent"
      followup_1_sent_at = now
      next_followup_date = now + 2 days (for WhatsApp)
      next_followup_type = "whatsapp_touch"
    ↓
    FAILURE → Error Handler → Don't update status
    ↓
    Wait 1 minute → Next batch
```

**Key improvement:** `lead_replied = false` check prevents sending follow-ups to leads who already responded.

---

### WF-1C: Cold Email — Follow-up 2 / Breakup (Day 8)

**Trigger:** Schedule — Daily at 10:15 AM  
**Rebuilds:** Current WF6

```
Flow:
  Schedule Trigger (10:15 AM)
    ↓
  Query Zoho CRM:
    cadence_status = "whatsapp_sent" OR cadence_status = "followup_1_sent"
    AND next_followup_date <= today
    AND next_followup_type = "email_followup_2"
    AND lead_replied = false
    AND opt_out_email ≠ true
    LIMIT 50
    ↓
  Loop (batch of 5):
    ↓
    Compliance Check → Build breakup email (threaded) → Send
    ↓
    SUCCESS:
      cadence_status = "cadence_completed"
      mailing = "completed"
      ↓
      Post-cadence routing:
        IF lead_replied = false AND email_opens > 0:
          → nurture_pool = true (they opened but didn't engage — nurture)
        IF lead_replied = false AND email_opens = 0:
          → cold_archive = true (no engagement at all — archive)
    ↓
    Wait 1 minute → Next batch
```

**Key improvement:** Post-cadence routing. Leads don't just disappear after 3 emails — they get routed to nurture pools or archives based on engagement.

---

### WF-1D: Cold WhatsApp Touchpoint (Day 5-6) — NEW

**Trigger:** Schedule — Daily at 10:30 AM  
**Status:** Entirely new workflow (doesn't exist today)  
**Provider:** WATI (WhatsApp Business API)

```
Flow:
  Schedule Trigger (10:30 AM)
    ↓
  Query Zoho CRM:
    cadence_status = "followup_1_sent"
    AND next_followup_type = "whatsapp_touch"
    AND next_followup_date <= today
    AND whatsapp_sent = false
    AND lead_replied = false
    AND opt_out_whatsapp ≠ true
    AND phone IS NOT NULL
    LIMIT 30
    ↓
  Loop (batch of 3, wait 2 min):
    ↓
    Compliance Check (WhatsApp channel)
    ↓
    Send via WATI API:
      Template (pre-approved by Meta):
        "Hi {first_name}, I reached out via email about how Utho can
         help {organization_name} save 60% on cloud costs.
         Would a quick 10-min call work this week?
         Book here: {calendar_link}
         Reply STOP to opt out."
    ↓
    SUCCESS:
      whatsapp_sent = true
      whatsapp_sent_at = now
      cadence_status = "whatsapp_sent"
      next_followup_date = now + 3 days
      next_followup_type = "email_followup_2"
    ↓
    FAILURE → Error Handler
```

**Cadence with WhatsApp:**
```
Day 0: Mail 1 (Email) — Value pitch
Day 3: Follow-up 1 (Email) — Gentle nudge
Day 5: WhatsApp Message ← NEW CHANNEL
Day 8: Follow-up 2 (Email) — Breakup
```

---

### WF-1E: Reply Detection & Intent Router — NEW (Critical)

**Trigger:** Brevo webhook — "Email Reply Received"  
**Status:** Most important new workflow. Currently missing entirely.

```
Flow:
  Brevo Webhook: reply received
    ↓
  Extract: reply_text, original_message_id, sender_email
    ↓
  Lookup Zoho CRM: find lead by email
    ↓
  CALL WF-0D: AI Intent Classification
    → Input: reply_text + lead context
    → Output: { intent, reason }
    ↓
  Switch on intent:

    INTERESTED:
      → CRM: lead_replied = true, reply_intent = "interested"
      → CRM: cadence_status = "replied_interested" (stops ALL follow-ups)
      → Trigger WF-3A: Lead Scoring (immediate score boost)
      → Route to Phase 2: Sales Pipeline
      → Slack: "🔥 Hot reply! {email} — Interested"

    NOT_NOW:
      → CRM: lead_replied = true, reply_intent = "not_now"
      → CRM: cadence_status = "replied_not_now"
      → Move to nurture pool (re-engage in 30 days)
      → Auto-reply: "No worries! We'll circle back when timing is better."

    NOT_INTERESTED:
      → CRM: lead_replied = true, reply_intent = "not_interested"
      → CRM: cadence_status = "replied_not_interested"
      → Stop all communication (but don't opt out — may change mind)

    UNSUBSCRIBE:
      → CRM: opt_out_email = true, opt_out_whatsapp = true
      → CRM: cadence_status = "unsubscribed"
      → Stop ALL communication
      → Auto-reply: "You've been unsubscribed."

    OUT_OF_OFFICE:
      → CRM: reply_intent = "ooo"
      → Reschedule next follow-up: today + 7 days
      → Cadence continues after OOO period

    AUTO_REPLY:
      → Log and ignore (don't treat as reply)

    QUESTION:
      → CRM: lead_replied = true, reply_intent = "question"
      → Route to sales with context
      → Slack: "❓ Lead asked a question: {reply_text}"
```

---

### WF-1F: Bounce & Engagement Handler — NEW

**Trigger:** Brevo webhooks for bounce, open, click events

```
Brevo Webhook: "hard_bounce"
  → CRM: email_status = "bounced", cadence_status = "bounced"
  → Stop all future emails

Brevo Webhook: "soft_bounce" (3rd time)
  → CRM: email_status = "soft_bounced"
  → Flag for review

Brevo Webhook: "opened"
  → CRM: email_opens += 1, last_opened_at = now
  → Feed to Lead Scoring (Phase 2)

Brevo Webhook: "clicked"
  → CRM: email_clicks += 1, last_clicked_at = now
  → If clicked "Book Meeting" → high engagement signal
  → Feed to Lead Scoring (Phase 2)
```

---

## 4. Phase 1: Inbound Signup Workflows

### WF-2A: Signup Event Handler (Replaces WF2)

**Trigger:** Webhook from Utho platform — "New User Signed Up"  
**Rebuilds:** Current WF2 (but fundamentally different architecture)

```
Flow:
  Webhook: new signup received
    ↓
  Deduplication: Check Zoho CRM — does contact exist?
    YES → Update existing record
          Check if in cold cadence → if yes, STOP cold cadence (they signed up!)
    NO  → Create new contact in CRM
    ↓
  Segment (AI-assisted or rule-based):
    HIGH_VALUE: Enterprise company, known brand, decision-maker title
    MEDIUM: SMB, clear use case
    EXPLORER: Individual, testing
    ↓
  Update CRM:
    lead_type = "inbound_signup", segment = result
    signup_date = now, nurture_status = "new"
    ↓
  Immediate actions:
    ALL → Send segment-appropriate welcome email (Brevo)
    HIGH_VALUE → Also WhatsApp welcome + Slack #sales-alerts
    MEDIUM → Slack DM to vishal.m
    ↓
  Set nurture schedule:
    next_nurture_date = now + 2 days
    next_nurture_type = "getting_started"
```

**What changed from WF2:** Webhook (not polling), AI segmentation (not manual), CRM (not Sheets), no code duplication, instant response, multi-channel for high-value.

### WF-2B: Inbound Nurture Sequence (Daily)

**Trigger:** Schedule — Daily at 10:45 AM

```
Nurture cadence for inbound signups:
  Day 0: Welcome email (sent by WF-2A — immediate)
  Day 2: "Getting Started" guide + video tutorials
  Day 5: Case study relevant to their industry/segment
  Day 10: "How {similar_company} saved 60% with Utho"
  Day 15: Direct CTA — "Book a call with our cloud consultant"

Flow:
  Query CRM: leads with nurture_status active AND next_nurture_date <= today
  → Check lead_replied, compliance
  → Send appropriate nurture email
  → Update CRM: next_nurture_date, nurture_step += 1
  → After last step: nurture_status = "completed"
```

### WF-2C: Hot Lead Fast-Track

**Trigger:** Event — when inbound lead shows high engagement (deployed resource, high scoring)

```
Flow:
  Triggered by CRM event or scoring change
  → Skip remaining nurture steps
  → Route directly to Phase 2 sales pipeline
  → Slack alert: "🔥 Inbound lead fast-tracked to sales"
```

---

## 5. Phase 2: AI Lead Scoring

### WF-3A: Lead Scoring Engine

**Trigger:** Hybrid — Event-driven (on engagement) + Daily recalculation at 9:00 AM

```
SCORING MODEL:

  PROFILE SCORE (static, set on lead creation):
    Title = CTO/VP/Director/Head          +20 points
    Title = Manager/Lead                  +10 points
    Company size > 200 employees          +15 points
    Company size 50-200                   +10 points
    Industry in target list               +10 points
    Has company website                    +5 points

  ENGAGEMENT SCORE (dynamic, updated on events):
    Opened email                           +5 points
    Clicked link in email                 +15 points
    Clicked "Book Meeting" link           +25 points
    Replied (any intent)                  +20 points
    Replied with INTERESTED intent        +40 points
    Signed up on Utho platform            +30 points
    Deployed a cloud resource             +50 points

  DECAY (reduces score over time):
    No activity in 14 days                -10 points
    No activity in 30 days                -25 points
    Replied NOT_INTERESTED                -50 points
    Email bounced                         -30 points

  LEAD GRADES:
    80+ points  = 🔥 HOT  → Route to sales immediately
    50-79       = 🟠 WARM → Higher priority nurture
    20-49       = 🟡 COOL → Standard nurture
    <20         = 🔵 COLD → Long-term nurture / archive

  On grade change to HOT → Trigger WF-3B (Auto-Router)
```

---

## 6. Phase 2: Lead Routing & Assignment

### WF-3B: AI Auto-Router (Replaces Pooja's Manual Assignment)

**Trigger:** Event — Lead grade changes to HOT

```
Flow:
  Lead qualifies as HOT
    ↓
  Routing logic:
    Industry specialization → match to rep with domain expertise
    Company size > 500 → assign to senior reps
    Default → round-robin among available reps
    ↓
  Capacity check:
    Rep's assigned_leads_today < max_capacity (5/day)?
    If all at capacity → Slack alert to manager
    ↓
  Update CRM:
    assigned_to = selected_rep
    assigned_at = now
    sla_deadline = now + 2 hours
    assignment_status = "assigned"
    ↓
  Trigger WF-3C: Sales Rep Notifier
```

### WF-3C: Sales Rep Notifier (Rebuilds WF3)

**Trigger:** Called by WF-3B on assignment

```
Flow:
  Lookup rep's Slack ID:
    Slack API → users.lookupByEmail (DYNAMIC — no hardcoding)
    ↓
  Generate AI Sales Brief (via OpenAI):
    "Lead: {name} | Company: {org} ({industry}, {size} employees)
     Score: {score}/100 (HOT 🔥) | Source: {lead_source}
     Key signals: {engagement_summary}
     Reply text: '{reply_text}' (if applicable)
     Recommended talking point: {AI_suggestion}
     ⏰ SLA: Act by {sla_deadline}
     📞 {phone} | 📧 {email}"
    ↓
  Slack DM to assigned rep
  Update CRM: notification_sent_at = now
```

### WF-3D: SLA Enforcer — NEW (Makes "2 Hour" Real)

**Trigger:** Schedule — Every 30 minutes

```
Flow:
  Query CRM:
    assignment_status = "assigned" AND sla_deadline < now AND rep_responded = false
    ↓
  For each overdue lead:
    First breach (15 min over):
      → Slack DM to rep: "⚠️ SLA breach! Lead {email} is overdue."
      → CRM: sla_warning_sent = true
    Second check (30+ min over):
      → Slack DM to rep: "🚨 Lead being reassigned."
      → Slack to manager: "Lead reassigned from {rep} — SLA breach."
      → Re-run WF-3B with exclude_rep = current_rep
      → CRM: sla_breaches += 1 (tracks rep performance)
```

---

## 7. Phase 2: Sales Enablement

### WF-3E: Meeting Booking Handler

**Trigger:** Google Calendar webhook — "Meeting Booked"

```
Flow:
  Extract: guest_email, meeting_time
  → Lookup CRM by email
  → Update CRM: meeting_booked = true, pipeline_stage = "meeting_scheduled"
  → Slack to rep: "📅 Meeting booked! {name} — {date}"
  → Send confirmation email to lead
```

### WF-3F: Pipeline Status Updater (Daily EOD)

**Trigger:** Schedule — Daily at 6:00 PM

```
Flow:
  Query CRM for all active pipeline leads
  → Flag stale leads (no activity in 7 days)
  → Generate daily summary → Slack #sales-pipeline
  → Stats: leads assigned today, meetings booked, SLA breaches
```

---

## 8. CRM Schema Requirements

### Key Zoho CRM Fields Needed

```
LEAD/CONTACT FIELDS:
  — Standard: email, first_name, last_name, phone, title
  — Organization: organization_name, industry, company_size, website
  — Location: city, state, country
  — Source: lead_type (cold/inbound), lead_source, lead_market

AUTOMATION FIELDS:
  — cadence_status: not_started → mail_1_sent → followup_1_sent →
                     whatsapp_sent → cadence_completed | replied_* | bounced
  — mail_1_sent_at, mail_1_message_id
  — followup_1_sent_at, whatsapp_sent_at
  — next_followup_date, next_followup_type
  — lead_replied (boolean), reply_intent, reply_text
  — opt_out_email (boolean), opt_out_whatsapp (boolean)

ENGAGEMENT FIELDS:
  — email_opens (count), email_clicks (count)
  — last_opened_at, last_clicked_at
  — whatsapp_sent (boolean)

SCORING FIELDS:
  — lead_score (number), lead_grade (HOT/WARM/COOL/COLD)
  — profile_score, engagement_score

ASSIGNMENT FIELDS:
  — assigned_to, assigned_at, sla_deadline
  — assignment_status, rep_responded (boolean)
  — sla_breaches (count), sla_warning_sent

PIPELINE FIELDS:
  — pipeline_stage, meeting_booked, meeting_date
  — nurture_status, nurture_step, next_nurture_date
  — automation_status, automation_error
```

---

## 9. Build Order & Timeline

### Recommended Build Sequence

```
WEEK 1: Foundation
  ├── Day 1-2: Set up Zoho CRM schema (all fields from Section 8)
  ├── Day 2-3: Build WF-0A (Error Handler) + WF-0B (Compliance Check)
  ├── Day 3-4: Build WF-0C (AI Email Gen) + WF-0D (AI Intent Classification)
  └── Day 4-5: Set up Brevo webhooks + credential management

WEEK 2: Phase 1 — Cold Outreach
  ├── Day 6-7: Build WF-1A (Mail 1) — the core cold email workflow
  ├── Day 7-8: Build WF-1B (Follow-up 1) + WF-1C (Follow-up 2)
  ├── Day 8-9: Build WF-1E (Reply Detection) — critical!
  └── Day 9-10: Build WF-1F (Bounce/Engagement Handler)

WEEK 3: Phase 1 — Inbound + WhatsApp
  ├── Day 11-12: Build WF-2A (Signup Event Handler)
  ├── Day 12-13: Build WF-2B (Inbound Nurture Sequence)
  ├── Day 13-14: Build WF-1D (WhatsApp Touchpoint) — requires WATI setup
  └── Day 14-15: Testing all Phase 1 workflows end-to-end

WEEK 4: Phase 2 — Scoring & Routing
  ├── Day 16-17: Build WF-3A (Lead Scoring Engine)
  ├── Day 17-18: Build WF-3B (AI Auto-Router) + WF-3C (Sales Rep Notifier)
  ├── Day 18-19: Build WF-3D (SLA Enforcer)
  ├── Day 19-20: Build WF-3E (Meeting Handler) + WF-3F (Pipeline Updater)
  └── Day 20-21: End-to-end testing + migration from old workflows

WEEK 5: Migration & Go-Live
  ├── Day 22-23: Migrate existing leads from Google Sheets to Zoho CRM
  ├── Day 23-24: Run new + old workflows in parallel (shadow mode)
  ├── Day 24-25: Disable old workflows, go live with new system
  └── Day 25-30: Monitor, debug, optimize
```

---

## 10. Complete System View

```
┌──────────────────────────────────────────────────────────────────────────┐
│                UTHO AI AUTOMATION — COMPLETE SYSTEM VIEW                 │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SOURCES           PHASE 1                      PHASE 2                  │
│  ═══════           ═══════                      ═══════                  │
│                                                                          │
│  Enriched    ─→ WF-1A: Mail 1 ──┐                                       │
│  Cold Data      WF-1B: FU 1     │                                       │
│                 WF-1C: FU 2     ├→ WF-1E: Reply ─→ WF-3A: Score         │
│                 WF-1D: WhatsApp │   Detection       → WF-3B: Route      │
│                                 │                   → WF-3C: Notify      │
│                 WF-1F: Bounce/  │                   → WF-3D: SLA         │
│                 Engagement ─────┘                   → WF-3E: Meeting     │
│                                                     → WF-3F: Pipeline    │
│  Platform   ─→ WF-2A: Signup                                            │
│  Signups       Handler ─→ WF-2B: Nurture ─→ WF-3A (same scoring)       │
│                           WF-2C: Fast-Track                              │
│                                                                          │
│  ┌─── SHARED INFRASTRUCTURE ──────────────────────────────────────┐     │
│  │ WF-0A: Error Handler     WF-0C: AI Email Generation           │     │
│  │ WF-0B: Compliance Check  WF-0D: AI Intent Classification      │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  DATA: Zoho CRM (single source of truth)                                │
│  EMAIL: Brevo (credential managed)                                       │
│  WHATSAPP: WATI                                                          │
│  INTERNAL: Slack                                                         │
│  AI: OpenAI (gpt-4o-mini)                                               │
│                                                                          │
│  TOTAL: 16 workflows (4 shared + 6 Phase 1 + 6 Phase 2)                │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Pre-Requisites Before Building

Before the first workflow is built, these must be confirmed/set up:

| Item | Status | Action Required |
|------|--------|----------------|
| Zoho CRM access & API credentials | ❓ Confirm | Need admin access to create fields |
| Zoho CRM schema setup | ❓ To build | Create all fields from Section 8 |
| Brevo API key — REVOKE old, generate new | 🔴 URGENT | Revoke exposed key immediately |
| Brevo webhook configuration | ❓ To set up | Enable reply, bounce, open, click webhooks |
| WATI account & API access | ❓ Confirm | Need API credentials + approved templates |
| OpenAI API key | ❓ Confirm | Need organization account + API key |
| Slack app permissions | ✅ Exists | May need additional scopes for channel posting |

---

*This implementation plan is the complete blueprint. Each workflow is specified with triggers, data flows, CRM updates, error handling, and AI integration points. The build order ensures dependencies are met — shared infrastructure first, then Phase 1, then Phase 2.*
