# ADTA 5760 — Contaminated Knowledge Base for LLM Testing

> **Course:** ADTA 5760 · NLP with Neural Networks · University of North Texas  
> **Live Demo:** [adta-5760-group-4.vercel.app](https://adta-5760-group-4.vercel.app)

## Overview

A curated knowledge base of **150 academic PDFs** embedded with **450 deliberate contaminants** across **3 contamination types**, built to evaluate whether Large Language Models can detect planted misinformation in domain-specific documents. The dataset spans two knowledge domains — supply chain management and medical research — and is served via an interactive web interface deployed on Vercel.

## Key Stats

| Metric | Value |
|--------|-------|
| Supply Chain PDFs | 100 |
| Medical PDFs | 50 |
| Total Contaminants | 450 |
| Contamination Types | 3 |

## How It Works

1. **Sourced** 150 real academic papers from supply-chain and medical domains.
2. **Injected** three types of knowledge contamination (factual errors, fabricated citations, statistical manipulation) into each paper — 450 contaminants total.
3. **Built** a searchable web interface (HTML + CSS + JS) listing every document with direct PDF viewing.
4. **Deployed** on Vercel for public access and LLM retrieval-augmented evaluation.

## Repository Structure

```
├── index.html       # Single-page web app (document browser)
├── css/
│   └── style.css    # Styling
├── js/
│   └── main.js      # Search, filtering & PDF viewer logic
└── pdfs/            # 150 contaminated PDFs (100 SC + 50 Medical)
```

## Tech Stack

`HTML` · `CSS` · `JavaScript` · `Vercel`

## Team — Group 4

| Member | Role |
|--------|------|
| **Karan Parekh** | Team Lead |
| Sanjana PR | Member |
| Sana Mhapsekar | Member |
| Medina Maloku | Member |
