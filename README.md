# NeyraIndex

<p align="center"><b>Reasoning-based RAG&nbsp; ◦ &nbsp;No Vector DB, No Chunking&nbsp; ◦ &nbsp;Context-Aware Retrieval&nbsp; ◦ &nbsp;Reads Like a Director</b></p>

**A memory index for films, in design.** The goal: one film, one memory. Agents and people ask it where, when and who, and every answer points at a scene or a shot.

> **Status: early.** What runs today is the document engine this repository is forked from, [PageIndex](https://github.com/VectifyAI/PageIndex) by Vectify AI (MIT). The film layer is a design and is not implemented yet.

## Why a film needs its own index

A document has a table of contents. A film has scenes, characters, places, story time, shots and takes, and its canon changes while the film is being made. Retrieval built for reports misses three things a production needs:

- **Exact lookups.** "Every scene where she wears the red coat" is a query over fields, not a similarity search.
- **Versions.** The answer depends on which draft is approved.
- **References you can open.** An answer cites a scene, a shot or a take, not a text chunk.

## What works today

The upstream PageIndex engine, unchanged: it builds a tree index for a text-based PDF and lets an LLM agent search the tree. It suits the documents around a film, such as a screenplay PDF, a story bible or a contract.

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

Limits of local mode today: text-based PDFs (Markdown through the command line), no scanned pages, no images, video or audio, no versions, and no incremental re-indexing.

## The film layer (planned)

| Part | What it is |
|---|---|
| Film Memory Format | An open JSON format: film → scene → shot → take, with registers for cast, world and documents, and a canon ledger with versions |
| Tools | `film_outline`, `film_read`, `film_find`, `film_search`, served over MCP |
| Agents | An indexer that updates changed nodes, a librarian that answers with references, a continuity keeper |
| Evaluation | A question set per film, scored against loading the whole film into the model |
| Standards | Fountain for screenplays first; a mapping to MovieLabs OMC, then OpenTimelineIO and C2PA |

Nothing in this table ships yet. A public format specification will be added to this repository when the first version is ready.

## Credits and license

NeyraIndex is a fork of [VectifyAI/PageIndex](https://github.com/VectifyAI/PageIndex), © 2025 Vectify AI, under the [MIT License](LICENSE). The tree-index retrieval method is their work:

```
Mingtian Zhang, Yu Tang and PageIndex Team,
"PageIndex: Next-Generation Vectorless, Reasoning-based RAG",
PageIndex Blog, Sep 2025.
```

Film layer by [Neyra Labs](https://neyra.ai).
