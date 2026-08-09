# Hollywood Airbnb Investment Strategy

Consulting-style analysis for Airbnb hosts in Hollywood, LA, answering: with a limited budget, should a host invest in **property upgrades** or **guest-management improvements** first?

Marketing Analytics

## The problem

Playing the role of consultants to a Hollywood Airbnb host, we set out to answer: **What specific property features, host behaviors, and neighborhood characteristics most influence guest satisfaction and occupancy in Hollywood Airbnb listings — and based on these drivers, where should a host with a limited budget invest?**

Hollywood guests are review-driven and expect cleanliness, modern amenities, good host communication, a trendy/safe space, and easy access to attractions. The question was which of those actually move the needle on occupancy and satisfaction, versus which just sound important.

## The data

[Inside Airbnb](http://insideairbnb.com/get-the-data/)'s Los Angeles listings and reviews dataset, filtered to Hollywood-area properties. The full detailed listings file alone runs 100+ variables and 100MB+ (property features, host attributes, review scores, pricing, availability calendars, and the full text of guest reviews). Getting from that raw scrape to an analysis-ready dataset meant:

- Filtering down to Hollywood-specific listings from the broader LA dataset
- Joining listing-level fields (property type, amenities, host status) with aggregated review data (ratings, review counts, review text) on `listing_id`
- Cleaning missing/inconsistent fields (unlisted amenities, missing review scores, price formatting) and reducing 100+ raw columns down to the variables actually relevant to occupancy and satisfaction
- Handling a large, high-dimensional feature set by reducing it with PCA rather than throwing every variable into a single messy regression

## What we did with it

Three analytical layers, each answering a different piece of the central question:

**1. Unsupervised learning (what predicts occupancy?)**
- Ran PCA on the listing features to collapse a large, correlated feature set into a handful of interpretable components
- Regressed occupancy on those PCA factors to rank which underlying drivers mattered most
- Ran K-means clustering (K=3) on the same features to segment listings into performance tiers and see whether the clusters told the same story as the regression

**2. Supervised learning (what predicts guest satisfaction?)**
- Built a logistic regression and a decision tree to classify listings as high vs. lower guest satisfaction, using property features, host behavior, and neighborhood characteristics as predictors
- Compared the two models on validation misclassification rate to pick the better classifier

**3. Text mining (what are guests actually saying?)**
- Word count and topic modeling on review text to surface recurring themes
- Sentiment analysis on review text, cross-checked against guests' numeric star ratings to confirm the sentiment scores were actually measuring what we thought they were

## Results

**Occupancy drivers (unsupervised learning)**

![PCA loading matrix](images/pca_loading_matrix.png)

- **Availability is the biggest lever on occupancy.** Unsold/open nights (PC1) was the strongest predictor in the regression (highest LogWorth, p < .0001) — the more open nights a listing carries, the lower its occupancy.

![Occupancy regression effect summary](images/occupancy_regression_effect_summary.png)

- **Amenities and space come second.** PC2 (space & amenities) was a significant positive driver of occupancy, and this showed up independently in both the regression and the clustering.
- **Clustering confirmed the same story from a different angle:** the amenity-rich, larger-space cluster averaged **76% occupancy**; the cluster with lots of open calendar and limited amenities averaged **36%**; a middle cluster, helped somewhat by location, landed at **59%**.

**Guest satisfaction drivers (supervised learning)**

![Satisfaction logistic regression effect summary](images/satisfaction_logistic_effect_summary.png)

- **Listing capacity (`accommodates`) was the single strongest predictor of guest satisfaction**, with Superhost status a close second — both far outweighed most individual property features. The logistic model correctly classified satisfaction ~78% of the time (22% validation misclassification rate).

![Model comparison ROC curve](images/model_comparison_roc.png)

- **The decision tree beat the logistic regression** on ROC/AUC at distinguishing high- from lower-satisfaction listings, and was selected as the final model.

**What guests are saying (text mining)**

![Review word cloud](images/review_word_cloud.png)

- Reviews are dominated by location, cleanliness, and host-related language — "stay," "place," "location," "clean," "host," and "Hollywood" are the most frequent terms across 45,000+ reviews.

![Sentiment vs. star rating](images/sentiment_vs_rating.png)

- **Review sentiment closely tracks star ratings**, which validated the text-mining approach: average sentiment ran strongly negative (~-30) at 1-star reviews and climbed to strongly positive (~68) at 5-star reviews.
- **Cleanliness is the single biggest source of guest complaints.** Topic modeling on negative reviews showed hygiene and maintenance issues drove the strongest negative reactions — more than property size or host communication tone.

## Recommendation

**Invest in filling the calendar and adding amenities before cosmetic upgrades.** Occupancy is driven far more by unsold-night availability and amenity richness than by anything else measured. On the guest-experience side, **cleanliness and maintenance should be the top operational priority** — it's guests' single strongest source of complaint, and addressing it protects the satisfaction drivers (listing capacity fit and Superhost status) the model identified as most important.

## Repo contents
| `images/` | JMP output screenshots referenced in this README (PCA, regression, model comparison, text mining) |

Raw data and the JMP project file aren't included — the detailed listings data includes host and reviewer names, which aren't appropriate to redistribute in bulk even though the source is public. Download the current LA dataset directly from [Inside Airbnb](http://insideairbnb.com/get-the-data/) to reproduce.

## Tools

**JMP** — statistical analysis software from SAS, used for the PCA, clustering, regression, and classification models. It's a GUI-driven point-and-click tool (no code required) common in business analytics, market research, and quality engineering — comparable to **SPSS**, **Minitab**, or **Stata**, and covers similar ground to what you'd otherwise do in R or Python with pandas/scikit-learn, just without writing code.

Text mining (word count, sentiment analysis, topic modeling) was also done using JMP's text analytics tools.

## Author

Savgun Kaur ([savscripts](https://github.com/savscripts)) 
