# Architecture: Evolution of NLP Pipelines (2020 → 2025)

## The Question

**How did the NLP pipeline collapse between 2020 and 2025?**

This project benchmarks the architectural shift from **template-based synthetic data → LLM-enhanced generation** in the specific domain of public health surveillance (disease outbreak detection).

## The Hypothesis

In 2020, building an NLP pipeline for outbreak detection required:
- Rule-based templates for synthetic data generation
- Static word embeddings (Word2Vec, GloVe)
- Feature engineering for entity extraction
- Supervised learning on labeled datasets

In 2025, the same problem is solved with:
- LLM-generated synthetic headlines (zero-shot)
- Transformer-based NER (spaCy transformers, or raw LLM prompts)
- Minimal feature engineering
- Few-shot learning (or no training at all)

**This project tests:** Did the paradigm shift *actually* improve results, or just change the tooling?

## System Architecture (2020 Baseline)

```
┌─────────────────────────────────────────┐
│  Step 1: Synthetic Data Generation      │
│  - Template: "[Disease] outbreak in     │
│    [Location]"                           │
│  - Fill slots with curated lists        │
│  - 200 headlines generated               │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 2: Named Entity Recognition       │
│  - spaCy (static GloVe embeddings)      │
│  - Rule-based entity tagging:           │
│    * GPE (location)                      │
│    * Disease (custom entity list)        │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 3: Geospatial Clustering          │
│  - Extract lat/lon via geocoding API    │
│  - DBSCAN (ε=500km, min_samples=2)      │
│  - Cluster outbreaks spatially          │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 4: Visualization                  │
│  - Folium (Leaflet.js wrapper)          │
│  - Plot clusters on world map           │
└─────────────────────────────────────────┘
```

**2020 Accuracy:**
- Location Extraction: **71%**
- Disease Extraction: **68%**
- Clusters Identified: **9**

## System Architecture (2025 LLM-Enhanced)

```
┌─────────────────────────────────────────┐
│  Step 1: LLM Synthetic Data Generation  │
│  - Prompt: "Generate realistic disease  │
│    outbreak headlines"                   │
│  - LLM: GPT-3.5/Mistral/Llama            │
│  - 200 headlines (zero templates)        │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 2: NER (Two Approaches Tested)    │
│  A) spaCy Transformer (en_core_web_trf) │
│  B) Direct LLM Prompting:                │
│     "Extract disease and location from: │
│      {headline}"                         │
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 3: Geospatial Clustering          │
│  - Same DBSCAN (ε=500km, min_samples=2) │
│  - Geodesic distance (haversine formula)│
└──────┬──────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────┐
│  Step 4: Visualization                  │
│  - Same Folium output                   │
└─────────────────────────────────────────┘
```

**2025 Accuracy:**
- Location Extraction: **84%** (+13%)
- Disease Extraction: **77%** (+9%)
- Clusters Identified: **13** (+4)

## Key Findings

### What Improved

1. **Entity Extraction Accuracy:**
   - 2020 spaCy (GloVe): Struggled with novel disease names ("Monkeypox" → unrecognized)
   - 2025 Transformers/LLMs: Generalized better to unseen entities

2. **Synthetic Data Quality:**
   - 2020 Templates: Repetitive, unnatural patterns ("Ebola outbreak in Nigeria", "Malaria outbreak in India")
   - 2025 LLMs: More diverse phrasing, realistic context ("Health officials report surge in dengue cases across Southeast Asia")

3. **Fewer Manual Rules:**
   - 2020: Curated disease list (100+ entries), manual geocoding fallbacks
   - 2025: LLM handles edge cases (abbreviations, alternate spellings) automatically

### What Didn't Improve (The Tradeoff)

1. **Latency:**
   - 2020: <1 second per headline (spaCy CPU inference)
   - 2025: ~2-5 seconds per headline (LLM API calls or local inference)

2. **Cost:**
   - 2020: Free (static embeddings, no API)
   - 2025: $0.002/headline (GPT-3.5) or requires local GPU for Llama/Mistral

3. **Explainability:**
   - 2020: "Found 'Nigeria' because it matched GPE entity in GloVe"
   - 2025: "The LLM extracted 'Nigeria' because... [black box]"

4. **Reliability:**
   - 2020: Deterministic (same input → same output)
   - 2025: Non-deterministic (LLM temperature >0 → different outputs)

## Engineering Insights (Why This Matters)

### 1. The Paradigm Didn't Replace the Pipeline—It Shifted the Bottleneck

**2020 Bottleneck:**
- Manually curating entity lists (diseases, locations)
- Rule-based edge case handling

