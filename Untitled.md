```
version: 2

sources:

- name: cib_raw

description: Existing CIB source tables in the Databricks raw schema

database: d4001-centralus-tdvip-tdsbi_mstrt_catalog

schema: raw

tables:

- name: cib_tbl_meetingtracker

description: Raw meeting-tracker data

- name: cib_lookup_strategist

description: Lookup table containing strategist details

columns:

- name: StrategistID

description: Unique strategist identifier

data_tests:

- not_null

- unique
```