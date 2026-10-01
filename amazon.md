---
layout: default
title: Amazon Review Near-Duplicate Detection
---

# Amazon Review Near-Duplicate Detection
#### Big Data and Cloud Computing, University of Chicago | Dec 2025

## Question
How much of Amazon review text is repeated, and what does the repeated text look like? I focused on Automotive, the largest category by number of products (1.7M).

## Data and Cleaning
The data has 64.7M reviews (52.4GB) and 4.3M product records, stored in Google Cloud Storage and processed with PySpark on a Dataproc cluster. Cleaning steps:

- Removed 88,032 reviews with empty text or a negative helpful-vote count
- Converted millisecond timestamps to datetimes
- Parsed price strings such as "from 49.50" into numbers, and set non-numeric prices to null
- Kept only the review and product columns used in the analysis

## Review Volume Over Time
![Daily review counts over time](images/amazon/daily_reviews.png)

Reviews run from 1998 to September 2023. Volume grows from 2012 and peaks between 2019 and 2021, with regular increases around December and July. From December 12, 2019 to January 12, 2020, daily counts stay near 40,000, well above any other holiday season in the data, so I flagged that period as an outlier and left it out of the trend plots.

## Exact Repeats
Using every Automotive review, 18.3% of reviews have text that appears more than once, and 60.8% have a title that appears more than once. The most repeated texts are one- or two-word phrases:

| Review text | Times it appears |
|---|---|
| Good | 53,254 |
| Great | 39,557 |
| Great product | 38,898 |
| Works great | 36,991 |
| Perfect | 28,270 |

The most repeated titles restate the star rating: "Five Stars" appears 1.59M times and "Four Stars" 248K times.

![Distribution of text duplicate counts](images/amazon/text_duplicate_counts.png)

Most texts appear once, but a small set of generic phrases repeats tens of thousands of times.

## Near-Duplicates with MinHash LSH
Exact matching misses reviews that are copied with small edits, so I used MinHash LSH on longer reviews:

1. Took a 1% random sample of Automotive reviews. The full category was too large for pairwise matching on the course cluster.
2. Removed punctuation and stopwords, and kept reviews with at least three remaining words, since short reviews are already covered by exact matching.
3. Turned each review into a word set with CountVectorizer and fit MinHashLSH with 5 hash tables.
4. Treated two reviews as near-duplicates when their Jaccard similarity was 0.8 or higher.

9,202 of the 286,750 sampled reviews (3.2%) were flagged as near-duplicates of another sampled review. This is a floor, not an estimate of the full rate:

- A review's match is only found if the match was also in the 1% sample, so most matches in the full data are missed.
- The count includes one review from each matched pair, not both.

## Conclusion
Repetition in Amazon reviews is common, and the most repeated text is short and generic. Nearly one in five Automotive reviews shares its exact text with another review, mostly phrases like "Good" or "Works great," and most titles just restate the star rating. These reviews add volume but carry about as much information as the star rating, so anyone building on review text, such as sentiment models, summaries, or fake-review detection, should separate them out first. Longer reviews are also near-copied, but the sampling design only gives a lower bound on how often.

## What I Would Change
- Sample by product instead of by review, keeping every review of each selected product, so near-duplicates stay together and the rate can be estimated rather than bounded
- Compare near-duplicate rates for short and long reviews on the same basis
- I first got interested in this because I wondered whether writing aids like autocomplete make reviews more alike. This analysis can't answer that: the notebook compares only two time periods of very different sizes (3.4% before 2022, 1.2% after, with the older group about four times larger) and leaves out the short reviews where autocomplete would show up. Tracking the exact-repeat rate of short reviews year by year on the full data would be a better test.

Code: [Data Cleaning](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/1_data_cleaning.ipynb) · [EDA](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/2_eda.ipynb) · [Analysis](https://github.com/sharon-lee78/portfolio/blob/main/amazon-reviews/3_analysis.ipynb)
