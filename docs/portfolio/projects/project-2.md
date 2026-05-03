---
title: "Listing Description Generator: AI Content That Sellers Actually Keep"
description: LLM pipeline that writes complete listing descriptions from structured data, with locale-aware prompts and a closed-loop evaluation system measuring real seller adoption across 7 locales.
---

# Listing Description Generator: AI Content That Sellers Actually Keep

!!! tip "Marketplace Application"
Empty or thin listing descriptions are one of the biggest conversion killers on any marketplace. Sellers skip writing them because it takes time. This system generates complete, market-appropriate descriptions automatically from structured listing data — and crucially, it measures whether sellers actually keep the output or rewrite it, so you can continuously improve quality. Applicable to any marketplace where listing quality drives buyer engagement: automotive, property, e-commerce, rental, or B2B.

!!! abstract "Project Summary"
**Domain**: Online Marketplace / Content Automation
**Role**: ML Engineer
**Scope**: 7 locales, production API + evaluation dashboard

<div class="metric-grid" markdown>
<div class="metric-card">
<span class="metric-value">7</span>
<span class="metric-label">Locales Supported</span>
</div>
<div class="metric-card">
<span class="metric-value">90%</span>
<span class="metric-label">Token Cost Reduction (Caching)</span>
</div>
<div class="metric-card">
<span class="metric-value">2</span>
<span class="metric-label">Evaluation Layers</span>
</div>
<div class="metric-card">
<span class="metric-value">3</span>
<span class="metric-label">Scoring Methods</span>
</div>
</div>

## The Problem

Creating compelling listing descriptions is time-consuming and a major friction point for sellers on marketplace platforms. Many listings go live with minimal or no descriptions, reducing buyer engagement and overall listing quality. The platform needed a way to automatically generate high-quality, structured descriptions that sellers would actually keep and use.

**The goal**: Build an AI system that generates listing descriptions so good that sellers adopt them with minimal editing - and build the evaluation infrastructure to prove it.

## System Architecture

