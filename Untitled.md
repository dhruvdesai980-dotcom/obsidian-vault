```sql
select

sha2

(

concat_ws(

'||',

cast(meeting_id as string),

cast(account_id as string),

cast(strategist_id as string)

),

256

) as meeting_activity_id,

*

from {{ ref('dbt_poc_stg_cib_meetingtracker') }}
```

```yaml
version: 2

models:

- name: fct_meeting_activity

description: Meeting activity fact table.

primary_key:

- meeting_activity_id

columns:

- name: meeting_activity_id

data_tests:

- not_null

- unique

- name: meeting_id

data_tests:

- not_null

- name: meeting_date

data_tests:

- not_null

- name: account_id

data_tests:

- not_null

- name: strategist_id

data_tests:

- not_null

- relationships:

to: ref('dim_strategist')

field: strategist_id

semantic_models:

- name: meeting_activity

description: Semantic model for meeting activity analysis.

model: ref('fct_meeting_activity')

defaults:

agg_time_dimension: meeting_date

entities:

- name: meeting_activity

type: primary

expr: meeting_activity_id

- name: meeting

type: foreign

expr: meeting_id

- name: account

type: foreign

expr: account_id

- name: strategist

type: foreign

expr: strategist_id

dimensions:

- name: meeting_date

type: time

type_params:

time_granularity: day

- name: data_upload_datetime

type: time

type_params:

time_granularity: second

- name: meeting_type

type: categorical

- name: primary_sales

type: categorical

- name: other_sales

type: categorical

- name: other_strategist

type: categorical

- name: topic_discussed

type: categorical

- name: desk

type: categorical

- name: client_country_state_provience

type: categorical

- name: client_area

type: categorical

- name: client_region

type: categorical

measures:

- name: meeting_record_count_measure

agg: count_distinct

expr: meeting_activity_id

- name: meeting_count_measure

agg: count_distinct

expr: meeting_id

- name: account_count_measure

agg: count_distinct

expr: account_id

- name: strategist_count_measure

agg: count_distinct

expr: strategist_id

metrics:

- name: meeting_record_count

label: Meeting Record Count

type: simple

type_params:

measure: meeting_record_count_measure

- name: meeting_count

label: Meeting Count

type: simple

type_params:

measure: meeting_count_measure

- name: account_count

label: Account Count

type: simple

type_params:

measure: account_count_measure

- name: strategist_count

label: Strategist Count

type: simple

type_params:

measure: strategist_count_measure
```