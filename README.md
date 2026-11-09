# Fashion AI: Roman Urdu Fashion Discovery System

An AI-assisted fashion discovery system for Pakistani shoppers. It  matches them to ethnic wear products, tags items by color/tone, checks current trend signals, and can generate a short spoken editorial summary about a style or trend.

---

## 1. Problem Statement

Online fashion shopping in Pakistan lacks tools that understand local language queries, regional style preferences, and current trends together in one place. Shoppers often search in Roman Urdu, but most catalog and recommendation systems are built for English queries and international markets.

This project builds a pipeline that combines several NLP and speech models to make fashion search more natural and locally relevant. **Phase 1 (this repo)** covers data cleaning, EDA, insights and preprocessing. Modeling comes in Phase 2.

---

## 2. Planned Techniques

- Text classification for clothing category tagging
- Roman Urdu intent and query understanding
- Color/tone classification (pastel, bright, dark, neutral)
- Summarization of product reviews and editorial text
- Podcast-style script generation
- Speech-to-text and text-to-speech for a voice interface
- Time-series forecasting for regional fashion trends
- Optional image tagging with a vision-language model

---

## 3. Datasets

Datasets are too large to upload to GitHub. Download them from the links below and place them in `data/raw/`.

| Dataset | Used for | Owner | Link |
|---|---|---|---|
| IndoFashion | Ethnic wear taxonomy, clothing category classification | ___ | <link> |
| Myntra Products | Product names, prices, brands | ___ | <link> |
| Pakistan's Largest Ecommerce Dataset | Local pricing and category signal | ___ | <link> |
| H&M Personalized Fashion Recommendations | Color/tone tagging | ___ | <link> |
| Synthetic Roman Urdu queries | Query understanding | ___ | <link or generation script> |
| Women's E-Commerce Clothing Reviews | Review/editorial summarization | ___ | <link> |
| Google Trends (pytrends) | Trend forecasting | ___ | pulled with the `pytrends` library |

### Dataset details

Fill in one block per dataset:

**<Dataset name>**
- **Source:** ___
- **Size:** ___ rows / ___ images
- **Features:** ___
- **Target variable:** ___
- **Used for:** ___

---

## 4. Data Cleaning Summary

Fill in one block per dataset:

**<Dataset name>**
- Missing values: ___
- Duplicates removed: ___
- Outliers handled: ___
- Format fixes: ___
- Scaling/encoding: ___
- Rows: ___ raw -> ___ clean

---

## 5. Key Findings

Fill in one block per dataset:

**<Dataset name>**
- **Distributions:** ___
- **Class imbalance:** ___
- **Trends/correlations:** ___
- **Implications for modeling:** ___

---

## 6. Visualizations

All graphs are saved in `images/`.

| File | What it shows |
|---|---|
| `<filename>.png` | ___ |
| `<filename>.png` | ___ |

---

## 7. Repository Structure

```
.
├── README.md
├── requirements.txt
├── data/
│   ├── raw/          # downloaded datasets (not uploaded)
│   └── processed/    # cleaned datasets
├── notebooks/
│   ├── <name>_cleaning.ipynb
│   ├── <name>_eda.ipynb
│   └── <name>_preprocessing.ipynb
└── images/           # saved graphs
```

---

## 8. How to Run

1. Clone the repo:
   ```bash
   git clone <your-repo-url>
   cd <repo-name>
   ```
2. Install the libraries:
   ```bash
   pip install -r requirements.txt
   ```
3. Download the datasets using the links in Section 3 and put them in `data/raw/`.
4. Open the notebooks in `notebooks/` and run them in order (cleaning, then EDA, then preprocessing). Cleaned data is saved to `data/processed/`.

> Some notebooks were run on Kaggle. If you run them there, add the dataset as an input and change the `DATA_DIR` path in the first cell.

---

## 9. Planned Models and Evaluation (Phase 2)

| Task | Model |
|---|---|
| Roman Urdu intent understanding | xlm-roberta-base / bert-base-multilingual-cased |
| Clothing category classification | distilbert-base-uncased |
| Summarization | bart-large-cnn / pegasus-xsum |
| Podcast script generation | GPT-4o-mini / Llama-3-8B-Instruct |
| Text-to-speech | Coqui TTS / facebook/mms-tts |
| Speech-to-text | Whisper (small/medium) |
| Trend forecasting | Prophet / pretrained TFT |
| Optional image tagging | CLIP (clip-vit-base-patch32) |

**Evaluation:**
- Accuracy for clothing category and color/tone tagging
- MAE/RMSE for trend forecasting
- Manual review of generated scripts and audio
- Query-to-result relevance checks for Roman Urdu searches
