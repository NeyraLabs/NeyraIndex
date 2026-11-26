# NeyraIndex

<p align="center"><b>Reasoning-based RAG&nbsp; ◦ &nbsp;No Vector DB, No Chunking&nbsp; ◦ &nbsp;Context-Aware Retrieval&nbsp; ◦ &nbsp;Reads Like a Director</b></p>

**Film memory: one film, one memory.** Chat, agents and prompt builders ask it where, when and who, and every answer points at a scene or a shot.

> **Status, 26 November 2026.** Two things exist today:
> 1. **The document engine** in this repository: [PageIndex](https://github.com/VectifyAI/PageIndex) by Vectify AI (MIT), kept in sync with upstream.
> 2. **The first parts of the film layer**, inside [Neyra Studio](https://neyra.ai). They are not in this repository yet (see [Progress](#progress)).
>
> The open format, the four retrieval tools and the evaluation set are still in design.

## Why a film needs its own memory

A document has a table of contents. A film has scenes, characters, places, story time, shots and takes, and its canon changes while the film is being made. Retrieval built for reports misses three things a production needs:

- **Exact lookups.** "Every scene where she wears the red coat" is a query over fields, not a similarity search.
- **Versions.** The answer depends on which draft is approved.
- **References you can open.** An answer cites a scene, a shot or a take, not a text chunk.

A film in a studio already has its structure in data, so the film layer does not parse layout the way PageIndex does for PDFs. It takes PageIndex's method instead: **read the outline, open the node you need, answer with a reference.**

## Progress

| Step | What it does | State |
|---|---|---|
| Canon reaches the crew | The scene pack for agents and the project chat read the logline and synopsis from the film's overview. Before, they read a field that does not exist. | Live in Neyra Studio |
| World in the scene | The scene pack names the location, how it looks and how it is lit, plus up to 8 linked world entries. Anything cut for size is listed, not dropped silently. | Live in Neyra Studio |
| One logline source | The project-brain functions and the score composer read the logline from the overview | Live in Neyra Studio. A few research and voice paths still read the old field |
| Location look in the master prompt | The shot blocks of the film's master prompt carry the whole look of a location written as one paragraph, not only lines 2–3 | In review |
| Lighting per scene | Lighting matched to time of day and scene number | Waiting for a structured field. Free-text lighting in several languages could not be parsed reliably |
| Shared memory layer | One context layer for every function, with the four tools below. The project chat reads the film on demand instead of a fixed first slice of scenes | Next |
| Documents, frame versions, scene history | Text of the film's documents, approved frames and scene revisions in memory | Later |

## What runs in this repository today

The PageIndex engine. It builds a tree index for a text-based PDF and lets an LLM agent search the tree. It suits the documents around a film, such as a screenplay PDF, a story bible or a contract.

Requires Python 3.10 or newer.

```bash
git clone https://github.com/NeyraLabs/NeyraIndex.git
cd NeyraIndex
pip install -e .
```

```python
import os
from pageindex import PageIndexClient

os.environ["ANTHROPIC_API_KEY"] = "your-key"

client = PageIndexClient(
    index="anthropic/claude-haiku-5-5",   # builds the tree
    chat="anthropic/claude-opus-5-5",     # searches it
)
doc_id = client.submit_document("story-bible.pdf")["doc_id"]
print(client.chat("What does the bible say about the main location?", doc_id=doc_id, citations=True))
```

The package is still named `pageindex`. `pip install pageindex` from PyPI installs Vectify's release, not this repository. For the full engine guide, see the [upstream README](https://github.com/VectifyAI/PageIndex#readme).

Limits of local mode: text-based PDFs (Markdown through the command line), no scanned pages, no images, video or audio, no versions, and no incremental re-indexing.

**Sync with upstream.** Last merged 26 November 2026: `PageSearchClient` and `PageExtractClient` expose their client as `.documents` (`.pageindex` still works, with a `DeprecationWarning`). The test suite passes on Python 3.14: 1104 passed, 180 skipped.

## The film layer (design)

### Format

An open JSON format: film → scene → shot → take, with registers for cast, world and documents, and a canon ledger with versions. Face reference photos and likeness-consent data are never part of the format or the public examples.

### Tools

| Tool | What it returns | Needs a model |
|---|---|---|
| `film_outline` | The film's outline with a short description per node | No |
| `film_read` | The full text of one node by id | No |
| `film_find` | Exact lookup by character, place, story time or prop | No |
| `film_search` | Keyword search across the whole film | No |

The agent picks the tool and answers with a scene or shot reference. Each quote is checked against the text of the node it cites. Vector search is added only if the evaluation shows the tools do worse without it.

### Agents

| Agent | Runs when | Output |
|---|---|---|
| Indexer | A film entity changes | Updated nodes. A model is called only for node descriptions |
| Librarian | A question comes from chat or from another agent | An answer with scene and shot references |
| Continuity keeper | A scene is edited, or before a render | Contradictions, each with quotes |
| Canon clerk | A package is approved at the gate | A proposed canon-ledger entry. A person approves it |
| Evaluator | Nightly and before each release | Accuracy, reference fidelity, cost and latency |

### Skills

| Skill | Use it for |
|---|---|
| `film-memory-ingest` | A new screenplay or bible: Fountain or Markdown into the format, with a dry run before writing |
| `film-memory-query` | Any question about the film: which tool to call in what order, and the reference rule |
| `film-continuity-audit` | Before a batch of renders: walk the scenes and report contradictions |
| `film-memory-eval` | Before merging a retrieval change: run the set and compare with the last report |

### Rhythms

| When | What happens |
|---|---|
| On every change | Re-index only the nodes whose fingerprint changed |
| At the gate | Propose a canon-ledger entry |
| Nightly | Full fingerprint check, refresh stale descriptions, run the evaluation |
| Weekly | Quality report to the film's owner |

### Evaluation is the gate

- 50–100 questions per film: facts, where and when, continuity, links between scenes, shot-level detail, and questions the film cannot answer.
- Three contestants: today's project chat, the whole film loaded into the model, and the new tools. Measures: accuracy, reference fidelity, tokens, latency.
- Release rule: accuracy no lower than loading the whole film, at a lower cost per question.
- External check: NovelQA (long narrative text).

### Standards

Fountain for screenplays first. Then a mapping to MovieLabs OMC, then OpenTimelineIO and C2PA.

## Roadmap

| Phase | Scope | State |
|---|---|---|
| P0 | Honest README, credit to PageIndex, sync with upstream | Done |
| P1 | Format v0, read-only export of a film from Neyra Studio, a question set for one film | Next |
| P2 | The four tools over MCP; project chat on the tools behind a flag; pass the evaluation gate | Planned |
| P3 | Indexer; document trees for the film's documents; shots and takes | Planned |
| P4 | Continuity keeper on the memory; canon ledger | Planned |
| P5 | Timecoded dialogue, shot boundaries in renders, questions across films, local models | Later |

The format specification and a Python reference implementation will be published in this repository with P1.

## Credits and license

NeyraIndex is a fork of [VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex), © 2025 Vectify AI, under the [MIT License](LICENSE). The tree-index retrieval method is their work:

```
Mingtian Zhang, Yu Tang and PageIndex Team,
"PageIndex: Next-Generation Vectorless, Reasoning-based RAG",
PageIndex Blog, Sep 2025.
```

Film layer by [Neyra Labs](https://neyra.ai).
