# Notes

## API - TheirStack Jobs API

Pulled on Nov 22, 2026 using a free TheirStack account (200 API credits, used about 155).

Endpoint: POST https://api.theirstack.com/v1/jobs/search
The key is stored in .env (not on GitHub) and sent as a Bearer token.

Search filters: jobs in Canada (country code CA), posted in the last 90 days, job title matching "data analyst", "data engineer" or "data scientist".

How it works: first a small test request (data analyst, 5 postings) to check the key and see what fields a posting has. Then a loop over the 3 roles, 2 pages each. The free plan only allows 25 results per request, so each role is page 0 + page 1 = 50 postings. Every response is saved unchanged as JSON in data/raw/theirstack/, one file per role and page, and each request prints a log line with the time and status.

Result: 7 files, 155 rows, 149 unique postings (6 duplicates, mostly the test postings showing up again in the data analyst pull). Posted between Nov 17 and Nov 22, 2026.

Problems / limitations:
- Only 50 postings per role because of the free plan, and they're the newest ones (TheirStack returns the most recent first), so all are from the last week even though we asked for 90 days. Not a random sample.
- Title search is loose, so some postings aren't really data jobs (for example "Business Analyst Co-op, Data Governance"). These get filtered in cleaning.
- Only 45 of 149 postings list a salary, and only 2 are tagged junior (93 mid level, 53 senior).
- 147 of 149 have coordinates, but 21 of those are a placeholder point TheirStack uses for postings that only say "Canada" (60.11, -113.64, middle of Canada). We blank those in cleaning, so 101 of the 123 cleaned postings have a real location.
- TheirStack terms (https://theirstack.com/en/docs/legal/terms-and-conditions, read Nov 22, 2026): nothing specific about academic or student use. They allow reposting and publicly showing job postings, and say aggregated stats or models made from the data belong to the user, as long as the original dataset can't be rebuilt from them. Not allowed: handing someone a whole dataset or using the data to compete with TheirStack. Our counts and charts are aggregated, and the 149 postings are a tiny sample, so our use fits. TheirStack keeps a database right over its datasets.
- Adzuna was tested first (Oct 2026) and rejected: descriptions cut at 500 characters, lots of non-data titles, and its terms restrict academic use. Those files are kept in data/raw/adzuna/ for reference.


## Web scraping - Job Bank wages

Scraped on Nov 21, 2026 from Job Bank's wage reports. The wages themselves come from Statistics Canada's Labour Force Survey (2024-2025).

Pages we used:
- data analyst -> data warehouse analyst, NOC 21223: https://www.jobbank.gc.ca/wagereport/occupation/17880
- data engineer -> data engineer, NOC 21211: https://www.jobbank.gc.ca/wagereport/occupation/24510
- data scientist -> data scientist, NOC 21211: https://www.jobbank.gc.ca/wagereport/occupation/227147

robots.txt doesn't block anything, it just says Crawl-delay: 5, so we waited 5 seconds between pages. Only 3 requests total. Screenshot is in docs/jobbank_robots_2026-11-22.png.

How it works: requests.get() each page and save the raw HTML to data/raw/wages_html/. Then BeautifulSoup grabs the region and province rows from the wage table (tr.wage-region and tr.wage-province) and pulls the low, median and high wage from each row. Saved to data/clean/jobbank_wages_by_region.csv, 255 rows (72 regions + 13 provinces/territories, for each of the 3 roles). Wages are per hour.

Problems / limitations:
- Job Bank gives wages by NOC, not by job title. Data engineer and data scientist are both 21211, so they have the same wages.
  
- 136 of the 255 rows have no wage. The site just shows N/A because there isn't enough data for that job in smaller regions. Regions with a median wage: analyst 39 of 72, engineer 26 of 72, scientist 26 of 72.

- No province wage for YT, NT, NU or PE. When a region is N/A we'll use the province wage instead.

- 
- The wages are from 2024-2025 but our API job postings are from Oct 2026.

## Cleaning (02_clean) - TheirStack postings and Job Bank wages

Postings: 149 -> 137 after removing 12 titles that matched "data" but aren't data jobs (data centre electrical/mechanical engineers, security engineers, QA testers, an EDI analyst) -> 124 after removing repeat postings (same title at the same company) -> 123 after removing one posting located only in New York. Final: 42 analyst, 35 engineer, 46 scientist.
- Province filled in from the location text when TheirStack gave "00". 19 postings have no province (they just say "Canada" or nothing).
- 21 placeholder coordinates blanked, 101 postings have a real location.
- 38 of 123 have a salary. Salary = middle of the posted range, per year. 12 were in USD, converted at 1.40. 11 had no currency, assumed CAD.
- Seniority: 76 mid, 45 senior, 2 junior. Our results describe mid/senior jobs, not entry level.
- 16 skill columns from a keyword search of the description. Keyword matching can miss skills written differently or count words used in another sense (e.g. "ML").
- Kept "Online Data Analyst - Canada (French Language)" in the data analyst group.

Wages: hourly wages converted to yearly using 1,950 hours (37.5 h x 52 weeks). Province rows kept so they can be used when a region is N/A.

Column definitions are in docs/data_dictionary.md.
