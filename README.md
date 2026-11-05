<a href="https://consensus.app"><img src="assets/logo.png" alt="Consensus" width="200" /></a>

# Consensus API: Scientific Paper Search API for RAG, AI Agents and Research Tools

[![API docs](https://img.shields.io/badge/docs-API%20reference-068EF1)](https://docs.consensus.app/api-get-started)
[![Get access](https://img.shields.io/badge/get%20access-self--serve-3BCDAA)](https://consensus.app/api-mcp)
[![Website](https://img.shields.io/badge/website-consensus.app-068EF1)](https://consensus.app)
[![Follow on X](https://img.shields.io/badge/follow-%40ConsensusNLP-000000?logo=x&logoColor=white)](https://x.com/ConsensusNLP)

The Consensus API is a REST API for semantic search over **400M+ scholarly articles, including 350M peer-reviewed papers and all of PubMed**. One call returns relevance-ranked papers with DOIs, citation counts, study type, sample size, journal quality (SJR quartile), a plain-language takeaway and, on paid plans, the full-text passages that answer your query.

Use it to ground LLM answers, RAG pipelines and AI agents in real, citable research, or to build literature review, systematic review and evidence monitoring tools. It's the same search engine that powers [Consensus](https://consensus.app), used by millions of researchers.

Every plan includes API access. The Free plan includes 30 calls a month, so you can make a request in a couple of minutes.

## Quickstart

1. Sign in at [consensus.app](https://consensus.app) and open the [API & MCP Dashboard](https://consensus.app/api-mcp)
2. Click **Create API key** and copy it (it's shown once)
3. Search:

```bash
export CONSENSUS_API_KEY="your-key"

curl -G "https://api.consensus.app/v1/search" \
  -H "x-api-key: $CONSENSUS_API_KEY" \
  --data-urlencode "query=Does creatine improve cognition?" \
  --data-urlencode "year_min=2015" \
  --data-urlencode "study_types=rct,meta-analysis"
```

**Python**

```python
import os
import requests

resp = requests.get(
    "https://api.consensus.app/v1/search",
    headers={"x-api-key": os.environ["CONSENSUS_API_KEY"]},
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

More languages (Go, PHP, Ruby, Java) are in the [getting started guide](https://docs.consensus.app/api-get-started).

## Endpoints

| Endpoint | What it does |
|----------|--------------|
| `GET /v1/search` | Semantic paper search with filters. Synchronous, one call. |
| `POST /v1/threads` | Hand a research question to the Consensus research agent. Returns a `thread_id`. |
| `GET /v1/threads/{thread_id}` | Poll a Thread for the agent's cited answer. |
| `POST /v1/threads/{thread_id}/interactions` | Ask a follow-up that builds on earlier answers. |
| `GET /v1/threads` | List your Threads. |

Base URL `https://api.consensus.app`. Authenticate every request with your key in the `x-api-key` header.

**Search or Threads?** Search returns papers and leaves the reasoning to your code: use it for RAG retrieval and pipelines. A Thread runs many steps (searching, citation crawls, reading full papers) and returns a written answer with citations and grounding quotes. [Threads quick start](https://docs.consensus.app/api-quickstart-threads).

> **Migrating from `/v1/quick_search`?** It's deprecated and will be removed on 2027-02-07. `/v1/search` is the same contract: update the path and you're done. [Details](https://docs.consensus.app/api-reference/quick-search).

## What a search result looks like

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

Every result includes title, abstract, authors, DOI, journal, publisher, publication year, volume and pages, and citation count. Depending on the paper you also get study type, sample size, study count, study duration, population type (human or animal), preprint status, countries of study, institutions, influential citation count and a key takeaway.

Two opt-in extras:

- `include_full_text_chunks=true`: query-relevant excerpts from the paper's full text, labeled by section (paid plans). Useful as RAG context. [Full text in the API](https://docs.consensus.app/api-full-text).
- `include_semantic_score=true`: a relevance score for the top 50 results

## Filters

| What you want | Parameters |
|---------------|------------|
| A publication window | `year_min`, `year_max`, `month_min`, `month_max` |
| Specific study designs | `study_types` (`rct`, `meta-analysis`, `systematic review`, `cohort study`, `case-control study`, ... 19 in total) |
| Methodological rigor | `human`, `controlled`, `sample_size_min`, `duration_min`, `duration_max`, `exclude_preprints` |
| Journal quality | `sjr_min` / `sjr_max` (SJR quartile, 1 = top), `citation_min` |
| Medical and clinical focus | `medical_mode` (top medical journals and guidelines, ~8M documents), `clinical_guideline` |
| Scope | `domain` (`med`, `bio`, `chem`, `cs`, `psych`, `econ`, ...), `country`, `journal_name`, `publisher_name`, `open_access` |

Full parameter reference: [docs.consensus.app/api-reference/search](https://docs.consensus.app/api-reference/search)

## Plans, pricing and rate limits

| | Free | Pro / Teams | Deep | Enterprise |
|---|---|---|---|---|
| Included calls per month | 30 | 500 | 2,000 | Custom |
| Max papers per request | 20 | 300 | 750 | 1,000 |
| Pagination | No | Yes | Yes | Yes |
| Full-text excerpts | No | Yes | Yes | Yes |
| Rate limit | 1 req/s | 1 req/s | 1 req/s | Custom |

A call is one request per 100 papers returned (20 papers = 1 call, 150 papers = 2 calls). Paid plans can turn on additional usage at $0.05 per call. The API and the [Consensus MCP server](https://github.com/Consensus-NLP/consensus-mcp) share one monthly pool. Details: [API plans and access](https://docs.consensus.app/api-plans-and-access). Plan prices: [consensus.app/pricing](https://consensus.app/pricing/).

For production volume, custom rate limits, zero data retention and a DPA, [talk to us about Enterprise](https://consensus.app/home/api/).

## What developers build

- **RAG and LLM grounding.** Retrieve peer-reviewed papers and full-text passages to ground a chatbot or assistant in real research, with DOIs to cite.
- **AI agents and scientific agents.** Give an agent a literature search tool, or delegate the whole research task to a Thread. [Phylo](https://docs.consensus.app/api-get-started) calls the API from its agents to screen about 20,000 drug targets across hundreds of diseases.
- **Pharma and biotech.** Target and mechanism-of-action research, pharmacovigilance and literature monitoring, HEOR and payer evidence, medical information.
- **Literature review and systematic review tooling.** Search by study design, screen abstracts, build evidence tables and monitor new publications.
- **Citation checking.** Verify that the claims in a draft are supported by the papers it cites.

Worked examples: [docs.consensus.app/use-cases](https://docs.consensus.app/use-cases).

## How it compares

PubMed (E-utilities), Europe PMC, OpenAlex, Semantic Scholar and Crossref all have free APIs, and they're great for metadata, bibliometrics and keyword lookups. The Consensus API is built for a different job: answering research questions and grounding AI.

| | Consensus API | Free scholarly metadata APIs |
|---|---|---|
| Coverage | 400M+ articles across all fields, including all of PubMed | Varies: biomedical only (PubMed, Europe PMC) or all fields (OpenAlex, Semantic Scholar, Crossref) |
| Search | Semantic ranking tuned for research questions | Mostly keyword, Boolean or field queries |
| Evidence signals per paper | Study type, sample size, journal quartile, takeaway | Usually publication type and citation counts |
| Full text | Licensed and open-access full-text passages | Abstracts and metadata; open-access full text in some |
| Research agent | Threads API returns cited, written answers | No |
| Cost | Free tier, then plan-based | Free |

If you need raw metadata at very high volume, a free metadata API may be the better fit. If you need the right papers for a question, with evidence signals and full text an LLM can use, use Consensus. More detail: [Consensus API vs PubMed, Europe PMC, OpenAlex and Semantic Scholar](https://docs.consensus.app/api-comparison).

## Security and data

SOC 2 Type II, hosted on Google Cloud. Customer data is not used to train AI models, and the LLM providers Consensus uses retain no request data. The API searches only the public research index. Enterprise agreements can include zero data retention and a DPA. See [Security and data handling](https://docs.consensus.app/enterprise/security).

## FAQ

**What's the best API for searching scientific papers for a RAG pipeline?**
If you need relevance-ranked, peer-reviewed papers with full-text passages and evidence metadata in one call, use the Consensus API. It's built for grounding LLMs.

**Is there a PubMed API alternative with semantic search?**
Yes. Consensus covers all of PubMed plus the rest of the literature, and ranks results semantically. Filter with `domain=med` or `medical_mode=true` to stay biomedical.

**Can I filter by evidence quality or study design?**
Yes. Use `study_types` (RCT, meta-analysis, systematic review and 16 more), `sample_size_min`, `human`, `controlled` and `sjr_max` for journal quality.

**Does the API return study type and sample size for each paper?**
Yes, when the paper reports them, along with a takeaway, journal quartile and citation counts.

**Can I track new publications on a drug, target or indication?**
Yes. Run the same query on a schedule with `year_min` and `month_min` to catch new papers. See the [scheduled research pipeline](https://docs.consensus.app/use-cases/scheduled-research-pipeline) example.

**Does Consensus train on my queries?**
No. Customer data is not used to train AI models. Enterprise agreements can add zero data retention.

**Is this the same as Consensus (goconsensus.com)?**
No. This is Consensus, the AI academic search engine at consensus.app. It isn't related to the sales demo platform at goconsensus.com.

## Consensus MCP

Want Consensus inside Claude, ChatGPT, Cursor or Microsoft 365 Copilot instead of your own code? The Consensus MCP server gives any MCP client the same search over a remote connection:

```
https://mcp.consensus.app/mcp
```

See the [consensus-mcp](https://github.com/Consensus-NLP/consensus-mcp) repo and the [MCP guide](https://docs.consensus.app/consensus-mcp).

---

Made by [Consensus](https://consensus.app), the AI search engine for research. Follow us on [X](https://x.com/ConsensusNLP) and [LinkedIn](https://www.linkedin.com/company/consensus-nlp).
