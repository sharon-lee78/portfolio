---
layout: default
title: Amazon Review Near-Duplicate Detection
---

# Amazon Review Near-Duplicate Detection
#### Big Data and Cloud Computing, University of Chicago | Dec 2025

## Question
Generative AI tools became widely available in late 2022. I wanted to know whether reviews written after that point repeat each other more often, which could be one sign of AI-written reviews. I focused on the Automotive category, which is the largest category in the product metadata.

## Data and Cleaning
The data has 64.7M reviews (52.4GB) and 4.3M product records, stored in Google Cloud Storage and processed with PySpark on a Dataproc cluster. Cleaning steps:

- Removed 88,032 reviews with empty text or a negative helpful-vote count
- Converted millisecond timestamps to datetimes
- Parsed price strings such as "from 49.50" into numbers, and set non-numeric prices to null
- Kept only the review and product columns used in the analysis

## Review Volume Over Time
![Daily review counts over time](images/amazon/daily_reviews.png)

Reviews run from 1998 to September 2023. Volume grows from 2012 and peaks between 2019 and 2021, with regular increases around December and July. From December 12, 2019 to January 12, 2020, daily counts stay near 40,000, well above any other holiday season in the data, so I flagged that period as an outlier and left it out of the trend plots instead of treating it as normal holiday demand.

## Exact Duplicates in Automotive
Across all Automotive reviews, 60.8% of review titles and 18.3% of review texts appear more than once. Most repeats are very short:

| Review text | Times it appears |
|---|---|
| Good | 53,254 |
| Great | 39,557 |
| Great product | 38,898 |
| Works great | 36,991 |
| Perfect | 28,270 |

![Distribution of text duplicate counts](images/amazon/text_duplicate_counts.png)

Most texts appear once, but a small set of generic phrases repeats tens of thousands of times.

## Near-Duplicates with MinHash LSH
Exact matching misses reviews that are copied with small edits, so I used MinHash LSH:

1. Took a 1% random sample of Automotive reviews. The full category was too large to run pairwise matching on the course cluster.
2. Removed punctuation and stopwords, and kept reviews with at least three remaining words, since one-word reviews were already covered above. I kept the original capitalization on purpose, since identical formatting is part of what AI-written reviews might share.
3. Turned each review into a word set with CountVectorizer and fit MinHashLSH with 5 hash tables.
4. Treated two reviews as near-duplicates when their Jaccard similarity was 0.8 or higher.

9,202 of the 286,750 sampled reviews (3.2%) had a near-duplicate in the sample. Within each of the five most-reviewed products, near-duplicates were almost absent (2 reviews in total).

## Did Duplication Go Up After 2022?
In the sample, 3.4% of pre-2022 reviews had a near-duplicate, compared with 1.2% of 2022–2023 reviews. On its face that points to less duplication, not more, but I don't think this data settles the question:

- The pre-2022 group is about four times larger, which by itself raises the chance of finding a match.
- The split is at January 2022, but ChatGPT was released at the end of November 2022, so most of the recent group was written before it. The data ends in September 2023, leaving only about ten months after the release.
- With random sampling, a review's match is only found if the match was also sampled, so every rate here is a lower bound.

## What I Would Change
- Split at December 2022 and compare equal-sized samples from each side
- Sample by product instead of by review, keeping every review of each selected product so duplicates within a product stay together

Code: [Data Cleaning](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/1_data_cleaning.ipynb) · [EDA](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/2_eda.ipynb) · [Analysis](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/3_analysis.ipynb)
