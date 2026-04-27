---
layout: post
title: "Reducing Streaming Churn with Personalized Recommendations"
author: austin
categories: [ development, software, cloud, AI ]
image: assets/images/recommendations.jpg
featured: True
---

{:.image-caption}
*Image courtesy of medium.com*

This project focuses on improving customer engagement for a streaming platform by replacing static or weakly related content recommendations with a more personalized recommendation workflow using AWS Personalize and an ETL pipeline for training data.

## Business Problem

Subscription platforms need users to keep finding relevant content. When users do not see titles that match their interests, they are less likely to return, less likely to explore the catalog, and more likely to churn.

The existing recommendation experience had two practical issues:

1. **Marketing engagement was limited** because outreach depended partly on third-party platforms with inconsistent open rates.
2. **Similar-video recommendations were inconsistent**, which made some playlists feel generic or unrelated to the user’s actual behavior.

## Objective

The goal was to direct users toward more relevant videos, increase engagement, and reduce subscription churn risk. A useful recommendation system needed to work with existing product constraints while giving engineers a clean way to retrieve ranked recommendations for the website.

## Technical Approach

### Step 1: Improve Similar Videos with AWS Personalize

AWS Personalize can use user-item interaction data to generate recommendations based on behavioral patterns. For a streaming product, useful interactions may include plays, watch duration, completion rate, genre preference, recency, and repeated engagement with similar content.

### Step 2: Build an ETL Pipeline for Training Data

The ETL pipeline prepares the interaction dataset needed for model training. The pipeline extracts watch and user activity data, transforms it into a schema suitable for AWS Personalize, and loads it into the training workflow.

Typical fields include:

- `USER_ID`
- `ITEM_ID`
- `TIMESTAMP`
- `EVENT_TYPE`
- `EVENT_VALUE`

The quality of this dataset matters as much as the model choice. Poorly defined events or noisy labels can produce recommendations that look technically valid but feel irrelevant to users.

### Step 3: Evaluate Recommendation Quality

The recommendation workflow should be evaluated before being promoted to production. Offline metrics can help compare model configurations, but product impact ultimately needs to be validated with engagement metrics such as click-through rate, watch starts, watch time, return rate, or churn-related outcomes.

### Step 4: Promote the Final Model to Production

Once a model configuration performs well enough, the recommendation campaign can be exposed to website engineers. The product can then request ranked recommendations in real time or near-real time and use those results to populate similar-video playlists.

## Possible Extensions

The same foundation can support additional recommendation surfaces:

- Personalized rows for favorite categories.
- Trending content adjusted by user segment.
- Automatically generated playlist titles.
- Personalized search ranking.
- Filters for subscription tier, content availability, region, or release recency.

## Conclusion

A recommendation system is not just a model. It is a product workflow that depends on clean interaction data, clear evaluation metrics, infrastructure for serving results, and product surfaces where recommendations can influence behavior. AWS Personalize is useful because it reduces the amount of custom modeling infrastructure needed, but the strategic work remains in defining the right events, constraints, and success metrics.
