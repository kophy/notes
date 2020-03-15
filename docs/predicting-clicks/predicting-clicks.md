# Predicting Clicks: Estimating the Click-Through Rate for New Ads

*Use features of ads, terms, and advertisers to learn a model that accurately predicts the click-though rate for new ads.*

## Background

- The key task for a search engine advertising system is to determine what advertisements should be displayed, and in what order, for each query that the search engine receives. 
- Assume the probability of an ad is clicked is independent of its position and the probability of an ad is viewed only depends on position. $P(click|ad,pos)=p(click|ad, seen) \cdot p(seen|pos)$. Then CTR is defined as $p(click|ad, seen)$.
- For ads that has been displayed enough times(up to 200-300 search pages), CTR can be estimated based on historical information, but this doesn't work for new ads.

## Model Basics

- The goal is to create a model to predict initial CTR for new ads.
- Training and testing data is split on advertiser level, so we can consider each ad and account as completely novel to the system.
- Use Logistics regression and cross-entropy loss function.
- For each feature $f_i$, add derived features $log(f_i+1)$ and $fi^2$. Feature values more than five standard deviations from the mean are truncated.

## Features

- Term CTR: the CTR of other ads with same or related bid terms.
- Ad quality feature set
  - manual features: appearance, attention capture, reputation, landing page quality, relevance.
  - unigram features: one-hot encoding of most common 10K words in ad title and body.
- Order specificity feature set: capture how targeted an order with the category entropy of bid terms.
- Search data feature set: the approximate frequency of bid term occurring on the Web and users query for the bid term.

## Others

- Although there is overlapping between features, we want to Include as many feature sets as possible for robustness in adversarial situations.
- Improvements: making the CTR estimation dependent on user query, adding human judges as new source of information, making the model time-dependent.