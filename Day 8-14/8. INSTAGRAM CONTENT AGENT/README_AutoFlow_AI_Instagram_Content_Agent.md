# AutoFlow AI — Autonomous Instagram Content Agent

An end-to-end, multi-agent Instagram content automation system built in **Zapier** for a fictional AI automation brand, **AutoFlow AI**.

This project goes beyond simple caption generation. The system acts as an autonomous social media strategist that analyzes recent content history, identifies content gaps, selects the best topic and audience, generates a complete Instagram post, sends it to a second AI critic for quality evaluation, automatically rewrites weak content up to two times, stores the final result in Zapier Tables, and sends it to a human reviewer.

> **Important:** The system does **not** automatically publish to Instagram. Human approval remains mandatory.

---

## Project Objective

The goal of this project was to build a portfolio-ready AI content system that demonstrates:

- Autonomous AI decision-making
- Zapier Agents
- Agent-to-agent communication
- Tool calling
- Retrieval from Zapier Tables
- Memory through historical content
- Strategic content selection
- Multi-agent orchestration
- AI Critic / AI-as-a-Judge
- Structured AI outputs
- Controlled rewrite loops
- Hallucination reduction
- Deterministic score-based routing
- Human-in-the-loop approval
- Zapier Tables
- Dynamic ID generation
- Production-oriented failure handling

---

# Fictional Brand

## Brand Name
**AutoFlow AI**

## Industry
AI Automation / Business Process Automation

## Brand Description
AutoFlow AI helps businesses, startups, freelancers, and aspiring automation professionals use AI agents and workflow automation to reduce repetitive work, improve processes, and build smarter digital systems.

## Brand Voice
- Practical
- Clear
- Modern
- Educational
- Professional
- Conversational
- Non-hype-driven
- Beginner-friendly
- Outcome-focused

## Main Products / Services
1. **AI Workflow Consulting**
2. **Custom AI Agent Development**
3. **AI Automation Academy**
4. **Automation Template Library**

## Target Audiences
- Small Business Owners
- Startup Founders
- Freelancers & Automation Consultants
- Marketing Teams
- Operations Managers
- Students & Aspiring AI Automation Professionals

## Content Goals
- Awareness
- Education
- Engagement
- Trust Building
- Lead Generation
- Product Promotion

---

# System Architecture

```text
Development / Final Trigger
        ↓
AutoFlow AI — Instagram Content Strategist
        ↓
Zapier Tables: Find Records
        ↓
Analyze Recent Content
        ↓
Evaluate Available Ideas
        ↓
Check Product Information
        ↓
Check Customer Personas
        ↓
Select Strategy
        ↓
Generate Instagram Content
        ↓
Call Agent → Instagram Content Critic
        ↓
Critic Score + Decision
        ↓
Paths by Zapier
        ├── Score >= 7
        │      ↓
        │   Save Final Content
        │      ↓
        │   Status = Awaiting Approval
        │      ↓
        │   Human Reviewer Email
        │
        └── Score < 7
               ↓
            Rewriter #1
               ↓
            Critic #2
               ↓
            Paths
               ├── Score >= 7
               │      ↓
               │   Save Revised Content
               │   Revision Count = 1
               │   Status = Awaiting Approval
               │
               └── Score < 7
                      ↓
                   Rewriter #2
                      ↓
                   Critic #3
                      ↓
                   Paths
                      ├── Score >= 7
                      │      ↓
                      │   Save Final Revised Content
                      │   Revision Count = 2
                      │   Status = Awaiting Approval
                      │
                      └── Score < 7
                             ↓
                          Save Final Version
                          Revision Count = 2
                          Status = Needs Human Review
                          ↓
                          Human Review Email
```

---

# Technology Stack

- **Zapier Agents**
- **Zapier Tables**
- **Zapier Paths**
- **Zapier Agents — Run Agent**
- **Zapier Agents — Call an Agent**
- **Zapier Tables — Find Records**
- **Zapier Tables — Create Record**
- **Zapier Tables — Increment or Decrement Value**
- **Code by Zapier**
- **Gmail**
- **Zapier Interfaces / development trigger where needed**
- **Schedule by Zapier** for final scheduled execution

---

# Zapier Tables

## 1. Content Ideas

Stores candidate content ideas for the Strategist.

### Fields
- Idea ID
- Topic
- Product
- Target Audience
- Content Type
- Goal
- Status
- Priority
- Last Used Date
- Times Used

### Status Values
- Available
- Selected
- Used
- Archived

### Example ID Format
```text
IDEA-001
IDEA-002
...
IDEA-010
IDEA-011
IDEA-012
```

---

