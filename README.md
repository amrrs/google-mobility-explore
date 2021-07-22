# google-mobility-explore
Exploring Google Mobility COVID dataset from GCP - BigQuery

`SELECT * FROM `bigquery-public-data.covid19_google_mobility.mobility_report` LIMIT 1000` 


Meta

* Understanding the data - Is it time series? What's the granularity of the data? 

Yes it's a TS - with daily data for each sub-region

```sql
SELECT * FROM `bigquery-public-data.covid19_google_mobility.mobility_report` 
where country_region_code = 'IN'
and sub_region_2 = 'Bangalore Urban'
order by date 

```


Business Questions

* Average change in Baseline Bangalore Urban for all indicators

```sql
SELECT 
avg(retail_and_recreation_percent_change_from_baseline) as retail_avg_change
FROM `bigquery-public-data.covid19_google_mobility.mobility_report` 
where 1=1
    and country_region_code = 'IN'
    and sub_region_2 = 'Bangalore Urban' 


```


```sql

SELECT
 substr(string(date),1,4) as year 
 ,avg(retail_and_recreation_percent_change_from_baseline) as retail_avg_change
 ,avg(parks_percent_change_from_baseline) as park_avg_change
 ,avg(transit_stations_percent_change_from_baseline) as transit_avg_change 
FROM
 `bigquery-public-data.covid19_google_mobility.mobility_report`
where 1=1
    and country_region_code = 'IN'
    and sub_region_2 = 'Bangalore Urban' 
group by 1 
```


* Compare US and India on all indicators 


```sql

SELECT
 substr(string(date),1,4) as year 
 ,country_region_code
 ,avg(retail_and_recreation_percent_change_from_baseline) as retail_avg_change
 ,avg(parks_percent_change_from_baseline) as park_avg_change
 ,avg(transit_stations_percent_change_from_baseline) as transit_avg_change 
 ,avg(workplaces_percent_change_from_baseline) as workplace_avg_change
 ,avg(residential_percent_change_from_baseline) as residential_avg_change 
 ,avg(grocery_and_pharmacy_percent_change_from_baseline) as grocery_avg_change
FROM
 `bigquery-public-data.covid19_google_mobility.mobility_report`
where 1=1
    and country_region_code in ('IN', 'US')
    --and sub_region_2 = 'Bangalore Urban' 
    and sub_region_1 is NULL 
group by 1,2
order by 1 

```

![image](https://user-images.githubusercontent.com/5347322/126631391-8457db75-8198-4dcf-a479-ca0af51a12f2.png)


* Peak and Dips of India (change from baseline) 


```sql


SELECT
date,
country_region_code
 ,retail_and_recreation_percent_change_from_baseline
 ,parks_percent_change_from_baseline 
 ,transit_stations_percent_change_from_baseline 
 ,workplaces_percent_change_from_baseline
 ,residential_percent_change_from_baseline
FROM
 `bigquery-public-data.covid19_google_mobility.mobility_report`
where 1=1
    and country_region_code in ('IN', 'US')
    and sub_region_1 is NULL 

```

![image](https://user-images.githubusercontent.com/5347322/126632795-9d9fb270-81fa-4b13-8040-45069c6b6e1d.png)


```sql

with country_retail_min_max as 
(SELECT
country_region_code
 ,max(retail_and_recreation_percent_change_from_baseline) as max_retail
 ,min(retail_and_recreation_percent_change_from_baseline) as min_retail
--  ,parks_percent_change_from_baseline 
--  ,transit_stations_percent_change_from_baseline 
--  ,workplaces_percent_change_from_baseline
--  ,residential_percent_change_from_baseline
FROM
 `bigquery-public-data.covid19_google_mobility.mobility_report`
where 1=1
    and country_region_code in ('IN', 'US')
    and sub_region_1 is NULL 
    and date > date('2020-04-01')
group by 1
)

select *  
FROM country_retail_min_max retail 
 join `bigquery-public-data.covid19_google_mobility.mobility_report` mob
on mob.country_region_code = retail.country_region_code
    and ( mob.retail_and_recreation_percent_change_from_baseline = retail.max_retail
    or mob.retail_and_recreation_percent_change_from_baseline = retail.min_retail)
where 1=1
    and mob.country_region_code in ('IN', 'US')
    and sub_region_1 is NULL 
    and date > date('2020-04-01')
order by date 
```


