```sql
{{ config(materialized='table') }}

  

select

explode(

sequence(

to_date('2020-01-01'),

to_date('2035-12-31'),

interval 1 day

)

) as date_day
```

```yaml
version: 2

models:

- name: dim_date

description: Daily time spine used by the dbt Semantic Layer.

primary_key:

- date_day

time_spine:

standard_granularity_column: date_day

custom_granularities: []

columns:

- name: date_day

description: One calendar date per row.

granularity: day

data_tests:

- not_null

- unique
```