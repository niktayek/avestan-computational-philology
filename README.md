# Avestan Computational Philology

**Tools for textual variant analysis, witness relationship mapping, and orthographic feature extraction**

**Status**: Experimental research tools  
**Related work**: [Avestan Text Processing Suite](https://github.com/niktayek/Avestan-Text-Processing-Suite) (Master's thesis, UC Berlin 2025)  
**Published models**: [Hugging Face](https://huggingface.co/Nikyek/avestan-ocr-kraken-v1)

---

## Overview

This repository contains computational tools developed during Master's thesis research for analyzing orthographic patterns in Avestan manuscripts. Rather than being finalized research contributions, these are experimental implementations of philological analysis methods explored during the thesis phase.

The toolkit includes:

- **Token alignment** (matchers) — Precise substitution detection and candidate correction for OCR post-correction
- **Textual error analysis** (leitfehler) — Systematic identification of manuscript variants (additions, omissions, permutations, substitutions)
- **Manuscript relationship analysis** (scribal schools) — Feature aggregation, similarity matrices, and manuscript clustering

These tools are designed to support questions in textual criticism:
- How do witness manuscripts relate genealogically?
- What orthographic patterns characterize different scribal traditions?
- Can computational methods identify meaningful textual variants automatically?

---

## Who Should Use This

- **Philologists** exploring computational support for textual analysis
- **Manuscript studies researchers** investigating scribal traditions and transmission patterns
- **Digital humanities practitioners** developing tools for historical language analysis
- **Colleagues** at partner institutions working on variant analysis in related texts
- **High-risk researchers** willing to use experimental tools and provide feedback

**Note**: These tools are not finalized. They represent exploratory methodology that did not make it into the final thesis scope. They are documented here for research transparency and potential reuse.

---

## Contents

### 1. Token Matching (matchers/)

Aligns OCR-generated tokens against canonical references to enable precise substitution detection and correction candidate extraction.

#### `dictionary_matcher/`
- Lexicon and rule-aware alignment
- Generates per-token substitution tables (CSV)
- Produces correction candidates for manual review
- Outputs: `(generated, reference, distance, changes, comment)`

#### `sequence_matcher/`
- Dynamic programming alignment across blocks/lines
- Handles merges, splits, and noisy spans
- Similarity scores for pair confidence
- Outputs: aligned pairs with merge/split flags

#### `spellchecker_test/`
- Experimental candidate suggestion system
- Baseline string similarity approach
- For exploration only (high false positive rate)

**Use case**: Pre-processing step to improve OCR accuracy before apparatus generation.

### 2. Error Analysis (leitfehler/)

Detects and classifies orthographic variants in aligned text streams using Martínez Porro's philological taxonomy.

- `addition_omission_new.py` — Find added/omitted characters
- `shared_permutation.py` — Identify character reorderings
- `shared_substitution_*.py` — Character and word-level substitutions
- `leitfehler_detector.py` — Unified error detection
- `matrix_builder*.py` — Build frequency matrices for variants
- `jaccard_clustermap.py` — Clustering analysis

**Outputs:**
- Binary/weighted omission matrices (for stemma construction)
- Error type frequency tables
- Dendrograms and cluster maps
- Leitfehler-tagged XML

**Use case**: Identifying shared error patterns across manuscripts to reconstruct transmission history.

### 3. Manuscript Relationship Analysis (scribal_school_analysis/)

Aggregates orthographic features across manuscripts to identify scribal traditions and manuscript groupings.

- `01_match_tokens-dictionary.py` — Align transcribed text
- `02_detect_features.py` — Extract orthographic features per token
- `03_create_frequency_matrix.py` — Build feature profiles per manuscript
- `04_create_similarity_matrix.py` — Compute manuscript similarities
- `05_propose_feature_catalog.py` — Generate feature catalogs (quantitative & qualitative)
- `06_scribal_school_prediction.py` — Classify new manuscripts based on features

**Outputs:**
- Frequency matrices (CSVs)
- Similarity matrices
- Dendrograms (phylogenetic-style trees)
- Feature catalogs with phonetic interpretations

**Use case**: Grouping manuscripts by scribal tradition (Iranian vs. Indian, Yazdi vs. Navsari, etc.).

---

## Installation

### Prerequisites
- Python 3.8+
- Poetry or pip

### Option 1: Poetry (Recommended)

```bash
git clone https://github.com/niktayek/avestan-computational-philology.git
cd avestan-computational-philology
poetry install
```

### Option 2: pip

```bash
git clone https://github.com/niktayek/avestan-computational-philology.git
cd avestan-computational-philology
pip install -r requirements.txt
```

### Dependencies

Core packages:
- `lxml` — XML parsing and generation
- `pandas` — Data manipulation and matrix operations
- `scipy` — Clustering and similarity metrics
- `pyyaml` — Configuration management
- `numpy` — Numerical operations

---

## Usage Examples

### Token Matching

```bash
# Dictionary-based alignment (high precision)
poetry run python src/matchers/dictionary_matcher/match_tokens.py \
  --generated ocr_output.txt \
  --reference canonical_text.txt \
  --output matches.csv

# Sequence-based alignment (robust to merges/splits)
poetry run python src/matchers/sequence_matcher/align_blocks.py \
  --witness witness_transcription.txt \
  --lemma canonical_text.txt \
  --output aligned_pairs.json
```

### Error Detection

```bash
# Detect all variants
poetry run python src/leitfehler/leitfehler_detector.py \
  --lemma canonical.xml \
  --witnesses witness1.xml witness2.xml witness3.xml \
  --output variant_report.csv

# Build omission matrix
poetry run python src/leitfehler/matrix_builder_no_filtering.py \
  --comparisons comparison_output.csv \
  --output omission_matrix.csv
```

### Manuscript Clustering

```bash
# Extract features from all manuscripts
poetry run python src/scribal_school_analysis/02_detect_features.py \
  --matched-tokens all_matches.csv \
  --feature-catalog features.yaml \
  --output manuscript_features.csv

# Compute similarities
poetry run python src/scribal_school_analysis/04_create_similarity_matrix.py \
  --frequency-matrix features.csv \
  --output similarity_matrix.csv \
  --metric jaccard

# Visualize clusters
poetry run python src/scribal_school_analysis/05_propose_feature_catalog.py \
  --similarity-matrix similarity.csv \
  --output dendrogram.png
```

See individual script `--help` for detailed options.

---

## Data Formats

### Input Formats

**Text files:**
- Plain UTF-8 text (one witness/reference per file)
- Avestan script (Unicode U+10130–U+1018E or Latin transliteration)

**XML files:**
- CAB XML format (preferred)
- eScriptorium ALTO XML (with preprocessing)
- TEI XML (custom ingestion)

**Configuration:**
- YAML files for feature catalogs and orthography families
- See `data/` for examples

### Output Formats

**Tabular:**
- CSV with standard Python escaping
- Columns: `lemma, witness, feature_type, distance, metadata`

**Hierarchical:**
- Newick format trees (`.nwk`)
- JSON dendrogram structures
- PNG dendrograms (via scipy)

**Annotated:**
- TEI/CAB XML with `@resp` and `@ana` attributes
- Leitfehler tags indicating error types

---

## Experimental Status & Limitations

**Why these tools are experimental:**

1. **Not in thesis scope** — The final thesis focused on the critical apparatus pipeline (configuration-driven classification). These tools represent exploratory work that diverged from the main research trajectory.

2. **Variable testing coverage** — Some scripts have been thoroughly tested on Yasna data; others are prototype implementations with known edge cases.

3. **Hardcoded parameters** — Many scripts have paths, feature lists, and thresholds specific to the Avestan manuscripts used during development. Adaptation to other texts/languages requires customization.

4. **Incomplete documentation** — While code is commented, some scripts lack detailed usage documentation. See inline comments and example notebooks for clarification.

**Known limitations:**

- Syncope (vowel deletion) detection is incomplete
- Matrix building assumes aligned input; unaligned comparisons may produce spurious results
- Clustering uses Jaccard similarity; other metrics may be more appropriate for different analyses
- Scribal school assignment remains a manual, human-driven process; automation is exploratory

---

## Reproducibility & Data

To reproduce analyses on Yasna data:

1. Contact Corpus Avesticum Berolinense (CAB) for manuscript corpus access
2. Use canonical lemma from `data/Yasna_Static.xml` (or CAB)
3. Run matchers on OCR output → witness alignments
4. Export leitfehler matrices
5. Aggregate features for scribal school analysis

Results should match Appendix X of the Master's thesis (2025) for comparative validation.

**Custom data:** To adapt these tools to other manuscripts or languages:
- Modify feature catalogs in `data/orthography_families.yaml`
- Adjust similarity thresholds in scribal_school_analysis config
- Retrain matcher models if moving to very different script families

---

## Output Examples

### Leitfehler Analysis

```
manuscript,canonical,variant,type,frequency,significance
0088,ā,ə,substitution-vowel,3,0.75
0090,ā,ə,substitution-vowel,1,0.25
0093,θ,t,substitution-consonant,5,0.95
```

### Similarity Matrix (Manuscript Clustering)

```
         0088   0090   0093   0110
0088   1.000  0.852  0.721  0.641
0090   0.852  1.000  0.693  0.658
0093   0.721  0.693  1.000  0.629
0110   0.641  0.658  0.629  1.000
```

### Dendrogram (Newick format)

```
((0088:0.15,0090:0.18):0.22,(0093:0.19,0110:0.21):0.25):0.0;
```

---

## Contributing & Feedback

These experimental tools are most useful when tested on new data and receive feedback. If you:

- Find bugs or unexpected behavior
- Test on manuscripts outside Avestan
- Adapt tools for related scripts (Pahlavi, Sogdian, etc.)
- Have suggestions for improvements

Please open an issue with:
- Tool name and script
- Input data characteristics
- Expected vs. actual output
- Suggested fix (if applicable)

---

## Citation

If using these tools in research, cite the thesis repository and this toolkit:

Yekrang Safakar, N. (2025). *Integrating OCR and Automated Apparatus Construction for Avestan Texts*. Master's thesis, Freie Universität Berlin. https://github.com/niktayek/Avestan-Text-Processing-Suite

For this toolkit:

Yekrang Safakar, N. (2025). Avestan Computational Philology. Experimental research tools. https://github.com/niktayek/avestan-computational-philology

---

## Related Repositories

- **[Avestan Text Processing Suite](https://github.com/niktayek/Avestan-Text-Processing-Suite)** — Master's thesis (finalized OCR and apparatus pipelines)
- **[avestan-manuscript-digitization-toolkit](https://github.com/niktayek/avestan-manuscript-digitization-toolkit)** — Colab notebooks for colleague training
- **[avestan-ocr-kraken-v1](https://huggingface.co/Nikyek/avestan-ocr-kraken-v1)** — Published model on Hugging Face

---

## Resources

- **[Martínez Porro (2020)](https://www.publications.uni-frankfurt.de/handle/pufa/123)** — Systematic orthographic analysis (theoretical foundation for feature definitions)
- **[Cantera Glera (2016)](https://www.avesta.uni-frankfurt.de/)** — Corpus Avesticum Berolinense documentation
- **[Skjærvø & Oktor Skjaervo](https://scholar.harvard.edu/skjærvø/publications)** — Avestan philological works

---

## License

MIT License. See LICENSE file for details.

These experimental tools are provided for academic and research use. Attribution to the thesis and primary repository is requested.

---

## Questions & Support

- **Bug reports**: GitHub Issues
- **Usage questions**: Check script documentation and inline comments
- **Methodology questions**: See Master's thesis for context
- **Contact**: niktaayekrang@gmail.com

---

**Last updated**: February 2026  
**Status**: Experimental research tools from Master's thesis  
**Maintained by**: Nikta Yekrang Safakar
