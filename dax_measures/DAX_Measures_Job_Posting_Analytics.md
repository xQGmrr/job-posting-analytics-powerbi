# Power BI Job Posting Analytics --- DAX Measures

## Overview

This document contains the DAX measures used in the **Power BI Job
Posting Analytics** project.

All measures are organized in a dedicated **measures table** in the
Power BI model.

> **Important:** The formulas below are documented exactly as
> implemented in the project. No optimization or logic changes have been
> applied.

## 1. Job & Skills KPIs

### Job count

``` dax
Job count = COUNT(job_postings_fact[job_id])
```

Counts the number of non-blank job IDs.

### Skills count

``` dax
Skills count = COUNT(skills_job_dim[skill_id])
```

Counts skill-job records in the bridge table.

### Skills per job

``` dax
Skills per job = DIVIDE([Skills count],[Job count])
```

Calculates the average number of recorded skills per job.

### All_Skills

``` dax
All_Skills = CALCULATE(COUNT(skills_job_dim[skill_id]),REMOVEFILTERS(skills_job_dim))
```

Calculates the total number of skill records while removing filters from
`skills_job_dim`.

### Skill proportion

``` dax
Skill proportion = DIVIDE([Skills count],[All_Skills],"Not Asked") 
```

Calculates the proportion of skill records represented by the current
context.

## 2. Salary Analysis

### Median salary

``` dax
Median salary = MEDIAN(job_postings_fact[salary_hour_or_year_avg])
```

Calculates the median salary.

### Average Salary

``` dax
Average Salary = AVERAGE(job_postings_fact[salary_hour_or_year_avg])
```

Calculates the arithmetic mean salary.

### Highest Salary

``` dax
Highest Salary = MAX(job_postings_fact[salary_hour_or_year_avg])
```

Returns the highest salary.

### Lowest Salary

``` dax
Lowest Salary = MIN(job_postings_fact[salary_hour_or_year_avg])
```

Returns the lowest salary.

## 3. Company & Platform KPIs

### Companies count

``` dax
Companies count = DISTINCTCOUNT(company_dim[company_id])
```

Counts distinct companies.

### Jobs Platform

``` dax
Jobs Platform = DISTINCTCOUNT(job_postings_fact[job_via])
```

Counts distinct job posting platforms.

## 4. Dynamic Ranking Measures

### Most Wanted Job

``` dax
Most Wanted Job = 

  VAR JobRanking =

      TOPN(

          1,

          VALUES(job_postings_fact[job_title_short]),

          CALCULATE(DISTINCTCOUNT(job_postings_fact[job_id])),

          DESC

      )

  RETURN

      CONCATENATEX(

          JobRanking,

          job_postings_fact[job_title_short],

          ", "

      )
```

Returns the job title with the highest number of distinct job postings
in the current filter context.

Functions used: `VAR`, `TOPN`, `VALUES`, `CALCULATE`, `DISTINCTCOUNT`,
`CONCATENATEX`.

### Most Wanted Skill

``` dax
Most Wanted Skill = 

  VAR SkillRanking =

      TOPN(

          1,

          VALUES(skills_dim[skills]),

          CALCULATE(DISTINCTCOUNT(skills_job_dim[job_id])),

          DESC

      )

  RETURN

      CONCATENATEX(

          SkillRanking,

          skills_dim[skills],

          ", "

      )
```

Returns the skill associated with the highest number of distinct job
postings in the current filter context.

Functions used: `VAR`, `TOPN`, `VALUES`, `CALCULATE`, `DISTINCTCOUNT`,
`CONCATENATEX`.

### Least Wanted Skill

``` dax
Least Wanted Skill = 

  VAR SkillTable =

      FILTER(

          ADDCOLUMNS(

              VALUES(skills_dim[skills]),

              "JobCount",

                  CALCULATE(

                      DISTINCTCOUNT(skills_job_dim[job_id])

                  )

          ),

          [JobCount] > 1

      )

  VAR MinCount =

      MINX(

          SkillTable,

          [JobCount]

      )

  VAR LeastSkill =

      TOPN(

          1,

          FILTER(

              SkillTable,

              [JobCount] = MinCount

          ),

          skills_dim[skills],

          ASC

      )

  RETURN

      MAXX(

          LeastSkill,

          skills_dim[skills]

      )
```

Returns the least requested skill among skills associated with more than
one job. If multiple skills share the minimum count, the alphabetical
ordering in `TOPN` is used.

Functions used: `VAR`, `FILTER`, `ADDCOLUMNS`, `VALUES`, `CALCULATE`,
`DISTINCTCOUNT`, `MINX`, `TOPN`, `MAXX`.

