# 🔥 AI AUTOMATION MASTER PLAN — DEEP ANALYSIS
## Utho Platform Private Limited (India Only)
### Analyzed for: AI Automation Engineer (n8n)
### Date: February 10, 2026

---

## 📌 TABLE OF CONTENTS
1. [Company Context & Why This Plan Exists](#1-company-context)
2. [The Big Picture — What This Plan Really Is](#2-big-picture)
3. [Phase 1: Marketing & Lead Generation](#3-phase-1)
4. [Phase 2: Sales Automation](#4-phase-2)
5. [Phase 3: Account Management & Retention](#5-phase-3)
6. [Phase 4: Support & Sales Bot](#6-phase-4)
7. [Phase 5: Billing & Revenue Protection (Missing!)](#7-phase-5)
8. [Phase 6: Leadership Intelligence](#8-phase-6)
9. [The 90-Day Action Plan — Your Execution Timeline](#9-90-day-plan)
10. [Non-Negotiables — The Rules You Cannot Break](#10-non-negotiables)
11. [Success Metrics — How You'll Be Measured](#11-success-metrics)
12. [Your Role as n8n Engineer — What YOU Build](#12-your-role)

---

## 1. COMPANY CONTEXT — WHY THIS PLAN EXISTS <a id="1-company-context"></a>

### What is Utho?
- **Utho** is an Indian cloud infrastructure company (like AWS/DigitalOcean but India-focused)
- They sell: Virtual Machines, Kubernetes, Managed Databases, Object Storage, Block Storage, Load Balancers, VPC, Cloud Firewalls
- **USP**: 60% cheaper than competition, 2X faster, 24/7 human support, deploy in 30 seconds
- **22,000+ users** already on platform
- **India-first** company — pricing, support, language all optimized for Indian market

### Why does Utho need this Master Plan?
Right now, Utho likely has:
- Manual marketing processes (someone writing emails, sending newsletters by hand)
- Sales team doing repetitive follow-ups manually
- No automated customer onboarding journey
- Support team overwhelmed with tickets
- Leadership has no real-time dashboard of what's happening

**This Master Plan = Automate EVERYTHING from the moment someone hears about Utho → signs up → buys → stays → pays → and leadership sees it all.**

---

## 2. THE BIG PICTURE — WHAT THIS PLAN REALLY IS <a id="2-big-picture"></a>

### The Customer Lifecycle (What This Plan Automates End-to-End)

```
STRANGER → LEAD → MEETING → CUSTOMER → HAPPY CUSTOMER → ADVOCATE
   ↑          ↑        ↑          ↑             ↑            ↑
Phase 1    Phase 1  Phase 2    Phase 3       Phase 4      Phase 3
Marketing  Lead Gen  Sales     Onboarding    Support      Retention
```

### Think of it as a FACTORY:
| Stage | Input | Your Automation Does | Output |
|-------|-------|---------------------|--------|
| Phase 1 | Strangers on internet | Attract, educate, nurture | Qualified Leads |
| Phase 2 | Qualified Leads | Score, route, follow-up | Paying Customers |
| Phase 3 | Paying Customers | Onboard, monitor, retain | Happy Long-term Customers |
| Phase 4 | Support Questions | AI bot answers, escalates | Resolved Issues |
| Phase 5 | Invoices & Payments | Automate billing, prevent revenue loss | Collected Revenue |
| Phase 6 | All Data | AI summaries for leadership | Smart Decisions |

### Your Role in This Factory:
**YOU are the person who builds this entire factory using n8n workflows.** Each "automation" = one or more n8n workflows that you design, build, test, and deploy.

---

## 3. PHASE 1: MARKETING & LEAD GENERATION <a id="3-phase-1"></a>

### 🎯 Goal: Turn strangers into qualified leads who are ready to talk to sales

### Automation 1: Inbound Signup Welcome & Nurturing Sequence

**What is this?**
When someone signs up on utho.com (creates a free account), they should NOT just get a boring "Welcome to Utho" email. Instead, they get a **smart, automated sequence** of emails over days/weeks that educates them and pushes them toward becoming a paying customer.

**The sequence (in order):**

| Day | Email | Purpose | n8n Implementation |
|-----|-------|---------|-------------------|
| Day 0 | **Welcome Email** | "Hey! Welcome to Utho. Here's how to get started." | Trigger: Webhook from Utho signup → Send email via SMTP/SendGrid |
| Day 2 | **Use-case Education** | "Here are 3 ways companies like yours use Utho" (e.g., hosting websites, running databases, deploying apps) | n8n Delay node → Conditional: What did they sign up for? → Send relevant email |
| Day 5 | **Quick-win Deployment Guide** | "Deploy your first cloud server in 30 seconds — here's how" | Step-by-step guide email with CTA to actually deploy |
| Day 8 | **Social Proof (India-focused)** | "See how [Indian Company X] reduced costs by 60% with Utho" | Case study email — must be INDIAN companies, not generic |
| Day 12 | **Soft CTA to Sales** | "Want a personalized demo? Book a call with our team" | Soft push — not aggressive, just offering help |

**Why India-focused social proof matters:** Indian buyers trust OTHER Indian companies' testimonials. Showing that Tata, Reliance, or Indian startups use Utho is 10X more effective than showing US companies.

**n8n Workflow Architecture:**
```
[Webhook: New Signup] → [Get User Data from CRM] → [Add to Nurture Sequence]
                                                           ↓
                                              [Delay 0 days] → [Send Welcome Email]
                                              [Delay 2 days] → [Send Use-case Email]
                                              [Delay 5 days] → [Send Quick-win Email]
                                              [Delay 8 days] → [Send Social Proof Email]
                                              [Delay 12 days] → [Send Soft CTA Email]
```

---

### Automation 2: Cold Outreach – India (Customers)

**What is this?**
Proactively reaching out to companies in India who have NEVER heard of Utho, to generate new leads.

**Channels:** Email + WhatsApp (WhatsApp is HUGE in India for business communication)

**"Problem-led messaging"** means:
- DON'T say: "Utho is great, buy our cloud"
- DO say: "Are you tired of paying too much for AWS? Are you frustrated with no human support? Is your cloud too complex?"
- Lead with their PAIN, then position Utho as the solution

**"AI reply intent detection"** means:
When a prospect replies to your cold email, AI reads the reply and classifies it:
- 😊 **Positive**: "Yes, I'm interested" → Route to sales immediately
- 🤔 **Maybe**: "Send me more info" → Send detailed info + follow up in 2 days
- 😠 **Negative**: "Not interested" → Stop sequence, mark in CRM
- 🚫 **Unsubscribe**: "Remove me" → Immediately remove (compliance!)

**n8n Implementation:**
```
[Trigger: Scheduled/Manual] → [Fetch Leads from CSV/CRM] → [Personalize Email with AI (OpenAI)]
        ↓
[Send Email via SMTP] → [Wait for Reply] → [AI Classify Reply Intent (OpenAI)]
        ↓                                          ↓
[Send WhatsApp via API]              [Positive] → [Create CRM Deal + Notify Sales]
                                     [Maybe] → [Send More Info + Schedule Follow-up]
                                     [Negative] → [Mark as Lost + Stop Sequence]
```

---

### Automation 3: Cold Partner Outreach – India

**What is this?**
Utho wants **partners** — companies that will RESELL Utho's cloud services.

**Target partners:**
- **MSP** (Managed Service Providers) — companies that manage IT for other businesses
- **SI** (System Integrators) — companies that build IT solutions for enterprises
- **Agencies** — web development, digital marketing agencies that need hosting for clients

**"Partner economics explained dynamically"** means:
The email dynamically calculates how much money the partner can make by reselling Utho.
Example: "If you have 50 clients and each spends ₹10,000/month on cloud, you'll earn ₹1,50,000/month in partner commissions."

**"Auto demo booking"** means:
The email includes a Calendly/Cal.com link that auto-books a demo with the partnerships team.

---

### Automation 4: Lost Lead Recycling

**What is this?**
Leads who showed interest 60-90 days ago but never converted. They went cold.

**Strategy:**
- Don't send the SAME emails. Send a **fresh angle** — maybe a new product launch, a price drop, a case study they haven't seen
- "Hey, we noticed you checked out Utho 2 months ago. Since then, we've launched Kubernetes support and reduced pricing by 20%!"

**n8n Implementation:**
```
[Scheduled Trigger: Weekly] → [Query CRM: Leads inactive 60-90 days]
        ↓
[Filter: Not opted out] → [AI Generate Fresh Angle Email (OpenAI)]
        ↓
[Send Email] → [Track Opens/Clicks] → [If Engaged: Move back to active pipeline]
```

---

### Automation 5: Monthly Newsletter Automation

**What is this?**
A monthly newsletter, but NOT one-size-fits-all. **Segmented** by audience:

| Segment | Content Focus |
|---------|---------------|
| **Prospects** (haven't bought yet) | Product highlights, pricing comparisons, case studies |
| **Customers** (already paying) | Product updates, tips & tricks, optimization guides |
| **Partners** (resellers) | Partner program updates, new tools, commission updates |

**n8n Implementation:**
```
[Scheduled: 1st of every month] → [Fetch Segments from CRM]
        ↓
[Branch by Segment] → [AI Generate Segment-specific Content]
        ↓
[Prospect Newsletter] / [Customer Newsletter] / [Partner Newsletter]
        ↓
[Send via Email Service]
```

---

### Phase 1 Key Outputs:
- ✅ **Qualified leads** (people who are genuinely interested in Utho)
- ✅ **Meeting-ready prospects** (people who want to talk to sales)
- ✅ **Partner conversations** (MSPs/SIs/Agencies interested in reselling)

---

## 4. PHASE 2: SALES AUTOMATION <a id="4-phase-2"></a>

### 🎯 Goal: Convert qualified leads into paying customers, faster

### Automation 1: AI Lead Scoring & Routing

**What is Lead Scoring?**
Not all leads are equal. A CTO from a 500-person company is MORE valuable than a student experimenting. Lead scoring assigns a NUMBER to each lead based on:

| Factor | Low Score (1-3) | Medium Score (4-6) | High Score (7-10) |
|--------|----------------|--------------------|--------------------|
| **Company Size** | 1-10 employees | 11-100 employees | 100+ employees |
| **Use Case** | Personal project | Small business hosting | Enterprise infra |
| **Engagement** | Opened 1 email | Clicked multiple links | Booked a demo |

**Routing rules:**
- Score 1-3 → Keep in marketing nurture (not ready for sales)
- Score 4-6 → Assign to **Sales team** (standard deals)
- Score 7-8 → Assign to **Strategic team** (bigger deals, more attention)
- Score 9-10 → Assign to **Partner team** (if they're a potential reseller)

**n8n Implementation:**
```
[Trigger: New Lead / Lead Updated] → [Fetch Lead Data from CRM]
        ↓
[AI Score Lead (OpenAI/Custom Logic)] → [Calculate Score]
        ↓
[IF Score 1-3] → [Keep in Nurture]
[IF Score 4-6] → [Assign to Sales Rep + Notify on Slack]
[IF Score 7-8] → [Assign to Strategic Rep + Notify on Slack]
[IF Score 9-10] → [Assign to Partner Team + Notify on Slack]
        ↓
[Update CRM with Score + Assignment]
```

---

### Automation 2: Meeting Validation System

**What is this?**
When a meeting is booked with a prospect, automate EVERYTHING around that meeting:

- **Auto reminders**: Send reminder 24hr before, 1hr before (Email + WhatsApp)
- **Agenda generation**: AI reads the lead's profile and generates a personalized meeting agenda
  - "Based on your interest in Kubernetes hosting, here's what we'll cover: 1) Current infra challenges 2) Utho K8s demo 3) Pricing comparison"
- **No-show follow-up**: If they don't attend, automatically send: "We missed you! Want to reschedule?"

---

### Automation 3: Post-Meeting Follow-ups

**What is this?**
After a sales call, the sales rep currently has to manually write follow-up emails. This automates it:

- **AI-generated summary**: Sales rep feeds call notes into the system → AI generates a professional summary
- **Next steps email**: Automatically sent to the prospect: "Great chatting! Here's what we discussed and next steps..."
- **Proposal nudges**: If a proposal was sent but no response in 3 days → gentle nudge email

---

### Automation 4: Deal Stage-Based Follow-ups

**What is this?**
Deals get stuck. This automation detects WHERE they're stuck and sends appropriate follow-ups:

| Deal Stuck Because | Automated Action |
|-------------------|-----------------|
| **Budget issue** | Send ROI calculator, cost comparison with AWS/Azure |
| **Delayed decision** | Send urgency content, time-limited offer |
| **Ghosted** (no response) | Multi-channel nudge: Email → WhatsApp → LinkedIn |

---

### Phase 2 Key Outputs:
- ✅ **Higher meeting show-up rate** (reminders + agenda = people actually attend)
- ✅ **Faster deal movement** (no deals sitting idle for weeks)
- ✅ **Reduced manual follow-ups** (sales team focuses on selling, not admin)

---

## 5. PHASE 3: ACCOUNT MANAGEMENT & RETENTION <a id="5-phase-3"></a>

### 🎯 Goal: Keep customers happy, reduce churn, increase revenue per customer

### Automation 1: Customer Onboarding Journey

**What is this?**
After someone PAYS and becomes a customer, they need to be ONBOARDED properly. A bad onboarding = customer leaves in 30 days.

| Day | Action | Purpose |
|-----|--------|---------|
| Day 0 | **Welcome from Account Manager (AM)** | Personal email: "Hi, I'm [Name], your dedicated AM" |
| Day 1 | **Setup Checklist** | "Here are the 5 steps to get fully set up on Utho" |
| Day 3 | **Best Practices Email** | "Top 5 mistakes new cloud users make (and how to avoid them)" |
| Day 7 | **Check-in** | "How's everything going? Need any help?" |

---

### Automation 2: Usage-Based Triggers

**What is this?**
Monitor how customers USE Utho's platform and trigger automated actions:

| Usage Pattern | What It Means | Automated Action |
|--------------|---------------|-----------------|
| **Low usage** | They signed up but aren't using it | Send education: "Did you know you can deploy in 30 seconds?" |
| **Cost spike** | Their bill suddenly jumped | Send advisory: "We noticed a 3X increase. Here's how to optimize." |
| **High usage** | They're using a LOT | Send recommendation: "You're growing fast! Here's how to scale efficiently with Utho" |

**n8n Implementation:**
```
[Scheduled: Daily] → [Fetch Usage Data from Utho API]
        ↓
[Analyze Usage Patterns] → [Classify: Low / Normal / Cost Spike / High]
        ↓
[Low] → [Send Education Email]
[Cost Spike] → [Send Optimization Advisory + Alert AM]
[High] → [Send Scale Recommendation + Alert Sales for Upsell]
```

---

### Automation 3: Loyalty & Advocacy Automation

**What is this?**
Turn happy customers into advocates who bring MORE customers:

- **Anniversary emails**: "You've been with Utho for 1 year! 🎉 Here's a special thank you."
- **NPS-based referral asks**: If a customer gives NPS score 9-10 (promoter) → Ask for referral. If NPS 1-6 (detractor) → Trigger escalation to AM.
- **Beta & early access invites**: Give loyal customers access to new features first

**NPS = Net Promoter Score**: "On a scale of 0-10, how likely are you to recommend Utho?"

---

### Automation 4: Lost Customer Re-engagement

**What is this?**
A customer has CANCELLED or stopped using Utho. Try to win them back:

1. **Exit feedback**: "We're sorry to see you go. What could we have done better?" (Automated survey)
2. **Cooling period**: Wait 30-60 days. Don't spam them immediately after cancellation.
3. **Win-back campaigns**: "We've improved! Here's what's new since you left + special comeback offer"

---

### Phase 3 Key Outputs:
- ✅ **Lower churn** (fewer customers leaving)
- ✅ **Higher lifetime value** (customers stay longer, spend more)
- ✅ **More referrals** (happy customers bring friends)

---

## 6. PHASE 4: SUPPORT & SALES BOT <a id="6-phase-4"></a>

### 🎯 Goal: Instant AI-powered support that reduces human workload

### Automation 1: AI Sales + Support Bot

**What is this?**
An AI chatbot (probably on utho.com and/or WhatsApp) that handles:

| Query Type | Bot Response | n8n Role |
|-----------|-------------|----------|
| **Product queries** | "What's the difference between Cloud and VPS?" | RAG over Utho docs → AI generates answer |
| **Pricing explanations** | "How much does 4GB RAM server cost?" | Fetch from pricing API → Format response |
| **Docs & how-to** | "How do I deploy Kubernetes?" | RAG over documentation → Step-by-step answer |
| **Ticket creation** | "My server is down!" | Create ticket in helpdesk system (Freshdesk/Zendesk) |

**RAG = Retrieval Augmented Generation**: The AI searches Utho's documentation database first, then generates an answer using that context. This ensures accurate, Utho-specific answers.

---

### Automation 2: Smart Escalation Logic

**What is this?**
Not everything should stay with the bot. Smart rules for when to escalate to a human:

| Condition | Action |
|-----------|--------|
| **High-value customer** (spending > ₹50K/month) | Immediately route to human agent |
| **Repeated issues** (same problem 3+ times) | Move to priority queue |
| **Negative sentiment detected** | Route to senior support + alert AM |
| **Bot confidence < 70%** | Admit "I'm not sure" → Route to human |

---

### Automation 3: Post-Ticket Automation

**What is this?**
After a support ticket is resolved:
- **Resolution feedback**: "Was your issue resolved? Rate 1-5" → If low rating → Reopen ticket
- **SLA breach alerts**: If a ticket takes longer than the promised SLA (e.g., 4 hours for critical issues) → Alert support manager immediately

---

### Phase 4 Key Outputs:
- ✅ **Faster responses** (bot responds in seconds, not hours)
- ✅ **Reduced support load** (bot handles 60-70% of queries)
- ✅ **Better customer experience** (24/7, instant, accurate)

---

## 7. PHASE 5: BILLING & REVENUE PROTECTION <a id="7-phase-5"></a>

### ⚠️ IMPORTANT NOTE: Phase 5 is mentioned in the overview but NOT detailed in the plan!

This is either:
1. **Still being defined** by leadership
2. **Was accidentally omitted** from the document
3. **Will be assigned later**

**What it LIKELY includes (based on industry standard):**
- Automated invoice generation and delivery
- Payment reminder sequences (before and after due date)
- Failed payment recovery (retry + notify customer)
- Revenue leakage detection (services used but not billed)
- Dunning management (escalating communication for overdue payments)

**🚨 ACTION FOR YOU:** Ask your manager about Phase 5 details on Day 1.

---

## 8. PHASE 6: LEADERSHIP INTELLIGENCE <a id="8-phase-6"></a>

### 🎯 Goal: Give leadership real-time visibility into business health

### Automation 1: Weekly AI Business Summary

**What is this?**
Every Monday morning, leadership receives an AI-generated business summary:

```
📊 UTHO WEEKLY BUSINESS SUMMARY — Week of Feb 10, 2026

LEADS:
- New leads this week: 342 (↑ 15% vs last week)
- Top source: Cold outreach (45%), Organic signup (30%), Partners (25%)

MEETINGS:
- Meetings booked: 28
- Meeting show-up rate: 82% (↑ from 74%)
- No-shows: 5 (auto follow-ups sent)

CONVERSION:
- Deals closed: 8
- Total new revenue: ₹4,20,000/month
- Average deal size: ₹52,500/month

REVENUE RISK:
- 3 customers showing declining usage (total revenue at risk: ₹1,80,000/month)
- 2 overdue invoices > 30 days (₹95,000)
```

**n8n Implementation:**
```
[Scheduled: Every Monday 8 AM] → [Fetch Data from CRM, Billing, Support]
        ↓
[Aggregate & Calculate Metrics] → [AI Generate Summary (OpenAI)]
        ↓
[Send via Email to Leadership] + [Post on Slack #leadership channel]
```

---

### Automation 2: Insight Alerts

**What is this?**
Real-time alerts when something concerning happens:

| Alert Type | Trigger | Action |
|-----------|---------|--------|
| **Drop in funnel performance** | Meeting bookings dropped 30% this week | Alert: "Meeting bookings down 30%. Possible cause: Cold outreach emails hitting spam." |
| **Campaign fatigue** | Email open rates below 10% for 2 consecutive campaigns | Alert: "Audience fatigue detected. Recommend refreshing email templates." |
| **Sales bottlenecks** | 15+ deals stuck in same stage for >7 days | Alert: "Sales bottleneck in 'Proposal Sent' stage. 15 deals stalled." |

---

### Phase 6 Key Outputs:
- ✅ **Faster decisions** (no waiting for end-of-month reports)
- ✅ **Clear visibility** (leadership sees problems BEFORE they become crises)

---

## 9. THE 90-DAY ACTION PLAN — YOUR EXECUTION TIMELINE <a id="9-90-day-plan"></a>

### This is YOUR roadmap. This is what you'll be doing day by day.

### 📅 Days 1–30: Marketing & Lead Generation

**What you're building:**
| Week | Focus | Specific n8n Workflows |
|------|-------|----------------------|
| Week 1 | **Setup & Discovery** | Understand CRM, email tools, data sources. Set up n8n instance. |
| Week 2 | **Inbound Nurturing** | Build the 5-email welcome sequence workflow |
| Week 3 | **Cold Outreach** | Build cold email + WhatsApp workflow with AI intent detection |
| Week 4 | **CRM Alignment** | Ensure all leads flow into CRM correctly, test & refine |

**"CRM Alignment"** means:
- Every lead, every email, every interaction gets logged in the CRM
- No data lives only in n8n — CRM is the single source of truth
- You'll need to set up CRM integrations (likely HubSpot, Salesforce, or Zoho CRM)

---

### 📅 Days 31–60: Sales Automation

**What you're building:**
| Week | Focus | Specific n8n Workflows |
|------|-------|----------------------|
| Week 5 | **Lead Scoring** | Build AI scoring workflow + routing logic |
| Week 6 | **Meeting Automation** | Reminders, agenda generation, no-show follow-ups |
| Week 7 | **Post-Meeting** | AI summaries, next-steps emails |
| Week 8 | **Deal Follow-ups** | Stage-based follow-up sequences |

---

### 📅 Days 61–90: Customer Success + Bot

**What you're building:**
| Week | Focus | Specific n8n Workflows |
|------|-------|----------------------|
| Week 9 | **Customer Onboarding** | Build onboarding email sequence |
| Week 10 | **Usage Triggers** | Build usage monitoring + automated actions |
| Week 11 | **AI Bot** | Build sales + support bot with RAG |
| Week 12 | **Testing & Polish** | Test everything end-to-end, fix bugs, optimize |

---

## 10. NON-NEGOTIABLES — THE RULES YOU CANNOT BREAK <a id="10-non-negotiables"></a>

### 1. Single CRM Source of Truth
- **ALL data lives in the CRM**. Not in spreadsheets, not in your head, not in n8n.
- Every workflow must READ from and WRITE to the CRM.
- If someone asks "what happened with Lead X?" — the answer is ALWAYS in the CRM.

### 2. Human Override at Every Stage
- **NO automation sends something without the ability for a human to stop it.**
- Every workflow should have an "OFF switch" — the ability to pause, cancel, or override.
- Example: If AI sends a wrong email, a human should be able to stop the sequence immediately.
- **Think of it as:** Automation is the car, but humans always have the steering wheel AND the brakes.

### 3. Clear Opt-out & Compliance
- **Every email MUST have an unsubscribe link.** This is LEGAL (IT Act, TRAI regulations, GDPR if dealing with international).
- **WhatsApp messages must follow WhatsApp Business API rules** (template messages only for first contact).
- **Respect DND (Do Not Disturb) registry** for Indian phone numbers.
- **When someone says "stop" → STOP. Immediately. No exceptions.**

### 4. Simple Language (India-first)
- **Write emails in simple, clear English.** Not academic English, not American slang.
- Many prospects are from Tier-2/Tier-3 cities. Keep language accessible.
- Consider Hindi/regional language options for WhatsApp messages.
- Avoid jargon unless the audience is technical.

---

## 11. SUCCESS METRICS — HOW YOU'LL BE MEASURED <a id="11-success-metrics"></a>

| Metric | What It Means | How Your Automation Impacts It | Target Direction |
|--------|--------------|------------------------------|-----------------|
| **Lead → Meeting ratio** | Of 100 leads, how many book a meeting? | Better nurturing + scoring = more meetings | ↑ Increase |
| **Meeting → Conversion** | Of meetings held, how many become customers? | Better preparation + follow-ups = more conversions | ↑ Increase |
| **Churn %** | What % of customers leave per month? | Better onboarding + usage triggers = fewer leaving | ↓ Decrease |
| **Collection cycle time** | How many days from invoice → payment? | Automated reminders = faster payment | ↓ Decrease |
| **Support resolution time** | How long to resolve a ticket? | AI bot = instant for simple queries | ↓ Decrease |

---

## 12. YOUR ROLE AS N8N ENGINEER — WHAT YOU ACTUALLY BUILD <a id="12-your-role"></a>

### Tools You'll Likely Use:

| Tool | Purpose |
|------|---------|
| **n8n** | Core automation engine — ALL workflows run here |
| **CRM** (HubSpot/Salesforce/Zoho) | Customer data, deals, pipeline |
| **OpenAI API** | AI for email writing, scoring, summaries, intent detection |
| **Email Service** (SendGrid/Mailgun) | Sending transactional & marketing emails |
| **WhatsApp Business API** | WhatsApp messaging |
| **Slack** | Internal notifications to sales/support teams |
| **Calendar** (Calendly/Cal.com) | Meeting booking |
| **Helpdesk** (Freshdesk/Zendesk) | Support ticket management |
| **Utho API** | Usage data, billing data, account data |

### Total Estimated Workflows to Build: ~25-35 n8n workflows

### Workflow Naming Convention (Suggested):
```
[Phase]-[Type]-[Description]
Examples:
P1-MKTG-Inbound-Welcome-Sequence
P1-MKTG-Cold-Outreach-India
P2-SALES-Lead-Scoring
P2-SALES-Meeting-Reminders
P3-AM-Customer-Onboarding
P4-BOT-Sales-Support-Bot
P6-INTEL-Weekly-Summary
```

---

## ⚠️ OBSERVATIONS & GAPS IN THE PLAN

1. **Phase 5 (Billing) is missing** — mentioned in overview, no details provided
2. **No mention of specific CRM** — you need to ask which CRM Utho uses
3. **No mention of specific email tool** — ask what email service is available
4. **No WhatsApp Business API provider mentioned** — need to identify (Gupshup, Twilio, etc.)
5. **No mention of data sources for cold outreach** — where do lead lists come from?
6. **No mention of Utho's API documentation** — you'll need API access for usage data
7. **Phase 6 numbering skips Phase 5** — the document goes 1,2,3,4,6 in details

---

## 🎯 YOUR DAY 1 CHECKLIST

- [ ] Get access to n8n instance (or set one up)
- [ ] Get access to the CRM system
- [ ] Get API keys for email service, WhatsApp, OpenAI
- [ ] Get access to Utho's internal API documentation
- [ ] Meet with marketing team to understand current processes
- [ ] Meet with sales team to understand their pain points
- [ ] Ask about Phase 5 (Billing) details
- [ ] Set up your workflow naming convention
- [ ] Create your first workflow: Inbound Signup Welcome Email

---

*This analysis was prepared as a comprehensive reference document. Refer back to it throughout your 90-day execution.*
