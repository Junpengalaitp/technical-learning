# Introduction
## Data Warehouse Definition
* A giant storehouse for your data. 
* ALL of your data. 
* Central repository, aggregate data from multiple systems.
* Single source of "the truth"

## Business Intelligence Definition
* Leveraging data you already have to convert knowledge into informed actions.
* Providing ways to measure the health of your business.
* Examining the data in your warehouse to look for three main areas of interest.
* Areas of Interest
  * Aggregations
  * Trends
  * Correlations

## Purpose of Data Warehouse
* Combination data from multiple systems
* Resolve inconsistencies between systems
* Make reporting easier
* Reduce the load on production systems
* Provide for long term storage of data
* Provide consistency among system transitions
* Make data available for analysis
* Ability to apply advanced analytic tools
* Extract further value from your data

# Warehousing Methodologies
* CIF - Corporate Information Factory
* Star Schema - Kimball Method

## Drawbacks of Reporting from a Transactional System
* OLTP - On Line Transaction Processing
* Designed for single record accesses
* Data is normalized
* Getting data can involve many joins
* Confusing for 'ad-hoc' reporting
* Slow, having an impact on the OLTP system

## Difference on Data Warehouse
* OLAP - Online Analytical Processing
* Data is de-normalized
* Number of tables is reduced
* Star Schema or Snowflake Schema

## Types of Tables in a Data Warehouse
* Facts
* Dimensions
* Surrogate Keys
  * A new key used as the primary key 
  * Adaptable to source system changes
  * often the key is not needed
  * Combine data from multiple sources

## Fact Tables
* Marker of an event
* Join dimensions, such as the who's and what's of an event
* Joins to the special date dimension
* Hold numeric, quantifiable values

## Dimensions
* Hold the values that describe facts
* "Lookup tables"
* Examples include geography, employees, product, customers and time
* Slowing Changing Dimension(SCD)
* Many types
  * Type 0: fixed static data like colors and sizes that will not change ever.
  * Type 1: when a value is updated, the old one is simply overwritten.
  * Type 2: when a value is updated, the old record is end dated and a new record is inserted.
  * Type 3: when a value is updated, records shift to a new column. (not recommended)
  * Type 4: when a value is updated, old record copied to history and current record is updated.
* Dimensional types are at the column level, not the row.
* The business should be the ones to determine which data is significant enough to track changes on.

## Storing data into a Data Warehouse
* ETL
  * Extract
  * Transform
  * Load
* SSIS - SQL Server Integration Services

## Retrieve data into a Data Warehouse
### Data Aggregation, Trending, Correlations
* SSAS - SQL Server Analysis Services
* Azure ML

### Reporting
* SSRS - SQL Server Reporting Services
* Excel
* PowerBI
