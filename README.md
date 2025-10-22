# Data Processing & Reporting Pipeline

This project sets up an automated pipeline to process tabular data using Python, Pandas, and GitHub Actions. It demonstrates best practices for data handling, code quality, and continuous integration/delivery for data-driven projects.

## Table of Contents

- [Project Overview](#project-overview)
- [Files in this Repository](#files-in-this-repository)
- [Data Conversion (`data.xlsx` to `data.csv`)](#data-conversion-dataxlsx-to-datacsv)
- [Python Script (`execute.py`)](#python-script-executepy)
  - [Error Fix Implemented](#error-fix-implemented)
- [GitHub Actions CI/CD Workflow](#github-actions-cicd-workflow)
- [Local Development](#local-development)
- [License](#license)

## Project Overview

The core of this project involves processing an Excel file (`data.xlsx`), converting it to CSV for easier programmatic access (`data.csv`), and then running a Python script (`execute.py`) to transform and aggregate this data. The final output is a `result.json` file, which is automatically published as a static asset via GitHub Pages.

## Files in this Repository

- `index.html`: A static web page serving as the project's landing page, explaining the pipeline and linking to the generated `result.json`.
- `execute.py`: The Python script responsible for reading, processing, and outputting data.
- `data.xlsx`: The initial raw data in Excel format.
- `data.csv`: The converted CSV version of `data.xlsx`, used by `execute.py`.
- `README.md`: This file.
- `LICENSE`: The MIT License for this project.
- `.github/workflows/ci.yml`: The GitHub Actions workflow definition for automated testing and deployment.

**Note:** The `result.json` file is *not* committed to the repository. It is generated dynamically by the CI/CD pipeline.

## Data Conversion (`data.xlsx` to `data.csv`)

The initial `data.xlsx` file has been converted into `data.csv`. This conversion simplifies the Python script's dependencies (avoiding the need for `openpyxl` at runtime if only CSV is consumed by `execute.py`) and is generally more suitable for plain-text version control.

To perform this conversion locally (if you were starting from scratch with only `data.xlsx`), you could use a simple Python script:

```python
import pandas as pd

# Load the Excel file
excel_file = 'data.xlsx'
df = pd.read_excel(excel_file)

# Save as CSV
csv_file = 'data.csv'
df.to_csv(csv_file, index=False)

print(f"'{excel_file}' successfully converted to '{csv_file}'.")
```

The `data.csv` file committed to this repository is the result of such a conversion.

## Python Script (`execute.py`)

The `execute.py` script is designed to:
1. Read the `data.csv` file.
2. Ensure data integrity by converting critical columns to numeric types, handling potential errors gracefully.
3. Calculate derived metrics (e.g., 'Revenue' from 'Quantity' and 'UnitPrice').
4. Aggregate data (e.g., total revenue per product).
5. Output the final processed data as a pretty-printed JSON to standard output.

The script is built to run on **Python 3.11+** and requires **Pandas 2.3**.

### Error Fix Implemented

The original `execute.py` contained a non-trivial error related to data type handling. Specifically, it assumed that certain columns (e.g., 'Quantity', 'UnitPrice') would always be numeric, which could lead to `TypeError` or `ValueError` if the source `data.csv` contained non-numeric entries (e.g., empty strings, text, or formatting issues).

The fix implemented involves:
- Using `pd.to_numeric(..., errors='coerce')` to convert these columns. This approach safely converts non-numeric values to `NaN` (Not a Number).
- Filling these `NaN` values with `0` using `.fillna(0)` before performing arithmetic operations. This ensures that calculations like revenue multiplication proceed without errors and treats missing/invalid numerical data as zero, preventing `NaN` propagation into the final aggregates.

This makes the script more robust against imperfect input data.

## GitHub Actions CI/CD Workflow

A GitHub Actions workflow (`.github/workflows/ci.yml`) automates several key aspects of this project:

- **Trigger:** Runs on every `push` event to any branch.
- **Environment:** Sets up Python 3.11.
- **Dependencies:** Installs `pandas==2.3.0` and `ruff` for linting.
- **Code Quality:**
  - Runs `ruff .` to check for code style issues and potential errors. The results are visible in the CI log.
- **Data Processing:**
  - Executes `python execute.py > result.json`. This command runs the main data processing script and redirects its JSON output to a `result.json` file.
- **Deployment:**
  - Uses `actions/upload-pages-artifact` to upload the generated `result.json` (along with `index.html` and other relevant files in the root directory) as an artifact.
  - Uses `actions/deploy-pages` to publish this artifact to GitHub Pages.

You can view the published `result.json` at:
`https://<YOUR_GITHUB_USERNAME>.github.io/<YOUR_REPO_NAME>/result.json`

*(Replace `<YOUR_GITHUB_USERNAME>` and `<YOUR_REPO_NAME>` with your actual GitHub username and repository name.)*
The `index.html` file provided in this repository will link to this output dynamically using a relative path `/result.json`, assuming it's deployed at the root of your GitHub Pages site.

## Local Development

To set up this project locally:

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>.git
    cd <YOUR_REPO_NAME>
    ```
2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv .venv
    source .venv/bin/activate  # On Windows: .venv\Scripts\activate
    ```
3.  **Install dependencies:**
    ```bash
    pip install pandas==2.3.0 ruff
    ```
4.  **Run the linter:**
    ```bash
    ruff .
    ```
5.  **Execute the data processing script:**
    ```bash
    python execute.py > result.json
    ```
    This will generate `result.json` in your project root.
6.  **View the web application:**
    Open `index.html` directly in your browser or serve it using a local HTTP server.

## License

This project is licensed under the MIT License. See the `LICENSE` file for full details.
