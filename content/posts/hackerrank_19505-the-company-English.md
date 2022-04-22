---
title: "HackerRank #1"
date: 2022-04-20T10:04:27-03:00
draft: false
tags: [hackerrank, sql]
---

This is my first post on a series in which I'll share my solutions to several HackerRank exercises. For this first exercise, you can read the complete statement [here](https://hackerrank-challenge-pdfs.s3.amazonaws.com/19505-the-company-English?AWSAccessKeyId=AKIAR6O7GJNX5DNFO3PV&Expires=1650460431&Signature=vTAupmzckGiDVuYbpaZnuWKDMxQ%3D&response-content-disposition=inline%3B%20filename%3Dthe-company-English.pdf&response-content-type=application%2Fpdf). That's said I present to you my solution for this problem.

### 1. SQL Server


```sql
with j_employee as (
    select distinct
        employee_code,
        manager_code,
        senior_manager_code,
        employee.lead_manager_code,
        employee.company_code,
        founder
    from employee
    inner join company
        on company.company_code = employee.company_code
    inner join lead_manager
        on lead_manager.lead_manager_code = employee.lead_manager_code
),

number_of_employees as (
    select
        company_code,
        founder,
        COUNT(*) as total_number_of_employee
    from j_employee
    group by company_code, founder
),

number_of_lead_managers as (
    select
        lead_manager.lead_manager_code,
        lead_manager.company_code,
        founder,
        total_number_of_employee,
        COUNT(distinct lead_manager.lead_manager_code) as total_lead_managers
    from lead_manager
    inner join number_of_employees
        on number_of_employees.company_code = lead_manager.company_code
    group by
        lead_manager.lead_manager_code,
        lead_manager.company_code,
        founder,
        total_number_of_employee
),

number_of_senior_managers as (
    select
        company_code,
        lead_manager_code,
        COUNT(distinct senior_manager_code) as total_senior_manager
    from senior_manager
    group by company_code, lead_manager_code
),

number_of_managers as (
    select
        company_code,
        lead_manager_code,
        COUNT(distinct manager_code) as total_manager
    from manager
    group by company_code, lead_manager_code
),

join_all_tables as (
    select
        number_of_lead_managers.company_code,
        founder,
        total_lead_managers,
        total_senior_manager,
        total_manager,
        total_number_of_employee
    from number_of_lead_managers
    inner join number_of_senior_managers
        on
            number_of_senior_managers.company_code = number_of_lead_managers.company_code and number_of_senior_managers.lead_manager_code = number_of_lead_managers.lead_manager_code
    inner join number_of_managers
        on
            number_of_managers.company_code = number_of_lead_managers.company_code and number_of_managers.lead_manager_code = number_of_lead_managers.lead_manager_code
)

select * from join_all_tables order by company_code asc
```
