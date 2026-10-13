```YAML
version: 2

models:

- name: dim_strategist

description: Strategist dimension containing organizational and classification details

primary_key:

- strategist_id

columns:

- name: strategist_id

description: Unique strategist identifier

data_tests:

- not_null

- unique

- name: strategist_name

description: Display name of the strategist

- name: position

description: Strategist position or role

- name: strategist_country

description: Source-system strategist country value

- name: strategist_team

description: Team assigned to the strategist

- name: product_covered

description: Product covered by the strategist

- name: strategist_region

description: Region assigned to the strategist

- name: active_status

description: Indicates whether the strategist is active

data_tests:

- accepted_values:

values:

- Active

- Inactive

- Unknown

semantic_models:

- name: strategist

description: Semantic dimension containing strategist details

model: ref('dim_strategist')

entities:

- name: strategist

type: primary

expr: strategist_id

dimensions:

- name: strategist_name

type: categorical

- name: position

type: categorical

- name: strategist_country

type: categorical

- name: strategist_team

type: categorical

- name: product_covered

type: categorical

- name: strategist_region

type: categorical

- name: city

type: categorical

- name: active_status

type: categorical
```