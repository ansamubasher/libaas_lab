# Libaas Lab: Roman Urdu Fashion Discovery System

An AI-assisted fashion discovery system for Pakistani shoppers. It matches them to ethnic wear products, tags items by color/tone, checks current trend signals, and can generate a short spoken editorial summary about a style or trend.

**Phase 1 (this repo):** data cleaning, EDA, insights and preprocessing. Modeling comes in Phase 2.

---

## 1. Problem Statement

Online fashion shopping in Pakistan lacks tools that understand local-language queries, regional style preferences and current trends together in one place. Shoppers often search in Roman Urdu, but most catalog and recommendation systems are built for English queries and international markets.

This project builds a pipeline that combines several NLP and speech models to make fashion search more natural and locally relevant.

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

## 3. Repository Structure

```
.
├── data/
│   ├── raw/                 # original data (trends.csv; large datasets are downloaded here, see Section 4)
│   └── processed/           # cleaned / preprocessed outputs created by the notebooks
├── images/
│   ├── trends/              # graphs from the Google Trends notebooks
│   └── indofashion/         # graphs from the IndoFashion notebook
├── notebooks/
│   ├── 01_trends_cleaning.ipynb
│   ├── 02_trends_eda.ipynb
│   ├── 03_trends_preprocessing.ipynb
│   └── 04_indofashion_cleaning_eda.ipynb
├── requirements.txt
└── README.md
```

---

## 4. Datasets

| Dataset | Used for | Phase 1 status | Link |
|---|---|---|---|
| Google Trends "Year in Search" (2001-2020) | Trend signals by country / category | **Cleaned, EDA, preprocessed** | https://www.kaggle.com/datasets/dhruvildave/google-trends-dataset (raw file included as `data/raw/trends.csv`) |
| IndoFashion | Ethnic-wear taxonomy, clothing category classification | **Cleaned, EDA, preprocessed** | https://www.kaggle.com/datasets/validmodel/indo-fashion-dataset |
| Pakistan's Largest E-commerce Dataset | Local pricing and category signal | Planned | https://www.kaggle.com/datasets/zusmani/pakistans-largest-ecommerce-dataset |
| H&M Personalized Fashion Recommendations | Color/tone tagging | Planned | https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations |
| Women's E-Commerce Clothing Reviews | Review/editorial summarization | Planned | https://www.kaggle.com/datasets/nicapotato/womens-ecommerce-clothing-reviews |
| Myntra Products | Product names, prices, brands | Planned | TODO: add link |
| Synthetic Roman Urdu queries | Query understanding | Planned | TODO: add generation script |

IndoFashion is too large for GitHub (106k images). Download it from the link above and put `train_data.json`, `val_data.json` and `test_data.json` in `data/raw/indofashion/` (images are not needed for Phase 1).

### Dataset details

**Google Trends "Year in Search"**
- **Source:** Google Trends yearly top searches, via Kaggle (link above)
- **Size:** 26,955 rows x 5 columns (raw)
- **Features:** `location` (Global + 82 countries), `year` (2001-2020), `category` (e.g. "Consumer Brands", "People"), `rank` (1-5), `query` (the search term)
- **Target variable:** none fixed; `rank` / trend popularity per category and country is the signal used for trend forecasting in Phase 2
- **Used for:** trend signals and forecasting

**IndoFashion**
- **Source:** IndoFashion dataset (Kaggle, link above)
- **Size:** 106,166 rows (train + val + test) x 6 columns, with one image per row
- **Features:** `image_url`, `image_path`, `brand`, `product_title`, `color` (95% missing)
- **Target variable:** `class_label` (15 ethnic-wear classes, e.g. saree, women_kurta, lehenga, sherwanis)
- **Used for:** clothing-category classification

---

## 5. Data Cleaning Summary

**Google Trends** (`notebooks/01_trends_cleaning.ipynb`)
- Missing values: none in the raw file; text columns are still filled with a placeholder so the notebook is safe on new data
- Format fixes: column names lower-cased; extra / double spaces trimmed in all text columns; locations title-cased; lower-case `category_clean` column added because the same category appears with different capitalisation
- Duplicates removed: 10 exact duplicate rows (checked *after* trimming spaces)
- Invalid values: only years 2001-2025 and ranks 1-5 kept
- Outliers: queries longer than the IQR limit (29 characters, 911 queries) are **flagged** in `is_length_outlier`, not removed, because they are real search terms
- Scaling/encoding (`03_trends_preprocessing.ipynb`): `year` Min-Max scaled; `query_len` and `query_word_count` log-transformed then standardised; `location` and `script` label-encoded; `category_clean` frequency-encoded (2,366 categories); `rank` left unscaled (ordinal, possible target)
- Rows: 26,955 raw -> 26,945 clean

