# China-Kazakhstan Chromium Supply Chains: Bilingual Text Analysis

Code for the text-analysis part of a research project on China's dependence on Kazakhstan for chromium, carried out under the CAPS Unlock research fellowship.

The notebooks take a set of English and Chinese research papers on chromium, critical minerals and Central Asia, and compare what the two literatures focus on, using topic modeling (BERTopic, LDA), TF-IDF and collocation analysis.

**Paper:** *China's Chromium Dependency on Kazakhstan: Computational Mapping of China–Kazakhstan Chromium Supply Chains After 2022*

## Research questions

The project asks three questions. This repo covers the third.

1. How dependent are China and Kazakhstan on each other in the chromium supply chain?
2. How did the geopolitical changes after 2022 affect that supply chain?
3. Do Chinese-language and English-language sources talk about chromium and critical minerals differently?

## What is in this repo

| File | Corpus | What it does |
| --- | --- | --- |
| `CA_China_new_analysis.ipynb` | Full corpus, 67 PDFs | Main analysis. Treats each paper as one document and runs BERTopic, TF-IDF, collocation analysis and LDA separately for each language. |
| `topic_modeling.ipynb` | Earlier set of 14 PDFs | Splits papers into short passages and runs one BERTopic model over both languages at once, to see whether topics cross the language boundary. |

Both notebooks include the saved outputs from their last run. The source PDFs are not included because they are copyrighted journal articles.

## Main analysis: `CA_China_new_analysis.ipynb`

### Corpus

67 PDF files: 34 classified as English and 33 as Chinese.

Language is detected automatically from the text: a paper counts as Chinese if at least 30% of the letters in its first 10,000 characters are Chinese characters.

### Method

1. **Extract text** from each PDF with PyMuPDF.
2. **Detect language** and split the corpus into an English set and a Chinese set.
3. **Prepare the text.** English: remove NLTK stopwords, and for collocations also remove punctuation, numbers and very short words. Chinese: segment into words with `jieba`, since Chinese is written without spaces.
4. **BERTopic** on each language, with `paraphrase-multilingual-MiniLM-L12-v2` embeddings.
5. **TF-IDF** to find the most characteristic terms in each language.
6. **Collocations**: two-word phrases ranked by pointwise mutual information (PMI) and by likelihood ratio, using NLTK.
7. **LDA** with 3 topics per language, using scikit-learn.

### Results

**BERTopic.** With default settings the English papers formed no topics at all. After removing stopwords and lowering the minimum topic size to 3, each language produced two topics:

| Language | Topic (top words) | Papers |
| --- | --- | --- |
| English | China, critical minerals, supply | 22 |
| English | Kazakhstan, raw-material trade statistics | 10 |
| English | not assigned | 2 |
| Chinese | critical minerals, United States, supply chain, security, strategy | 14 |
| Chinese | economy, risk, suppliers, procurement | 14 |
| Chinese | not assigned | 5 |

**Top TF-IDF terms.**

| Rank | English | Chinese |
| --- | --- | --- |
| 1 | china | 矿产 (minerals) |
| 2 | critical | 中国 (China) |
| 3 | kazakhstan | 铬矿 (chromite ore) |
| 4 | minerals | 资源 (resources) |
| 5 | central | 关键 (critical) |
| 6 | mineral | 中亚 (Central Asia) |
| 7 | supply | 哈萨克斯坦 (Kazakhstan) |
| 8 | mining | 合作 (cooperation) |
| 9 | asia | 我国 ("our country") |
| 10 | 2025 | 国家 (country) |

**Strongest collocations by likelihood ratio.**

- English: central asia, critical minerals, united states, raw materials, rare earth, supply chain
- Chinese: 关键矿产 (critical minerals), 一带一路 (Belt and Road), 新能源汽车 (new energy vehicles), 中亚五国 (the five Central Asian countries), 供应风险 (supply risk), 地缘政治 (geopolitics), 关键金属 (critical metals)

**LDA topics (top words).**

| | English | Chinese |
| --- | --- | --- |
| Topic 1 | central, kazakhstan, asia, critical, global, eu, foreign | suppliers, procurement, risk, supply, new energy, management, chromite ore |
| Topic 2 | mining, secondary, primary, processed, steel, iron, materials | economy, minerals, trade, Central Asia, cooperation, risk, global |
| Topic 3 | critical, resource, global, mining, materials, lithium, rare, risk | resources, demand, "our country", world, strategy, reserves |

### What the results show

- Both literatures share a core vocabulary: China, Kazakhstan, Central Asia, critical minerals, supply.
- Chromite ore is the third most characteristic term in the Chinese papers. No chromium term is in the English top 10, where the vocabulary is about critical minerals in general.
- The Chinese papers pair these subjects with the Belt and Road, new energy vehicles and supply risk. The English papers pair them with the United States, rare earths and raw materials.
- The Chinese corpus contains a group of company-level studies on chromite procurement and supplier management (14 papers in BERTopic, and LDA topic 1). Nothing similar shows up in the English topics.

