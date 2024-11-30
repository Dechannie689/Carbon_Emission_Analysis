# Carbon Emission Analysisn - SQL
## 1. Abstract
This SQL project delves into the fascinating realm of product carbon footprints (PCFs) to explore the environmental impact of various products. The dataset, sourced from nature.com, provides valuable insights into the greenhouse gas emissions associated with different companies and their products.
The primary focus of this analysis is to investigate and quantify the carbon footprint of products across different stages of production.
## 2. Methodology
### a. Data patterns
The dataset consists of four tables containing information on carbon emissions generated during the production of goods. Below is a snapshot of the data structure, highlighting key fields and relationships within the data:
![](https://github.com/Dechannie689/Carbon_Emission_Analysis/blob/main/carbon_emissions_data_struture.png)
### b. Solutions
Before analyzing, we need to clean the data step by step as follows:
#### Step 1: Identification of Duplicates
Duplicate entries are identified using a COUNT(1) > 1 query in SQL. This approach groups records by key identifiers to check for rows with the same values, which could distort the accuracy of carbon emission calculations.
```sql
select id, 
	count(id) as Num_Id
from product_emissions
group by id
having count(id) > 1
```
Result:
| id           | Num_Id | 
| -----------: | -----: | 
| 10056-1-2014 | 2      | 
| 10056-1-2015 | 2      | 
| 10222-1-2013 | 2      | 
| 10261-1-2017 | 2      | 
| 10261-2-2017 | 2      | 
| ... | ...      | 

There are a total of 171 rows with duplicate values
#### Step 2: Handling Missing and Invalid Values
Rows containing missing values or irrelevant data (e.g., records with entries such as “N/A”) are identified using a WHERE clause with NOT LIKE '%N/A%'. This filters out incomplete or non-numeric data that might otherwise skew results.
```sql
select *
from product_emissions
where upstream_percent_total_pcf not like '%N/A%'
	or operations_percent_total_pcf not like '%N/A%'
	or downstream_percent_total_pcf not like '%N/A%'
```
Result:

| id           | company_id | country_id | industry_group_id | year | product_name                                                    | weight_kg | carbon_footprint_pcf | upstream_percent_total_pcf | operations_percent_total_pcf | downstream_percent_total_pcf | 
| -----------: | ---------: | ---------: | ----------------: | ---: | --------------------------------------------------------------: | --------: | -------------------: | -------------------------: | ---------------------------: | ---------------------------: | 
| 10056-1-2014 | 82         | 28         | 2                 | 2014 | Frosted Flakes(R) Cereal                                        | 0.7485    | 2                    | 57.50                      | 30.00                        | 12.50                        | 
| 10056-1-2015 | 82         | 28         | 15                | 2015 | "Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)" | 0.7485    | 2                    | 57.50                      | 30.00                        | 12.50                        | 
| 10222-1-2013 | 83         | 28         | 8                 | 2013 | Office Chair                                                    | 20.68     | 73                   | 80.63                      | 17.36                        | 2.01                         | 
| 10261-1-2017 | 14         | 16         | 25                | 2017 | Multifunction Printers                                          | 110       | 1488                 | 30.65                      | 5.51                         | 63.84                        | 
| 10261-2-2017 | 14         | 16         | 25                | 2017 | Multifunction Printers                                          | 110       | 1818                 | 25.08                      | 4.51                         | 70.41                        | 

There are a total of 480 rows without missing values or irrelevant data.
## 3. Findings
### 1. Most contributed carbon emissions product
```sql
select product_name, 
	sum(carbon_footprint_pcf) as sum_car
from product_emissions
group by product_name
order by sum_car desc
```
Result:
| product_name                 | sum_car | 
| ---------------------------: | ------: | 
| Wind Turbine G128 5 Megawats | 3718044 | 
### 2. The industry groups of these products
```sql
select i.industry_group , p.product_name
from product_emissions p
left join industry_groups i on p.industry_group_id = i.id
group by i.industry_group, p.product_name
```
Result:
| industry_group                                       | product_name                       | 
| ---------------------------------------------------: | ---------------------------------: | 
| "Consumer Durables, Household and Personal Products" | "Sayl Work Chair, Suspension Back" | 
| "Consumer Durables, Household and Personal Products" | Aeron Chair                        | 
| "Consumer Durables, Household and Personal Products" | Caper Stacking Chair               | 
| "Consumer Durables, Household and Personal Products" | Celle Chair                        | 
| "Consumer Durables, Household and Personal Products" | Embody Chair                       | 
### 3. The industries with the highest contribution to carbon emissions
```sql
select i.industry_group, 
	sum(p.carbon_footprint_pcf) as highes_car
from product_emissions p
left join industry_groups i on p.industry_group_id = i.id
group by i.industry_group
order by highes_car desc
limit 1
```
Result:
| industry_group                     | highes_car | 
| ---------------------------------: | ---------: | 
| Electrical Equipment and Machinery | 9801558    | 
### 4. The companies with the highest contribution to carbon emissions
```sql
select c.company_name, 
	sum(p.carbon_footprint_pcf) as highes_car
from product_emissions p
left join companies c on p.company_id = c.id
group by c.company_name
order by highes_car desc
limit 1
```
Result:
| company_name                           | highes_car | 
| -------------------------------------: | ---------: | 
| "Gamesa Corporación Tecnológica, S.A." | 9778464    | 
### 5. The countries with the highest contribution to carbon emissions
```sql
select c.country_name, 
	sum(p.carbon_footprint_pcf) as highes_car
from product_emissions p
left join countries c on p.country_id = c.id
group by c.country_name
order by highes_car desc
limit 1
```
Result:
| country_name | highes_car | 
| -----------: | ---------: | 
| Spain        | 9786130    | 
### 6. The trend of carbon footprints (PCFs) over the years
```sql
select year, 
    sum(carbon_footprint_pcf) as sum_car
from product_emissions 
group by year
order by year desc
```
Result:
| year | sum_car  | 
| ---: | -------: | 
| 2017 | 340271   | 
| 2016 | 1640182  | 
| 2015 | 10840415 | 
| 2014 | 624226   | 
| 2013 | 503857   | 
