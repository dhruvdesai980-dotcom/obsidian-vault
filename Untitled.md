```sql
select

strategist_id,

strategist_name,

strategist_name_comma_format,

position,

strategist_country,

strategist_team,

product_covered,

strategist_region,

city,

active_status

  

from {{ ref('dbt_poc_stg_cib_strategist') }}
```