**IndoFashion** (`notebooks/04_indofashion_cleaning_eda.ipynb`)
- Missing values: placeholders (`NA`, `None`, `-`, empty string) converted to NaN; rows missing `class_label` or `product_title` dropped; missing `brand` filled with `unknown`; `color` dropped (95% missing)
- Format fixes: titles, brands and labels lower-cased; special characters removed from titles; repeated spaces collapsed
- Duplicates removed: 2,894 rows with a repeated image URL, including 723 images that appeared in more than one split (train/test leakage); the test/val copy is kept
- Outliers: titles with fewer than 2 words removed; very long titles (above the IQR limit of 21 words) kept and flagged in `long_title_flag`
- Scaling/encoding: `class_label` -> `label_id`; `title_words` and `title_chars` log-transformed (`log1p`, to reduce right skew) then standardised with `StandardScaler` **fit on train only** and applied to val/test
- Rows: 106,166 raw -> ~96.8k clean (exact number is printed in the notebook's final-check cell)

---

## 6. Key Findings

**Google Trends**
- **Distributions:** coverage grows strongly over time; 2001-2010 have few entries, 2012-2020 have most of the data. Query length is right-skewed (most queries are short names of people, brands or events).
- **Geography:** the United States has the most entries, followed by Japan and Canada. Only ~4% of rows are "Global".
- **Class balance:** ranks 1-5 appear exactly equally, because every list is a top-5. `rank` is an ordering inside each list, not something to predict from metadata.
- **Correlations:** `year`, `query_len` and `is_global` have almost zero correlation with `rank`; the meaning of the query matters, not its length.
- **Scripts:** ~85% of queries are in Latin script, ~6% Chinese/Japanese/Korean, ~4% Cyrillic, plus Arabic, Thai, Hebrew and Greek; category names are multilingual too (e.g. `películas`); text models must handle more than English.
- **Implications for modeling:** category text is messy and very high-cardinality (2,366 values), so categories will need grouping (e.g. a fashion/clothing subset) before trend forecasting; recent years carry most of the signal.

**IndoFashion**
- **Class imbalance:** 15 classes, from ~11.7k (saree) down to ~2.5k (sherwanis), a 4.7x ratio. Phase 2 will use class weights / stratified sampling and report macro-F1, not only accuracy.
- **Splits:** val and test are nearly balanced (~500 per class); all imbalance is in train.
- **Data leakage:** 723 images were shared between splits; these were removed from train so test scores are honest.
- **Title length:** right-skewed; blouse, petticoat, dhoti and saree titles are longest (~10-12 words), palazzo, sherwani and men's kurta titles shortest (~7 words). `title_words` and `title_chars` are strongly correlated, so one is enough.
- **Vocabulary:** generic words (`women`, `s`, `with`, `cotton`) dominate and should be removed or down-weighted (TF-IDF); the class name itself is almost always among the top words for its class, so titles are a strong signal for category classification.
- **Brands:** 8,000+ unique brands, very long-tailed (`generic` and `unknown` largest); too high-cardinality to one-hot encode.
- **Color:** 95% missing, so color/tone tagging will rely on the H&M dataset or on images.

---

## 7. Visualizations

All graphs are shown in the notebooks and saved in `images/`. Each graph has a short "What the graph shows" note under it in its notebook.

| File | What it tells us |
|---|---|
| `images/trends/01_query_length_outliers.png` | Query length is right-skewed; the red line is the IQR outlier limit |
| `images/trends/02_entries_per_year.png` | Data coverage grows a lot after 2010 |
| `images/trends/03_top10_countries.png` | US, Japan and Canada have the most entries |
| `images/trends/04_top10_categories.png` | The most common (cleaned) trend categories |
| `images/trends/05_top10_global_queries.png` | Queries that appear most often in the global top-5 lists |
| `images/trends/06_correlation_heatmap.png` | Numeric metadata is almost uncorrelated with rank |
| `images/trends/07_before_after_scaling.png` | log1p + StandardScaler removes most of the skew in query length |
| `images/indofashion/01_missing_values.png` | `color` is ~95% missing; every other column is complete |
| `images/indofashion/02_title_length_outliers.png` | Title length is right-skewed; very short titles removed, long ones flagged |
| `images/indofashion/03a_class_distribution_after_cleaning.png` | Class counts after cleaning |
| `images/indofashion/03_class_distribution.png` | 15 classes, moderately imbalanced (saree ~11.7k vs sherwanis ~2.5k) |
| `images/indofashion/04_split_distribution.png` | Val/test are nearly balanced; all imbalance is in train |
| `images/indofashion/05_title_length_by_class.png` | Blouse/saree titles are longest, palazzo/sherwani shortest |
| `images/indofashion/06_top_words.png` | Generic words dominate titles and should be treated as stop words |
| `images/indofashion/07_top_brands.png` | 8,000+ brands, long-tailed (`generic` and `unknown` largest) |
| `images/indofashion/08_correlation.png` | `title_words` and `title_chars` are strongly correlated |
| `images/indofashion/09_before_after_scaling.png` | log1p + StandardScaler removes most of the skew in title length (created when the notebook is re-run) |

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
3. Google Trends: the raw file is already in `data/raw/trends.csv`.
   IndoFashion: download it (Section 4) and put the three JSON files in `data/raw/indofashion/`.
4. Open Jupyter **from the `notebooks/` folder** and run the notebooks in order:
   `01_trends_cleaning` -> `02_trends_eda` -> `03_trends_preprocessing` -> `04_indofashion_cleaning_eda`.
   Cleaned data is saved to `data/processed/` and graphs to `images/`.

> The IndoFashion notebook can also be run on Kaggle: add the IndoFashion dataset as an input and run all cells. The first code cell detects Kaggle and switches paths automatically; download the outputs from `/kaggle/working/` and copy them into `data/processed/` and `images/indofashion/`.

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
- Accuracy and macro-F1 for clothing category and color/tone tagging
- MAE/RMSE for trend forecasting
- Manual review of generated scripts and audio
- Query-to-result relevance checks for Roman Urdu searches
