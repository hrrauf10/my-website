---
title: "Pricing Intelligence: Detecting Misleading Listing Prices Across 5 Markets"
description: Fine-tuned transformer classifier that automatically flags conditional and misleading prices in marketplace listings — 20x faster and 5x cheaper than LLMs, across 5 markets and 6 languages.
---

# Pricing Intelligence: Detecting Misleading Listing Prices Across 5 Markets

!!! tip "Marketplace Application"
Online marketplaces routinely receive listings where the advertised price is misleading — tied to financing deals, leasing conditions, or VAT exclusions that most buyers don't qualify for. This system automatically detects those listings at scale, protecting buyer trust and flagging them for review before they go live. Applicable to any marketplace where price transparency matters: automotive, property, rental, B2B, or e-commerce.

!!! abstract "Project Summary"
**Domain**: Online Marketplace / Pricing Transparency
**Role**: ML Engineer (sole data scientist)
**Scope**: 5 markets, 6 languages, production API + batch pipeline

<div class="metric-grid" markdown>
<div class="metric-card">
<span class="metric-value">20x</span>
<span class="metric-label">Faster than Claude</span>
</div>
<div class="metric-card">
<span class="metric-value">&gt;2x</span>
<span class="metric-label">Cheaper at Scale</span>
</div>
<div class="metric-card">
<span class="metric-value">90–95%</span>
<span class="metric-label">Detection Rate</span>
</div>
<div class="metric-card">
<span class="metric-value">5</span>
<span class="metric-label">Markets, 6 Languages</span>
</div>
</div>

!!! success "Key Result: In-house model is 20x faster and >2x cheaper than Claude while maintaining comparable detection performance"

## The Problem

In online marketplaces, some listings advertise prices that come with conditions - financing requirements, leasing terms, special buyer restrictions, or VAT exclusions. These conditional prices are misleading for standard buyers who expect transparent pricing. Manually reviewing thousands of listings across multiple markets and languages was unsustainable.

**The goal**: Build an automated system that flags conditional pricing across 5 international markets with high accuracy and low latency.

## Model Architecture

