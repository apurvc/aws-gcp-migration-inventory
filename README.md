# GitHub .NET Repo Analyzer for AWS-to-GCP Migration

[![Python Version](https://img.shields.io/badge/python-3.7%2B-blue)](https://www.python.org/)
[![GCP Migration](https://img.shields.io/badge/Migration-AWS%20to%20GCP-orange)](https://cloud.google.com/dotnet)

An automated inventory and assessment tool designed to streamline the migration of .NET application portfolios from AWS to Google Cloud. It performs deep static analysis of repository structures, dependencies, and architectural patterns to generate a comprehensive migration strategy.

---

## 🚀 Key Features

- **Recursive Tree Discovery**: Uses the GitHub Trees API to analyze entire repository structures (including nested microservices) in a single API call.
- **Minimal Data Overhead**: Implements "Lazy Loading"—only high-signal files (`.csproj`, `appsettings.json`, `web.config`) are downloaded for deep inspection.
- **Deep Cloud Readiness Metrics**:
  - **Databases**: Detects SQL Server, PostgreSQL, MySQL, CosmosDB, and DynamoDB.
  - **Authentication**: Identifies Windows Auth (GCE indicator) vs. OAuth/OIDC (Cloud Run friendly).
  - **Logging**: Detects Serilog, NLog, and Application Insights.
  - **Secrets**: Identifies hardcoded AWS keys (AKIA/ASIA) and existing secret management patterns.
- **Subfolder Support**: Directly analyze specific subfolders or branches using standard GitHub URLs.
- **Smart Target Mapping**: Automatically suggests optimal GCP services (Cloud Run, GCE, App Engine) based on auth and framework version.
- **Multi-Format Export**: Generates professional Excel and CSV reports with 20+ intelligence data points.

---

## 🛠️ Setup

### 1. Prerequisites
- **Python 3.7+**
- A **GitHub Personal Access Token (PAT)** with `repo` scope.

### 2. Installation
```bash
# Clone the repository
git clone https://github.com/apurvachandra1990/repo_analysis.git
cd repo_analysis

# Create and activate virtual environment
python -m venv env
source env/Scripts/activate  # Windows: .\env\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

---

## 📖 Usage

### Quick Start
1. Set your GitHub token as an environment variable:
   ```powershell
   $env:GITHUB_TOKEN = "ghp_your_token_here"
   ```
2. Edit `gh_repo_analysis.py` to add your targets to `REPOS_TO_ANALYZE`.
3. Run the analyzer:
   ```bash
   python gh_repo_analysis.py
   ```

### Targeting Subfolders
You can analyze specific projects within a monorepo by providing the folder URL:
```python
REPOS_TO_ANALYZE = [
    "owner/repo/tree/main/src/MyProject"
]
```

---

## 📊 Inventory Output

The tool generates a file named `AWS_to_GCP_Migration_Inventory.xlsx` with detailed metrics:

| Field | Description |
| :--- | :--- |
| **Application Type** | REST API, Windows Service, SOAP, etc. |
| **Target Framework** | Detects .NET Framework (4.x) vs .NET Core/5+. |
| **Suggested GCP** | Mapping to Cloud Run, GCE, or GKE. |
| **Database Engine** | Detected drivers (SQL, Postgres, MySQL). |
| **Auth Strategy** | Identifies blocking patterns like Windows Auth. |
| **AWS Services** | List of specific AWSSDK services detected. |
| **CI/CD Pipeline** | GitHub Actions, Jenkins, AppVeyor, etc. |

---

## 🔒 Security & Privacy
- **Metadata-First**: The tool fetches file lists before downloading content.
- **No Private Data Stored**: Only extracts technical patterns and frameworks.
- **Safe Export**: Automatically handles Excel file-locking errors by creating timestamped backups.

---
