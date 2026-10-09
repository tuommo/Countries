# REST Countries – population, borders and official languages

A small data-analysis project in a Jupyter notebook. It downloads data about every country
from the [REST Countries API](https://restcountries.com/) and answers these questions:

1. Connect to the REST Countries API.
2. Download the data for all available countries.
3. Save the country names in an appropriate object.
4. List the countries by population size in descending order.
5. Find the country bordered by the largest number of neighbours.
6. Which 10 languages are most often listed as official languages?
7. Create a function `top_languages` with the variable `nr_of_top_languages`.
8. How does the list change when official languages are ranked by population?
9. Show graphically how the list changed between questions 6 and 8.

## Repository contents

| File | Purpose |
|---|---|
| `countries_analysis.ipynb` | The notebook: code, comments, tables and charts |
| `requirements.txt` | Python packages needed to run it |
| `.env.example` | Template showing which secret is required (no real key inside) |
| `.gitignore` | Keeps `.env`, cached data and temporary files out of git |

## How to run it

**1. Get an API key.** Since 2026 the REST Countries API (v5) requires a key.
Sign up at <https://restcountries.com/sign-up> and create a key in the dashboard.
The free plan is enough for this project.

**2. Clone the repository and install the packages.**

```bash
git clone https://github.com/<your-user>/<your-repo>.git
cd <your-repo>

python -m venv .venv
# Windows:      .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate

pip install -r requirements.txt
```

**3. Store your API key locally.** Copy `.env.example` to `.env` and paste your key:

```
RESTCOUNTRIES_API_KEY=rc_live_xxxxxxxxxxxxxxxx
```

**4. Open `countries_analysis.ipynb` in VS Code**, select the `.venv` interpreter as the
kernel and choose **Run All**.

The first run calls the API (3 requests) and stores the raw response in
`data/countries_raw.json`. Later runs read that file, so they need neither the key nor any
API quota. To download fresh data, delete the file or set `FORCE_REFRESH = True`.

## API-key handling – best practices

The key is a secret, like a password. The project follows these rules:

- **Never hard-code the key.** The notebook reads it from the environment variable
  `RESTCOUNTRIES_API_KEY` using `os.getenv`.
- **Keep it in a local `.env` file that is not tracked by git.** `.env` is listed in
  `.gitignore`; `python-dotenv` loads it at runtime.
- **Commit a template, not the secret.** `.env.example` documents the variable name with a
  placeholder value.
- **Send the key in the `Authorization` header, not in the URL.** URLs are stored in logs,
  history and error messages.
- **Never print or log the key.** The notebook only reports that a key was loaded.
  Remember that notebook *outputs* are saved in the `.ipynb` file and get committed too.
- **Fail early and clearly.** If the key is missing, a readable error is raised before any
  request is sent.
- **Check before every commit.** Run `git status` and make sure `.env` is not listed.
- **If a key leaks, rotate it.** Revoke it in the dashboard and create a new one. Deleting
  it in a later commit is not enough, because it remains in the git history.
- **In CI or the cloud, use the platform's secret store** (e.g. GitHub Actions secrets,
  Azure Key Vault) to set the same environment variable. No code change is needed.

## How it was done

1. **Connection.** A `requests.Session` carries the `Authorization: Bearer <key>` header.
   Every request has a timeout, and HTTP errors raise an exception.
2. **Download.** The list endpoint `GET https://api.restcountries.com/countries/v5` is
   paginated (max. 100 records per page on the free plan). The notebook requests pages with
   `limit`/`offset` until `meta.more` is `false`. Only the needed fields are requested via
   `response_fields`. The raw result is cached to disk to save quota.
3. **Transformation.** The nested JSON is flattened into a pandas `DataFrame` with one row
   per country: name, ISO code, population, list of border codes, list of language names.
4. **Country names.** Stored as a sorted Python `list`.
5. **Population ranking.** `sort_values` on population, descending.
6. **Most neighbours.** The number of neighbours is the length of the `borders` list. All
   countries sharing the maximum are returned, in case of a tie.
7. **Top languages.** `explode()` creates one row per country–language pair, then languages
   are grouped and counted. This logic is wrapped in
   `top_languages(nr_of_top_languages=10, by="countries")`.
8. **Population-weighted ranking.** The same function with `by="population"` sums the
   population of all countries where the language is official.
9. **Visualisation.** Two matplotlib charts: side-by-side bar charts of both top-10 lists,
   and a slope chart showing each language's rank in both lists.

## Assumptions and limitations

- Every language the API lists for a country is treated as an official language.
- For question 8 the whole population of a country is assumed to speak each of its official
  languages. Multilingual countries are therefore counted once per language, so the totals
  are an upper bound and add up to more than the world population.
- Ties are broken alphabetically so that results are reproducible.
- The dataset contains dependencies and territories as well as sovereign states.

## Data source

Data provided by [REST Countries](https://restcountries.com/).
