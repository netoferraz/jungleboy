---
title: "101 SQLFluff"
date: 2022-04-22T09:06:26-03:00
draft: false
tags: [sql, sqlfluff, linter]

---

## 1. Introduction

Are you tired of people nitpicking when doing code reviews? Did your energy drain by those that had written comments in your Pull Request talking about comma positions?

By the way, this kind of behavior is not healthy at all for you and the organization as well. Instead of that, people should bring to light the logic of the business and try to improve the code base with comments that try to deliver more value, instead to put so much energy inside the aesthetic of the codebase.

But recently we have seen an open-source project called SQLFluff bring a proposal to make a linter for SQL. The project has almost 4k stars on Github on the day I’m writing this post. As mentioned before there are different flavors of SQL and according to the actual state of the project, these are the dialects supported by the linter.

The process of reviewing the style guide must be done by the machine, once we have a pattern or a set of rules to guide us. For decades several programming languages took advantage of linters to make this done, otherwise, when we look at the RDBMS ecosystem we see a lot of different flavors of SQL that make some standardization a little harder to achieve.

But recently we have seen a open source project called [SQLFluff](https://github.com/sqlfluff/sqlfluff) bring a proposal to make a linter for SQL. The project has almost 4k stars in Github at the day i'm writing this post. As mentioned before there are different flavors of SQL and according to the actual state of the project, these are the dialects supported by the linter.

- `ANSI SQL`
- `BigQuery`
- `Exasol`
- `Hive`
- `MySQL`
- `Oracle`
- `PostgreSQL` (aka Postgres)
- `Redshift`
- `Snowflake`
- `SparkSQL`
- `SQLite`
- `Teradata`
- `Transact-SQL` (aka T-SQL)

I'll guide you guys how to setup it and make some basic demonstration of usage.

## 2. Installation

`SQLFluff` is a python project and the easy way of installing is through [PyPi](https://pypi.org/project/sqlfluff/). 

First of all, let's create a directory to host your project.

```shell
mkdir sqlfluff_demo
```

As a second step, it's a good practice to create a virtual environment and then activate it.

```shell
python3 -m venv .venv
source .venv/bin/activate
```

Once our python environment was created and activated we'll use pip to install SQLFluff.

```shell
pip install sqlfluff
```

You can type `sqlfluff` in you terminal to see which options are available through the CLI.

```shell
Usage: sqlfluff [OPTIONS] COMMAND [ARGS]...

  Sqlfluff is a modular sql linter for humans.

Options:
  --version   Show the version and exit.
  -h, --help  Show this message and exit.

Commands:
  dialects  Show the current dialects available.
  fix       Fix SQL files.
  lint      Lint SQL files via passing a list of files or using stdin.
  parse     Parse SQL files and just spit out the result.
  rules     Show the current rules in use.
  version   Show the version of sqlfluff.
```

A simple way to check the current version of SQLFluff is typing `sqlfluff --version`

```shell
sqlfluff, version 0.12.0
``` 

As you noticed this software hasn't have a stable API because its current release is below 1.0 and the maintainers [wrote](https://github.com/sqlfluff/sqlfluff) a statement about it.

The two major commands that we'll use during our development are: `lint` and `fix`. The former analyzes and shows up which [rules](https://docs.sqlfluff.com/en/stable/rules.html) our SQL break, while the latter modify our code to make it adherent to those rules.

## 3. Setup

By default, the linter has several default parameters but you can customize them to better fit the desired style code of your organization. You can read [here](https://docs.sqlfluff.com/en/stable/configuration.html#defaultconfig) some of the possible customizations you could make. 

Let's create a `.sqlfluff` file to make little modifications in the default linter behavior to show how this works.

```shell
touch .sqlfluff
```

Let's set as default the dialect `snowflake`, then exclude [L027 rule](https://docs.sqlfluff.com/en/stable/rules.html#sqlfluff.core.rules.Rule_L027) and finally setup that we desire to format our code with leading commas, instead of trailling ones.

```text
[sqlfluff]
dialect = snowflake
exclude_rules = L027

[sqlfluff:rules]
comma_style = leading
```

## 4. Usage

For the purpose of this demo, we're going to use a compiled SQL version of this [dbt model](https://github.com/cotatenis/dbt_cotatenis/blob/main/models/dev/staging/sneakers/stg_lr_gdlp.sql).

The following SQL code is our start point to format then into a `sqlfluff` style.

```sql
-- sample.sql
WITH col_format AS (
    SELECT
        DISTINCT
          t1.sku
        , t1.url
        , t1.price
        , t1.product
        , t1.brand
        , t1.size
        , t1.in_stock
        , t1.description
        , t1.collected_at
    FROM (
            SELECT
                 DISTINCT
                      sku
                    , url
                    , prdi_2.value:element:productPrice::float as price
                    , prdi_2.value:element:productName::text as product
                    , prdi_2.value:element:categoryName::text as brand
                    , d2.value:element::text as description
                    , ss3.value:in_stock::boolean as in_stock
                    , ss3.value:size::float as size
                    , collected_at
            FROM COTATENIS_DEV.ANALYTICS.stg_gdlp as store,
            lateral flatten(input => sizes_and_stock) ss1,
            lateral flatten(input => ss1.value) ss2,
            lateral flatten(input => ss2.value) ss3,
            lateral flatten(input => product_info) prdi_1,
            lateral flatten(input => prdi_1.value) prdi_2,
            lateral flatten(input => parse_json(description)) d1,
            lateral flatten(input => d1.value) d2
            WHERE prdi_2.value:element:productName NOT LIKE '%CHINELO%'
            AND ss3.value:size is NOT NULL
      ) t1
), most_recent_records as (
    SELECT * 
    FROM (
        SELECT
          DISTINCT sku
        , url
        , price
        , product
        , brand
        , description
        , size
        , in_stock
        , MAX(collected_at) as collected_at
    FROM col_format
    GROUP BY sku, url, price, product, brand, description, size, in_stock
    ) 
) SELECT 
      SUBSTR(sku, 0, 6) as sku
    , url
    , price
    , product
    , description
    , collected_at
    , brand
    , in_stock
    , -1 as qty_stock
    , size
    , 'GDLP' as store
    FROM most_recent_records
    where true
    qualify ROW_NUMBER() OVER(PARTITION BY sku, size ORDER BY collected_at DESC) = 1
```
Let's parse this code with the linter and take a look in the output.

```bash
sqlfluff lint sample.sql
```

```text
== [sample.sql] FAIL
L:   1 | P:   1 | L050 | Files must not begin with newlines or whitespace.
L:   3 | P:   5 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:   4 | P:   9 | L003 | Expected 1 indentations, found 2 [compared to line 03]
L:   5 | P:  11 | L003 | Expected 3 indentations, found 2 [compared to line 03]
L:  15 | P:  13 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:  15 | P:  13 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:  16 | P:  18 | L003 | Expected 3 indentations, found 4 [compared to line 15]
L:  17 | P:  23 | L003 | Expected 6 indentations, found 5 [compared to line 15]
L:  18 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  19 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  19 | P:  64 | L010 | Keywords must be consistently upper case.
L:  20 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  20 | P:  62 | L010 | Keywords must be consistently upper case.
L:  21 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  21 | P:  63 | L010 | Keywords must be consistently upper case.
L:  22 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  22 | P:  46 | L010 | Keywords must be consistently upper case.
L:  23 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  23 | P:  51 | L010 | Keywords must be consistently upper case.
L:  24 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  24 | P:  45 | L010 | Keywords must be consistently upper case.
L:  25 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  26 | P:  18 | L014 | Unquoted identifiers must be consistently lower case.
L:  26 | P:  32 | L014 | Unquoted identifiers must be consistently lower case.
L:  26 | P:  51 | L010 | Keywords must be consistently upper case.
L:  26 | P:  54 | L025 | Alias 'store' is never used in SELECT statement.
L:  26 | P:  54 | L031 | Avoid aliases in from clauses and join conditions.
L:  26 | P:  59 | L019 | Found trailing comma. Expected only leading.
L:  27 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  27 | P:  13 | L010 | Keywords must be consistently upper case.
L:  27 | P:  55 | L011 | Implicit/explicit aliasing of table.
L:  27 | P:  55 | L025 | Alias 'ss1' is never used in SELECT statement.
L:  27 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  28 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  28 | P:  13 | L010 | Keywords must be consistently upper case.
L:  28 | P:  49 | L011 | Implicit/explicit aliasing of table.
L:  28 | P:  49 | L025 | Alias 'ss2' is never used in SELECT statement.
L:  28 | P:  52 | L019 | Found trailing comma. Expected only leading.
L:  29 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  29 | P:  13 | L010 | Keywords must be consistently upper case.
L:  29 | P:  49 | L011 | Implicit/explicit aliasing of table.
L:  29 | P:  52 | L019 | Found trailing comma. Expected only leading.
L:  30 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  30 | P:  13 | L010 | Keywords must be consistently upper case.
L:  30 | P:  52 | L011 | Implicit/explicit aliasing of table.
L:  30 | P:  52 | L025 | Alias 'prdi_1' is never used in SELECT statement.
L:  30 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  31 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  31 | P:  13 | L010 | Keywords must be consistently upper case.
L:  31 | P:  52 | L011 | Implicit/explicit aliasing of table.
L:  31 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  32 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  32 | P:  13 | L010 | Keywords must be consistently upper case.
L:  32 | P:  63 | L011 | Implicit/explicit aliasing of table.
L:  32 | P:  63 | L025 | Alias 'd1' is never used in SELECT statement.
L:  32 | P:  65 | L019 | Found trailing comma. Expected only leading.
L:  33 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  33 | P:  13 | L010 | Keywords must be consistently upper case.
L:  33 | P:  48 | L011 | Implicit/explicit aliasing of table.
L:  35 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 34]
L:  35 | P:  32 | L010 | Keywords must be consistently upper case.
L:  36 | P:   7 | L003 | Expected 2 indentations, found 1 [compared to line 14]
L:  36 | P:   9 | L011 | Implicit/explicit aliasing of table.
L:  37 | P:   4 | L022 | Blank line expected but not found after CTE closing
                       | bracket.
L:  37 | P:  24 | L010 | Keywords must be consistently upper case.
L:  38 | P:  13 | L001 | Unnecessary trailing whitespace.
L:  40 | P:   9 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:  41 | P:  11 | L003 | Expected 2 indentations, found 2 [compared to line 40]
L:  41 | P:  11 | L021 | Ambiguous use of 'DISTINCT' in a 'SELECT' statement with
                       | 'GROUP BY'.
L:  42 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  43 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  44 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  45 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  46 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  47 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  48 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  49 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  49 | P:  11 | L030 | Function names must be consistently lower case.
L:  49 | P:  29 | L010 | Keywords must be consistently upper case.
L:  50 | P:   5 | L003 | Expected 2 indentations, found 1 [compared to line 40]
L:  51 | P:   5 | L003 | Expected 2 indentations, found 1 [compared to line 40]
L:  52 | P:   6 | L001 | Unnecessary trailing whitespace.
L:  53 | P:   3 | L022 | Blank line expected but not found after CTE closing
                       | bracket.
L:  53 | P:   3 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:  53 | P:   9 | L001 | Unnecessary trailing whitespace.
L:  54 | P:   7 | L003 | Expected 2 indentations, found 1 [compared to line 53]
L:  54 | P:   7 | L030 | Function names must be consistently lower case.
L:  54 | P:  25 | L010 | Keywords must be consistently upper case.
L:  62 | P:  10 | L010 | Keywords must be consistently upper case.
L:  64 | P:  14 | L010 | Keywords must be consistently upper case.
L:  65 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  66 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  66 | P:   5 | L010 | Keywords must be consistently upper case.
L:  67 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  67 | P:   5 | L010 | Keywords must be consistently upper case.
L:  67 | P:  13 | L030 | Function names must be consistently lower case.
L:  67 | P:  85 | L009 | Files must end with a single trailing newline.
All Finished!
```

😲 we've got more than 100 warnings!

The next step is to resort with the `fix` command to try to resolve these inconsistencies. This option has the default behavior of opening a prompt asking the programmer if you are certain to do these modifications in your code. If you want to skip this prompt we could add a flag `-f` to the command.

```shell
sqlfluff fix -f sample.sql
```

```text
==== finding fixable violations ====
== [sample.sql] FAIL                                                                                                        
L:   1 | P:   1 | L050 | Files must not begin with newlines or whitespace.                                                  
L:   3 | P:   5 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:   4 | P:   9 | L003 | Expected 1 indentations, found 2 [compared to line 03]
L:   5 | P:  11 | L003 | Expected 3 indentations, found 2 [compared to line 03]
L:  15 | P:  13 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:  15 | P:  13 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:  16 | P:  18 | L003 | Expected 3 indentations, found 4 [compared to line 15]
L:  17 | P:  23 | L003 | Expected 6 indentations, found 5 [compared to line 15]
L:  18 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  19 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  19 | P:  64 | L010 | Keywords must be consistently upper case.
L:  20 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  20 | P:  62 | L010 | Keywords must be consistently upper case.
L:  21 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  21 | P:  63 | L010 | Keywords must be consistently upper case.
L:  22 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  22 | P:  46 | L010 | Keywords must be consistently upper case.
L:  23 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  23 | P:  51 | L010 | Keywords must be consistently upper case.
L:  24 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  24 | P:  45 | L010 | Keywords must be consistently upper case.
L:  25 | P:  21 | L003 | Expected 3 indentations, found 5 [compared to line 15]
L:  26 | P:  18 | L014 | Unquoted identifiers must be consistently lower case.
L:  26 | P:  32 | L014 | Unquoted identifiers must be consistently lower case.
L:  26 | P:  51 | L010 | Keywords must be consistently upper case.
L:  26 | P:  54 | L025 | Alias 'store' is never used in SELECT statement.
L:  26 | P:  59 | L019 | Found trailing comma. Expected only leading.
L:  27 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  27 | P:  13 | L010 | Keywords must be consistently upper case.
L:  27 | P:  55 | L011 | Implicit/explicit aliasing of table.
L:  27 | P:  55 | L025 | Alias 'ss1' is never used in SELECT statement.
L:  27 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  28 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  28 | P:  13 | L010 | Keywords must be consistently upper case.
L:  28 | P:  49 | L011 | Implicit/explicit aliasing of table.
L:  28 | P:  49 | L025 | Alias 'ss2' is never used in SELECT statement.
L:  28 | P:  52 | L019 | Found trailing comma. Expected only leading.
L:  29 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  29 | P:  13 | L010 | Keywords must be consistently upper case.
L:  29 | P:  49 | L011 | Implicit/explicit aliasing of table.
L:  29 | P:  52 | L019 | Found trailing comma. Expected only leading.
L:  30 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  30 | P:  13 | L010 | Keywords must be consistently upper case.
L:  30 | P:  52 | L011 | Implicit/explicit aliasing of table.
L:  30 | P:  52 | L025 | Alias 'prdi_1' is never used in SELECT statement.
L:  30 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  31 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  31 | P:  13 | L010 | Keywords must be consistently upper case.
L:  31 | P:  52 | L011 | Implicit/explicit aliasing of table.
L:  31 | P:  58 | L019 | Found trailing comma. Expected only leading.
L:  32 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  32 | P:  13 | L010 | Keywords must be consistently upper case.
L:  32 | P:  63 | L011 | Implicit/explicit aliasing of table.
L:  32 | P:  63 | L025 | Alias 'd1' is never used in SELECT statement.
L:  32 | P:  65 | L019 | Found trailing comma. Expected only leading.
L:  33 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 26]
L:  33 | P:  13 | L010 | Keywords must be consistently upper case.
L:  33 | P:  48 | L011 | Implicit/explicit aliasing of table.
L:  35 | P:  13 | L003 | Expected 4 indentations, found 3 [compared to line 34]
L:  35 | P:  32 | L010 | Keywords must be consistently upper case.
L:  36 | P:   7 | L003 | Expected 2 indentations, found 1 [compared to line 14]
L:  36 | P:   9 | L011 | Implicit/explicit aliasing of table.
L:  37 | P:   4 | L022 | Blank line expected but not found after CTE closing
                       | bracket.
L:  37 | P:  24 | L010 | Keywords must be consistently upper case.
L:  38 | P:  13 | L001 | Unnecessary trailing whitespace.
L:  40 | P:   9 | L041 | 'SELECT' modifiers (e.g. 'DISTINCT') must be on the same
                       | line as 'SELECT'.
L:  41 | P:  11 | L003 | Expected 2 indentations, found 2 [compared to line 40]
L:  42 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  42 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  43 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  43 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  44 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  44 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  45 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  45 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  46 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  46 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  47 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  47 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  48 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  48 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  49 | P:   9 | L003 | Expected 3 indentations, found 2 [compared to line 40]
L:  49 | P:   9 | L005 | Commas should not have whitespace directly before them.
L:  49 | P:  11 | L030 | Function names must be consistently lower case.
L:  49 | P:  29 | L010 | Keywords must be consistently upper case.
L:  50 | P:   5 | L003 | Expected 2 indentations, found 1 [compared to line 40]
L:  51 | P:   5 | L003 | Expected 2 indentations, found 1 [compared to line 40]
L:  52 | P:   6 | L001 | Unnecessary trailing whitespace.
L:  53 | P:   3 | L022 | Blank line expected but not found after CTE closing
                       | bracket.
L:  53 | P:   3 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:  53 | P:   9 | L001 | Unnecessary trailing whitespace.
L:  54 | P:   7 | L003 | Expected 2 indentations, found 1 [compared to line 53]
L:  54 | P:   7 | L030 | Function names must be consistently lower case.
L:  54 | P:  25 | L010 | Keywords must be consistently upper case.
L:  62 | P:  10 | L010 | Keywords must be consistently upper case.
L:  64 | P:  14 | L010 | Keywords must be consistently upper case.
L:  65 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  66 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  66 | P:   5 | L010 | Keywords must be consistently upper case.
L:  67 | P:   5 | L003 | Expected 0 indentations, found 1 [compared to line 53]
L:  67 | P:   5 | L010 | Keywords must be consistently upper case.
L:  67 | P:  13 | L030 | Function names must be consistently lower case.
L:  67 | P:  85 | L009 | Files must end with a single trailing newline.
==== fixing violations ====
103 fixable linting violations found
FORCE MODE: Attempting fixes...
Persisting Changes...
== [sample.sql] PASS
Done. Please check your files to confirm.
  [1 unfixable linting violations found]
```

Then we can sneak peek at our code to evaluate the changes that linter has done.

```sql
-- sample.sql
WITH col_format AS (
    SELECT DISTINCT
        t1.sku
        , t1.url
        , t1.price
        , t1.product
        , t1.brand
        , t1.size
        , t1.in_stock
        , t1.description
        , t1.collected_at
    FROM (
            SELECT DISTINCT
                sku
                , url
                , collected_at
                , prdi_2.value:element:productPrice::float AS price
                , prdi_2.value:element:productName::text AS product
                , prdi_2.value:element:categoryName::text AS brand
                , d2.value:element::text AS description
                , ss3.value:in_stock::boolean AS in_stock
                , ss3.value:size::float AS size
            FROM cotatenis_dev.analytics.stg_gdlp
            ,  LATERAL flatten(input => sizes_and_stock)
            ,  LATERAL flatten(input => ss1.value)
            ,  LATERAL flatten(input => ss2.value) AS ss3
            ,  LATERAL flatten(input => product_info)
            ,  LATERAL flatten(input => prdi_1.value) AS prdi_2
            ,  LATERAL flatten(input => parse_json(description))
            ,  LATERAL flatten(input => d1.value) AS d2
            WHERE prdi_2.value:element:productName NOT LIKE '%CHINELO%'
                AND ss3.value:size IS NOT NULL
        ) AS t1
)

, most_recent_records AS (
    SELECT *
    FROM (
        SELECT DISTINCT
            sku
            , url
            , price
            , product
            , brand
            , description
            , size
            , in_stock
            , max(collected_at) AS collected_at
        FROM col_format
        GROUP BY sku, url, price, product, brand, description, size, in_stock
    )
)

SELECT
    url
    , price
    , product
    , description
    , collected_at
    , brand
    , in_stock
    , size
    , 'GDLP' AS store
    , substr(sku, 0, 6) AS sku
    , -1 AS qty_stock
FROM most_recent_records
WHERE true
QUALIFY row_number() OVER(PARTITION BY sku, size ORDER BY collected_at DESC) = 1
```
Once our code was modified we can run the linter again to check out the result.

```shell
sqlfluff lint sample.sql
```

```text
== [sample.sql] FAIL
L:  24 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  24 | P:  14 | L039 | Unnecessary whitespace found.
L:  25 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  25 | P:  14 | L039 | Unnecessary whitespace found.
L:  26 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  26 | P:  14 | L039 | Unnecessary whitespace found.
L:  27 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  27 | P:  14 | L039 | Unnecessary whitespace found.
L:  28 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  28 | P:  14 | L039 | Unnecessary whitespace found.
L:  29 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  29 | P:  14 | L039 | Unnecessary whitespace found.
L:  30 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  30 | P:  14 | L039 | Unnecessary whitespace found.
L:  39 | P:  16 | L021 | Ambiguous use of 'DISTINCT' in a 'SELECT' statement with
                       | 'GROUP BY'.
All Finished!
```

We've reduced our warnings from a hundred to a couple of units 😀. Most of them are regarding are about whitespaces and commas ([L008](https://docs.sqlfluff.com/en/stable/rules.html#sqlfluff.core.rules.Rule_L008) and [L039](https://docs.sqlfluff.com/en/stable/rules.html#sqlfluff.core.rules.Rule_L039)).

Furthermore was detected a violation about the rule [L021](https://docs.sqlfluff.com/en/stable/rules.html#sqlfluff.core.rules.Rule_L021). One the coolest thing about that rules documentation is that it can be thought as a collection of anti-pattern for SQL Code. Each one of them has some examples of anti-patterns and best practices to follow. The following figure shows an example for the rule `L021`.

![sqlfluff_rule_l021](/sqlfluff_intro/sqlfluff_RL021.png)

Our next step is to get rid of the DISTINCT clause from line 39 and to run again the `fix` command.

```shell
sqlfluff fix -f sample.sql
```

```text
==== finding fixable violations ====
== [sample.sql] FAIL
L:  24 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  25 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  26 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  27 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  28 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  29 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
L:  30 | P:  14 | L008 | Commas should be followed by a single whitespace unless
                       | followed by a comment.
==== fixing violations ====
7 fixable linting violations found
FORCE MODE: Attempting fixes...
Persisting Changes...
== [sample.sql] PASS
Done. Please check your files to confirm.
  [1 unfixable linting violations found]
```

Unfortunately, it remains a couple of warnings regarding the rule L008 and the linter printed out an important message at the end: [1 unfixable linting violations found]. A final workaround is to manually edit to insert only one space after the commas through lines 24 to 30. As a final check, we must run the `lint` command to guarantee that our code meets the specification.

```shell
sqlfluff lint sample.sql                                
> All Finished 📜 🎉!    
```

👌 Well done! We've achieved a code style that doesn't generate any warning against the linter specification. Then we can take a look at our code.

```sql
WITH col_format AS (
    SELECT DISTINCT
        t1.sku
        , t1.url
        , t1.price
        , t1.product
        , t1.brand
        , t1.size
        , t1.in_stock
        , t1.description
        , t1.collected_at
    FROM (
            SELECT DISTINCT
                sku
                , url
                , collected_at
                , prdi_2.value:element:productPrice::float AS price
                , prdi_2.value:element:productName::text AS product
                , prdi_2.value:element:categoryName::text AS brand
                , d2.value:element::text AS description
                , ss3.value:in_stock::boolean AS in_stock
                , ss3.value:size::float AS size
            FROM cotatenis_dev.analytics.stg_gdlp
            , LATERAL flatten(input => sizes_and_stock)
            , LATERAL flatten(input => ss1.value)
            , LATERAL flatten(input => ss2.value) AS ss3
            , LATERAL flatten(input => product_info)
            , LATERAL flatten(input => prdi_1.value) AS prdi_2
            , LATERAL flatten(input => parse_json(description))
            , LATERAL flatten(input => d1.value) AS d2
            WHERE prdi_2.value:element:productName NOT LIKE '%CHINELO%'
                AND ss3.value:size IS NOT NULL
        ) AS t1
)

, most_recent_records AS (
    SELECT *
    FROM (
        SELECT
            sku
            , url
            , price
            , product
            , brand
            , description
            , size
            , in_stock
            , max(collected_at) AS collected_at
        FROM col_format
        GROUP BY sku, url, price, product, brand, description, size, in_stock
    )
)

SELECT
    url
    , price
    , product
    , description
    , collected_at
    , brand
    , in_stock
    , size
    , 'GDLP' AS store
    , substr(sku, 0, 6) AS sku
    , -1 AS qty_stock
FROM most_recent_records
WHERE true
QUALIFY row_number() OVER(PARTITION BY sku, size ORDER BY collected_at DESC) = 1
```

For today that's all folks 🤟! I hope that you've enjoyed so far and I'll try to bring in my next post a configuration for sqlfluff working together with [pre-commit](https://pre-commit.com/) to work with this linter with a better flow and better automated.