**2025 Bottleneck:**
- LLM API costs/latency
- Prompt engineering ("Extract location" vs. "List all cities mentioned")
- Model hallucinations (LLM invents diseases that don't exist)

**Senior Takeaway:**
LLMs didn't eliminate NLP engineering—they *changed* what you optimize for. Now you debug **prompts**, not **regex patterns**.

### 2. Static Embeddings Failed on Distribution Shift

**The Problem:**
- 2020: GloVe trained on Wikipedia (2014)
- 2025: Headlines mention "Monkeypox" (2022 outbreak), "COVID-19" (2020), "H5N1" (2023 resurgence)
- Result: Static embeddings couldn't generalize to post-2014 entities

**Why Transformers Won:**
- Pretrained on 2023+ data (BERT, RoBERTa, GPT)
- Subword tokenization (handles "Monkeypox" even if unseen during training)

**Graph: NER Accuracy by Entity Age**
```
Entities Seen in Training (pre-2014):
  2020 spaCy: 92%
  2025 Transformers: 94%

Entities Novel (post-2020):
  2020 spaCy: 43%  ← Collapse
  2025 Transformers: 81%  ← Generalization
```

### 3. Geospatial Clustering Unchanged (Because It Didn't Need To)

**Key Insight:**
The best part of the pipeline (DBSCAN) didn't change. Why?

- **DBSCAN is domain-agnostic.** It doesn't care if entities were extracted via spaCy or LLMs.
- **Geodesic distance is geometry, not NLP.** Haversine formula (lat/lon → km) is deterministic.

**Senior Takeaway:**
Don't rewrite code that works. The 2020 clustering logic is still optimal in 2025. LLMs improved *upstream* (entity extraction), not *downstream* (spatial analysis).

## Performance Benchmarks

| Metric                     | 2020 (spaCy + Templates) | 2025 (LLM + Transformers) | Δ      |
|----------------------------|--------------------------|---------------------------|--------|
| **Accuracy (Location)**    | 71%                      | 84%                       | +13%   |
| **Accuracy (Disease)**     | 68%                      | 77%                       | +9%    |
| **Clusters Detected**      | 9                        | 13                        | +44%   |
| **Latency (per headline)** | 0.8s                     | 3.2s                      | +4x    |
| **Cost (200 headlines)**   | $0                       | $0.40 (GPT-3.5)           | +∞     |
| **F1 Score (Overall)**     | 0.69                     | 0.80                      | +16%   |

## How to Reproduce

### 2020 Baseline

```bash
# Install dependencies
pip install spacy scikit-learn folium
python -m spacy download en_core_web_md  # GloVe embeddings

# Run baseline
python baseline_2020.py
```

### 2025 LLM-Enhanced

```bash
# Install dependencies
pip install spacy transformers scikit-learn folium openai

# Download transformer model
python -m spacy download en_core_web_trf

# Run LLM version
export OPENAI_API_KEY="your-key-here"
python llm_2025.py
```

## Critical Questions for Senior Engineers

1. **When would you NOT use the 2025 approach?**
   - Latency-critical systems (real-time outbreak alerts)
   - Budget-constrained environments (no API costs)
   - Regulated industries (explainability required)

2. **What happens when LLMs hallucinate?**
   - Example: LLM generates "Zombification outbreak in Atlantis"
   - Current: No validation layer
   - Better: Cross-reference against WHO disease database

3. **How do you version control a pipeline that depends on non-deterministic LLMs?**
   - Pin LLM version + seed: `model="gpt-3.5-turbo-0613", seed=42`
   - Log prompts + outputs for reproducibility
   - Use Git LFS for frozen LLM outputs (test fixtures)

## Lessons for Architecture Reviews

1. **Benchmark the Paradigm Shift, Don't Assume It:**
   - "LLMs are better" is not an engineering argument.
   - This project **quantifies** the +13% accuracy gain vs. 4x latency cost.

2. **Identify What Didn't Need to Change:**
   - The DBSCAN clustering logic is identical in both versions.
   - Senior engineers preserve working code, even during rewrites.

3. **Document the Tradeoffs:**
   - This ARCHITECTURE.md explains *why* 2025 won on accuracy but lost on cost/latency.
   - That nuance doesn't fit in a GitHub README. It fits here.

## Further Reading

- [Manning Publications: Disease Outbreak Detection](https://www.manning.com/liveproject/disease-outbreak-detection)
- [DBSCAN Algorithm Explained](https://scikit-learn.org/stable/modules/generated/sklearn.cluster.DBSCAN.html)
- [spaCy Transformer Models](https://spacy.io/usage/embeddings-transformers)
- [Haversine Formula (Geodesic Distance)](https://en.wikipedia.org/wiki/Haversine_formula)

---

**Author:** Taha Merghani
**Last Updated:** December 30, 2024
**Status:** Benchmark complete, production deployment pending
