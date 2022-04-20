---
title: "HackerRank SQL #2 and #3"
date: 2022-04-20T17:15:46-03:00
draft: false
tags: [hackerrank, sql]
---

### 1. [Top Competitors](https://hackerrank-challenge-pdfs.s3.amazonaws.com/19599-full-score-English?AWSAccessKeyId=AKIAR6O7GJNX5DNFO3PV&Expires=1650488203&Signature=QnhiRxPLGN0NIfdFf3ZF5UUMRzE%3D&response-content-disposition=inline%3B%20filename%3Dfull-score-English.pdf&response-content-type=application%2Fpdf)

#### 1.1 SQL Server

```sql
with join_tables as (
    select
        submission_id,
        submissions.hacker_id,
        name,
        submissions.challenge_id,
        submissions.score as hacker_score,
        challenges.difficulty_level,
        difficulty.score as challenge_score
    from submissions
    inner join challenges
        on challenges.challenge_id = submissions.challenge_id
    inner join difficulty
        on difficulty.difficulty_level = challenges.difficulty_level
    inner join hackers
        on hackers.hacker_id = submissions.hacker_id
),

add_flag_column as (
    select
        *,
        case
            when hacker_score = challenge_score then 1
            else 0
        end as is_full_score
    from join_tables
),

filter_only_full_scores as (
    select * from add_flag_column where is_full_score = 1
),

count_num_challenges as (
    select
        hacker_id,
        name,
        COUNT(*) as total_num_challenges
    from filter_only_full_scores group by hacker_id, name having COUNT(*) >= 2
)

select
    hacker_id,
    name
from count_num_challenges order by total_num_challenges desc, hacker_id asc
```

### 2. [Weather Observation Station 20](https://hackerrank-challenge-pdfs.s3.amazonaws.com/9355-weather-observation-station-20-English?AWSAccessKeyId=AKIAR6O7GJNX5DNFO3PV&Expires=1650486045&Signature=9FbwDUUGfzM9v%2F%2FMRh2MEgvfSpM%3D&response-content-disposition=inline%3B%20filename%3Dweather-observation-station-20-English.pdf&response-content-type=application%2Fpdf)

#### 2.1 SQL Server

```sql
SELECT DISTINCT CAST(PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY LAT_N) OVER() AS NUMERIC(10, 4)) AS MEDIAN
FROM STATION
```



