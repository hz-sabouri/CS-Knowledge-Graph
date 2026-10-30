# CS Knowledge Graph Builder (Showcase)

An LLM-based pipeline that turns research-paper abstracts into a searchable **knowledge graph** in Neo4j, with an interactive 3D web UI for exploring it.


| | |
|---|---|
| **Input** | CSV of papers (title, abstract, authors, venue, ...) |
| **Data** | 3,000 arXiv AI/ML abstracts (2025-2026, Kaggle) |
| **Output** | Neo4j graph: ~30K nodes, ~49K edges, 19 entity types, 27 relation types |
| **Stack** | Python · Gemini API · embeddings · Neo4j (Cypher) · FastAPI · JavaScript |

## Demo

The video has no audio and the interface is in Persian (Farsi). The table below shows what happens when.

https://github.com/user-attachments/assets/f12220cb-f6cf-4237-9147-355a2d1b3424

| Time (approx.) | What you see |
|---|---|
| 0:00 | Home page with the four sections: view graph, build graph, Q&A, about |
| 0:06 | Q&A page (prototype preview) |
| 0:12 | Entering the 3D "hologram" explorer |
| 0:18 | Inspecting nodes: type, properties, related entities and the rules behind each relation |
| 0:36 | Full graph in 3D, clustered and colored by entity type |
| 0:42 | Entity search (query "ml"), then opening a paper with its authors and extracted entities |
| 1:06 | Statistics panel: entity counts per type |
| 1:18 | Building a graph from a CSV: upload, validation rules, 8-step pipeline, live log, JSON export |

## The problem

Computer-science knowledge is mostly locked in unstructured text such as paper abstracts. Keyword search finds documents, but it does not answer questions like *"which methods were evaluated on this benchmark?"* or *"what extends this model?"*. A knowledge graph stores concepts as entities and the relations between them, which makes those questions directly queryable.

## How it works

1. Reads a structured CSV of papers.
2. Extracts typed entities from each abstract with an LLM (Gemini API).
3. Merges duplicates in two steps: exact matches first, then embedding similarity with final LLM verification.
4. Extracts typed relations between entities, using the entity list plus the source abstract and few-shot prompting.
5. Adds structural metadata (papers, authors, journals) with a script.
6. Stores everything in Neo4j and serves it through an interactive UI.

<p align="center">
  <img src="https://github.com/user-attachments/assets/5406d010-984b-4b88-a510-86322820e398" alt="Pipeline architecture" width="420">
</p>

<details>
<summary><b>Ontology</b> (19 entity classes, 27 relation types)</summary>

**Entity classes:** Model, Method / Framework, Concept / Technology, Task / Application, Dataset / Benchmark, Metric, Software / Platform, Hardware / Component, Robot / System, Organization / Affiliation, Author / Person, Venue, Regulation / Policy, Region / Location, Study / Experiment, Material / Chemical, Biomolecule / Gene, Competition / Challenge, Other

**Relations:** USES, APPLIED_TO, EVALUATED_ON, TRAINED_ON, FINE_TUNED_FROM, DISTILLED_FROM, IMPLEMENTED_IN, RUNS_ON, OUTPERFORMS, COMPARED_WITH, ACHIEVES, MEASURED_BY, PART_OF, BASED_ON, EXTENDS, ALTERNATIVE_TO, INTRODUCES, AFFILIATED_WITH, PUBLISHED_IN, LOCATED_IN, PARTICIPATES_IN, TARGETS, CONTROLS, GOVERNED_BY, RELATED_OTHER, SAME_AS, IS_A

</details>

## Interface

- **3D graph view** with nodes colored by entity type; click a node to inspect it, double-click to focus
- **Search** by entity name, with filters by entity type
- **Entity and relationship browsing**, including the rule behind each relation
- **Statistics** per entity type
- **Graph builder:** upload a CSV, validate it, run the pipeline and download the result as JSON

## Results

| | Count |
|---|---|
| Nodes, full graph (incl. papers, authors, journals) | 30,407 |
| Edges, full graph | ~49,000 |
| Entities extracted from text | 12,979 |
| Relations extracted from text | 15,997 |
| Duplicate reduction (entity resolution) | 16,777 → 12,979 (22.6%) |
| Multi-hop query latency | < 130 ms warm, ~500 ms cold |

**Extraction quality** (automatic proxy metrics, see limitations):

| Metric | Value | How it was measured |
|---|---|---|
| NER precision | 99.7% | Share of extracted entities whose name or abbreviation appears in the source text |
| NER recall | 87% | Share of regex-detected named terms in abstracts that are present in the graph |
| **NER F1** | **93%** | Harmonic mean of the two above |
| RE precision | 77% | Share of relations whose two endpoints co-occur in a short span (about two sentences) |
| RE recall | 60% | Share of entity pairs linked by a cue verb in one sentence (uses, based on, outperforms, ...) that have a relation in the graph |
| **RE F1** | **67%** | Harmonic mean of the two above |

## Challenges

- **The LLM renamed entities** instead of copying them from the text. Controlled with prompt engineering.
- **Wrong merges in entity resolution:** entities that look similar but mean different things were merged. Reduced by sending embedding-based candidates to an LLM for a final check.
- **Free-tier API limits** on requests and tokens, plus slow LLM responses, which capped the corpus at 3,000 abstracts (7,000 were planned).
- **Graph size:** the full graph made rendering and queries slower, so the UI loads and filters by type.

## Limitations

- Evaluation is **proxy-based, not against a human-annotated gold set**. Relation precision checks co-occurrence only; relation **type and direction were not verified**.
- Entity-resolution **precision was not measured**.
- Schema consistency (100%) means adherence to the ontology, not semantic correctness.
- About 57% of the nodes (papers, authors, journals) were added by script from metadata; the LLM-extracted part is the 12,979 entities and 15,997 relations above.
- One domain (AI/ML papers) and abstracts only, not full texts.

## Next steps

- Human-annotated evaluation set and measured entity-resolution precision
- Larger corpus and more CS sub-domains
- Cheaper extraction (lighter models or fine-tuning) and contradiction detection across sources
- A question-answering assistant grounded in the graph

## Contact

Zahra Sabouri · [GitHub](https://github.com/hz-sabouri)
