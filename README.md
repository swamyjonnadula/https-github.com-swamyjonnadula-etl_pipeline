# ETL Pipeline with DABs & GitHub Actions CI/CD

End-to-end ETL pipeline that extracts sales data from a CSV file in a Unity Catalog volume, transforms it, and loads it into a Delta table — all deployed via Declarative Automation Bundles (DABs) with GitHub Actions CI/CD.

## Project Structure

```
etl_pipeline/
├── databricks.yml              # Main DAB bundle configuration
├── resources/
│   └── etl_job.job.yml         # Job resource definition
├── src/
│   ├── notebooks/
│   │   ├── extract.ipynb       # Extract: Read CSV from UC volume
│   │   ├── transform.ipynb     # Transform: Clean, enrich, validate
│   │   ├── load.ipynb          # Load: Write to Delta table
│   │   └── main.ipynb          # Orchestration: inline ETL steps
│   └── tests/
│       └── test_transform.ipynb  # Unit tests for transform logic
├── .github/
│   └── workflows/
│       └── ci-cd.yml           # GitHub Actions CI/CD pipeline
├── requirements.txt            # Python dependencies
├── .gitignore                  # Git ignore rules
└── README.md                   # This file
```

## Pipeline Steps

### 1. Extract
Reads raw sales data from a CSV file stored in a Unity Catalog volume at:
`/Volumes/workspace/default/etl_source_data/sales_data.csv`

### 2. Transform
Cleans and enriches the data:
- **Deduplication**: Removes duplicate order records
- **Text cleaning**: Trims whitespace, standardizes region casing
- **Data quality**: Filters out null order IDs and negative quantities
- **Derived columns**: Calculates `total_amount`, extracts `order_month`/`order_year`

### 3. Load
Writes the transformed data to a Delta table in Unity Catalog:
`workspace.default.sales_delta`

## CI/CD Pipeline (GitHub Actions)

The `.github/workflows/ci-cd.yml` workflow runs on push to `main` and on PRs:

| Stage | Action | Trigger |
| ----- | ------ | ------- |
| **Lint** | Run `ruff` linter and formatter check | All pushes & PRs |
| **Validate** | `databricks bundle validate --strict` | All pushes & PRs |
| **Deploy** | `databricks bundle deploy --target prod` | Push to main only |
| **Run ETL** | `databricks bundle run etl_pipeline_job` | Push to main only |

### Required GitHub Secrets

Set these in your GitHub repo (Settings → Secrets and variables → Actions):

- `DATABRICKS_HOST` — Your Databricks workspace URL (e.g., `https://your-workspace.cloud.databricks.com`)
- `DATABRICKS_TOKEN` — A Databricks personal access token

## Local Development

### Validate the Bundle
```bash
cd etl_pipeline
databricks bundle validate --strict --target dev
```

### Deploy to Dev
```bash
databricks bundle deploy --target dev
```

### Run the ETL Job
```bash
databricks bundle run etl_pipeline_job --target dev
```

### Run Unit Tests
Open the `test_transform` notebook in Databricks and run all cells.

## Getting Started

### Step 1 — Create a GitHub Repository
1. Go to [github.com/new](https://github.com/new) and create a new repo (e.g., `etl_pipeline`)
2. Do **not** initialize with a README or .gitignore (the project already has them)
3. Copy the repo URL (e.g., `https://github.com/your-username/etl_pipeline.git`)

### Step 2 — Set Up GitHub Secrets
In your GitHub repo, go to **Settings → Secrets and variables → Actions → New repository secret**:

| Secret Name | Value |
| ----------- | ----- |
| `DATABRICKS_HOST` | Your Databricks workspace URL (e.g., `https://your-workspace.cloud.databricks.com`) |
| `DATABRICKS_TOKEN` | A Databricks personal access token (generate from User Settings → Access Tokens) |

### Step 3 — Push the Project to GitHub
Clone the repo locally, copy the project files, and push:

```bash
git clone https://github.com/your-username/etl_pipeline.git
cp -r /path/to/etl_pipeline/* etl_pipeline/
cd etl_pipeline
git add .
git commit -m "Initial commit: ETL pipeline with DABs and CI/CD"
git push origin main
```

Alternatively, you can connect the repo as a Databricks Git Folder:
1. In Databricks workspace, go to **Repos → Add Repo**
2. Paste the GitHub repo URL
3. The project files will appear in the workspace

### Step 4 — CI/CD Pipeline Triggers Automatically
On push to `main`, GitHub Actions will:
1. **Lint** Python code with `ruff`
2. **Validate** the DAB bundle configuration
3. **Deploy** the bundle to Databricks (prod target)
4. **Run** the ETL job automatically

## DABs Targets

| Target | Mode | Description |
| ------ | ---- | ----------- |
| `dev` | development | Default target for development/testing |
| `prod` | production | Production deployment target |