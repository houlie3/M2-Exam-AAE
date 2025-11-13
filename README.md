# M2-Exam-AAE
## Dataset Used and Rationale for Subset Choice
The dataset used is kurry/sp500_earnings_transcripts, a publicly available Hugging Face dataset containing quarterly earnings call transcripts for S&P 500 companies. Each record includes the company symbol, quarter, year, transcript text, and metadata such as company name and structured content.
Because the dataset is quite large, a subset of 20 transcripts was used for initial experimentation and model development.
The subset was randomly sampled from the full dataset after shuffling with a fixed seed (seed=42) to ensure reproducibility.
Converting the subset into mutable Python dictionaries allowed for inline modification (e.g., adding filtered text fields).
This smaller subset provided a manageable yet representative sample of companies and sectors for validating the risk extraction pipeline before scaling up to the entire dataset.
## LLM Extraction Process, Network Construction, and Analysis
1. Preprocessing and Filtering
To focus the language model on relevant content, the transcripts were pre-filtered using spaCy with a custom keyword-based sentence extraction step.
A list of ~60 risk-related keywords (e.g., risk, uncertainty, compliance, bankruptcy, safety, breach, contamination) was defined and matched case-insensitively against each transcript.
Only sentences containing these keywords were retained and concatenated to create a reduced “risk-focused” transcript for each company.
This filtering step reduced noise (general corporate language, greetings, etc.) and significantly improved LLM efficiency and precision.
2. LLM Risk Extraction (Gemini 2.5 Flash)
Each filtered transcript was passed to Google’s Gemini 2.5 Flash model via the client.chat.completions.create() API.
A structured prompt instructed the model to:
Identify and classify all risks discussed, and
Return results strictly as a JSON array, where each element contained:
"company" — company symbol,
"risk_type" — e.g., market, operational, regulatory, legal, or supply-chain, and
"sentence" — the exact sentence mentioning the risk.
To ensure stability, the script included:
Automatic retry logic with exponential backoff (to handle 503 errors / overloads), and
JSON cleaning to extract valid arrays even if the model’s output included preamble text.
Originally, transcripts were processed in 800-word chunks, but after filtering, they became short enough to process as full texts without further splitting.
3. Network Construction and Analysis
The extracted JSON records were combined into a structured dataframe, where each row represented one risk mention.
From this, a risk-company bipartite network can be constructed:
Nodes: companies and risk types,
Edges: indicate that a company discussed a given risk.
Such a network allows for:
Identifying clusters of companies exposed to similar risks,
Measuring risk centrality (which risk types dominate across firms), and
Visualizing industry-level patterns of concern (e.g., cybersecurity vs. supply-chain issues).
Summary of Findings and Limitations
## Findings
The keyword filtering successfully reduced transcript length by 85–95% while retaining high-relevance content.
Gemini extracted coherent, categorized risks with good sentence alignment, especially for explicit mentions (e.g., “subject to risks and uncertainties” or “market volatility”).
Even with a small sample, recurring risk categories emerged — particularly general business uncertainty, regulatory compliance, and market risk.
Limitations
Company labeling issue: During initial runs, all outputs appeared under the same company symbol (“A”) due to dataset indexing, later fixed by randomizing and correctly mapping symbols.
Duplicate sentences: Gemini occasionally repeated similar risk sentences when phrasing varied slightly.
Keyword bias: The initial filtering relies on exact keyword matches, potentially missing implicit risk discussions (e.g., “headwinds” or “macroeconomic challenges”).
Limited scale: The subset of 20 transcripts limits statistical generalization — future iterations should process the full dataset.
Model dependency: Outputs depend heavily on the Gemini model’s ability to interpret and format text precisely as JSON; minor prompt tuning or model variation can affect consistency.
