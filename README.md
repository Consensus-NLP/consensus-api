# Consensus API

[![API Reference](https://img.shields.io/badge/docs-API%20reference-068EF1)](https://docs.consensus.app/reference/v1_search)
[![Get access](https://img.shields.io/badge/get%20access-self--serve-3BCDAA)](https://consensus.app/home/api/)
[![Website](https://img.shields.io/badge/website-consensus.app-068EF1)](https://consensus.app)
[![Follow on X](https://img.shields.io/badge/follow-%40ConsensusNLP-000000?logo=x&logoColor=white)](https://x.com/ConsensusNLP)

The Consensus API is a REST API for searching academic literature: one call returns relevance-ranked, peer-reviewed papers from a corpus of 220M+, with citation counts, study metadata, journal quality signals, and query-relevant full-text excerpts.

It is the same search engine behind [Consensus](https://consensus.app), used by millions of researchers every month.

## Quickstart

```bash
curl -G "https://api.consensus.app/v1/search" \
  -H "x-api-key: $CONSENSUS_API_KEY" \
  --data-urlencode "query=Does creatine improve cognition?" \
  --data-urlencode "year_min=2015" \
  --data-urlencode "study_types=rct,meta-analysis"
```

**Python**

```python
import requests

resp = requests.get(
    "https://api.consensus.app/v1/search",
    headers={"x-api-key": "YOUR_API_KEY"},
    params={
        "query": "Does creatine improve cognition?",
        "year_min": 2015,
        "study_types": "rct,meta-analysis",
    },
)
for paper in resp.json()["results"]:
    print(paper["publish_year"], paper["citation_count"], paper["title"])
```

**JavaScript**

```javascript
const params = new URLSearchParams({
  query: "Does creatine improve cognition?",
  year_min: "2015",
  study_types: "rct,meta-analysis",
});
const resp = await fetch(`https://api.consensus.app/v1/search?${params}`, {
  headers: { "x-api-key": process.env.CONSENSUS_API_KEY },
});
const { results } = await resp.json();
```

## Endpoint

```
GET https://api.consensus.app/v1/search
```

Authenticate with your API key in the `x-api-key` header.

> **Migrating from `/v1/quick_search`?** It is deprecated and will be removed on 2027-02-07. `/v1/search` is the same contract: update the path and you are done. [Details](https://docs.consensus.app/reference/v1_quick_search).

## What a result looks like

```json
{
  "results": [
    {
      "title": "The effects of creatine supplementation on cognitive performance: a randomised controlled study",
      "authors": ["Sandkühler, J.F.", "..."],
      "publish_year": 2023,
      "doi": "10.1186/s12916-023-03146-5",
      "journal_name": "BMC Medicine",
      "citation_count": 47,
      "study_type": "rct",
      "sample_size": 123,
      "sjr_best_quartile": 1,
      "takeaway": "Creatine supplementation showed small positive effects on cognitive performance...",
      "abstract": "...",
      "url": "https://consensus.app/papers/..."
    }
  ],
  "page": 0,
  "page_size": 20,
  "is_end": false,
  "next_page": 1
}
```

Every result includes title, abstract, authors, DOI, journal, publication year, volume and pages, and citation count. Depending on the paper you also get study type, sample size, study count, population type (human or animal), preprint status, countries of study, institutions, influential citation count, and a plain-language key takeaway.

Two opt-in extras:

- `include_semantic_score=true`: a relevance score for the top results
- `include_full_text_chunks=true`: query-relevant excerpts from licensed full text (paid plans)

## Filters

| What you want | Parameters |
|---------------|------------|
| A publication window | `year_min`, `year_max`, `month_min`, `month_max` |
| Specific study designs | `study_types` (`rct`, `meta-analysis`, `systematic review`, `cohort study`, ...) |
| Methodological rigor | `human`, `controlled`, `sample_size_min`, `exclude_preprints` |
| Journal quality | `sjr_min` / `sjr_max` (SJR quartile, 1 = top), `citation_min` |
| Medical focus | `medical_mode` (top medical journals and guidelines, ~8M documents), `clinical_guideline` |
| Scope | `domain` (`med`, `bio`, `cs`, `psych`, `econ`, ...), `country`, `journal_name`, `publisher_name`, `open_access` |

Full parameter reference: [docs.consensus.app/reference/v1_search](https://docs.consensus.app/reference/v1_search)

## Getting access

The Consensus API is self-serve: create an API key from your Consensus account and start building. Paid plans (Pro, Deep, Teams) include monthly API usage. For production workloads, high volume, and custom rate limits, [talk to us about Enterprise](https://consensus.app/home/api/).

## What developers build

- LLM and RAG applications grounded in citable, peer-reviewed sources
- Literature discovery and systematic review tooling
- Research copilots, writing assistants, and reference managers
- Evidence dashboards for clinical and policy teams

## Consensus MCP

Looking to use Consensus inside an AI assistant instead of your own code? The Consensus MCP server gives Claude, ChatGPT, Cursor, and any MCP client the same search over a remote connection:

```
https://mcp.consensus.app/mcp
```

See the [consensus-mcp](https://github.com/Consensus-NLP/consensus-mcp) repo and the [MCP guide](https://docs.consensus.app/docs/mcp).

---

Made by [Consensus](https://consensus.app), the AI search engine for research. Follow us on [X](https://x.com/ConsensusNLP) and [LinkedIn](https://www.linkedin.com/company/consensus-nlp).
