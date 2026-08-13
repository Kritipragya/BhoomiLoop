# Carbon Credit Compliance RAG Assistant

A retrieval-augmented generation (RAG) system that answers questions about
India's Carbon Credit Trading Scheme (CCTS) offset rules for **waste
projects** — grounded in an official BEE/CCTS knowledge document, with source
citations.

Built from scratch to understand RAG end to end: not just the happy path, but
the failure modes and the fixes.

> **Prototype for guidance only.** Final carbon-credit eligibility is decided by
> BEE and an accredited verification agency (ACVA). This tool helps a user
> understand the rules and find the relevant section — it does not certify a claim.

---

## What it does

Ask a question like *"Can a housing society earn carbon credits for composting?"*
and the assistant:

1. Retrieves the most relevant passages from the CCTS waste methodology document
2. Answers using **only** those passages (grounded, not from model memory)
3. Cites the document sections the answer came from
4. Refuses questions whose answer is not in the document

## The pipeline

```
Chunk  ->  Embed  ->  Store        (indexing, done once)
Retrieve  ->  Generate             (query, every question)
```

| Stage | Choice |
|---|---|
| Chunk | Structure-aware (splits on headings/paragraphs, tagged with section) |
| Embed | `all-mpnet-base-v2` (sentence-transformers) |
| Store | FAISS (`IndexFlatL2`) |
| Retrieve | Top-3 nearest chunks |
| Generate | `Qwen2.5-1.5B-Instruct`, grounded prompt |

## What I learned building this

1. **Retrieval is imperfect** — top-3 chunks + the LLM together are robust; no
   single chunk is reliable on its own.
2. **Chunking strategy decides retrieval quality** — blind word-count chunking
   mixed concepts together; structure-aware chunking fixed several cases.
3. **The embedding model matters** — moving from MiniLM to mpnet improved
   nuanced retrieval.
4. **Query phrasing matters** — short, abstract questions ("what is X?") are the
   hardest, because the answer sits inside a larger block (the *semantic gap*).
5. **Grounding actually works** — verified with a hallucination test: the system
   refuses out-of-document questions (e.g. a carbon-credit price) instead of
   inventing an answer.

The notebook keeps every iteration and failure, because that is where the
understanding is.

## Files

| File | What it is |
|---|---|
| `AIPL_RAG_Practice_V2_documented.ipynb` | The full build, documented stage by stage |
| `ccts_waste_methodology.md` | The knowledge base (sourced from BEE/CCTS material) |
| `requirements.txt` | Dependencies |

## Run it

```bash
pip install -r requirements.txt
```

Then open the notebook (Colab recommended, with a T4 GPU), make sure
`ccts_waste_methodology.md` is in the working directory, and run the cells top to
bottom. The final cell launches a Gradio demo UI.

## Source

Knowledge base compiled from Bureau of Energy Efficiency (BEE) material on the
CCTS Offset Mechanism (Detailed Procedure, Version 1, March 2025) and related
Ministry of Power / MoEFCC notifications. Simplified for a prototype.