<style>
.ma-wrap{font-family:'Segoe UI',system-ui,sans-serif;margin:1rem 0}
.ma-note{background:#fefce8;border:1.5px solid #fde68a;border-radius:8px;padding:10px 14px;font-size:12.5px;color:#713f12;margin-bottom:14px}
.ma-flow{display:flex;align-items:center;flex-wrap:wrap;gap:0}
.ma-box{border-radius:8px;padding:10px 14px;font-size:12px;line-height:1.4;text-align:center;font-weight:500;color:#1e293b;flex-shrink:0}
.ma-plain{background:#f1f5f9;border:2px solid #94a3b8;min-width:120px}
.ma-trim{background:#fce7f3;border:2px solid #db2777;min-width:130px}
.ma-model{background:#dbeafe;border:2px solid #1d4ed8;min-width:150px}
.ma-bin{background:#dcfce7;border:2px solid #16a34a;min-width:140px}
.ma-multi{background:#fef9c3;border:2px solid #ca8a04;min-width:140px}
.ma-arrow{width:28px;flex-shrink:0;display:flex;align-items:center;justify-content:center;color:#64748b;font-size:18px;font-weight:bold}
.ma-split{display:flex;flex-direction:column;gap:10px;flex-shrink:0}
.ma-why{background:#f0f9ff;border:1.5px solid #7dd3fc;border-radius:8px;padding:10px 14px;font-size:12px;margin-top:12px;color:#0c4a6e}
.ma-why strong{display:block;margin-bottom:4px;font-size:12.5px}
.ma-tag{display:inline-block;background:#fef08a;border-radius:4px;padding:1px 5px;font-size:11px;font-weight:700;color:#713f12;margin-top:2px}
</style>

<div class="ma-wrap">
  <div class="ma-note">🧠 Encoders learn contextual word meaning with a tiny memory footprint — enabling <strong>quick iteration and faster fine-tuning</strong> on a single 24 GB GPU</div>
  <div class="ma-flow">
    <div class="ma-box ma-plain">Listing<br>Description</div>
    <div class="ma-arrow">→</div>
    <div class="ma-box ma-trim">✂ Trimmer<br><small>Extracts sentences with<br><span class="ma-tag">pricing-relevant</span> keywords</small></div>
    <div class="ma-arrow">→</div>
    <div class="ma-split" style="gap:8px">
      <div class="ma-box ma-model">🤖 mDeBERTa-v3-base<br><small>Encoder · 12 Layers<br><span class="ma-tag">86M Parameters</span> <span class="ma-tag">Multilingual</span><br>Used for: DE</small></div>
      <div class="ma-box ma-model">🤖 DeBERTa-v3-large<br><small>Encoder · 24 Layers<br><span class="ma-tag">304M Parameters</span><br>Used for: IT, AT, CA, BE</small></div>
    </div>
    <div class="ma-arrow">→</div>
    <div class="ma-split">
      <div class="ma-box ma-bin">🔲 Binary Classifier<br><small><span class="ma-tag">Conditional</span><br>Non-Conditional</small></div>
      <div class="ma-box ma-multi">⊞ Multi Classifier<br><small><span class="ma-tag">Financing</span> · Incentives<br>Leasing · SpecialBuyers · Others</small></div>
    </div>
  </div>
  <div class="ma-why">
    <strong>💡 Why Two Heads?</strong>
    Fine-tuning both tasks simultaneously forces the encoder to learn richer shared representations — the binary signal sharpens category boundaries, and the category signal anchors binary decisions. Single-task models for each head performed worse individually than this joint approach.
  </div>
</div>

## My Approach

### Dual-Head Transformer Encoder Fine-Tuning

Rather than fine-tuning two separate models, I designed a **single dual-head classifier** that learns both tasks simultaneously in one forward pass:

- **Binary head**: Conditional vs. Non-Conditional — detects whether the advertised price is achievable by a standard buyer
- **Multi-label head**: 7 condition categories — Financing, Leasing, Incentives, Special Buyers, VAT Excluded, Other, OK

Joint fine-tuning improved accuracy on both tasks: the binary signal sharpens category boundaries, and the category signal anchors the binary decision. One model, two outputs, better performance than either standalone.

### Language-Aware Model Selection

Rather than forcing a single model across all markets, I selected architecturally appropriate models:

| Model                | Markets        | Rationale                                                                |
| -------------------- | -------------- | ------------------------------------------------------------------------ |
| **DeBERTa V3 Large** | IT, AT, CA, BE | Superior performance on English-centric and Romance language text        |
| **mDeBERTa V3 Base** | DE             | Better multilingual representations for German compound words and syntax |

### Feature Engineering

- **Country-specific keyword extraction**: 70–71 priority stems per market to extract pricing-relevant sentences before feeding into the model

### Hybrid Inference: Transformers + LLM Fallback

For predictions where the model confidence fell below 0.8, the system routes to **Claude via AWS Bedrock** for validation.

<style>
.inf-wrap{font-family:'Segoe UI',system-ui,sans-serif;margin:1rem 0}
.inf-row{display:flex;align-items:center;flex-wrap:wrap;gap:0;margin-bottom:6px}
.inf-box{border-radius:8px;padding:8px 14px;font-size:12px;font-weight:500;text-align:center;color:#1e293b;flex-shrink:0}
.inf-blue{background:#bfdbfe;border:2px solid #1d4ed8;min-width:130px}
.inf-green{background:#dcfce7;border:2px solid #16a34a;min-width:130px}
.inf-purple{background:#e9d5ff;border:2px solid #7c3aed;min-width:130px}
.inf-arrow{width:26px;flex-shrink:0;text-align:center;color:#64748b;font-size:16px;font-weight:bold}
.inf-branch{display:flex;flex-direction:column;gap:6px;flex-shrink:0}
.inf-label{font-size:11px;color:#64748b;padding:2px 6px;background:#f1f5f9;border-radius:4px;margin-right:4px;font-weight:600}
</style>

<div class="inf-wrap">
  <div class="inf-row">
    <div class="inf-box inf-blue">Listing</div>
    <div class="inf-arrow">→</div>
    <div class="inf-box inf-blue">Transformer<br>Inference</div>
    <div class="inf-arrow">→</div>
    <div class="inf-box inf-blue">Confidence<br>Check</div>
    <div class="inf-arrow">→</div>
    <div class="inf-branch">
      <div class="inf-row" style="margin:0"><span class="inf-label">≥ 0.8</span><div class="inf-box inf-green">Return Prediction<br><small>(sub-second)</small></div></div>
      <div class="inf-row" style="margin:0"><span class="inf-label">&lt; 0.8</span><div class="inf-box inf-purple">Claude Validation<br><small>(+2–5s)</small></div><div class="inf-arrow">→</div><div class="inf-box inf-green">Return Prediction</div></div>
    </div>
  </div>
</div>

This hybrid approach optimises for speed on high-confidence predictions while using LLM reasoning only for genuinely ambiguous edge cases.

### Automated Evaluation Pipeline

I built a stratified validation system using Claude as ground truth:

1. Sample 250 listings per country (stratified by predicted class)
2. Generate Claude labels with structured reasoning
3. Evaluate transformer predictions against Claude ground truth
4. Track per-country precision/recall over time

## Production System

### Deployment Pipeline

<style>
.dp-wrap{font-family:'Segoe UI',system-ui,sans-serif;margin:1rem 0}
.dp-lane{border-radius:10px;padding:14px 16px;margin-bottom:10px}
.dp-lane-ci{background:#f0fdf4;border:1.5px dashed #86efac}
.dp-lane-aws{background:#fff7ed;border:1.5px dashed #fdba74}
.dp-label{font-size:11px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;margin-bottom:8px}
.dp-label-ci{color:#15803d}
.dp-label-aws{color:#c2410c}
.dp-flow{display:flex;align-items:center;flex-wrap:wrap;gap:0}
.dp-box{border-radius:8px;padding:8px 12px;font-size:12px;line-height:1.35;text-align:center;font-weight:500;color:#1e293b;flex-shrink:0;min-width:115px}
.dp-ci{background:#bbf7d0;border:2px solid #15803d}
.dp-aws{background:#fed7aa;border:2px solid #c2410c}
.dp-arrow{width:26px;flex-shrink:0;display:flex;align-items:center;justify-content:center;color:#64748b;font-size:16px;font-weight:bold}
</style>

<div class="dp-wrap">
  <div class="dp-lane dp-lane-ci">
    <div class="dp-label dp-label-ci">🔁 CI/CD Pipeline</div>
    <div class="dp-flow">
      <div class="dp-box dp-ci">FastAPI App<br><small>(classifier service)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-ci">Docker Image<br><small>(containerised)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-ci">GitHub Actions<br><small>(build &amp; push)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-ci">AWS ECR<br><small>(container registry)</small></div>
    </div>
  </div>
  <div class="dp-lane dp-lane-aws">
    <div class="dp-label dp-label-aws">☁ AWS Infrastructure (CloudFormation)</div>
    <div class="dp-flow">
      <div class="dp-box dp-aws">ECR Image<br><small>(pulled on deploy)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-aws">CloudFormation<br><small>(stack launch)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-aws">Auto Scaling<br>Group<br><small>(EC2 GPU)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-aws">Target Group<br><small>(load balancer)</small></div>
      <div class="dp-arrow">→</div>
      <div class="dp-box dp-aws">Route 53<br><small>(DNS endpoint)</small></div>
    </div>
  </div>
</div>

**Real-Time API** (FastAPI):

- Accepts listing description, price, and country code
- Returns classification with confidence score in <2s
- Falls back to Claude for uncertain predictions
- Includes business rule overrides (e.g., known leasing providers auto-flagged)

**Monitoring**: Full Datadog integration tracking prediction distributions, Claude API costs, conditional listing rates per market, and model latency.

## Fine-Tuning Under Constraints

Fine-tuning a 304M parameter transformer encoder on a single **24 GB GPU** required engineering around every memory bottleneck. Real constraints from the fine-tuning run:

| Constraint            | Problem                                | Solution                                                                                           |
| --------------------- | -------------------------------------- | -------------------------------------------------------------------------------------------------- |
| 24 GB VRAM            | Model + activations exceeded memory    | Gradient checkpointing — recompute activations on backward pass instead of storing them            |
| Max batch size 8      | Too noisy for stable convergence       | Gradient accumulation over 4 steps → effective batch of 64 without extra memory                    |
| Long input sequences  | Full 512-token inputs too slow         | Input truncation to **384 tokens** — cut training time significantly with negligible accuracy loss |
| FP32 precision        | Doubled memory for weights/activations | FP16 mixed precision training throughout                                                           |
| DeBERTa-v3-large size | 304M params barely fit                 | Combined all four techniques together to make fine-tuning feasible on a single GPU                 |

The disentangled attention mechanism in DeBERTa encodes each token using separate content and position embeddings — this is what gives it strong contextual understanding with a relatively small parameter count compared to its performance.

## Tech Stack

`Python` `PyTorch` `HuggingFace Transformers` `DeBERTa V3` `mDeBERTa V3` `FastAPI` `AWS Bedrock` `Claude` `AWS Athena` `AWS S3` `AWS EC2 (GPU)` `Papermill` `Datadog` `Docker`

## Why In-House Over Claude?

The core business case: fine-tuning a transformer model instead of routing everything through Claude delivered dramatic cost and speed gains at no meaningful accuracy cost.

### Detection Performance

| Method            | Detection Rate | False Positive Rate |
| ----------------- | -------------- | ------------------- |
| Claude            | 99%            | 1%                  |
| Transformer Model | 90% – 95%      | 4% – 7%             |

A small accuracy trade-off in exchange for 20x faster inference and a cost curve that stays flat as volume scales.

### Inference Speed

| Method            | Listings per Day | Inference Time |
| ----------------- | ---------------- | -------------- |
| Claude            | 60K              | 2 – 3 hours    |
| Transformer Model | 60K              | ~10 minutes    |

### Cost at Scale

| Listings per Day | GPU Cost p.a. (Transformer) | Claude Cost p.a. |
| ---------------- | --------------------------- | ---------------- |
| 20K              | $14K                        | $10K             |
| 40K              | $14K                        | $23K             |
| 60K              | $14K                        | $42K             |
| 80K              | $14K                        | $56K             |
| 100K             | $14K                        | $70K             |

The transformer's GPU cost is **fixed** at $14K/year regardless of volume. Claude's cost scales linearly — at 100K listings/day the in-house model is **5x cheaper**.

## Key Takeaways

- **Right-sizing models matters**: DeBERTa vs mDeBERTa selection per market improved accuracy without unnecessary compute
- **Confidence-based routing** between fast local models and LLMs is a production pattern I now use everywhere
- **Country-specific feature engineering** (keyword stems, text extraction) outperformed language-agnostic approaches
- **Automated evaluation pipelines** with LLM-as-judge provide scalable quality assurance across markets
