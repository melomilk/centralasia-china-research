# China–Kazakhstan Chromium Supply Chains: Bilingual Topic Modeling

Code for the text-analysis part of a research project on China's dependence on Kazakhstan for chromium, carried out under the CAPS Unlock research fellowship.

The notebook in this repo takes a set of English and Chinese research papers, splits them into passages, and uses BERTopic with multilingual sentence embeddings to find out which topics each language's literature focuses on.

**Paper:** *China's Chromium Dependency on Kazakhstan: Computational Mapping of China–Kazakhstan Chromium Supply Chains After 2022*

## Research questions

The project asks three questions. This repo covers the third.

1. How dependent are China and Kazakhstan on each other in the chromium supply chain?
2. How did the geopolitical changes after 2022 affect that supply chain?
3. Do Chinese-language and English-language sources talk about chromium and critical minerals differently?

## What is in this repo

| File | What it does |
| --- | --- |
| `topic_modeling.ipynb` | Full pipeline: PDF text extraction, chunking, multilingual embeddings, BERTopic, topic share by language. Saved outputs from the last run are included in the notebook. |

The source PDFs are not included because they are copyrighted journal articles.

## How the pipeline works

1. **Extract text** from each PDF with `pdfplumber`.
2. **Split into chunks** of up to 800 characters, cut at sentence boundaries. The splitter handles both English and Chinese punctuation (`.` `!` `?` `。` `！` `？`).
3. **Embed every chunk** with `paraphrase-multilingual-mpnet-base-v2`. This model places English and Chinese text in the same vector space, so passages on the same subject can end up in the same topic regardless of language.
4. **Cluster with BERTopic** (UMAP for dimensionality reduction, HDBSCAN for clustering, minimum topic size 8). UMAP uses a fixed random seed so the topics are the same on every run.
5. **Label topics** with a custom tokenizer: `jieba` for Chinese, whitespace split for English. BERTopic's default tokenizer splits on spaces, which does not work for Chinese and leaves Chinese topics with empty labels.
6. **Compare languages:** compute what share of each language's chunks falls into each topic.

All chunks are used, not only those that mention chromium, because the goal is to see what each literature talks about overall.

## Results from the saved run

Corpus in this run:

| | English | Chinese | Total |
| --- | --- | --- | --- |
| Documents | 9 | 5 | 14 |
| Chunks | 1,076 | 244 | 1,320 |

BERTopic found 29 topics. 298 chunks (23%) were not assigned to any topic.

**Topics are mostly split by language.** Only 3 of the 29 topics hold at least 1% of the chunks in both languages:

| Topic (top words) | English share | Chinese share |
| --- | --- | --- |
| chromium, China, chromite, resources | 3.9% (42 chunks) | 2.0% (5 chunks) |
| energy, renewable, demand | 1.6% (17 chunks) | 3.3% (8 chunks) |
| China, Central Asia | 1.2% (13 chunks) | 1.6% (4 chunks) |

**What each language's sources focus on:**

- **English sources:** critical mineral resources and deals (15.0% of English chunks), trade and political alignment between countries (7.6%), international relations theory (6.7%), land rights (5.0%), and several Kazakhstan-specific topics (foreign policy hedging, the period after 2022, the Eurasian Economic Union).
- **Chinese sources:** reserve and production figures in tonnes (10.2% of Chinese chunks), national security strategy and US policy toward Central Asia (8.2%), supply diversification and the US Defense Production Act (6.6%), C5+1 diplomacy (3.7%), and production capacity in other supplier countries such as Australia, Indonesia and Chile (2.5%).

In this corpus, the Chinese sources spend more of their text on supply figures and great-power strategy, while the English sources spend more on trade, deals and Kazakhstan's own position.

## Limitations

- **Small and unbalanced corpus.** 14 documents, and the Chinese side has only 244 chunks, so one chunk is 0.4% of the Chinese total. The shares above describe these documents, not the two literatures in general.
- **The language split may be partly a model effect.** Multilingual embeddings still carry information about which language a text is in, so chunks can cluster by language even when the subject is similar. The split should not be read as proof that the two literatures discuss different things.
- **Noisy topics.** Judging by their top words, 9 of the 29 topics (173 chunks) are PDF extraction noise, not real subjects: URLs and reference lists, regression table fragments, text extracted in reverse order from one PDF, and full-width Latin characters from the Chinese PDFs.
- **No stopword removal.** Many English topic labels are dominated by words like "the", "of" and "and".
- **No topic quality score.** Coherence was not measured, and topics were not manually validated.

Planned fixes: strip reference sections and URLs before chunking, normalize full-width characters, add English and Chinese stopword lists, and report topic coherence.

## How to run

The notebook was built and run in Google Colab with a T4 GPU.

1. Put the PDFs in Google Drive in this structure:

   ```
   chromium_project/
   ├── pdfs/
   │   ├── en/    English PDFs
   │   └── zh/    Chinese PDFs
   └── data/      outputs are written here
   ```

2. Open `topic_modeling.ipynb` in Colab and change `BASE` if your folder is somewhere else.
3. Run the first two cells. Model weights download from Hugging Face on the first run.

Dependencies installed by the notebook: `pdfplumber`, `sentence-transformers`, `bertopic`, `umap-learn`, `hdbscan`, `jieba`, `transformers`, `torch`. The notebook pins `pillow==10.4.0`, which makes pip print a version warning for `pdfplumber`; the run still completes.

Outputs written to `data/`:

| File | Contents |
| --- | --- |
| `topic_modeling_chunks.csv` | Every chunk with its source file, language and assigned topic |
| `topic_modeling_topics.csv` | Topic list with sizes and top words |
| `topic_by_language.csv` | Share of each language's chunks in each topic |

## Other parts of the project

These are part of the same research but their code is not in this repo yet:

- **Framing classifier.** A zero-shot classifier (`mDeBERTa-v3-base-mnli-xnli`) that labels each chromium-related passage as security-framed or economic-framed.
- **Supply chain concentration.** Herfindahl–Hirschman Index and trade network analysis on UN Comtrade data.

## Author

Milana Pak, Kazakh-British Technical University. Research conducted with a co-researcher under the CAPS Unlock fellowship.