## Earlier run: `topic_modeling.ipynb`

This notebook tests a different design on a smaller, earlier set of 14 papers (9 English, 5 Chinese).

- Papers are split into chunks of up to 800 characters, cut at sentence boundaries: 1,320 chunks in total (1,076 English, 244 Chinese).
- All chunks from both languages are embedded with `paraphrase-multilingual-mpnet-base-v2` and clustered in **one** BERTopic model, so a topic can contain passages from both languages.
- Topic labels use a custom tokenizer (`jieba` for Chinese, whitespace for English), because BERTopic's default tokenizer leaves Chinese topics with empty labels.
- UMAP uses a fixed random seed, so the topics are the same on every run.

BERTopic found 29 topics, and 298 chunks (23%) were not assigned to any topic. **The topics are mostly split by language.** Only 3 of the 29 hold at least 1% of the chunks in both languages:

| Topic (top words) | English share | Chinese share |
| --- | --- | --- |
| chromium, China, chromite, resources | 3.9% (42 chunks) | 2.0% (5 chunks) |
| energy, renewable, demand | 1.6% (17 chunks) | 3.3% (8 chunks) |
| China, Central Asia | 1.2% (13 chunks) | 1.6% (4 chunks) |

## Limitations

- **Few documents for topic modeling.** The main analysis uses whole papers as documents, so each language has only about 33. BERTopic needs more than that: it found no English topics with default settings and only two per language after tuning.
- **Topics are not matched across languages.** The main analysis fits a separate model for each language, so the English and Chinese topics are compared by reading them side by side, not by a shared model.
- **Language split in the shared model.** In the earlier run the topics separated by language. Multilingual embeddings still carry information about which language a text is in, so this split does not prove the two literatures discuss different things.
- **PDF extraction noise.** Words broken by line-end hyphens, reference lists, URLs and table fragments get into the results. The PMI collocation ranking is dominated by this noise, which is why the likelihood-ratio ranking is reported above. Years and "https" appear among the top terms.
- **Corpus cleaning.** Two Chinese papers appear in the folder twice under slightly different filenames. One PDF with a Chinese title was classified as English by the language detector.
- **Uneven preprocessing.** English text has stopwords removed and Chinese text does not, because no Chinese stopword list is applied.
- **No quality scores.** Topic coherence was not measured, and the number of LDA topics (3) was chosen by hand.

Planned fixes: remove duplicate files, strip reference sections and URLs before analysis, add a Chinese stopword list, run the full corpus at chunk level so there are enough documents for BERTopic, and report topic coherence.

## How to run

Both notebooks were built and run in Google Colab and read PDFs from Google Drive.

**`CA_China_new_analysis.ipynb`**

1. Put all PDFs in one Drive folder and set `PDF_FOLDER` to its path. English and Chinese files can be mixed.
2. Run the install and import cells, then the cells that define `extract_pdf_text` and build `papers_df`, and then the rest. The two cells that add and print the `language` column sit above the cell that builds `papers_df`, so they must be run after it.

Dependencies: `bertopic`, `sentence-transformers`, `pymupdf`, `jieba`, `nltk`, `scikit-learn`, `pandas`. No GPU is needed.

**`topic_modeling.ipynb`**

1. Put the PDFs in Drive in this structure and change `BASE` if your folder is somewhere else:

   ```
   chromium_project/
   ├── pdfs/
   │   ├── en/    English PDFs
   │   └── zh/    Chinese PDFs
   └── data/      outputs are written here
   ```

2. Run the first two cells. A GPU runtime makes the embedding step faster.

Dependencies: `pdfplumber`, `sentence-transformers`, `bertopic`, `umap-learn`, `hdbscan`, `jieba`. The notebook pins `pillow==10.4.0`, which makes pip print a version warning for `pdfplumber`; the run still completes.

Outputs written to `data/`: `topic_modeling_chunks.csv` (every chunk with its file, language and topic), `topic_modeling_topics.csv` (topic list with sizes and top words), `topic_by_language.csv` (share of each language's chunks in each topic).

## Other parts of the project

These are part of the same research but their code is not in this repo yet:

- **Framing classifier.** A zero-shot classifier (`mDeBERTa-v3-base-mnli-xnli`) that labels each chromium-related passage as security-framed or economic-framed.
- **Supply chain concentration.** Herfindahl–Hirschman Index and trade network analysis on UN Comtrade data.

## Author

Milana Pak, Kazakh-British Technical University. Research conducted with a co-researcher under the CAPS Unlock fellowship.