## 5. DAX Functions Used

  Function          Purpose
  ----------------- ------------------------------------------------------------
  `COUNT`           Counts non-blank values
  `DISTINCTCOUNT`   Counts unique values
  `DIVIDE`          Performs safe division
  `MEDIAN`          Calculates the median
  `AVERAGE`         Calculates the average
  `MAX`             Returns the maximum
  `MIN`             Returns the minimum
  `CALCULATE`       Evaluates an expression in a modified filter context
  `REMOVEFILTERS`   Clears filters from a table or column
  `VALUES`          Returns distinct values in the current context
  `TOPN`            Returns the top N rows according to an ordering expression
  `CONCATENATEX`    Converts table values into text
  `FILTER`          Filters a table expression
  `ADDCOLUMNS`      Adds calculated columns to a table expression
  `MINX`            Returns the minimum evaluated across a table
  `MAXX`            Returns the maximum evaluated across a table
  `VAR`             Stores intermediate results

## 6. Filter Context

The measures are designed to respond to Power BI slicers and visual
filters such as job title, country, salary rate, work mode, company,
skill, and date.

This is especially important for the dynamic measures `Most Wanted Job`,
`Most Wanted Skill`, and `Least Wanted Skill`.

`CALCULATE` evaluates an expression in a modified filter context, while
`REMOVEFILTERS` clears filters from specified tables or columns.

## 7. Measure Table Organization

All DAX measures are stored in one dedicated measures table rather than
being distributed across the model tables.

``` text
Power BI Model
├── job_postings_fact
├── company_dim
├── skills_dim
├── skills_job_dim
├── schedule_dim
├── date_dim
└── Measures Table
    ├── Job & Skills KPIs
    ├── Salary Measures
    ├── Company & Platform Measures
    └── Dynamic Ranking Measures
```

## 8. Measure Summary

  -----------------------------------------------------------------------
  Measure                 Category                Purpose
  ----------------------- ----------------------- -----------------------
  `Job count`             Job KPI                 Count job postings

  `Skills count`          Skill KPI               Count skill-job records

  `Skills per job`        Skill KPI               Average skills per job

  `All_Skills`            Skill Analysis          Total skill records
                                                  ignoring bridge-table
                                                  filters

  `Skill proportion`      Skill Analysis          Proportion of total
                                                  skill records

  `Median salary`         Salary                  Median salary

  `Average Salary`        Salary                  Average salary

  `Highest Salary`        Salary                  Highest salary

  `Lowest Salary`         Salary                  Lowest salary

  `Companies count`       Company KPI             Distinct company count

  `Jobs Platform`         Platform KPI            Distinct platform count

  `Most Wanted Job`       Ranking                 Job title with the most
                                                  distinct postings

  `Most Wanted Skill`     Ranking                 Skill associated with
                                                  the most distinct jobs

  `Least Wanted Skill`    Ranking                 Least requested skill
                                                  among skills with more
                                                  than one associated job
  -----------------------------------------------------------------------

## 9. Dashboard Usage

### Global Job Market Overview

Key measures include `Job count`, `Median salary`, `Companies count`,
and `Skills per job`.

### Salary Analysis

Key measures include `Average Salary`, `Median salary`,
`Highest Salary`, `Lowest Salary`, and `Skills per job`.

### Skills Analysis

Key measures include `Skills count`, `Skills per job`,
`Most Wanted Skill`, `Least Wanted Skill`, and `Skill proportion`.

### Company & Location Analysis

Key measures include `Companies count`, `Median salary`,
`Lowest Salary`, and `Jobs Platform`.

### Date & Time Analysis

Key measures include `Companies count`, `Median salary`,
`Lowest Salary`, and `Most Wanted Job`.

## 10. Key DAX Concepts Demonstrated

-   Aggregation functions
-   Distinct counting
-   Safe division
-   Filter context
-   Filter removal
-   Table expressions
-   Variables
-   Iterator functions
-   Dynamic ranking
-   Text generation from table expressions
-   Part-to-whole calculations
-   KPI development
-   Context-aware analytical measures

## References

-   Microsoft Learn --- DAX overview:
    https://learn.microsoft.com/en-us/dax/dax-overview
-   Microsoft Learn --- DAX function reference:
    https://learn.microsoft.com/en-us/dax/dax-function-reference
-   Microsoft Learn --- CALCULATE:
    https://learn.microsoft.com/en-us/dax/calculate-function-dax
-   Microsoft Learn --- TOPN:
    https://learn.microsoft.com/en-us/dax/topn-function-dax
