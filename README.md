# Fundamentals of dbt

This project will be used to help train aspiring Analytics Engineers on the fundamentals of dbt.

This repo has been forked from [gwenwindflower's](https://github.com/gwenwindflower) [octocatalog](https://github.com/gwenwindflower/octocatalog): an open-source, open-data data-platform-in-a-box[^1] based on [DuckDB](https://duckdb.org/) + [dbt](https://www.getdbt.com/) + [Evidence](https://evidence.dev/), offering a simple script to extract and load (EL) data from the [GitHub Archive](https://www.gharchive.org/), a dbt project built on top of this data inside a DuckDB database, and BI tooling via Evidence to analyze and present the data.

This project runs completely local, or inside of a devcontainer or Github Codespaces.

## Quickstart
1. Install [uv](https://docs.astral.sh/uv/getting-started/installation/) (already installed if you're using _Github Codespaces_ or _VS Code Devcontainers_)
2. Go to the `extract-and-load-with-python` folder and run `uv run python el.py -e`
3. Go to the `extract-and-load-with-python` folder and run `uv run python el.py -l`
4. Go to the `transform-with-dbt` folder and run  `uv run dbt deps`
5. Go to the `transform-with-dbt` folder and run  `uv run dbt seed`
6. Go to the `transform-with-dbt` folder and run  `uv run dbt build`


## Project Structure
Each part of the project has a dedicated folder:
1. **transform-with-dbt** folder
    1. It is inside this folder that you need to run all dbt commands (for ex. 'dbt run', 'dbt debug', ...)
2. **create-reports-with-evidence** folder
    1. From inside this folder you can launch the commands to launch [Evidence](https://evidence.dev/) dashboards
3. **extract-and-load-with-python** folder
    1. You need to be inside this folder to launch the Extraction and Load script (el.py) that will get github public data into the local Data Warehouse
4. Inside the **store-and-compute-with-duckdb** is where all the files of the Local duckDB Data Warehouse will be stored.
5. The **data-landing-zone** folder will be used for temporarily storing the raw github data, before loading it into the Data Warehouse.


## Setup

There are a few steps to get started with this project if you want to develop locally. We'll need to:

1. [Clone the project locally](#clone-the-project-locally).
2. [Insatll uv (python), then install the dependencies and other tooling](#python).
3. [Extract and load the data locally](#extract-and-load).
4. [Transform the data with dbt](#transform-the-data-with-dbt).
5. [Build the BI platform with Evidence](#build-the-bi-platform-with-evidence).


### Clone the project locally


### Python

You need to install uv : https://docs.astral.sh/uv/getting-started/installation/


## Extract and Load
> extract-and-load-with-python

Extract and load is the process of taking data from one source, like an API, and loading it into another source, typically a data warehouse. In our case our source is the GitHub Archive, and our load targets is a local DuckDB database.

### Local usage
> extract-and-load-with-python/el.py

If you run the script directly, it takes two arguments: a start and end datetime string, both formatted as `'YYYY-MM-DD-HH'`. It is inclusive of both, so for example running `uv run python el.py '2023-09-01-01' '2023-09-01-02'` will load _two_ hours: 1am and 2am on September 9th 2023. Pass the same argument for both to pull just that hour.
Please note that the GitHub Archive is available from 2011-02-12 to the present day and that being event data it is very large. Running more than a few days or weeks will push the limits of DuckDB (that's part of the interest and goal of this project though so have at).

> [!NOTE]
> **Careful of data size**. DuckDB is an in-process database engine, which means it runs primarily in memory. This is great for speed and ease of use, but it also means that it's (somewhat) limited by the amount of memory on your machine. The GitHub Archive data is event data that stretches back years, so is very large, and you'll likely run into memory issues if you try to load more than a few days of data at a time. We recommend using a single hour locally when developing. When you want to go bigger for production use you'll probably want to leverage the option below.

### Running the EL script

You can manually run the `el.py` script with `uv run python el.py [args]` to pull a custom date range, run on small test data file, and isolate the extract or load steps. P

The args are:

```shell
python el.py [start_date in YYYY-MM-DD format, defaults to yesterday] [end_date in YYYY-MM-DD format, defaults to today] [-e --extract Run the extract part only] [-l --load Run the load part only] [-p --prod Run in production mode against MotherDuck]
```

Running the the `el.py` script without an `-e` or `-l` flag is a no-op as all flags default to `false`. Combine the flags to create the commands you want to run. For example:

```shell
uv run python el.py -e # extract the data for the past day
uv run python el.py -lp # load any data into the production database
uv run python el.py 2023-09-20 2023-09-23 -elp # extract and load 3 days of data into the production database
```

## Store the extracted data in local duckDB
> store-and-compute-with-duckdb

In order for Evidence to work the DuckDB file needs to be built into the `store-and-compute-with-duckdb/` directory. If you're looking to access it via the DuckDB CLI you can find it at `store-and-compute-with-duckdb/duckdb_data_store.db`.

## Transform the data with dbt
> transform-with-dbt

dbt is the industry-standard control plane for data transformations. We use it to get our data in the shape we want for analysis.

The task runner is configured to run dbt for you `task transform`, but if you'd like to run it manually you can do so by running these commands in the virtual environment:

```shell
uv run dbt deps # install the dependencies
uv run dbt build # build and test the models
uv run dbt run # just build the models
uv run dbt test # just test the models
uv run dbt run -s marts # just build the models int the marts folder
```

## Build the BI platform with Evidence
> create-reports-with-evidence

Evidence is an open-source, code-first BI platform. It integrates beautifully with dbt and DuckDB, and lets analysts author version-controlled, literate data products with Markdown and SQL. Like the other steps, it's configured to run via the task runner with `task bi`, but you can also run it manually with:

```shell
cd create-reports-with-evidence
npm install # install the dependencies
npm run sources # build fresh data from the sources
npm run dev # run the development server
```

### Developing pages for Evidence

Evidence uses Markdown and SQL to create beautiful data products. It's powerful and simple, focusing on what matters: the _information_. You can add and edit markdown pages in the `create-reports-with-evidence/pages/` directory, and SQL queries those pages can reference in the `create-reports-with-evidence/queries/` directory. You can also put queries inline in the Markdown files inside of code fences, although stylistically this project prefers queries go in SQL files in the `queries` directory for reusability and clarity. Because Evidence uses a WASM DuckDB implementation to make pages dynamic, you can even chain queries together, referencing other queries as the input to your new query. We recommend you utilize this to keep queries tight and super readable. CTEs in the BI section's queries are a sign that you might want to chunk your query up into a chain for flexibility and clarity. Sources point to the raw tables, either in your local DuckDB database file or in MotherDuck if you're running prod mode. You add a `select * [model]` query to the `create-reports-with-evidence/sources/` directory and re-run `npm run sources` and you're good to go.

Evidence's dev server uses hot reloading, so you can see your changes in real time as you develop. It's a really neat tool, and I'm excited to see what you build with it.

---

## Modeling the event data

Schemas for the event data [are documented here](https://docs.github.com/en/rest/overview/github-event-types?apiVersion=2022-11-28).

So far we've modeled:

- [x] Issues
- [x] Pull Requests
- [x] Users
- [x] Repos
- [ ] Stars
- [ ] Forks
- [ ] Comments
- [ ] Pushes

[^1]: Based on the patterns developed by Jacob Matson for the original [MDS-in-a-box](https://duckdb.org/2022/10/12/modern-data-stack-in-a-box.html).
