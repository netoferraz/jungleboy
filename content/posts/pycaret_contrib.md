---
title: "My first contribution to Pycaret"
date: 2022-03-20T15:01:19-03:00
draft: false
tags: [machine learning, pycaret, data science, open source]
---

I'm thrilled to announce that my first contribution to PyCaret has been accepted. PyCaret is such an incredible project that helps to democratize machine learning to a broad audience! PyCaret already has a nice integration with mlflow, but with this contribution, I believe I made it even better.

PyCaret has different modules for different tasks like:
- ✔️ Regression
- ✔️ Classification
- ✔️ Clustering
- ✔️ Anomaly Detection
- ✔️ Natural Language Processing

In our [Pull Request](https://github.com/pycaret/pycaret/pull/1526), we implemented a feature for users willing to send custom tags to mlflow tracking API using these modules. Hence, with this addition, PyCaret users can send customized tags for each task that they want to send to their mlflow tracking server.

This new feature is available from version [2.3.7](https://github.com/pycaret/pycaret/releases/tag/2.3.7). For those interested in code I'll share some pictures showing use cases of this functionality.

![Regression](/pycaret_contrib/pycaret_1.png)

![Classification](/pycaret_contrib/pycaret_2.png)

![Clustering](/pycaret_contrib/pycaret_3.png)

![Anomaly Detection](/pycaret_contrib/pycaret_4.png)

![NLP](/pycaret_contrib/pycaret_5.png)






