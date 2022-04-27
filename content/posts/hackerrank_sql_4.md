---
title: "HackerRank SQL #4"
date: 2022-04-26T22:50:46-03:00
draft: false
tags: [hackerrank, sql]
---

### 1. [Ollivander's Inventory](https://hackerrank-challenge-pdfs.s3.amazonaws.com/19502-harry-potter-and-wands-English?AWSAccessKeyId=AKIAR6O7GJNX5DNFO3PV&Expires=1651028509&Signature=9QTQ2WY1D5fI%2FfyBou9OeqHrAPU%3D&response-content-disposition=inline%3B%20filename%3Dharry-potter-and-wands-English.pdf&response-content-type=application%2Fpdf)

#### 1.1 SQL Server

```sql
with join_tables as (
    select
        id,
        age,
        coins_needed,
        power
    from wands
    inner join wands_property
        on wands_property.code = wands.code
    where is_evil = 0
),

agg_price as (
    select
        id,
        age,
        power,
        MIN(coins_needed) as min_price
 from join_tables
 group by id, power, age
),

dedup_step as (
    select
        id,
        age,
        min_price,
        power,
        row_number() over(
            partition by age, power order by min_price asc
        ) as rownumber
 from agg_price
)

select
    id,
    age,
    min_price,
    power
from dedup_step
where rownumber = 1
order by power desc, age desc
```

