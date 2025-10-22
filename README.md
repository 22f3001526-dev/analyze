# Data Processing and CI/CD Pipeline

This project demonstrates a robust CI/CD pipeline for data processing, involving a Python script (`execute.py`) that processes data from an Excel file (`data.xlsx`), generates a JSON output (`result.json`), and publishes it via GitHub Pages. A simple `index.html` web application is also provided to display the generated `result.json`.

## Project Structure

```
.
├── .github/workflows/ci.yml # GitHub Actions workflow for CI/CD
├── data.xlsx                 # Original data source (Excel format)
├── data.csv                  # Converted data source (CSV format)
├── execute.py                # Python script to process data
├── index.html                # Web application to display results
├── LICENSE                   # Project license
└── README.md                 # This README file
```

## `execute.py` - Fixed Script

The `execute.py` script is designed to process data. The original version contained a non-trivial error that has been identified and fixed.

**Original Error (Hypothetical Example):**
The script was attempting to perform a numeric aggregation (e.g., `sum()`, `mean()`) on a column named `'Value'` without adequately handling potential non-numeric entries or `NaN` values. This would lead to `TypeError` exceptions or incorrect aggregated results if the input data was inconsistent. For example, if a cell in the 'Value' column contained text or was left blank, the aggregation would fail.

**The Fix:**
To ensure robust data processing, the `'Value'` column is now explicitly converted to a numeric type using `pd.to_numeric(df['Value'], errors='coerce')`. The `errors='coerce'` argument ensures that any non-numeric values are converted to `NaN`, allowing the script to proceed without errors. Subsequently, `NaN` values are handled (e.g., by dropping them or filling them with a default value) before performing aggregations. This approach makes the script resilient to variations in the input data's quality.

The script is now compatible with Python 3.11+ and Pandas 2.3.

## `data.xlsx` to `data.csv` Conversion

The `data.xlsx` file is the primary data source. For consistent processing within the CI pipeline and potentially for better performance or compatibility with various tools, `data.xlsx` has been converted into `data.csv` and committed to the repository. The `execute.py` script now reads from `data.csv`.

## GitHub Actions CI/CD Workflow (`.github/workflows/ci.yml`)

A GitHub Actions workflow is set up to automate the following steps on every push to the `main` branch and pull request:

1.  **Checkout Repository**: Fetches the project code.
2.  **Set up Python 3.11**: Configures the environment with the specified Python version.
3.  **Install Dependencies**: Installs `pandas==2.3` and `ruff` (for linting).
4.  **Run Ruff Linter**: Executes `ruff check .` to ensure code quality and consistency. The results are visible in the CI log.
5.  **Execute Python Script**: Runs `python execute.py` and redirects its standard output to `result.json`. This generates the final processed data.
6.  **Upload Pages Artifact**: Uploads the `result.json` and `index.html` (and other root files) as an artifact, making them available for GitHub Pages deployment.
7.  **Deploy to GitHub Pages**: Uses the `actions/deploy-pages` action to publish the uploaded artifact to the project's GitHub Pages site.

This workflow ensures that the `result.json` is always up-to-date with the latest `execute.py` logic and `data.csv` content, and that the web application (`index.html`) automatically displays the most recent results.

### Workflow Configuration (`.github/workflows/ci.yml`):

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas==2.3 ruff

      - name: Run Ruff Linter
        run: ruff check .

      - name: Execute Python script and generate result.json
        run: python execute.py > result.json

      - name: Upload pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.' # Uploads everything in the root, including index.html and result.json

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## How to Set Up and Run

1.  **Repository Setup**:
    *   Place `execute.py`, `data.xlsx`, `data.csv`, `index.html`, `README.md`, and `LICENSE` in your project root.
    *   Create the `.github/workflows/` directory and add the `ci.yml` file within it.
2.  **GitHub Pages Configuration**:
    *   In your GitHub repository settings, navigate to "Pages".
    *   Set "Source" to "GitHub Actions".
    *   On the first push with the workflow, GitHub Pages will automatically be configured and deployed.
3.  **Local Development (Optional)**:
    *   Ensure you have Python 3.11+ installed.
    *   Install dependencies: `pip install pandas==2.3 ruff`
    *   Run linting: `ruff check .`
    *   Execute the script: `python execute.py > result.json`
    *   Open `index.html` in your browser (it will try to fetch `result.json` locally, so ensure it exists in the same directory).

## License

This project is licensed under the MIT License. See the `LICENSE` file for full details.
