# Account Engagement Reporter

**Recurring task:** producing an account-level sales and marketing engagement report ahead of a renewal call, QBR, or stalled-deal review.

An AI skill that turns raw CRM and marketing activity for an account into a short, sales-ready report. It sits on top of a sales and marketing activity taxonomy I built separately: about 90 rules that classify every logged touch into a consistent hierarchy. The output is either a one-screen summary or a full breakdown, depending on what the reader needs.

**My contribution:** I designed the taxonomy, wrote the classification rules, and designed, built, and corrected the report logic end to end.

**Status:** It's built as a reusable skill so sales reps and other analysts can run it themselves.

## Why I built it

Answering "what's been going on with this account?" meant opening the CRM and the marketing platform separately, scrolling through months of raw activity, and sorting out in your head what mattered. It was slow. It also depended on whoever did it already knowing the account's history, so two people doing the same pull would make different calls about what counted as real engagement.

The request kept coming back, for different accounts, on no fixed schedule. A reusable skill made the judgment consistent and put it in writing instead of in one person's head.

## What it does

**Used when:** someone asks, usually before a renewal call, a QBR, a deal-health check, or any "has anyone talked to this account lately?" question.

**Inputs:**

- The account's own identifier, so related or child entities aren't missed
- A time window, using either the calendar or the company's fiscal calendar
- Which activity categories matter: sales, marketing, pipeline and support, or all of them
- A condensed or detailed view

**Output:** a short list of the most important things that happened. Pipeline moves, new leads, and support tickets are counted separately so they don't distort the engagement numbers. Outreach is broken down by category and by one-to-one versus bulk or automated. The detailed view adds the five most engaged people and what they did.

**Scope:** it reports what happened. It doesn't predict what's next, and it doesn't decide what action to take. Anything that was never logged, like an unrecorded conversation, is outside what it can see.

## How it works

1. **Understand the request.** Ask explicitly for the account, time window, activity categories, and depth, because each answer changes what gets pulled.
2. **Apply the context.** Run every activity through the taxonomy: a real pipeline move or routine logging, one-to-one or automated, and which domains belong to the account.
3. **Do the work.** A script applies the rules, computes every count, and detects a short list of notable patterns. A model never guesses row by row.
4. **Check the result.** Flag any activity that matched zero rules or more than one. Either case means the rules need fixing before the numbers can be trusted.
5. **Write the output.** Match the requested depth, and label pattern-based call-outs as "worth a look" rather than fact.

## How I designed and refined it
Most of my time went into classifying the activities. Every count, call-out, and recommendation in the report depends on that classification, so I treated it as the foundation and built the report on top of it.

**Layers instead of one label per activity.** The business needed to answer several questions about the same touch at once: what kind of activity it was, which direction it came from, and whether a person or a system did it. One flat label couldn't carry all of that. I built the taxonomy as five stacked layers:

1. **Top-level category**: sales, marketing, or CRM record-keeping
2. **Direction**: inbound or outbound
3. **Channel**: calls, emails, meetings, events, web visits, content downloads, and so on
4. **Driver**: rep-driven or automated for sales, customer-driven or automated for marketing
5. Exact activity type

Each layer answers one question, so it can be read on its own or combined with the others.

**Two depths of output instead of one fixed report.** A rep prepping for a call in five minutes wants three sentences. Someone investigating a stalled deal wants the full trail. One format satisfied neither.

## How I checked its quality

- Every rule change was re-run against all real activity collected so far.
- A sample of computed counts was checked against original systems manually.
- The same real account was re-run across several rounds of changes to confirm the numbers stayed stable.

The pattern-based call-outs still need a human. A person decides whether a fast-moving deal is progress or a data artifact, and owns the judgment on any contact the report surfaces.

## Value

Generating this report went from manual pill to a deterministic processs taking ±1 minute. Now the classification runs in about two seconds. The bigger gain is reproducibility: the same input always produces the same classification.

**Current limitation:** the report is only as complete as what's logged in the company's systems. 

## Illustrative example

*Fictional example. No company data. Not actually run.*

**Request:** "Give me a quick read on Acme Robotics' engagement this month. High level is fine."

**Expected output:**

> **Acme Robotics, March 2026**
> 142 touches (18 sales, 124 marketing) · 1 support ticket · 1 pipeline move · 6 people engaged beyond automated email
>
> **Top action:** The Acme Robotics expansion opportunity moved to Negotiation on 3/14. A contact submitted a demo request on 3/22 with no open opportunity on file. Follow up the same day.

## What I learned

Get the classification right first. Every count, call-out, and recommendation in the report is built on how each activity is classified, so if that's wrong, everything downstream is wrong too. I spent a lot of time here: learning the business context and testing the rules against enough real examples to catch edge cases before anyone relied on the numbers.

Share it with stakeholders before rolling it out. Feedback from the sales team before a wider release changed what the report includes. They wanted to know what was happening outside our systems, like leadership changes, acquisitions, and news coverage, pulled from the web. They also wanted to see exactly what content the account had consumed, so their conversations could be more personal.

## Keywords

`account engagement reporting` · `sales enablement analytics` · `event taxonomy design` · `deterministic AI workflows` · `Claude skills` · `MCP` · `CRM activity data` · `agentic report generation`

## Confidentiality

Company name, internal data fields, and customer account details have been removed or replaced with generic placeholders throughout. The illustrative example uses a fictional account and was not run against real data. The underlying taxonomy, scripts, and real report output remain the company's property and are not reproduced here.
