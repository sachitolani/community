---
id: 2022-06-23-How-Short-video-Platform-Likee-Removes-Duplicate-Videos-with-Milvus.md
title: How Short-video Platform Likee Removes Duplicate Videos with Milvus
author: Xinyang Guo, Baoyu Han
date: 2022-06-23
desc: Learn how Likee uses Milvus to identify duplicate videos in milliseconds.
cover: assets.zilliz.com/How_Short_video_Platform_Likee_Removes_Duplicate_Videos_with_Milvus_07bd75ec82.png
tag: Scenarios
tags: Use Cases of Milvus
canonicalUrl: http://milvus.io/blog/2022-06-23-How-Short-video-Platform-Likee-Removes-Duplicate-Videos-with-Milvus.md
---

![Cover image](https://assets.zilliz.com/How_Short_video_Platform_Likee_Removes_Duplicate_Videos_with_Milvus_07bd75ec82.png "How Short-video Platform Likee Removes Duplicate Videos with Milvus")

> This article is written by Xinyang Guo and Baoyu Han, engineers at BIGO, and translated by [Rosie Zhang](https://www.linkedin.cn/incareer/in/rosie-zhang-694528149).

[BIGO Technology](https://www.bigo.sg/) (BIGO) is one of the fastest-growing Singapore technology companies. Powered by artificial intelligence technology, BIGO's video-based products and services have gained immense popularity worldwide, with over 400 million users in more than 150 countries. These include [Bigo Live](https://www.bigo.tv/bigo_intro/en.html?hk=true) (live streaming) and [Likee](https://likee.video/) (short-form video). 

Likee is a global short video creation platform where users can share their moments, express themselves, and connect with the world. To increase user experience and recommend higher-quality content to users, Likee needs to weed out duplicate videos from a tremendous amount of videos generated by users every day, which poses no straightforward task. 

This blog presents how BIGO uses [Milvus](http://Milvus.io), an open-source vector database, to effectively remove duplicate videos.

**Jump to:**
- [Overview](#Overview)
- [Video deduplication workflow](#Video-deduplication-workflow)
- [System Architecture](#System-architecture)
- [Using Milvus to power similarity search](#Using-Milvus-vector-database-to-power-similarity-search)

# Overview
Milvus is an open-source vector database featuring ultra-fast vector search. Powered by Milvus, Likee is able to complete a search within 200ms while ensuring a high recall rate. Meanwhile, by [scaling Milvus horizontally](https://milvus.io/docs/v2.0.x/scaleout.md#Scale-a-Milvus-Cluster), Likee successfully increases the throughput of vector queries, further improving its efficiency.

# Video deduplication workflow
How does Likee identify duplicate videos? Every time a query video is input into Likee's system, it will be cut into 15-20 frames and each frame will be converted to a feature vector. Then Likee searches in a database of 700 million vectors to find the top K most similar vectors. Each one of the top K vectors corresponds to a video in the database. Likee further conducts refined searches to get the final results and determine the videos to be removed.

# System architecture
Let's take a closer look at how Likee's video de-duplication system works using Milvus. As is shown in the diagram below, new videos uploaded to Likee will be written to Kafka, a data storage system, in real-time and consumed by Kafka consumers. The feature vectors of these videos are extracted through deep learning models, where unstructured data (video) is converted into feature vectors. These feature vectors will be packaged by the system and sent to the similarity auditor.

![Architechure of Likee's video de-duplication system](https://assets.zilliz.com/Likee_1_6f7ebcd8fc.png "Architechure of Likee's video de-duplication system")

The feature vectors extracted will be indexed by Milvus and stored in Ceph, before being [loaded by the Milvus query node](https://milvus.io/blog/deep-dive-5-real-time-query.md) for further search. The corresponding video IDs of these feature vectors will also be stored simultaneously in TiDB or Pika according to the actual needs.

### Using Milvus vector database to power similarity search
When searching for similar vectors, billions of existing data, along with large amounts of new data generated every day, pose great challenges to the functionality of the vector search engine. After thorough analysis, Likee eventually chose Milvus, a distributed vector search engine of high performance and high recall rate, to conduct vector similarity search.

As is shown in the diagram below, the procedure of a similarity search goes as follows:

1. First, Milvus performs a batch search to recall the top 100 similar vectors for each of the multiple feature vectors extracted from a new video. Each similar vector is bound to its corresponding video ID. 

2. Second, by comparing the video IDs, Milvus removes the duplicate videos and retrieves the feature vectors of the remaining videos from TiDB or Pika. 

3. Finally, Milvus calculates and scores the similarity between each set of the feature vectors retrieved and the feature vectors of the query video. The video ID with the highest score is returned as the result. Thus the video similarity search is concluded.

![Procedure of a similarity search](https://assets.zilliz.com/02_a24d251c8f.png "Procedure of a similarity search")

As a high-performance vector search engine, Milvus has done an extraordinary job in Likee's video de-duplication system, greatly fueling the growth of BIGO's short-video business. In terms of video businesses, there are many other scenarios where Milvus can be applied to, such as illegal content blocking or personalized video recommendation. Both BIGO and Milvus are looking forward to future cooperation in more areas.