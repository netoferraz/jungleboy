---
title: "Sneakers Web Crawlers"
date: 2022-03-27T18:47:22-03:00
draft: false
tags: [sneakers, cotatenis, web scraping, scrapy]

---

📣📣📣 A couple of days ago we released our first contribution to the open-source community from the **cotatenis** project. It's was a collection of sneakers images as a [Kaggle dataset](https://www.kaggle.com/datasets/ferraz/cotatenis-sneakers).

🕸️ 🕸️🕸️ And today I'm thrilled to announce that we've decided to release 17 code repositories that were the ingestion layer of the cotatenis project.

These projects were a collection of web crawlers who were in charge of collecting data through a series of stores and websites and dump into our data lake.

- artwalk (https://github.com/cotatenis/artwalk)
- authenticfeet (https://github.com/cotatenis/authenticfeet)
- dafiti (https://github.com/cotatenis/dafiti)
- farfetch (https://github.com/cotatenis/farfetch)
- gdlp (https://github.com/cotatenis/gdlp)
- goat (https://github.com/cotatenis/goat)
- kanui (https://github.com/cotatenis/kanui)
- kings (https://github.com/cotatenis/kings)
- maze (https://github.com/cotatenis/maze)
- centauro (https://github.com/cotatenis/centauro)
- newbalance (https://github.com/cotatenis/newbalance)
- pineapple (https://github.com/cotatenis/pineapple)
- shop2gether (https://github.com/cotatenis/shop2gether)
- sneakers123 (https://github.com/cotatenis/sneakers123)
- solecollector (https://github.com/cotatenis/solecollector)
- yourid (https://github.com/cotatenis/yourid)

😲 That's a lot of code, isn't it? All of that code is organized and scheduled by Airflow. Each one of these stores/sites was organized as DAGs. So we do some health checks to ensure that our spiders will encounter those targets in a way that we expected to perform to make the data collection successfully.

To encapsulate this logic we have a code that is in charge of that. It runs before each data gathering process inside of our DAGs.

- crawl_validators (https://github.com/cotatenis/crawl_validators)

I'm very excited 🤩 to hear from you guys about what you think about that launch and hope you enjoy it!
