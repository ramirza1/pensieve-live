# 🧠 Pensieve

A semantic search tool for academic papers and research notes, powered by ChromaDB and OpenAI embeddings.

Just like Dumbledore stored memories in the Pensieve, this tool helps retrieve knowledge from notes and readings, making your research instantly searchable and intelligently summarized.

**Live App:** [https://pensieve.rehanmirza.net]

---

## ✨ Features

- **Semantic Search**: Find relevant content across all your notes and papers using natural language queries
- **Auto-Summarization**: LLM-generated summaries for each paper and note section
- **AI Query Snippets**: On-demand, query-focused insights that explain how each result relates to your search
- **Incremental Indexing**: Only processes new or changed files — fast weekly updates
- **Cloud Sync**: Automatically deploys to Streamlit Cloud via Backblaze B2

---

## 📁 Organizing Your Content

### Notes (`data/inbox/notes/`)

Save your research notes as `.docx` files. Structure them as follows:
```
data/inbox/notes/
├── Experimental Methods/
│   ├── Causal Inference Notes.docx
│   └── Survey Design Notes.docx
├── Political Communication/
│   └── Media Effects Notes.docx
└── Literature Reviews/
    └── Deliberation Quality.docx
```

**Within each notes file:**

| Heading Level | Purpose | Example |
|---------------|---------|---------|
| **Heading 2** | Topic/Theme | `## Causal Inference` |
| **Heading 3** | Paper reference | `### "Why Do Humans Reason?", Mercier & Sperber, 2011` |
| Body text | Your notes on that paper | Bullet points, summaries, quotes |

**Example structure inside a .docx file:**
```
## Internal Validity                          ← Heading 2 (Theme)

### "Experimental Design", Shadish et al., 2002   ← Heading 3 (Paper)
- Key points about threats to validity
- Notes on randomization

### "Causal Inference", Pearl, 2009              ← Heading 3 (Paper)  
- DAGs and counterfactuals
- Do-calculus basics

## External Validity                          ← Heading 2 (Theme)
...
```

### Papers (`data/inbox/papers/`)

Save PDF versions of papers you read:
```
data/inbox/papers/
├── Mercier_Sperber_2011_Argumentative_Theory.pdf
├── Pearl_2009_Causality.pdf
└── subfolder/
    └── Another_Paper.pdf
```

The system automatically extracts metadata (title, authors, year) from PDFs via Crossref lookup.

---

## 🚀 Running the Pipeline

All commands run from the main Pensieve directory:
```powershell
cd "C:\Users\...\Pensieve"
.\venv\Scripts\Activate
```

### Full Pipeline (Index → Summarize → Deploy)
```powershell
python scripts/update_and_deploy.py
```

Runs all steps:
1. Index new/changed notes into ChromaDB
2. Index new/changed papers into ChromaDB
3. Generate LLM summaries for new content
4. Upload ChromaDB to Backblaze B2 (incremental)

### Command Options

| Command | Description |
|---------|-------------|
| `python scripts/update_and_deploy.py` | Full pipeline |
| `python scripts/update_and_deploy.py --upload-only` | Just upload to B2 (skip indexing) |
| `python scripts/update_and_deploy.py --skip-upload` | Index locally only (no deploy) |
| `python scripts/update_and_deploy.py --notes-only` | Only process notes |
| `python scripts/update_and_deploy.py --papers-only` | Only process papers |
| `python scripts/update_and_deploy.py --cleanup` | Remove orphaned entries first |
| `python scripts/update_and_deploy.py --sync-deletions` | Also delete orphaned B2 files |
| `python scripts/update_and_deploy.py --dry-run` | Preview changes without executing |
| `python scripts/update_and_deploy.py --force` | Force full reprocess (ignore cache) |

### Example Workflows

**Weekly update (most common):**
```powershell
python scripts/update_and_deploy.py
```

**After deleting files from inbox:**
```powershell
python scripts/update_and_deploy.py --cleanup --sync-deletions
```

**Preview what would change:**
```powershell
python scripts/update_and_deploy.py --dry-run
```

**Just test indexing locally:**
```powershell
python scripts/update_and_deploy.py --skip-upload
```

---

## 🔍 Using the App

### Search

1. Enter a topic or question in the search bar
   - Example: `"motivated reasoning"`, `"threats to internal validity"`, `"deliberation quality"`
2. Toggle **📝 Notes** and/or **📄 Papers** to filter results
3. Adjust **Number of results** (1-25) as needed

### Understanding Results

**For Notes:**
| Field | Description |
|-------|-------------|
| **Theme** | Your Heading 2 topic (e.g., "Internal Validity") |
| **Title/Authors/Year** | Paper info from your Heading 3 |
| **Summary** | Auto-generated LLM summary of that section |

**For Papers:**
| Field | Description |
|-------|-------------|
| **Title/Authors/Year** | Extracted from PDF metadata |
| **Summary** | Auto-generated LLM summary of the full paper |

### AI Query Snippets

Toggle **✨ AI snippets** to enable query-focused insights.

For each result, click **"Generate / refresh"** to get:
- A direct answer explaining how this content relates to your query
- 3-6 specific bullet points of relevant insights
- Confidence rating (High/Medium/Low)

> Note: AI snippets call the OpenAI API on-demand, so they take a few seconds to generate.

### Additional Controls

- **🌗** Toggle light/dark mode
- **🐛** Toggle debug mode (shows chunk IDs, distances, metadata)

---

## 🏗️ Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     YOUR LOCAL MACHINE                          │
├─────────────────────────────────────────────────────────────────┤
│  data/inbox/notes/*.docx  ──┐                                   │
│  data/inbox/papers/*.pdf  ──┼──► Indexing Scripts ──► ChromaDB  │
│                             │         │                         │
│                             │         ▼                         │
│                             │    LLM Summaries                  │
│                             │         │                         │
│                             │         ▼                         │
│                             └──► Upload to B2                   │
└─────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                     BACKBLAZE B2                                │
│                 (Cloud ChromaDB Storage)                        │
└─────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                   STREAMLIT CLOUD                               │
│          Downloads DB from B2 → Serves App                      │
│          https://pensieve-live.streamlit.app                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

- **Frontend**: Streamlit
- **Vector Database**: ChromaDB
- **Embeddings**: OpenAI `text-embedding-3-small`
- **Summarization**: OpenAI `gpt-4.1-mini`
- **Cloud Storage**: Backblaze B2
- **Hosting**: Streamlit Cloud

---

## 📋 Requirements

- Python 3.10+
- OpenAI API key
- Backblaze B2 account (for deployment)

### Local Development
```powershell
# Clone and setup
git clone https://github.com/ramirza1/pensieve-live.git
cd pensieve-live
python -m venv venv
.\venv\Scripts\Activate
pip install -r requirements.txt

# Add your API key
echo "OPENAI_API_KEY=sk-your-key" > .env

# Run locally
streamlit run app/streamlit_app.py
```

---

## 📝 Tips

- **Consistent formatting**: Use Heading 2/3 consistently in your notes for best results
- **Paper titles in H3**: Include full citation info for better metadata display
- **Folder organization**: Group related notes into folders — they appear in search results
- **Regular updates**: Run the pipeline weekly to keep your search index fresh
- **Cleanup after deletions**: Use `--cleanup --sync-deletions` after removing files

---

Made by Rehan Mirza
