# Web Scraping & RAG Data Enrichment Pipeline

This project is a Python script designed to enrich a list of companies from a CSV file with detailed financial and business data. It automatically scrapes each company's website, uses a local RAG (Retrieval-Augmented Generation) pipeline to find the most relevant information, and then employs the Google Gemini API to extract specific data points into a new, structured CSV file.

## 🚀 Features

  * **Automated Web Scraping:** Uses `requests` and `BeautifulSoup` to fetch and parse text content from company websites.
  * **Mini-RAG Pipeline:**
      * Chunks scraped text into manageable pieces.
      * Uses `sentence-transformers` (`all-MiniLM-L6-v2`) to create vector embeddings for text chunks.
      * Builds an in-memory `faiss` vector index to perform fast semantic search.
      * Finds the most relevant context for the LLM, rather than feeding it the entire (often noisy) website text.
  * **LLM Data Extraction:**
      * Uses Google's `gemini-2.5-flash` model to analyze the relevant context.
      * Employs detailed prompt engineering to force the LLM to return data in a strict JSON format.
  * **Robust Error Handling:** Includes an exponential backoff and retry mechanism (`query_llm`) to gracefully handle API rate limits (`429` errors).
  * **Data Summarization:** Generates a final CSV with the enriched data and prints a "Data Completeness Summary" to show how many fields were successfully populated.

## 🏛️ Architecture & Workflow

The script follows this step-by-step process for each company in the input file:

1.  **Load Data:** Reads the input CSV (`assignment_uno_southAfrica.csv`) into a pandas DataFrame.
2.  **Iterate:** Loops through each company (row) in the DataFrame.
3.  **Scrape:** Cleans the company's URL and uses the `scrape_site` function to fetch the raw text content from the website.
4.  **Contextualize (RAG):** The `extract_relevant_context` function is called:
      * The raw text is split into overlapping chunks.
      * These chunks are embedded using `SentenceTransformer`.
      * A `faiss` index is built from these embeddings.
      * A set of queries (e.g., "company loan terms," "company headquarters") are used to search the index.
      * The most relevant text chunks are combined to create a dense, focused `context`.
5.  **Build Prompt:** A detailed prompt is constructed, providing the LLM with the company's name, the RAG-generated `context`, and a precise JSON schema of the 14 data points to extract (e.g., `headquarters_city`, `founded_year`, `avg_interest_rate`).
6.  **Extract Data:** The `query_llm` function sends the prompt to the Gemini API. If the API is busy, it automatically waits and retries.
7.  **Parse Response:** The script cleans and parses the text response from the LLM to extract the valid JSON object.
8.  **Store Results:** The extracted data is stored in a list. If any step fails, `null` values are recorded for that company.
9.  **Rate Limiting:** A `time.sleep(8)` is hardcoded in the loop to respect the free-tier API limits and prevent failures.
10. **Save Output:** After processing all companies, the results are compiled into a new DataFrame and saved as `populated_results.csv`.

## 🛠️ Technology Stack

  * **Data Handling:** `pandas`, `numpy`
  * **Web Scraping:** `requests`, `beautifulsoup4`
  * **LLM:** `google-generativeai` (for Gemini 2.5 Flash)
  * **Embedding/RAG:** `sentence-transformers`, `faiss-cpu`
  * **Utilities:** `tldextract`, `json`, `re`, `time`
