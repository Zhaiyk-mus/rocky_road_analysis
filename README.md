# Ozon Data Intelligence Pipeline: From Scraping to Predictive Modeling

This project represents the full data lifecycle for the **Ozon** marketplace: from bypassing advanced anti-bot systems and data scraping to deep cleaning, feature engineering, and building predictive models.

## 🛠 2. Tech Stack
* **Data Sourcing:** Python, Selenium, `undetected-chromedriver`, `requests`.
* **Data Processing:** Pandas, NumPy.
* **Feature Engineering & Modeling:** Scikit-learn, XGBoost, Random Forest.
* **Visualization:** Matplotlib, Seaborn.

---

## 🚀 Quick Start

### 1. Install Dependencies
Ensure you have Python 3.9+ installed and run the following command:
```bash
pip install -r requirements.txt
```

### 2. Parser Configuration (Bypassing Protection)
Ozon utilizes robust anti-fraud systems. To prevent the script from getting banned instantly, you must mimic a real user session by extracting your personal cookies and headers:

1. Open [Ozon](https://www.ozon.ru) in your browser.
2. Open Developer Tools (**F12**) and navigate to the **Network** tab.
3. Filter the requests by **XHR**.
4. Look for a request where the URL contains `pixis` (this is Ozon's internal API for fetching product data).
5. Right-click on the request -> **Copy** -> **Copy as cURL (bash)**.
6. Go to [convertcurl.com](https://convertcurl.com/) and paste your copied cURL command.
7. Extract the `cookies` and `headers` from the output and paste them into your parser's configuration file or environment variables.

### 3. Why Undetected Chromedriver?
We use the `undetected-chromedriver` library because standard Selenium leaves specific "fingerprints" (webdriver flags) that are immediately flagged and banned by Ozon and Cloudflare. This library patches the Chrome binary on the fly, allowing us to bypass automation detection and scrape safely.

---

## 🔄 Data Lifecycle (Workflow)

### Stage 1: Data Scraping (Parsing)
The scraping process is split into two specialized scripts located in `src/parser/` for maximum speed and stealth:
1. **`get_links.py`**: First, run this script to bypass Cloudflare/Ozon protections using `undetected_chromedriver`. It scrolls through paginated categories (Laptops, Smartphones, Tablets) and collects raw product URLs.
2. **`fetch_cards.py`**: Next, execute this script to iterate through the collected URLs. 
   * *Note:* We do not parse heavy HTML blocks. Instead, the script uses `requests` to intercept Ozon's **internal JSON API** (`entrypoint-api.bx`), extracting deep, structured hardware specifications (`webCharacteristics`).

### Stage 2: Data Cleaning
Open the main **`all.ipynb`** notebook to begin the processing pipeline:
* Extracts and flattens nested JSON data from the raw API responses.
* Drops rows with critical missing target variables (e.g., missing Price).
* Cleans text formatting (e.g., stripping "ГБ" or "Гц" from numeric strings and converting them to float/int types).
* Imputes missing secondary specifications using median values.

### Stage 3: Feature Engineering
Continued within **`all.ipynb`**:
* Standardizes column names by mapping Russian specification titles to unified English keys (`cpu_freq`, `ram`, `storage_ssd`).
* Applies **One-Hot Encoding** and **Label Encoding** for categorical features (Brands, Processors, OS).
* Normalizes continuous variables using **StandardScaler** to prepare the data for machine learning algorithms.

### Stage 4: Visualization, EDA & Hypothesis Testing
* Generates `matplotlib` and `seaborn` visualizations to analyze price distributions across different tech categories.
* Plots correlation matrices to identify relationships (e.g., RAM size vs. Price).
* Conducts **A/B Hypothesis Testing** (e.g., two-sample t-tests) to prove statistically significant pricing differences between competitor brands.

### Stage 5: Predictive Modeling
Training Machine Learning models in **`all.ipynb`** to estimate hardware pricing based on extracted features:
* Splits the dataset into Train/Test subsets.
* Trains a baseline **Random Forest Regressor**.
* Implements an advanced **XGBoost Regressor** with hyperparameter tuning (`n_estimators=200`, `learning_rate=0.1`, `max_depth=6`).
* Evaluates model performance using standard metrics (MAE, RMSE, R² Score) to compare algorithm efficiency.

---

## 📂 Data Storage

All datasets are stored in the `data/` directory and are strictly versioned according to their pipeline stage:

* `data/raw/url/` — Raw text/CSV files containing the initial scraped product links.
* `data/raw/categories/` — Unprocessed JSON dumps directly from Ozon's internal API containing dirty product specifications.
* `data/processing/` — Intermediate and final sanitized datasets. This folder contains the cleaned CSVs with engineered features, ready to be fed into the XGBoost and Random Forest models.