## 2. Product Information

Trusted source of truth for product information.

### Fields
- Product ID
- Product Name
- Description
- Target Audience
- Key Benefits
- Problems Solved
- Features
- Pricing
- Approved Claims
- CTA Options
- Product URL
- Status

### Purpose
The AI is not allowed to invent:
- Prices
- Statistics
- Guarantees
- Features
- Customer results
- Product capabilities

If a claim is not supported by trusted product data, it must not be used.

---

## 3. Customer Personas

Stores audience context for more relevant content generation.

### Fields
- Persona ID
- Persona Name
- Role
- Industry
- Pain Points
- Goals
- Challenges
- Preferred Content
- Objections
- Messaging Style
- Relevant Products
- Status

### Example Personas
- Busy Small Business Owner
- Growth-Focused Startup Founder
- Aspiring AI Automation Freelancer
- Efficiency-Focused Operations Manager

---

## 4. Instagram Content Calendar

Stores both historical and newly generated Instagram content.

### Fields
- Content ID
- Date
- Topic
- Product
- Target Audience
- Content Type
- Goal
- Hook
- Caption
- CTA
- Hashtags
- Visual Concept
- Score
- Critic Feedback
- Revision Count
- Status
- Generated At
- Approved At
- Posted At
- Post URL
- Approval Notes

### Official Status Values
- Draft
- Needs Human Review
- Awaiting Approval
- Approved
- Rejected
- Published

### Status Meaning

| Status | Meaning |
|---|---|
| Draft | Optional manually-created or early-stage content |
| Needs Human Review | Failed quality threshold after maximum rewrites or required manual inspection |
| Awaiting Approval | Passed AI quality review and is waiting for human approval |
| Approved | Human reviewer approved the post |
| Rejected | Human reviewer rejected the post |
| Published | Post was manually published to Instagram |

---

## 5. System Counters

Utility table used for automatic Content ID generation.

### Fields
- Counter Name
- Current Number
- Prefix

### Example Record
```text
Counter Name: Instagram Content
Current Number: 12
Prefix: IG-2026-
```

---

# Automatic Content ID Generation

Hard-coded IDs were replaced with an automatic counter system.

## Process
1. Increment `Current Number` in **System Counters**
2. Pass the new number to **Code by Zapier**
3. Generate a padded ID
4. Save the generated ID in Instagram Content Calendar

### JavaScript

```javascript
const num = Number(inputData.number);

return {
  content_id: `IG-2026-${String(num).padStart(3, '0')}`
};
```

### Example Output
```text
IG-2026-013
IG-2026-014
IG-2026-015
```

---

# Main Agent

## Agent Name
**AutoFlow AI — Instagram Content Strategist**

## Responsibility
Decide what AutoFlow AI should post today.

This agent is not a simple caption generator.

It analyzes:
- Recent posts
- Content repetition
- Content type usage
- Product promotion frequency
- Audience frequency
- Content gaps
- Available content ideas
- Product facts
- Customer personas

Then it independently selects:
- Topic
- Product
- Target Audience
- Content Type
- Goal
- Hook Direction
- CTA Direction

Finally it generates:
- Hook
- Caption
- CTA
- Hashtags
- Visual Concept

---

# Dynamic Table Retrieval

Zapier Agents allowed only one reusable **Find Records** tool in this implementation.

Therefore:

**Zapier Tables → Find Records**

was configured with:

```text
Table ID → Let your agent select a value for this field
```

The Strategist dynamically chooses which table it needs:

- Content Ideas
- Instagram Content Calendar
- Product Information
- Customer Personas

This makes the tool reusable and more agentic.

---

# Content Diversity Logic

The Strategist analyzes recent content before generating a new post.

Examples of diversity rules:

- Avoid using the same content type repeatedly
- Avoid repeating the same topic too frequently
- Avoid repeatedly targeting the same audience
- Avoid promoting the same product too often
- Avoid making every post promotional
- Maintain variety while still supporting business goals

Example:

```text
Educational
Educational
Educational
```

The next post should normally prefer another suitable format such as:

- Case Study
- Storytelling
- Problem / Solution
- Myth vs Fact
- Industry Insight
- Engagement

The AI decides this strategically rather than using hard-coded Paths.

---

# Content Critic Agent

## Agent Name
**AutoFlow AI — Instagram Content Critic**

The Critic independently evaluates the generated content.

### Evaluation Dimensions
1. Hook Quality
2. Audience Relevance
3. Content Usefulness
4. CTA Quality
5. Goal Alignment
6. Originality
7. Repetition / Diversity
8. Claim Safety
9. Instagram Readability
10. Visual Concept Alignment

