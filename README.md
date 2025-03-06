# Tracking Lexical Complexity in English Fiction Over Time  
**DATA 34200 Final Project** – Analyzing how the complexity of words in English-language fiction has evolved over time.

## Overview
This project explores the historical evolution of word complexity in English fiction using Google Books N-grams and the EFLLex dataset. The analysis measures lexical difficulty trends over centuries and identifies patterns in word usage across different CEFR difficulty levels (A1–C1).

## Key Findings
### Lexical Complexity Trends  
- The average word difficulty (weighted by frequency) has remained relatively flat over time, with minor fluctuations in the 1600s–1700s.  
- A1-level words (easiest) dominate, making up more than 50% of words, with a rising trend in recent decades.  
- C1-level words (most complex) have gradually increased from approximately 3.5% in 1700 to 4.2% in the 2010s.  
- A2-level vocabulary has steadily declined over the centuries.

## Data Sources
This project integrates two datasets:

1. **[Google Books N-grams](https://cloud.google.com/bigquery/public-data/google-books)**  
   - Dataset: `eng_fiction_1` (6.93GB)  
   - Contains word frequencies across centuries in fiction books.  

2. **[EFLLex Dataset](https://cental.uclouvain.be/cefrlex/efllex/)**  
   - 15,000+ words labeled with CEFR difficulty levels (A1–C1).  
   - Provides word complexity information based on language proficiency.  

## Methodology
### Data Cleaning & Preprocessing
- Converted words to lowercase and removed duplicates.  
- Joined the Google Books N-grams dataset with the EFLLex dataset to assign CEFR difficulty levels.  
- Used SQL (BigQuery) for data transformation.

### Computing Lexical Complexity Over Time
- Mapped CEFR levels to numeric scores (A1 = 1, A2 = 2, …, C1 = 5).  
- Unnested the yearly n-grams data using SQL to structure it for analysis.  
- Weighted the CEFR scores by word frequency to compute an average difficulty score per year and per decade.

### Data Visualization
- Generated yearly and decade-wise trends for word complexity.  
- Computed proportions of words at each CEFR level over time.  
- Visualized results in a Colab Notebook.

## SQL Queries Used
Example query for computing average CEFR score per year:
```sql
WITH unnested_data AS (
  SELECT 
    CASE 
      WHEN cefr_label = 'A1' THEN 1 
      WHEN cefr_label = 'A2' THEN 2 
      WHEN cefr_label = 'B1' THEN 3 
      WHEN cefr_label = 'B2' THEN 4 
      WHEN cefr_label = 'C1' THEN 5 
      ELSE 0 
    END AS cefr_score, 
    y.year AS year,
    y.term_frequency AS term_frequency 
  FROM `de-hw1-447917.final_proj_dataset.ngrams_cefr_labeled` AS t 
  CROSS JOIN UNNEST(t.years) AS y
)
SELECT year, SUM(cefr_score * term_frequency) / SUM(term_frequency) AS avg_cefr_score 
FROM unnested_data 
GROUP BY year 
ORDER BY year;