<style>
.arch-diagram{display:flex;flex-direction:column;gap:12px;font-family:'Segoe UI',system-ui,sans-serif;margin:1rem 0}
.arch-label{font-size:11px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;margin-bottom:4px;padding:0 4px}
.arch-lane{border-radius:10px;padding:14px 16px;display:flex;flex-direction:column;gap:8px}
.arch-lane-gen{background:#eff6ff;border:1.5px dashed #93c5fd}
.arch-lane-eval{background:#f0fdf4;border:1.5px dashed #86efac}
.arch-lane-infra{background:#fff7ed;border:1.5px dashed #fdba74}
.arch-lane-gen .arch-label{color:#1d4ed8}
.arch-lane-eval .arch-label{color:#15803d}
.arch-lane-infra .arch-label{color:#c2410c}
.arch-flow{display:flex;align-items:center;flex-wrap:wrap;gap:0}
.arch-box{border-radius:8px;padding:8px 12px;font-size:12px;line-height:1.35;text-align:center;min-width:110px;max-width:148px;font-weight:500;color:#1e293b;flex-shrink:0}
.arch-gen{background:#bfdbfe;border:2px solid #1d4ed8}
.arch-eval{background:#bbf7d0;border:2px solid #15803d}
.arch-infra{background:#fed7aa;border:2px solid #c2410c}
.arch-out{background:#dbeafe;border:2px solid #1d4ed8}
.arch-langfuse{background:#e9d5ff;border:2px solid #7c3aed;color:#4c1d95}
.arch-arrow{width:28px;flex-shrink:0;display:flex;align-items:center;justify-content:center;color:#64748b;font-size:16px;font-weight:bold}
.arch-langfuse-col{display:flex;flex-direction:column;align-items:center;flex-shrink:0}
.arch-v-arrow{font-size:16px;color:#7c3aed;line-height:1;text-align:center}
.arch-loop-note{font-size:10.5px;color:#64748b;font-style:italic;border-top:1.5px dashed #94a3b8;padding-top:6px;margin-top:4px;text-align:center}
</style>

<div class="arch-diagram">

  <div class="arch-lane arch-lane-gen">
    <div class="arch-label">⚙ Generation Pipeline</div>
    <div class="arch-flow">
      <div class="arch-box arch-gen">Listing Service<br><small>(GraphQL API)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-gen">Data Loader<br><small>&amp; Field Filter</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-langfuse-col">
        <div class="arch-box arch-gen">Prompt Builder<br><small>(YAML Config)</small></div>
        <div class="arch-v-arrow">↓</div>
        <div class="arch-box arch-langfuse">Langfuse<br><small>(Trace Logging)</small></div>
      </div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-gen">AWS Bedrock<br><small>invoke_model()</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-gen">Pydantic AI<br><small>Validated Schema</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-out">Generated<br>Description</div>
    </div>
    <div class="arch-loop-note">↩ AI output logged to Datadog → feeds Evaluation Pipeline</div>
  </div>

  <div class="arch-lane arch-lane-eval">
    <div class="arch-label">📊 Evaluation Pipeline</div>
    <div class="arch-flow">
      <div class="arch-box arch-eval">Datadog Logs<br><small>(gen events)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-eval">Log Fetcher<br><small>&amp; Matcher</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-eval">Cosine Similarity<br><small>(semantic score)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-eval">LLM Judge<br><small>(Claude Sonnet)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-eval">Adoption Score<br><small>(0–100)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-eval">Streamlit<br>Dashboard</div>
    </div>
  </div>

  <div class="arch-lane arch-lane-infra">
    <div class="arch-label">🏗 Infrastructure</div>
    <div class="arch-flow">
      <div class="arch-box arch-infra">Docker Image</div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-infra">AWS ECR<br><small>(Container Registry)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-infra">Route 53<br><small>(DNS / Publish)</small></div>
      <div class="arch-arrow">→</div>
      <div class="arch-box arch-infra">SSO<br><small>(Access Control)</small></div>
    </div>
  </div>

</div>

## My Approach

### Stakeholder Workshops & Locale-Aware Prompt Engineering

Before writing a single prompt, I ran **workshops with internal stakeholders** — marketplace operations, seller success, and locale market managers — to understand what "good" looked like in each market. These sessions surfaced requirements that no spec document would have captured: preferred vocabulary, tone expectations, and the vocabulary sellers in each country actually use when writing descriptions themselves.

This translated directly into prompt engineering decisions:

- **Locale-specific lexicon**: Each locale config encodes preferred phrasing and restricted words, so the model writes the way local sellers and buyers speak — not generic AI-English
- **Accent and dialect calibration**: For markets with regional language variants, the prompt steers toward the accepted standard rather than a literal translation
- **YAML-based hierarchical configuration**: System prompt defines role and output schema; locale configs layer on tone, field availability, and vocabulary rules; field configs exclude attributes not tracked in that market (e.g., accident history absent in some locales) to prevent hallucination

This architecture makes onboarding a new locale a configuration change, not a code change.

### AWS Bedrock with Claude Haiku 4.5 — Cost-Optimised at Scale

The generation API calls **Claude Haiku 4.5 via AWS Bedrock** with two key optimisations:

- **Adaptive retries** (5 attempts): the API retries with adjusted parameters on malformed or low-quality outputs before failing, improving reliability without manual intervention
- **System prompt caching**: the system prompt — the largest and most stable part of each request — is cached at the Bedrock layer, reducing the cost of cached tokens by **90%** at production call volumes

### Evaluation System with DeepEval — Three Scoring Methods

Generating descriptions is straightforward. **Proving they work in production** required a separate engineering effort: an evaluation pipeline built on **DeepEval**, pulling real logs from Datadog and scoring AI output against what sellers actually published.

The evaluation logs are extracted from Datadog (generation events + final seller descriptions), matched by listing ID, then scored three ways:

**Method 1 — Word overlap** (fast, structural):
Computes overlapping words, word overlap %, and AI words retained %. Simple but fast — gives an immediate signal on how much of the AI text survived seller editing.

**Method 2 — Cosine similarity** (semantic):
Catches cases where sellers paraphrase rather than copy verbatim — high meaning adoption that word overlap would miss.

**Method 3 — LLM as judge** (DeepEval):
Claude Sonnet evaluates each AI/seller description pair and returns a structured score: `adoption_score` (0–100), `adoption_category`, and reasoning. Categories: `adopted` | `partially_adopted` | `replaced` | `non_relevant`. This catches nuanced rewrites that neither lexical method detects.

### Interactive Evaluation Dashboard — Production Feedback Loop

Built a **Streamlit dashboard** as a feedback channel from the production environment back to the development team:

- Surfaces failing cases — descriptions where sellers replaced or discarded the AI output — so prompt issues can be investigated with real examples
- Per-locale and per-user-type (professional dealers vs. private sellers) breakdowns expose where the prompt underperforms
- Automated recommendation engine analyses patterns (what sellers keep, remove, add) and proposes targeted prompt changes
- Monitoring successful and failing cases alike is what makes it possible to measure the success metrics of an AI product — not just output quality in isolation, but actual adoption in the field

## Production System

**Generation API** (FastAPI):

- Accepts vehicle ID, fetches attributes from the listing service via GraphQL
- Applies locale-specific field filtering and YAML-driven prompt assembly
- Calls Claude Haiku 4.5 via AWS Bedrock with adaptive retry logic (5 attempts) and system prompt caching
- Returns description + token usage metrics

**Evaluation Pipeline** (DeepEval + Streamlit + Datadog):

- Fetches generation events and final seller descriptions from Datadog
- Scores pairs using word overlap, cosine similarity, and LLM-as-judge (DeepEval)
- Surfaces failing cases in the dashboard for prompt iteration
- Generates actionable recommendations for prompt tuning per locale

## Tech Stack

`Python` `FastAPI` `Streamlit` `Claude Haiku 4.5` `Claude Sonnet` `AWS Bedrock` `DeepEval` `Datadog` `Plotly` `Pydantic` `YAML Config` `Docker`

## Key Takeaways

- **Stakeholder workshops before prompts**: understanding locale-specific vocabulary and tone expectations shaped every prompt decision — no spec document would have surfaced this
- **Prompt engineering for locale accents and lexicon** is a distinct discipline from general prompt engineering — what reads as natural in one market sounds foreign in another
- **System prompt caching** turned AWS Bedrock costs from a scale concern into a non-issue — 90% reduction on cached token costs
- **Evaluation is harder than generation**: building the proof that AI content gets adopted required more engineering than generating the content itself
- **Three scoring methods** (word overlap, cosine similarity, LLM-as-judge) catch different adoption patterns — no single method is sufficient
- **A production→dev feedback loop** via the evaluation dashboard is what turns a one-time launch into a continuously improving product
