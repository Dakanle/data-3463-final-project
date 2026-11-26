# Data dictionary (Ken's sources)

## postings_clean.csv (data/clean/)

One row = one job posting from the TheirStack API. 123 rows. Made in 02_clean, section 1.

- id (integer): TheirStack's id for the posting
- role (text): data analyst, data engineer or data scientist. The job title we searched for, after removing non-data titles
- noc (text): NOC code for the role. Analyst 21223, engineer 21211, scientist 21211. Used to join to the wages
- job_title (text): title as posted
- company (text): employer name
- seniority (text): junior, mid or senior. From TheirStack's label, staff counted as senior
- location (text): location as posted
- province (text): 2-letter province code. Blank if the posting only says "Canada" or nothing
- latitude, longitude (decimal): job location. Blank when TheirStack only gave its placeholder point for "Canada" (60.11, -113.64)
- remote (True/False): remote job. TheirStack flag, or {{REMOTE}} in the location
- hybrid (True/False): hybrid job, TheirStack flag
- date_posted (date): date the job was posted, YYYY-MM-DD
- salary_annual_cad (decimal): yearly salary in CAD. Middle of the posted range, USD converted at 1.40. Blank if no salary was posted
- salary_currency (text): currency the salary was posted in, CAD or USD. A blank currency was assumed to be CAD
- skill_sql, skill_python, skill_r, skill_excel, skill_power_bi, skill_tableau, skill_spark, skill_airflow, skill_dbt, skill_aws, skill_azure, skill_gcp, skill_snowflake, skill_databricks, skill_machine_learning, skill_statistics (True/False): True if the job description mentions that skill. Keyword search, not case sensitive
- url (text): link to the posting

## wages_clean.csv (data/clean/)

One row = wages for one role in one region or province, from the scraped Job Bank wage pages. 255 rows. Made in 02_clean, section 2.

- role (text): data analyst, data engineer or data scientist
- noc (text): NOC code of the wage page, 21223 or 21211
- level (text): region (economic region) or province
- province (text): 2-letter province code
- region (text): economic region name, or the province name for province rows
- wage_low, wage_median, wage_high (decimal): hourly wage in CAD from Job Bank (StatCan Labour Force Survey, 2024-2025). Blank where Job Bank shows N/A
- wage_low_annual, wage_median_annual, wage_high_annual (decimal): hourly wage x 1,950 hours (37.5 hours a week x 52 weeks)
