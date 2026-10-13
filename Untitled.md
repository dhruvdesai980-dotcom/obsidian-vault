```
with source_data as (

select *

from {{ source('cib_raw', 'cib_tbl_meetingtracker') }}

),

  

renamed as (

select

cast(MeetingID as bigint) as meeting_id,

cast(`Date` as date) as meeting_date,

cast(DataUploadDateTime as timestamp) as data_upload_datetime,

  

cast(StrategistID as bigint) as strategist_id,

cast(AccountID as bigint) as account_id,

  

trim(MeetingType) as meeting_type,

trim(PrimarySales) as primary_sales,

trim(OtherSales) as other_sales,

trim(OtherStrat) as other_strategist,

  

trim(TopicDiscussed) as topic_discussed,

trim(Desk) as desk,

  

trim(ClientCountryStateProv) as client_country_state_provience,

trim(ClientArea) as client_area,

trim(ClientRegion) as client_region

from source_data

)

  

select *

from renamed
```

```
with source_data as (

  

select *

from {{ source('cib_raw', 'cib_lookup_strategist') }}

  

)

  

select

cast(StrategistID as bigint) as strategist_id,

nullif(trim(Strategists), '-') as strategist_name_comma_format,

case

when StrategistID = 1000 then 'Unknown Strategist'

else nullif(trim(StrategistsNoComma), '-')

end as strategist_name,

nullif(trim(Position), '-') as position,

nullif(trim(StrategistCountry), '-') as strategist_country,

nullif(trim(StrategistTeam), '-') as strategist_team,

nullif(trim(ProductCovered), '-') as product_covered,

nullif(trim(Region), '-') as strategist_region,

nullif(trim(City), '-') as city,

case

when lower(trim(ActiveInactive)) = 'active' then 'Active'

when lower(trim(ActiveInactive)) = 'inactive' then 'Inactive'

else 'Unknown'

end as active_status

  

from source_data
```