### Score Range
```text
0–10
```

### Decision
```text
APPROVE
REWRITE
```

### Deterministic Routing Rule
```text
overall_score >= 7 → APPROVE
overall_score < 7 → REWRITE
```

Paths by Zapier enforce this rule outside the AI model.

---

# Agent-to-Agent Communication

The Main Strategist uses:

**Call an Agent → AutoFlow AI — Instagram Content Critic**

The Strategist sends:
- Topic
- Product
- Audience
- Content Type
- Goal
- Hook
- Caption
- CTA
- Hashtags
- Visual Concept
- Recent content context
- Source Idea ID
- Data notes

The Critic returns:
- Individual scores
- Overall score
- Feedback
- Rewrite instructions
- Decision

The Strategist exposes those Critic values as structured Zap outputs.

---

# Rewriter Agent

## Agent Name
**AutoFlow AI — Instagram Content Rewriter**

The Rewriter is only used when the Critic score is below 7.

It receives:
- Original strategy
- Current content
- Critic score
- Critic feedback
- Rewrite instructions
- Revision count

It returns:
- Revised Hook
- Revised Caption
- Revised CTA
- Revised Hashtags
- Revised Visual Concept
- Revision Summary

The Rewriter does not randomly change the entire strategy.

It fixes only the weaknesses identified by the Critic.

---

# Controlled Rewrite Loop

To prevent infinite AI loops, the workflow allows a maximum of:

```text
2 rewrites
```

## Rewrite Flow

```text
Original Content
   ↓
Critic #1
   ↓
If < 7
   ↓
Rewrite #1
   ↓
Critic #2
   ↓
If < 7
   ↓
Rewrite #2
   ↓
Critic #3
```

After Critic #3:

```text
Score >= 7
→ Awaiting Approval

Score < 7
→ Needs Human Review
```

No third rewrite is allowed.

---

# Human Approval

Human approval remains mandatory.

## Passed AI Review
When AI score is >= 7:

```text
Status = Awaiting Approval
```

The reviewer receives an email containing:
- Content ID
- Topic
- Product
- Audience
- Content Type
- Goal
- Hook
- Caption
- CTA
- Hashtags
- Visual Concept
- Quality Score
- Critic Feedback
- Revision Count

The reviewer then manually changes the status to:
- Approved
- Rejected

If later posted manually:
```text
Status = Published
```

---

# Needs Human Review Flow

If content still scores below the threshold after 2 rewrites:

```text
Status = Needs Human Review
Revision Count = 2
```

The reviewer receives a dedicated email containing:
- Final Critic Score
- Critic Feedback
- Final revised content
- Reason manual review is required

No additional AI rewrite occurs.

---

# Output Structure

The Strategist returns structured fields such as:

```text
selected_topic
selected_product
target_audience
content_type
goal
hook
caption
cta
hashtags
visual_concept
source_idea_id
requires_human_review
critic_overall_score
critic_feedback
critic_rewrite_instructions
critic_decision
```

These are defined as **Agent Output Fields** in the Zapier Agents → Run Agent action, allowing later Zap steps to map them individually.

---

# Development vs Final Trigger

## Development
The agent was initially tested using:
```text
On Demand
```

For workflow execution through a Zap, the Strategist was configured with:
```text
Trigger via Zap
```

## Final Production Setup
Recommended final trigger:

```text
Schedule by Zapier
Every day at 9:00 AM
```

The system generates content for review but does not publish automatically.

---

# Testing

All major end-to-end tests were completed successfully.

## Test 1 — Original Post Passes Critic #1

### Input
```text
Generate today's Instagram content using the normal production rules.
This is End-to-End Test 1.
```

### Result
- Critic Score: `8`
- Revision Count: `0`
- Status: `Awaiting Approval`
- Content ID: `IG-2026-017`
- Human reviewer email sent
- No rewrite executed
- No auto-publishing

### Result
**PASS**

---

## Test 2 — Original Fails, Rewrite #1 Passes

To force the rewrite branch during testing:

```text
Temporary threshold:
Approve >= 9
Rewrite < 9
```

Critic #2 remained at the normal production threshold.

### Result
- Rewrite #1 executed
- Critic #2 Score: `7.8`
- Revision Count: `1`
- Status: `Awaiting Approval`
- Content ID: `IG-2026-018`
- Rewriter #1 content was saved
- Human reviewer email sent
- Rewrite #2 did not execute

### Result
**PASS**

---

## Test 3 — Rewrite #2 Required

Temporary thresholds:

```text
Critic #1:
Approve >= 9
Rewrite < 9

Critic #2:
Approve >= 9
Rewrite #2 < 9

Critic #3:
Approve >= 7
Human Review < 7
```

