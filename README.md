# Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
This is the SQL Module Final Project for Lighthouse Labs' Immersive Data Science Bootcamp.

The goals of this project are to put together all the component knowledge about SQL that has been learned in the first 14 days of the bootcamp, including the following tasks:
- extracting data from a SQL database
- cleaning, transforming and analyzing data
- loading data into a database
- developing and implementing a QA process to validate transformed data against raw data

## Process
### Examine Data within Excel in CSV format
### Determine recipient tables for the Excel CSV files
### Determine datatypes for columns in the recipient tables
### Import 4 CSV files into 4 tables in PostgreSQL
### Examine data within columns of each table, and run preliminary EDA SQL queries, to determine data cleaning steps for each table
### Create 3 new views for 3 out of 4 tables with data cleaning steps implemented.  Data cleaning steps included such things as:
#### Removing columns with no data in any rows
#### String manipulations to remove erroneous data
1. Removing extranneous and erroneous words like "Google", "YouTube", "Android", "Waze" from product names
2. Removing an erroneous single entry in the pagetitle column
3. De-duplicating data (e.g. setting "/asearch.html/" to "/asearch.html", etc.)
4. Removing leading and traliing whitespaces from product name
#### Standardizing / normalizing / scaling very large values for price/revenue (i.e. dividing by 1,000,000)
#### Interpolating data (replacing NULL values for sentimentscore and sentimentmagnitude with average values - please note that this was done "on the fly" during SQL querying, on a subset of rows)
#### Making note of rows of data that require further investigation with members of the business to fill in missing data
### Writing and executing SQL to carry-out QA to ensure cleaned table integrity and consistency with original data, after cleaning
### Writing and executing SQL to carry-out EDA to understand how the tables and keys are related in the data, given that there is no data dictionary available for the data
### Writing and executing SQL to answer questions posed in the "starting_with_questions"
### Formulating additional questions about the data, recorded within "starting_with_data", and writing and executing SQL to answer those questions

## Results
(fill in what you discovered this data could tell you and how you used the data to answer those questions)

## Challenges 
This project had several challenges:

1. No data dictionary to explain what the columns mean, and how the columns/keys and tables relate to each other.  This caused a lot of guessing and caveats on the validity of the conclusions from the data analysis.
2. No "business" to ask questions about the data, the application, domain, context under which this data was gathered and how the data might be used to draw conclusions.  No ability to validate that the application of the analysis would be correct.
3. Short time-frame under which to complete the project (done over 48 hours non-stop).
4. Data quality is extremely poor; many null values, a lot of inconsistencies in data, corrupt data, numbers and sums do not add up or agree between tables.

## Future Goals

If I had more time and tools, I would:

1. Do further data cleaning after more EDA type interrogation of the data.
2. Find a "business person" to interview to ask more questions about the way this data is sourced, the business domain, and to validate that the assumptions I have made about how the data connects, is accurate.
3. Take some of my findings from the LHL questions and do statistical analysis looking for patterns and trends that are more subtle than stand out to the "naked eye".  This would also include doing graphical analysis (visualizations, histograms, etc.) of the results.
4. Follow up with "the business" about the tentative conclusions I drew from the questions I posed to the data, in order to get feedback and see if further data analysis or further data procurement would help with developing further and stronger conclusions.
