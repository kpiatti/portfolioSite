---
description: ""
featured_image: "/images/sentiment_feature.webp"
title: "Mobile Device Sentiment Analysis with AWS & R"
show_reading_time: false
draft: true
---

## What is Sentiment Analysis?

Sentiment Analysis is a subset of the broader class of text mining analytics strategies that attempts to gather insights by analysing large volumes of text data (e.g. websites).

Sentiment analysis uses text mining strategies to uncover insights, trends, and patterns about peoples' feelings/sentiments.


### Project Motivation

Helio is working with a government health agency to create a suite of smart phone medical apps for use by aid workers in developing countries. The government agency will be providing workers with technical support services, but they need to limit the support to a single model of smart phone and operating system. This will also help to limit purchase costs and ensure uniformity when training aid workers to use the device. After completing an initial investigation, Helio has created a short list of devices that are all capable of executing the app suite's functions.

To narrow this list down to one device, Helio has engaged Alert Analytics to conduct a broad-based web sentiment analysis to gain insight into the attitudes toward the devices. Your job is to conduct this analysis.

The goal of this project is to develop a model that represents how people feel (sentiment) about particular mobile devices based on what they say about those devices online.


### Objective
Analyze online sentiment toward four mobile devices



## Data Collection
The body of text mined for this project comes from the [CommonCrawl](https://commoncrawl.org/), a non-profit organization
The data for this project comes from scrapping the content of tens of thousands of webpages to identify those that express a view/attitude/sentiment toward one or more of the five mobile devices being considered (Apple iPhone, Samsung Galaxy, Sony Xperia, Nokia Lumia, or HTC phone)—these websites are deemed relevant.

  * More explicitly, a website is deemed relevant if and only if it mentions at least one of the target mobile devices *and* also contains words that indicate the content is a review.

If a webpage is deemed relevant, than data from the site is collected about the sentiment(s) towards the device that are expressed.


## Methodology

```r
library(tidyverse)
```

{{< figure src="/images/coord_flip_ex.png" title="Data Vizualization" >}}

[Link to Project GitHub Repository](https://github.com/kpiatti/Mobile-Device-Sentiment-Analysis) &nbsp; &nbsp; [Link to RPubs](https://rpubs.com/kpiatti)
