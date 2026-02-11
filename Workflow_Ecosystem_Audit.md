# Utho AI Automation — Workflow Ecosystem Audit

**Document Version:** 1.0  
**Date:** 11 February 2026  
**Prepared By:** AI Automation Architect  
**Purpose:** Complete current-state analysis of all existing n8n workflows before Master Plan implementation  
**Scope:** 6 workflows analyzed across Cold Outreach and Inbound Signup journeys

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Ecosystem Visual Map](#2-ecosystem-visual-map)
3. [Lead Journey Maps](#3-lead-journey-maps)
4. [Per-Workflow Analysis](#4-per-workflow-analysis)
5. [Dependencies & Integrations](#5-dependencies--integrations)
6. [People & Roles Involved](#6-people--roles-involved)
7. [Security Findings](#7-security-findings)
8. [Patterns Worth Keeping](#8-patterns-worth-keeping)
9. [Critical Issues Summary](#9-critical-issues-summary)
10. [Gaps vs. Master Plan](#10-gaps-vs-master-plan)
11. [Workflow Verdicts Summary](#11-workflow-verdicts-summary)
12. [Recommendations](#12-recommendations)

---

## 1. Executive Summary

Utho currently operates **6 n8n workflows** across **2 distinct lead journeys** — Cold Outreach and Inbound Signup processing. These workflows were built by a previous employee and are currently **active in production**.

### Key Findings at a Glance

| Finding | Severity |
|---------|----------|
| Brevo API key exposed in plain text across 3 workflows | 🔴 CRITICAL |
| Google Sheets used as primary data source instead of CRM (Zoho exists but unused) | 🔴 CRITICAL |
| Zero AI usage across all workflows (Master Plan requires AI at every stage) | 🔴 CRITICAL |
| No unsubscribe/opt-out in any email across all workflows | 🔴 COMPLIANCE RISK |
| No reply detection — follow-ups sent even after lead replies | 🔴 CRITICAL |
| 1 workflow targets wrong market (US instead of India) | 🔴 WRONG MARKET |
| Failed emails silently marked as "Sent" in multiple workflows | 🟠 HIGH |
| No WhatsApp integration (Master Plan requires Email + WhatsApp) | 🟠 HIGH |
| Hardcoded Slack IDs (team changes require code edits) | 🟠 HIGH |
| Massive code duplication in Workflow 2 | 🟠 HIGH |
| No bounce handling | 🟡 MEDIUM |
| Hindi comments in production code | 🟡 LOW |

### What Works

- India cold outreach cadence structure (3 emails, 8 days) is solid
- Email threading via In-Reply-To/References headers — smart
- Lead segmentation concept in inbound workflow
- Enriched data usage for personalization
- Rate limiting across workflows
- Slack error notifications (partial)

### Bottom Line

**1 workflow needs to be SCRAPPED entirely (WF1 — US Cold Outreach).**  
**5 workflows need SIGNIFICANT REBUILDING** — the concepts and patterns are salvageable, but the architecture, security, AI integration, CRM integration, compliance, and error handling are all fundamentally insufficient for the Master Plan.

---

## 2. Ecosystem Visual Map

### 2.1 Complete Workflow Inventory

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     UTHO n8n WORKFLOW ECOSYSTEM                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─── COLD OUTREACH ───────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │  WF1: Cold Email Outreach (US)              ❌ WRONG MARKET          │   │
│  │  └── Google Sheet: "US Cold OutReach"                                │   │
│  │      └── Standalone. No connections to other workflows.              │   │
│  │                                                                      │   │
│  │  WF4: Cold Enriched Data - India (Personalized)   ✅ INDIA           │   │
│  │  WF5: Cold Enriched Data - India (Follow up 1)    ✅ INDIA           │   │
│  │  WF6: Cold Enriched Data - India (Follow up 2)    ✅ INDIA           │   │
│  │  └── All 3 share: Google Sheet "Cold Enriched Data (India)"          │   │
│  │      └── Connected via Mail-1 / Mail-2 / Mail-3 columns             │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─── INBOUND SIGNUP ──────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │  WF2: Inbound Signup Lead Follow-up                                  │   │
│  │  └── Google Sheet: "Mailing Data (Signup Leads) - Inbound"           │   │
│  │      └── Outputs "Connected" leads to ──┐                            │   │
│  │                                          │                            │   │
│  │  WF3: Signup Leads Message by Slack  ◄───┘                           │   │
│  │  └── Google Sheet: "Signup Leads > Distribution"                     │   │
│  │      └── Reads leads assigned by Pooja manually                      │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─── NOT BUILT (Master Plan Requires) ────────────────────────────────┐   │
│  │                                                                      │   │
│  │  • AI Lead Scoring & Auto-Routing                                    │   │
│  │  • AI Reply Intent Detection                                         │   │
│  │  • WhatsApp Outreach (WATI integration)                              │   │
│  │  • AI-Powered Email Generation                                       │   │
│  │  • Meeting Booking Automation                                        │   │
│  │  • Zoho CRM Integration                                              │   │
│  │  • Nurturing Sequences                                               │   │
│  │  • Churn Detection & Retention                                       │   │
│  │  • Support Bot                                                       │   │
│  │  • Billing & Revenue Protection                                      │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow Between Workflows

```
                        ┌──────────────┐
                        │ GOOGLE SHEETS │  (Acting as pseudo-CRM)
                        └──────┬───────┘
                               │
              ┌────────────────┼─────────────────────┐
              │                │                      │
              ▼                ▼                      ▼
    ┌─────────────────┐ ┌──────────────┐   ┌──────────────────┐
    │ "US Cold        │ │ "Cold        │   │ "Mailing Data    │
    │  OutReach"      │ │  Enriched    │   │ (Signup Leads)   │
    │                 │ │  Data        │   │  - Inbound"      │
    │  Used by: WF1   │ │  (India)"   │   │                  │
    │  Status: New →  │ │              │   │  Used by: WF2    │
    │  Mail Sent      │ │  Used by:    │   │  Status: New →   │
    └─────────────────┘ │  WF4, WF5,   │   │  Mail Sent       │
                        │  WF6         │   └────────┬─────────┘
                        │              │            │
                        │  Columns:    │            │ "Connected" leads
                        │  lead_status │            │ get added to:
                        │  Mail-1      │            ▼
                        │  Mail-2      │   ┌──────────────────┐
                        │  Mail-3      │   │ "Signup Leads >  │
                        │  mailing     │   │  Distribution"   │
                        │  message_id  │   │                  │
                        │  added_to_   │   │  Used by: WF3    │
                        │  ZOHO        │   │  Status: New →   │
                        └──────────────┘   │  Processed       │
                                           └──────────────────┘

    ┌─────────────────────────────────────────────────────────┐
    │  ZOHO CRM — EXISTS but NOT integrated with any workflow │
    │  Evidence: "added_to_ZOHO" column in Cold Enriched Data │
    └─────────────────────────────────────────────────────────┘
```

### 2.3 External Services Map

```
    ┌──────────────────────────────────────────────────────┐
    │                   EXTERNAL SERVICES                   │
    ├──────────────────────────────────────────────────────┤
    │                                                       │
    │  📧 BREVO (SendInBlue)                               │
    │  ├── Used by: WF1, WF2, WF4, WF5, WF6               │
    │  ├── WF1, WF2: via n8n Brevo node (credential mgmt)  │
    │  ├── WF4, WF5, WF6: via raw HTTP Request             │
    │  │   └── ⚠️ API KEY EXPOSED IN PLAIN TEXT            │
    │  └── Senders:                                         │
    │      ├── lalitmohan@utho.io (Cold Outreach India)     │
    │      ├── bharatpundhir@utho.com (Inbound - Bharat)    │
    │      └── srishti@utho.com (Inbound - Srishti)         │
    │                                                       │
    │  💬 SLACK                                             │
    │  ├── Used by: WF2, WF3, WF4, WF6                     │
    │  ├── Error alerts → vishal.m                          │
    │  ├── Lead assignment alerts → pooja.k                 │
    │  └── Sales rep notifications → 12 reps (hardcoded)    │
    │                                                       │
    │  📊 GOOGLE SHEETS                                     │
    │  ├── Used by: ALL 6 WORKFLOWS                         │
    │  └── Acting as CRM/database (⚠️ Anti-pattern)        │
    │                                                       │
    │  📅 GOOGLE CALENDAR                                   │
    │  └── "Book your slot" links in cold outreach emails   │
    │                                                       │
    │  📱 WATI (WhatsApp)                                   │
    │  └── NOT USED in any workflow                         │
    │  └── Evidence: "Added to WATI" column in WF2 sheet    │
    │                                                       │
    │  🏢 ZOHO CRM                                          │
    │  └── NOT USED in any workflow                         │
    │  └── Evidence: "added_to_ZOHO" column in WF4 sheet    │
    │                                                       │
    └──────────────────────────────────────────────────────┘
```

---

## 3. Lead Journey Maps

### 3.1 Journey A: India Cold Outreach (WF4 → WF5 → WF6)

This is a **3-email, 8-day cold outreach cadence** for enriched Indian leads.

```
                    INDIA COLD OUTREACH JOURNEY
                    ═══════════════════════════

    Lead Source: "Cold Enriched Data (India)" Google Sheet
    (likely enriched from Apollo/Lusha/similar tool)

    ┌────────────────────────────────────────────────────────────┐
    │  DAY 0 — Mail-1 (WF4) — 10:02 AM Daily                   │
    │  ──────────────────────────────────────────────────────    │
    │  Filter: lead_status = "new" → Email not empty →          │
    │  Take first 50 leads → Batch of 5                         │
    │                                                            │
    │  EMAIL: Heavy personalization                              │
    │  • First name, Organization name, Industry, Job title     │
    │  • India-first emotional pitch                             │
    │  • Social proof: 22,000+ users, named Indian brands       │
    │  • CTA: Book discovery call / Reply to email               │
    │  • Sender: Lalit Mohan (Sr. Manager – Client Engagement)  │
    │                                                            │
    │  AFTER SEND:                                               │
    │  • Save Brevo messageId                                    │
    │  • Mail-1 = "Sent"                                         │
    │  • Mail-2 = today + 3 days (date for follow-up)           │
    │  • lead_status = "old"                                     │
    │                                                            │
    │  Wait 1 min between batches of 5                           │
    └───────────────────────┬────────────────────────────────────┘
                            │ 3 days later
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  DAY 3 — Follow-up 1 (WF5) — 10:10 AM Daily              │
    │  ──────────────────────────────────────────────────────    │
    │  Filter: Mail-1 = "Sent" →                                │
    │  Code: Mail-1 = "Sent" AND Mail-2 ≠ "Sent"               │
    │        AND Mail-3 ≠ "Sent" →                              │
    │  If: Mail-2 date = today? → Batch of 5                    │
    │                                                            │
    │  EMAIL: Gentle nudge (THREADED via In-Reply-To)           │
    │  • "Just checking in - did you see my previous email?"    │
    │  • Only uses first_name (lost org/industry/title)          │
    │  • CTA: Book slot / Reply                                  │
    │  • Sender: Lalit Mohan                                     │
    │                                                            │
    │  AFTER SEND:                                               │
    │  • Mail-2 = "Sent"                                         │
    │  • Mail-3 = today + 5 days                                 │
    │                                                            │
    │  Wait 1 min between batches of 5                           │
    └───────────────────────┬────────────────────────────────────┘
                            │ 5 days later
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  DAY 8 — Follow-up 2 (WF6) — 10:15 AM Daily              │
    │  ──────────────────────────────────────────────────────    │
    │  Filter: Mail-2 = "Sent" →                                │
    │  Code: Mail-2 = "Sent" AND Mail-3 ≠ "Sent" →             │
    │  If: Mail-3 date = today? → Batch of 5                    │
    │                                                            │
    │  EMAIL: Breakup email (THREADED via In-Reply-To)          │
    │  • "Before I close this loop..."                           │
    │  • "If now's not the right time, I'll reconnect later"    │
    │  • Only uses first_name                                    │
    │  • CTA: Book slot / Reply / or walk away                   │
    │  • Sender: Lalit Mohan                                     │
    │                                                            │
    │  AFTER SEND:                                               │
    │  • Mail-3 = "Sent"                                         │
    │  • mailing = "Completed"                                   │
    │                                                            │
    │  ⚠️ Lead journey ENDS here. No further action.            │
    └────────────────────────────────────────────────────────────┘
                            │
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  AFTER CADENCE — ??? NOTHING                               │
    │  ──────────────────────────────────────────────────────    │
    │  • No reply detection at any stage                         │
    │  • No bounce handling                                      │
    │  • No engagement tracking (opens/clicks)                   │
    │  • No AI scoring                                           │
    │  • No routing to sales                                     │
    │  • No nurturing for non-responsive leads                   │
    │  • Lead sits in sheet with mailing = "Completed" forever   │
    └────────────────────────────────────────────────────────────┘
```

### 3.2 Journey B: Inbound Signup Follow-up (WF2 → WF3)

This handles leads who **sign up on the Utho platform** and are called by the onboarding team.

```
                   INBOUND SIGNUP JOURNEY
                   ══════════════════════

    Lead Source: "Mailing Data (Signup Leads) - Inbound" Google Sheet
    (leads come from Utho platform signups)

    ┌────────────────────────────────────────────────────────────┐
    │  STEP 1: Onboarding Exec Calls Lead (MANUAL)              │
    │  ──────────────────────────────────────────────────────    │
    │  • Bharat Singh Pundhir or Srishti calls the lead         │
    │  • Records call outcome in "Segment" column:              │
    │    - COMP/IND_ACC (Company/Individual Account)             │
    │    - INV_NUM (Invoice Number — paying customer)            │
    │    - MAIL_ONLY (Send email only)                           │
    │    - PAID_ACC (Paid Account)                               │
    │    - Connected/Sales (Hot lead, transfer to sales)         │
    │  • Sets "First Touch OE" to Bharat or Srishti             │
    │  • Sets Status = "New"                                     │
    └───────────────────────┬────────────────────────────────────┘
                            │
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  STEP 2: Automated Follow-up Email (WF2) — Every 5 min   │
    │  ──────────────────────────────────────────────────────    │
    │  Filter: Status = "New" → Validate data →                 │
    │  Switch on "First Touch OE":                               │
    │    Owner A → Bharat's templates                            │
    │    Owner A1 → Srishti's templates                          │
    │  Then Switch on "Segment":                                 │
    │    5 segments → 5 different static email templates          │
    │                                                            │
    │  FOR SEGMENTS (COMP/IND_ACC, INV_NUM, MAIL_ONLY, PAID):  │
    │  • Send segment-specific email via Brevo                   │
    │  • Update Status = "Mail Sent"                             │
    │  • If send fails → Slack alert to vishal.m                 │
    │                                                            │
    │  FOR "Connected/Sales" SEGMENT:                            │
    │  • Add lead to "Signup Leads > Distribution" sheet         │
    │  • Slack DM to Pooja: "New lead! Please assign."          │
    │  • Update Status = "Mail Sent"                             │
    │                                                            │
    │  ⚠️ ARCHITECTURE: Entire workflow is DUPLICATED           │
    │  for Bharat and Srishti (copy-pasted switch + templates)   │
    └───────────────────────┬────────────────────────────────────┘
                            │ "Connected/Sales" leads only
                            │ go to Distribution sheet
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  STEP 3: Pooja Assigns Lead (MANUAL)                       │
    │  ──────────────────────────────────────────────────────    │
    │  • Pooja receives Slack DM                                 │
    │  • Pooja manually types sales rep name in                  │
    │    "Assigned To" column in Distribution sheet              │
    │  • No SLA, no automation, no AI                            │
    └───────────────────────┬────────────────────────────────────┘
                            │
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  STEP 4: Sales Rep Notification (WF3) — Every 1 min       │
    │  ──────────────────────────────────────────────────────    │
    │  Filter: Status = "New" AND "Assigned To" not empty →     │
    │  Loop one by one →                                         │
    │  Map name to hardcoded Slack ID →                          │
    │  Slack DM to sales rep:                                    │
    │    "Hey [Name], a new lead has been assigned to you        │
    │     on FlowAura. Act within 2 hours or lead will be       │
    │     reallocated."                                          │
    │  Update Status = "Processed"                               │
    │                                                            │
    │  ⚠️ "2 hour reallocation" is an EMPTY THREAT              │
    │  ⚠️ No enforcement workflow exists                         │
    │  ⚠️ "FlowAura" — unclear what this reference means        │
    └────────────────────────────────────────────────────────────┘
                            │
                            ▼
    ┌────────────────────────────────────────────────────────────┐
    │  AFTER ASSIGNMENT — ??? NOTHING AUTOMATED                  │
    │  ──────────────────────────────────────────────────────    │
    │  • Sales rep is expected to act manually                   │
    │  • No follow-up tracking                                   │
    │  • No meeting booking automation                           │
    │  • No conversion tracking                                  │
    │  • No SLA enforcement                                      │
    └────────────────────────────────────────────────────────────┘
```

### 3.3 Journey C: US Cold Outreach (WF1) — DEPRECATED

```
                   US COLD OUTREACH (WRONG MARKET)
                   ════════════════════════════════

    ┌────────────────────────────────────────────────────────────┐
    │  WF1: Cold Email Outreach (US)                             │
    │  ──────────────────────────────────────────────────────    │
    │  • STANDALONE — no connections to any other workflow       │
    │  • Targets US market (violates Master Plan: India only)   │
    │  • Single static email, no cadence                         │
    │  • Minimal personalization (first name only)               │
    │  • VERDICT: ❌ SCRAP ENTIRELY                              │
    └────────────────────────────────────────────────────────────┘
```

---

## 4. Per-Workflow Analysis

### WF1: Cold Email Outreach (US)

| Attribute | Detail |
|-----------|--------|
| **Name** | Cold Email Outreach (US) |
| **Status** | Active |
| **Trigger** | Schedule (interval not specified in JSON, likely periodic) |
| **Data Source** | Google Sheet: "US Cold OutReach" → "Automatic Mailing" |
| **Email Provider** | Brevo (via n8n node with credential management) |
| **Filter** | Status = "New" |
| **Batch Size** | Up to 100 leads, batch processing with 2-min wait |
| **Target Market** | ❌ USA (should be India) |
| **Personalization** | First name only |
| **Email Count** | 1 (single email, no cadence) |
| **Reply Detection** | None |
| **WhatsApp** | None |
| **AI Usage** | None |
| **Unsubscribe** | None |
| **Error Handling** | ❌ `onError: continueRegularOutput` — failed emails marked as "Mail Sent" |
| **Connections** | Standalone — no connections to other workflows |

**Critical Issues:**
1. Wrong market (US, not India)
2. Static, hardcoded email content
3. Failed emails silently marked as sent (leads lost)
4. No unsubscribe link
5. No AI, no cadence, no reply handling
6. Google Sheets instead of CRM

**Verdict:** 🆕 **SCRAP AND REBUILD FROM SCRATCH**

---

### WF2: Inbound Signup Lead Follow-up

| Attribute | Detail |
|-----------|--------|
| **Name** | Inbound Signup Lead Follow-up |
| **Status** | Active |
| **Trigger** | Schedule: Every 5 minutes |
| **Data Source** | Google Sheet: "Mailing Data (Signup Leads) - Inbound" |
| **Email Provider** | Brevo (via n8n node with credential management) |
| **Filter** | Status = "New" |
| **Target Market** | ✅ India |
| **Personalization** | First name only, segment-based template selection |
| **Email Count** | 1 per lead (not a nurturing sequence) |
| **Segmentation** | 5 segments: COMP/IND_ACC, INV_NUM, MAIL_ONLY, PAID_ACC, Connected/Sales |
| **Owner Routing** | Switch on "First Touch OE" (Bharat / Srishti) |
| **Reply Detection** | None |
| **WhatsApp** | None (but "Added to WATI" column exists in sheet) |
| **AI Usage** | None |
| **Unsubscribe** | None |
| **Error Handling** | Partial — Slack alerts to vishal.m on email failure |
| **Connections** | Outputs "Connected/Sales" leads to → WF3 (via Distribution sheet + Pooja DM) |

**Critical Issues:**
1. Runs every 5 minutes → 288 times/day → excessive Google Sheets API usage
2. **Massive code duplication** — entire Switch→Templates→Send structure is copy-pasted for Bharat and Srishti
3. Sends only 1 email — not the nurturing sequence Master Plan requires
4. All 10 email templates are static (no AI)
5. Race condition: if status update fails, lead gets re-processed → duplicate emails
6. Manual lead routing to Pooja via Slack DM
7. No unsubscribe link in any email
8. Missing attachments referenced in email body ("I've attached...")
9. "Book a Meeting" links point to same calendar for both owners
10. Inconsistent sender phone numbers

**Positives:**
- India-focused ✅
- Good segmentation concept (5 segments) ✅
- Slack error alerts ✅
- Data validation node ✅
- Retry on fail ✅
- Lead distribution awareness (handoff to sales) ✅

**Verdict:** 🔧 **NEEDS SIGNIFICANT REBUILD**

---

### WF3: Signup Leads Message by Slack

| Attribute | Detail |
|-----------|--------|
| **Name** | Signup Leads Message by Slack |
| **Status** | Active |
| **Trigger** | Schedule: Every 1 minute |
| **Data Source** | Google Sheet: "Signup Leads" → "Distribution" tab |
| **Filter** | Status = "New" |
| **Target Market** | ✅ India (internal workflow) |
| **Function** | Notify sales reps via Slack DM when leads are assigned |
| **Slack ID Resolution** | Hardcoded map of 12 sales rep names → Slack User IDs |
| **SLA Claim** | "Act within 2 hours or lead reallocated" — NOT ENFORCED |
| **Error Handling** | ❌ All error outputs go to nothing (silent failures) |
| **Connections** | Fed by → WF2 (via Distribution sheet + Pooja manual assignment) |

**Critical Issues:**
1. Runs every 1 minute → 1,440 times/day → most aggressive trigger
2. Hardcoded Slack ID map — requires code changes when team changes
3. "2-hour reallocation" is an empty threat — no enforcement exists
4. Slack message says "FlowAura" — unclear/unprofessional reference
5. All error outputs silently discarded
6. Google Sheets instead of CRM

**Positives:**
- Slack DM concept is correct ✅
- SLA awareness (even if not enforced) ✅
- Clean, simple flow ✅
- Status tracking prevents duplicate notifications ✅

**Sales Reps in Hardcoded Map:**
Heena, Pravesh, Sivesh, Ashish, Vikrant, Jeevan, Thota, Paul, Devendra, Siddharth, Anushree, Sunny

**Verdict:** 🔧 **NEEDS SIGNIFICANT CHANGES**

---

### WF4: Cold Enriched Data - India (Personalized) — Mail-1

| Attribute | Detail |
|-----------|--------|
| **Name** | Cold Enriched Data - India (Personalized) |
| **Folder** | India Campaign |
| **Status** | Active |
| **Trigger** | Schedule: Daily at 10:02 AM |
| **Data Source** | Google Sheet: "Cold Enriched Data (India)" → Sheet1 |
| **Email Provider** | Brevo via raw HTTP Request (⚠️ API key exposed) |
| **Filter** | lead_status = "new" |
| **Batch Size** | 50 leads max (node says "100"), batch of 5 |
| **Target Market** | ✅ India |
| **Personalization** | Rich — first_name, organization_name, organization_industry, title |
| **Sender** | Lalit Mohan (Sr. Manager – Client Engagement, lalitmohan@utho.io) |
| **Message Tracking** | Saves Brevo messageId |
| **Reply Detection** | None |
| **WhatsApp** | None |
| **AI Usage** | None |
| **Unsubscribe** | None |
| **Error Handling** | ❌ `onError: continueRegularOutput` on send; Partial — Slack alert on sheet update error |
| **Connections** | Sets Mail-2 date → triggers WF5 three days later |

**Enriched Data Columns Available:**
`id, first_name, last_name, title, email, phone, linkedin, city, state, country, organization_name, organization_website, organization_industry, organization_size, lead_status, created_at, message_id, Mail-1, Mail-2, Mail-3, mail_status, mailing, added_to_ZOHO`

**Critical Issues:**
1. 🔴 **Brevo API key exposed in plain text** in HTTP Request headers
2. Node name "Only 100 rows" but code takes 50 — mismatch
3. Double email validation (redundant If nodes)
4. Failed emails silently marked as sent
5. No unsubscribe
6. Hindi comments in production code

**Positives:**
- Best email content quality across all workflows ✅
- Rich personalization using enriched data ✅
- India-focused with emotional storytelling ✅
- Rate limiting (batch 5 + 1 min wait) ✅
- messageId tracking ✅
- Multi-mail cadence tracking columns ✅
- Business hours scheduling ✅

**Key Discovery:** `added_to_ZOHO` column confirms **Zoho CRM exists** at Utho.

**Verdict:** 🔧 **BEST FOUNDATION — NEEDS SECURITY FIX + UPGRADES**

---

### WF5: Cold Enriched Data - India (Follow up 1) — Mail-2

| Attribute | Detail |
|-----------|--------|
| **Name** | Cold Enriched Data - India (Follow up 1) |
| **Folder** | India Campaign |
| **Status** | Active |
| **Trigger** | Schedule: Daily at 10:10 AM |
| **Data Source** | Same sheet as WF4: "Cold Enriched Data (India)" |
| **Email Provider** | Brevo via raw HTTP Request (⚠️ same exposed API key) |
| **Filter** | Mail-1 = "Sent" → Code: Mail-2 ≠ "Sent" AND Mail-3 ≠ "Sent" → If: Mail-2 date = today |
| **Threading** | ✅ Uses In-Reply-To and References headers (same email thread) |
| **Personalization** | First name only (lost org/industry/title from Mail-1) |
| **Sender** | Lalit Mohan |
| **Timing** | 3 days after Mail-1 |
| **Error Handling** | ❌ `onError: continueRegularOutput` on sheet update |
| **Connections** | Fed by → WF4 (Mail-2 date column); Sets Mail-3 date → triggers WF6 |

**Email Content:** Gentle follow-up nudge — "Just checking in – did you see my previous email?"

**Critical Issues:**
1. 🔴 Same exposed API key
2. Follow-up loses all personalization (only first_name)
3. No reply detection — sends even if lead already replied to Mail-1
4. No bounce handling — sends to bounced addresses
5. Date comparison may be fragile (string quoting issues)
6. `onError: continueRegularOutput` — dangerous

**Positives:**
- Email threading via In-Reply-To ✅
- Date-based scheduling ✅
- Rate limiting ✅

**Verdict:** 🔧 **NEEDS CHANGES (Part of cadence rebuild)**

---

### WF6: Cold Enriched Data - India (Follow up 2) — Mail-3

| Attribute | Detail |
|-----------|--------|
| **Name** | Cold Enriched Data - India (Follow up 2) |
| **Folder** | India Campaign |
| **Status** | Active |
| **Trigger** | Schedule: Daily at 10:15 AM |
| **Data Source** | Same sheet as WF4/WF5: "Cold Enriched Data (India)" |
| **Email Provider** | Brevo via raw HTTP Request (⚠️ same exposed API key) |
| **Filter** | Mail-2 = "Sent" → Code: Mail-3 ≠ "Sent" → If: Mail-3 date = today |
| **Threading** | ✅ Uses In-Reply-To and References headers |
| **Personalization** | First name only |
| **Sender** | Lalit Mohan |
| **Timing** | 5 days after Follow-up 1 (8 days after Mail-1) |
| **Cadence Completion** | Sets `mailing = "Completed"` — marks end of sequence |
| **Error Handling** | Partial — Slack alert on sheet read error to vishal.m |
| **Connections** | Fed by → WF5 (Mail-3 date column); FINAL in cadence — no output |

**Email Content:** Breakup email — "Before I close this loop... if now's not the right time, I'll reconnect later."

**Critical Issues:**
1. 🔴 Same exposed API key
2. No reply detection
3. No bounce handling
4. After cadence completes → lead is ABANDONED (no nurturing, no scoring, nothing)
5. Same date comparison fragility

**Positives:**
- Email threading ✅
- Breakup email concept ✅
- Cadence completion tracking ✅
- Slack error alert on sheet read ✅

**Verdict:** 🔧 **NEEDS CHANGES (Part of cadence rebuild)**

---

## 5. Dependencies & Integrations

### 5.1 Currently Used

| Service | How It's Used | Used By | Authentication |
|---------|--------------|---------|----------------|
| **Google Sheets API** | Primary data store (pseudo-CRM) | All 6 workflows | OAuth2 credential in n8n |
| **Brevo (SendInBlue)** | Email sending | WF1, WF2, WF4, WF5, WF6 | WF1/WF2: n8n credential mgmt ✅; WF4/5/6: Plain text API key ❌ |
| **Slack API** | Notifications & alerts | WF2, WF3, WF4, WF6 | n8n credential mgmt ✅ |
| **Google Calendar** | Meeting booking links | WF4, WF5, WF6 | Static links (no API) |

### 5.2 Available But Not Integrated

| Service | Evidence of Existence | Status |
|---------|----------------------|--------|
| **Zoho CRM** | `added_to_ZOHO` column in Cold Enriched Data sheet | Exists, NOT integrated with any n8n workflow |
| **WATI (WhatsApp)** | `Added to WATI` column in Inbound Signup sheet | Exists, NOT integrated with any n8n workflow |

### 5.3 Required by Master Plan But Missing Entirely

| Service | Master Plan Requirement |
|---------|------------------------|
| **OpenAI / LLM API** | AI email generation, lead scoring, intent detection |
| **WhatsApp Business API** | Email + WhatsApp outreach cadence |
| **CRM (Zoho)** | Single source of truth (Non-Negotiable #1) |
| **Meeting Scheduling API** | Automated meeting booking |
| **Webhook/Event System** | Real-time triggers instead of polling |

---

## 6. People & Roles Involved

### 6.1 Email Senders

| Person | Role | Email | Used In |
|--------|------|-------|---------|
| **Lalit Mohan** | Sr. Manager – Client Engagement | lalitmohan@utho.io | WF4, WF5, WF6 (India Cold Outreach) |
| **Bharat Singh Pundhir** | Onboarding Executive | bharatpundhir@utho.com | WF2 (Inbound — Owner A) |
| **Srishti** | Onboarding Executive | srishti@utho.com | WF2 (Inbound — Owner A1) |

### 6.2 Internal Notification Recipients

| Person | Slack User ID | Role in Workflows |
|--------|--------------|-------------------|
| **vishal.m** | U09FR9SV42X | Receives error alerts and workflow status messages |
| **pooja.k** | U08LM3TB6ET | Receives lead assignment requests, manually assigns leads to sales reps |

### 6.3 Sales Team (Hardcoded in WF3)

| Name | Slack User ID |
|------|--------------|
| Heena | U0786CY5FTJ |
| Pravesh | U07LU4Z17FF |
| Sivesh | U09GDRQV2CW |
| Ashish | U09G9KH7UM9 |
| Vikrant | U08NWGYNM1P |
| Jeevan | U08NGQBRJ1K |
| Thota | U09G9JEPELT |
| Paul | U091EBD9H46 |
| Devendra | U091M8E26BU |
| Siddharth | U07SH1ER3EK |
| Anushree | U05Q6FFMRFT |
| Sunny | U056UG759SS |

**⚠️ Note:** This mapping is hardcoded in a JavaScript Code node. Any team change requires a developer to edit the workflow code.

---

## 7. Security Findings

### 🔴 CRITICAL: Brevo API Key Exposed in Plain Text

**Affected Workflows:** WF4, WF5, WF6 (all India Cold Outreach cadence workflows)

**Location:** HTTP Request node → Header Parameters → `api-key` value

**The exposed key:** `xkeysib-XXXX...XXXX-REDACTED` *(full key redacted from documentation for security — original found in WF4/WF5/WF6 HTTP Request nodes)*

**Risk Assessment:**
- Anyone who exports, backs up, or shares these workflows can see the full API key
- This key grants **full access** to Utho's Brevo account:
  - Send emails as any Utho sender
  - Read contact lists
  - Read email templates
  - View sending statistics
  - Potentially delete data
- If this key is committed to any git repository, it may already be in version control history

**Contrast:** WF1 and WF2 correctly use n8n's built-in credential management system (`sendInBlueApi` credential reference), which stores the key encrypted in n8n's database. WF4/5/6 bypassed this by using raw HTTP Request nodes.

**Immediate Actions Required:**
1. Revoke and regenerate the Brevo API key immediately
2. Migrate WF4/5/6 to use n8n's Brevo credential management
3. Audit if this key has been shared or committed to any repository
4. Review Brevo account for any unauthorized sending activity

### 🟠 HIGH: Hardcoded Slack User IDs

**Affected Workflow:** WF3

12 sales rep Slack IDs are hardcoded in JavaScript. Any team reshuffling requires code changes. Should use Slack's `users.lookupByEmail` API or maintain mapping in a config sheet.

---

## 8. Patterns Worth Keeping

These patterns from the current workflows are well-designed and should be preserved in the rebuilt system:

### 8.1 Email Threading (WF5, WF6) ⭐

```json
"headers": {
    "In-Reply-To": "{{ $json.msg_id }}",
    "References": "{{ $json.msg_id }}"
}
```

Using `In-Reply-To` and `References` headers to keep follow-ups in the same email thread. This significantly improves open rates and makes communication look natural. **Must keep.**

### 8.2 Rate Limiting (WF1, WF4, WF5, WF6)

Batch processing (5 items) with 1-2 minute waits between batches. Prevents hitting API rate limits and improves email deliverability. **Must keep.**

### 8.3 Lead Segmentation Concept (WF2)

Categorizing inbound leads into segments (COMP/IND_ACC, INV_NUM, MAIL_ONLY, PAID_ACC, Connected/Sales) and tailoring the response per segment. The concept is excellent — the implementation (static templates, code duplication) needs rebuilding. **Keep the concept, rebuild the implementation.**

### 8.4 Message ID Tracking (WF4)

Saving Brevo's `messageId` for each sent email enables future open/click/bounce tracking. **Must keep and actually use it.**

### 8.5 Date-Based Follow-up Scheduling (WF4 → WF5 → WF6)

Setting a specific date for the next follow-up (instead of counting days from a trigger) ensures proper spacing even if a workflow fails to run one day. **Smart approach, must keep.**

### 8.6 Staggered Trigger Times (WF4: 10:02, WF5: 10:10, WF6: 10:15)

Prevents race conditions on shared data. **Must keep.**

### 8.7 Progressive Email Strategy (WF4 → WF5 → WF6)

Value pitch → Gentle nudge → Breakup email. This is textbook cold outreach best practice. **Must keep.**

### 8.8 Enriched Data Usage (WF4)

Using organization_name, organization_industry, title, and other enriched fields for personalization. **Must keep and extend to follow-ups.**

### 8.9 Slack Error Notifications (WF2, WF4, WF6)

Alerting vishal.m on failures via Slack DM. **Must keep and make consistent across all workflows.**

### 8.10 Data Validation (WF2, WF4)

Checking for empty/invalid fields before processing. **Must keep and standardize.**

---

## 9. Critical Issues Summary

### 9.1 Issues Affecting ALL Workflows

| # | Issue | Severity | Affected |
|---|-------|----------|----------|
| 1 | Google Sheets as data source instead of CRM | 🔴 CRITICAL | ALL |
| 2 | No AI usage — all content is static/templated | 🔴 CRITICAL | ALL |
| 3 | No unsubscribe/opt-out link in any email | 🔴 COMPLIANCE | WF1,2,4,5,6 |
| 4 | No WhatsApp channel integration | 🟠 HIGH | ALL |

### 9.2 Issues by Workflow

| # | Issue | Severity | Workflow |
|---|-------|----------|----------|
| 5 | Brevo API key exposed in plain text | 🔴 CRITICAL | WF4, WF5, WF6 |
| 6 | Wrong target market (US not India) | 🔴 CRITICAL | WF1 |
| 7 | No reply detection — follow-ups sent after lead replies | 🔴 CRITICAL | WF4, WF5, WF6 |
| 8 | Failed emails silently marked as "Sent" | 🟠 HIGH | WF1, WF4, WF5 |
| 9 | Trigger runs every 1 minute (1,440x/day) | 🟠 HIGH | WF3 |
| 10 | Trigger runs every 5 minutes (288x/day) | 🟠 HIGH | WF2 |
| 11 | Massive code duplication (2x of entire structure) | 🟠 HIGH | WF2 |
| 12 | Hardcoded Slack ID map (12 reps) | 🟠 HIGH | WF3 |
| 13 | "2-hour reallocation" not enforced | 🟠 HIGH | WF3 |
| 14 | Manual lead routing (Pooja assigns manually) | 🟠 HIGH | WF2, WF3 |
| 15 | No bounce handling | 🟡 MEDIUM | WF4, WF5, WF6 |
| 16 | Follow-ups lose personalization (only first_name) | 🟡 MEDIUM | WF5, WF6 |
| 17 | Post-cadence leads abandoned (no nurture) | 🟡 MEDIUM | WF6 |
| 18 | Race condition on status update | 🟡 MEDIUM | WF2 |
| 19 | Date comparison string quoting fragility | 🟡 MEDIUM | WF5, WF6 |
| 20 | "FlowAura" reference in Slack message | 🟡 LOW | WF3 |
| 21 | Hindi comments in production code | 🟡 LOW | WF3, WF4 |
| 22 | Node name "100 rows" but code takes 50 | 🟡 LOW | WF4 |
| 23 | Missing email attachments referenced in body | 🟡 LOW | WF2 |

---

## 10. Gaps vs. Master Plan

### Phase 1: Marketing & Lead Generation (Days 1-30)

| Master Plan Requirement | Current State | Gap |
|------------------------|---------------|-----|
| AI-powered cold outreach (India) | Exists but NO AI — static templates | 🔴 Need AI email generation |
| Problem-led messaging per persona | Generic messaging, same for all leads | 🔴 Need persona-based AI content |
| Multi-channel: Email + WhatsApp | Email only | 🔴 Need WhatsApp integration (WATI) |
| Multi-step nurturing sequence | 3-email cadence exists (WF4→5→6) | 🟡 Need to extend with AI + WhatsApp |
| Reply intent detection | None | 🔴 Need AI intent classification |
| Inbound signup nurturing | Single email per segment (WF2) | 🔴 Need multi-step journey |
| Open/click engagement tracking | messageId saved but never used | 🟡 Need tracking integration |
| Unsubscribe/compliance | None | 🔴 Must add immediately |
| CRM as data source | Google Sheets (Zoho CRM unused) | 🔴 Must migrate to Zoho |

### Phase 2: Sales Automation (Days 31-60)

| Master Plan Requirement | Current State | Gap |
|------------------------|---------------|-----|
| AI lead scoring | None | 🔴 Must build from scratch |
| AI auto-routing to sales reps | Manual (Pooja assigns via Slack) | 🔴 Must build AI routing |
| Meeting booking automation | Static calendar links | 🟡 Need API integration |
| Sales rep notification | Exists (WF3 Slack DMs) | 🟡 Need to rebuild (dynamic IDs, SLA enforcement) |
| SLA enforcement (2 hours) | Claimed but not enforced | 🔴 Must build enforcement + reallocation |
| Meeting → Conversion tracking | None | 🔴 Must build from scratch |
| AI sales assistance | None | 🔴 Must build from scratch |

### Phases 3-6 (Not Yet Analyzed)

| Phase | Status |
|-------|--------|
| Phase 3: Account Management & Retention | No existing workflows |
| Phase 4: Support & Sales Bot | No existing workflows |
| Phase 5: Billing & Revenue Protection | No existing workflows (Phase details undefined in Master Plan) |
| Phase 6: Leadership Intelligence | No existing workflows |

---

## 11. Workflow Verdicts Summary

| Workflow | Name | Market | Verdict | Action |
|----------|------|--------|---------|--------|
| **WF1** | Cold Email Outreach (US) | ❌ US | 🆕 SCRAP | Rebuild from scratch for India (or use WF4's foundation) |
| **WF2** | Inbound Signup Lead Follow-up | ✅ India | 🔧 SIGNIFICANT REBUILD | Keep segmentation concept, rebuild architecture, add AI, add nurturing sequence |
| **WF3** | Signup Leads Message by Slack | ✅ India | 🔧 SIGNIFICANT CHANGES | Keep Slack notification concept, add dynamic IDs, SLA enforcement, AI routing |
| **WF4** | Cold Enriched Data - India (Personalized) | ✅ India | 🔧 BEST FOUNDATION | Fix security, add AI, add CRM, add WhatsApp, add reply detection |
| **WF5** | Cold Enriched Data - India (Follow up 1) | ✅ India | 🔧 PART OF CADENCE | Fix security, add personalization, add reply detection, add bounce handling |
| **WF6** | Cold Enriched Data - India (Follow up 2) | ✅ India | 🔧 PART OF CADENCE | Fix security, add post-cadence routing, add reply detection |

---

## 12. Recommendations

### Immediate Actions (Before Any Rebuild)

1. **🔴 SECURITY:** Revoke the exposed Brevo API key, generate a new one, store in n8n credentials
2. **🔴 COMPLIANCE:** Add unsubscribe links to ALL email templates across all workflows
3. **🟠 STABILITY:** Fix `onError: continueRegularOutput` on all send-email nodes — failed sends should NOT update status

### Architecture Decisions Needed

1. **CRM:** Confirm Zoho CRM as the single source of truth and plan migration from Google Sheets
2. **WhatsApp Provider:** Confirm WATI as the WhatsApp Business API provider
3. **AI Provider:** Confirm OpenAI/LLM provider for email generation and intent detection
4. **Event-Driven System:** Replace polling (every 1-5 min) with webhook/event-driven triggers where possible

### Rebuild Priority Order (Based on Master Plan Phases 1 & 2)

1. **Phase 1A:** Rebuild India Cold Outreach cadence (WF4→5→6) with AI, CRM, reply detection, WhatsApp
2. **Phase 1B:** Rebuild Inbound Signup journey (WF2) with AI nurturing sequences
3. **Phase 2A:** Build AI Lead Scoring & Auto-Routing (replace manual Pooja assignment)
4. **Phase 2B:** Rebuild Sales Rep Notification (WF3) with dynamic Slack lookup & SLA enforcement
5. **Phase 2C:** Build Meeting Booking & Conversion Tracking

---

*This document is the current-state analysis. The Implementation Plan (how to rebuild) will be a separate document created after confirming CRM, WhatsApp, and AI provider decisions.*
