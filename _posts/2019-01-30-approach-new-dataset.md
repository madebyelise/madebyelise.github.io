---
published: false
---
---
layout: post
title: How do I approach a new dataset?
---

Let's face it. We work with new datasets all the time as data professionals. Engineering added new tables to an existing database. The boss suggested augmenting our analysis with a public data source. As data professionals, we are tempted to dive right in. However, before going in too deep, make sure that we understand the data thoroughly.

In fact, I made a mistake in a recent project working with a household survey dataset. After I skimmed through the available documentation and column names, I believed I understood the data well and spent a ton of time building two Tableau dashboards based on the said dataset. 

However, upon talking to someone who has much more experience with this dataset, I realized I glossed over a key detail that was vaguely mentioned in the documentation: depending on the questions, survey responses may be replicated across household members! If I create a pivot table using these questions, the result would be unwieldy and often unexplicable. This means most of my time generating dashboards was wasted simply because I missed a small technicality.

To avoid mistakes like mine, regardless of whether you have "data" in your title, make sure you go through the following steps when given a new dataset.  