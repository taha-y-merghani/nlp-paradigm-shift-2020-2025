# Evolution of NLP Architectures: Benchmarking the 5-Year Collapse of Static Embeddings

**The Question:** How did the NLP pipeline collapse between 2020 and 2025?

This project benchmarks the architectural shift from **template-based synthetic data → LLM-enhanced generation** in public health surveillance (disease outbreak detection). Not a tutorial—a quantified analysis of what changed and why.

**Read the full architecture breakdown:** [`ARCHITECTURE.md`](./ARCHITECTURE.md)

---

## The Paradigm Shift

**2020 Approach (Manning LiveProject):**
- Template-based synthetic data ("Zika outbreak in Miami")
- spaCy NER with static GloVe embeddings (2014)
- Manual disease list curation (100+ entries)
- DBSCAN clustering
- Deterministic, zero-cost, fast

**2025 Replication (LLM-Enhanced):**
- LLM-generated synthetic headlines
- Transformer NER / LLM prompts
- Zero-shot entity recognition
- Same DBSCAN clustering (geometry unchanged)
- Non-deterministic, API costs, slower

**2025 Measured Results:**
- **Entity Extraction:** 168 locations (84%), 154 diseases (77%) identified
- **Spatial Clustering:** 13 outbreak clusters via DBSCAN
- **Geographic Coverage:** Southeast Asia (Jakarta: 27, Bangkok: 17), Americas (Miami: 18)

**The Architectural Insight:** LLMs didn't eliminate NLP engineering—they *shifted the bottleneck* from manual entity curation to prompt engineering and hallucination handling. The tradeoff: better generalization, but new costs (API fees, non-determinism, explainability loss).

## Replication Results (2025)

### Data Processing Pipeline
Successfully replicated the core analysis pipeline with modern tools:
- **Data Loading**: 200 LLM-generated headlines (Jan 1 - Jul 18, 2024)
- **Entity Extraction**: 168 locations (84%) and 154 diseases (77%) identified
- **Geolocation**: All locations successfully geocoded using Nominatim
- **Spatial Clustering**: DBSCAN identified 13 distinct outbreak clusters

### Key Findings
1. **Geographic Distribution**: 
   - Southeast Asia: Jakarta (27), Bangkok (17), Manila, Singapore
   - Americas: Miami (18), San Juan, Bogotá, Rio de Janeiro
   - No European or Middle Eastern outbreaks in dataset

2. **Disease Patterns**:
   - Zika virus: 54 cases (35%)
   - TB: 49 cases (32%)
   - Other diseases: 51 cases (33%)

3. **Cluster Analysis**:
   - 13 spatial clusters identified using DBSCAN (eps=400km, min_samples=3)
   - Zero noise points suggests clustering parameters may be too lenient
   - Clusters primarily represent geographic proximity rather than epidemiological relationships

### Architecture Comparison: 2020 vs 2025

| Component | 2020 (Static Embeddings) | 2025 (LLM-Enhanced) | Shift |
|-----------|--------------------------|---------------------|-------|
| **Data Generation** | Template-based | LLM-generated | Repetitive → Natural phrasing |
| **Entity Extraction** | spaCy (GloVe 2014) | Transformer NER / LLM prompts | Static → Contextual embeddings |
| **Entity Coverage** | Manual list (100+ diseases) | Zero-shot recognition | Manual → Automatic |
| **Clustering** | DBSCAN (ε=500km) | DBSCAN (ε=400km) | **Unchanged** (geometry) |
| **Determinism** | Reproducible outputs | Non-deterministic (temperature >0) | Stable → Stochastic |
| **Cost** | Free (local CPU) | API fees or GPU required | $0 → Usage-based |
| **Explainability** | Rule-based (transparent) | LLM-based (black box) | Clear → Opaque |

### Limitations Identified
1. **Geographic Bias**: Current dataset lacks European/Middle Eastern outbreaks
2. **Clustering Interpretation**: Clusters based purely on spatial proximity, not epidemiological relationships
3. **Temporal Analysis**: Limited time-series analysis compared to original
4. **Disease Diversity**: Focus on tropical diseases, missing global health threats

## Project Structure
```
my-ddo/
├── data/                    # Generated synthetic headlines
│   ├── template_headlines.txt  # Basic template-generated headlines
│   └── llm_headlines.txt      # LLM-enhanced headlines
├── 2020Analysis/           # Original analysis notebooks
│   └── step*.ipynb         # Original implementation steps
├── step4_llm_analysis.py   # Replication analysis script
├── outbreak_map.html       # Interactive visualization
├── generate_synthetic_headlines.py  # Template-based generator
└── generate_llm_headlines.py        # LLM-based generator
```

## Recent Changes
1. Data Generation Enhancement:
   - Created two synthetic data generators:
     - Template-based for baseline comparison
     - LLM-based for improved quality
   - Generated parallel datasets for comparative analysis

2. Project Restructuring:
   - Archived original 2020 analysis in `2020Analysis/`
   - Created dedicated `data/` directory for synthetic datasets
   - Set up new analysis pipeline structure

3. Replication Implementation:
   - Successfully replicated core analysis pipeline
   - Generated interactive map visualizations
   - Identified key differences from original approach

## Dependencies
- Python 3.9+
- Core packages: pandas, spaCy, scikit-learn, matplotlib/Seaborn, GeoPy
- Additional requirements in `requirements.txt` and `environment.yml`

## Next Steps
- Implement comparative analysis notebooks
- Evaluate data quality metrics between approaches
- Enhance visualization and dashboard components
- Document improvements and insights
- Address geographic bias in data generation
- Improve clustering interpretation with epidemiological context

## Getting Started
1. Clone the repository
2. Create virtual environment: `python -m venv my-ddo-env`
3. Install dependencies: `pip install -r requirements.txt`
4. Generate synthetic data using provided scripts
5. Run analysis notebooks for comparison

**Quick Visualization:**
After running the analysis script (`python step4_llm_analysis.py`), the interactive outbreak map (`outbreak_map.html`) will automatically open in your default web browser for immediate viewing. No manual steps required!

## License
[Original Project License] 