Central Asia–China Chromium Supply Chain Research

Research code for a paper on China–Kazakhstan chromium supply chains, submitted to CAPS Unlock (Central Asia Policy Studies).

Research questions:

RQ1: Industrial dependency structures between Kazakhstan and China
RQ2: Post-2022 geopolitical fragmentation effects on the supply chain
RQ3: Cross-lingual framing divergence — does Chinese-language discourse securitize chromium access more than English-language discourse frames it in economic/trade terms?
What's in this repo
notebooks/framing_classifier.ipynb — bilingual zero-shot framing classifier (English vs. Chinese sources) using mDeBERTa-v3-base-mnli-xnli. Extracts and chunks PDF text with pdfplumber, filters chunks with a chromium-specific keyword list, classifies each chunk as security-framed vs. economic-framed.
notebooks/topic_modeling.ipynb — BERTopic pipeline using paraphrase-multilingual-mpnet-base-v2 embeddings to find cross-lingual topic clusters.
notebooks/supply_chain_network.ipynb — dependency mapping / HHI concentration index stub (USGS production data; Kazakhstan export destination data pending from ERG annual reports or UN Comtrade HS code 2610).
data/ — processed outputs only (classification results, topic tables). Raw PDFs are NOT stored here — see below.
The literature corpus (PDFs)

The raw PDF corpus (English + Chinese sources) is not in this repo — journal articles are copyrighted, and GitHub isn't an appropriate place to redistribute them.

The corpus lives in Google Drive: chromium_project/pdfs/. Ask Milana to share that folder with you directly.

To run the notebooks against the real corpus:

Mount your own Google Drive in Colab (from google.colab import drive; drive.mount('/content/drive'))
Point the notebook's PDF_DIR variable at wherever you've placed the shared pdfs/ folder
Chinese-language PDFs from CNKI go in the same folder, tagged/named consistently (see naming convention below)

File naming convention: [lang]_[shortcite].pdf, e.g. en_brautigam2023.pdf, zh_wang2022.pdf — the pipeline uses the en_/zh_ prefix to route documents to the right language model.

Running the code

Everything was built and tested in Google Colab (T4 GPU runtime). Recommended path:

Open the notebook in Colab (upload from this repo, or open directly from GitHub via Colab's "File → Open notebook → GitHub" and paste this repo URL)
Mount Drive and set PDF_DIR as above
Run cells top to bottom — model weights (mDeBERTa, sentence-transformers) download automatically from Hugging Face on first run

To run locally instead of Colab:

bash
pip install -r requirements.txt

Then run the notebooks with Jupyter. A GPU is recommended for the classifier and embeddings step but not strictly required — it'll just be slower on CPU.

Outputs
data/framing_results.csv — per-chunk classification (security vs. economic framing), by language
data/topic_info.csv, data/topic_by_lang.csv — BERTopic outputs
Current headline result: English sources ~22% security framing vs. Chinese sources ~31%, consistent with the RQ3 hypothesis. Manual validation on a 20-chunk sample showed ~90% agreement with the classifier. Known limitation: implicit security framing (e.g. "managing risk," "dispersed patterns") is systematically under-detected by the zero-shot classifier — see the methodology/limitations section of the paper draft for full discussion.
Notes for collaborators
Use Google Sheets, not Excel, for editing any bilingual (Chinese-character) data — Excel corrupts UTF-8 Chinese text on save.
Citation management is in Zotero (Google Docs plugin) — avoid triple-clicking sources in the citation dialog, it's a known bug that duplicates entries.