### Result
- Rewrite #1 executed
- Rewrite #2 executed
- Critic #3 Score: `8`
- Revision Count: `2`
- Status: `Awaiting Approval`
- Content ID: `IG-2026-019`
- Rewriter #2 content saved
- Human reviewer email sent
- No additional rewrite occurred

### Result
**PASS**

---

## Test 4 — Maximum Rewrites → Human Review

For branch testing, all three thresholds were temporarily raised to 9.

```text
Critic #1:
Approve >= 9
Rewrite < 9

Critic #2:
Approve >= 9
Rewrite #2 < 9

Critic #3:
Approve >= 9
Human Review < 9
```

### Result
- Rewrite #1 executed
- Rewrite #2 executed
- Final Critic Score: `8`
- Because the temporary test threshold was `9`, final routing went to Human Review
- Revision Count: `2`
- Status: `Needs Human Review`
- Human review email sent
- No third rewrite occurred

### Result
**PASS**

---

# Final Production Thresholds

After testing, all temporary thresholds must be restored to:

```text
Critic #1:
Approve >= 7
Rewrite < 7

Critic #2:
Approve >= 7
Rewrite #2 < 7

Critic #3:
Final Approved >= 7
Needs Human Review < 7
```

---

# Test Summary

| Test | Scenario | Expected Final Status | Revision Count | Result |
|---|---|---|---:|---|
| Test 1 | Original content passes | Awaiting Approval | 0 | PASS |
| Test 2 | Rewrite #1 passes | Awaiting Approval | 1 | PASS |
| Test 3 | Rewrite #2 passes | Awaiting Approval | 2 | PASS |
| Test 4 | Still fails after max rewrites | Needs Human Review | 2 | PASS |

**All end-to-end routing tests passed successfully.**

---

# Important Safety Rules

The workflow explicitly prevents:

- Unsupported product claims
- Fake statistics
- Guaranteed outcomes
- Fake customer results
- Invented pricing
- Excessive marketing hype
- Automatic Instagram publishing
- Unlimited rewrite loops
- Automatic approval without human review

---

# Key Design Decisions

## Why use multiple agents?
Each agent has one clear responsibility:

- Strategist → Decide and generate
- Critic → Evaluate
- Rewriter → Improve

This separation improves maintainability and makes the architecture easier to explain and debug.

## Why use deterministic Paths?
AI evaluates quality, but Zapier controls routing.

This prevents the model from being the only decision-maker for workflow execution.

## Why maximum 2 rewrites?
This prevents infinite loops and ensures weak content reaches a human instead of consuming unlimited AI runs.

## Why keep human approval mandatory?
Even high-scoring AI-generated social content should be reviewed before public publishing.

---

# Final Workflow Capabilities

The completed system can:

- Analyze recent Instagram history
- Detect repetitive content patterns
- Identify underused audiences
- Identify underused content types
- Select a strategic content idea
- Verify product information
- Adapt messaging to customer personas
- Generate complete Instagram content
- Generate a visual concept
- Evaluate content with a separate Critic Agent
- Score content from 0 to 10
- Automatically rewrite weak content
- Stop after 2 rewrites
- Generate unique Content IDs
- Save the correct content version
- Maintain Revision Count
- Route low-quality content to human review
- Send approval emails
- Keep all public publishing manual

---

# Portfolio Value

This project demonstrates practical experience with:

- AI Agent architecture
- Autonomous AI decision-making
- Multi-agent orchestration
- Zapier Agents
- Structured outputs
- Dynamic tool usage
- Zapier Tables
- AI content strategy
- AI-as-a-Judge
- Controlled AI loops
- Hallucination protection
- Deterministic workflow routing
- Human-in-the-loop design
- Production safety considerations
- End-to-end testing

---

# Future Improvements

Possible future upgrades:

- Schedule the Strategist to run automatically every morning
- Add Slack or Zapier Human in the Loop approval
- Update Content Idea status automatically after selection
- Increment Times Used and Last Used Date automatically
- Add Instagram performance metrics
- Feed real performance data back into the Strategist
- Add competitor research
- Add controlled web research
- Add trend analysis
- Add approved-post publishing workflow after explicit human approval
- Add analytics dashboard
- Add automated reporting for content diversity and performance

---

# Final Status

**Project Status:** Completed and Tested

**All major workflow branches:** PASS

**Human approval:** Enabled

**Automatic Instagram publishing:** Disabled

**Maximum rewrites:** 2

**AI quality threshold:** 7 / 10

**Final architecture:** Production-oriented multi-agent content system built entirely in Zapier.
