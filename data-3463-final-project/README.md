# What Do Data Jobs Actually Ask For?

DATA 3463 Final Project: Multi-Source Data Acquisition · Ken Reyes & David

**Question:** Which technical skills are most frequently required in data analyst, data scientist and data engineer job postings in Canada (with a closer look at Calgary), and how do these requirements differ between roles?

Everyone "knows" analysts use SQL and engineers use the cloud. We want to test that against real postings and dig deeper: which skills **set each role apart**, which skills **pay more**, and how blurry the job titles really are.

> Status: in progress. The proposal is with the instructor, and data collection is starting.

## Sources

| Modality | Source | What it adds |
|---|---|---|
| API | [Adzuna API](https://developer.adzuna.com/) | Postings, salary (and whether it is estimated), lat/long, category, posting date |
| Database / open data | [Job Bank postings dataset](https://open.canada.ca/data/en/dataset/ea639e28-c0fc-48bf-b5dd-b8899bd43072) (monthly CSVs, loaded into SQLite) | Postings with occupation (NOC) code, salary, requirements, several months of history |
| Web scraping | [Job Bank wage pages](https://www.jobbank.gc.ca/marketreport/wages-occupation/17882/ca) | Median wage by occupation and economic region (the benchmark) |
| Geospatial | Adzuna coordinates + StatCan economic region boundaries | The region of each posting, which is the key used to join the wage benchmark |
| RSS | Jobicy (https://jobicy.com/jobs/feed) | Additional Canada eligible remote job postings, including descriptions for extracting technical skills and requirements.|

## The dataset

**One row = one job posting**, in `data/final/jobs_final.csv`. The main column groups:

- **Who and where:** `role`, `seniority`, `noc_code`, `employer`, `sector`, `city`, `market` (Calgary / rest of Alberta / other), `econ_region`, `work_mode`
- **Skills:** 0/1 flags (`skill_python`, `skill_sql`, `skill_spark`, `skill_powerbi`, ...), rolled up into skill-family counts (programming, BI, cloud, big data, ML/stats)
- **Pay:** `salary_annual`, `salary_is_predicted`, `region_median_wage`, `wage_ratio` (posted salary ÷ regional median)
- **Requirements:** `years_exp_min`, `education_min`, `degree_field`
- **Data quality:** `source`, `text_source` (snippet or full text), `text_length`, `dup_group_id`

## What we'll analyze

1. **Distinctive skills:** a lift score (how much more often a skill appears in one role than across all postings). This goes beyond a plain "most common skills" chart, where SQL comes out on top for every role.
2. **Skill pay premium:** the median salary with vs without each skill, within a role, using only real (not estimated) salaries.
3. **Title vs skills:** how often an "analyst" posting asks for engineering tools.
4. **Calgary angle:** Calgary vs other markets, and whether energy-sector employers ask for different skills.

**Planned charts:** a role × skill-family heatmap, a skill pay premium dot plot, and Calgary vs other markets by sector and wage ratio.

## How to run

Run the notebooks in order from inside `notebooks/`. Each one reads the output of the notebook before it and writes the input for the next one.

| Notebook | Does | Reads → Writes |
|---|---|---|
| `01_acquire` | Downloads raw data | internet → `data/raw/` |
| `02_clean` | Turns raw text into columns | `data/raw/` → `data/clean/` |
| `03_integrate` | Stacks sources, removes duplicates, joins wages | `data/clean/` → `data/final/` |
| `04_visualize` | Makes the charts | `data/final/` → `figures/` |

`data/raw/` is never edited by hand, so everything can be rerun from there.

```
pip install requests pandas matplotlib jupyter
```

Adzuna needs a free `app_id` and `app_key`. These go in a `.env` file, which is gitignored. The large raw Job Bank CSVs are also gitignored. Their download links are in `docs/notes.md`.

## Folders

```
notebooks/   the four phase notebooks
data/        raw/ → clean/ → final/
figures/     chart images for the report
docs/        proposal, report, notes and planning files
```

## Known limitations

- Adzuna returns only a snippet of each description, so it will find fewer skills per posting than full text does. The `text_source` column tracks which postings have only a snippet.
- Adzuna fills in missing salaries with its own estimates. These are flagged and left out of the pay analysis.
- Calgary has a few hundred data postings at most, and a few large employers can dominate them.

## Licensing

Job Bank data is under the Open Government Licence – Canada. Adzuna data is used under its API terms. No job boards are scraped.
