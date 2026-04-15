# GitHub .NET Repo Analyzer - Setup Guide
## Overview
This automation tool analyzes your GitHub repositories to create a migration inventory for AWS-to-GCP transitions. It identifies .NET application types, detects AWS SDK usage, and generates an Excel inventory with migration complexity estimates.
---
## Prerequisites
### 1. GitHub Personal Access Token (PAT)
You need a GitHub token with **read access** to your repositories.
**Steps:**
1. Go to https://github.com/settings/tokens
2. Click "Generate new token" → "Generate new token (classic)"
3. **Scopes needed:**
- `repo` (full repo access) OR
- `public_repo` + `read:org` (if you have public repos only)
4. Copy the token and save it safely
### 2. Python Environment
- **Python 3.7+** installed on your system
- Dependencies:
- `requests` - For GitHub API calls
- `openpyxl` - For Excel output (optional, falls back to CSV)
**Install dependencies:**
```bash
pip install requests openpyxl
```
---
## Usage
### Quick Start
#### **Option 1: Using the Python Script Directly**
```bash
# Set your GitHub token as environment variable
export GITHUB_TOKEN="ghp_your_token_here" # macOS/Linux
# or on Windows:
set GITHUB_TOKEN=ghp_your_token_here
# Run the analyzer
python github_repo_analyzer.py
```
Edit the script to add your repos:
```python
REPOS_TO_ANALYZE = [
"myorg/service-api",
"myorg/batch-processor",
"myorg/windows-service",
# Add your repos here
]
```
#### **Option 2: Using the Batch Script (Windows)**
```powershell
# Set token
$env:GITHUB_TOKEN = "ghp_your_token_here"
# Run script
python github_repo_analyzer.py
```
#### **Option 3: Automated Scheduling**
**On Windows (Task Scheduler):**
```powershell
# Create a scheduled task script
$task = @{
TaskName = "GitHub Migration Inventory"
Trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 9am
Action = New-ScheduledTaskAction -Execute "python" -Argument "C:\path\to\github_repo_analyzer.py"
RunLevel = "Limited"
}
Register-ScheduledTask @task
```
**On Linux/macOS (cron):**
```bash
# Edit crontab
crontab -e
# Add line to run weekly:
0 9 * * 1 cd /path/to/script && GITHUB_TOKEN=ghp_xxx python github_repo_analyzer.py
```
---
## Configuration
### Adding Repositories
Edit the `REPOS_TO_ANALYZE` list in the script. You can now use subfolder URLs:
```python
REPOS_TO_ANALYZE = [
    "organization/repo-name",               # Analyze entire repo
    "myorg/mono-repo/tree/main/src/api",    # Analyze specific subfolder
    "myorg/service/tree/develop",           # Analyze specific branch
]
```
**Bulk Loading from File:**
Create `repos.txt`:
```
organization/repo1
organization/repo2
organization/repo3
```
Then modify the script:
```python
# Load repos from file
with open("repos.txt") as f:
REPOS_TO_ANALYZE = [line.strip() for line in f if line.strip()]
```
### Environment Variables
| Variable | Description | Required |
|----------|-------------|----------|
| `GITHUB_TOKEN` | GitHub Personal Access Token | Yes |
| `OUTPUT_FILE` | Excel output filename (default: `migration_inventory.xlsx`) | No |
---
## Output
### Excel Spreadsheet Format
The analyzer creates a spreadsheet with these columns:
| Column | Description |
|--------|-------------|
| **Repository Name** | Name of the GitHub repo |
| **Repository URL** | Direct link to GitHub |
| **Description** | Repo description (from GitHub) |
| **Application Type** | Detected type (REST API, Windows Service, etc.) |
| **Target Framework** | .NET version (e.g., net8.0, v4.7.2) |
| **Suggested GCP Service** | Recommended GCP target (Cloud Run, GCE, etc.) |
| **Database Engine** | Detected DB (SQL Server, Postgres, etc.) |
| **Auth Architecture** | Windows Auth vs OAuth/OIDC |
| **Logging Framework** | Serilog, NLog, AppInsights, etc. |
| **Secrets Management** | Env vars vs Cloud Secret Manager |
| **CI/CD Pipeline** | GitHub Actions, Jenkins, etc. |
| **AWS SDK Detected** | Yes/No - Whether repo uses AWS SDK |
| **AWS Services Used** | List of AWS services detected |
| **Hardcoded AWS Found** | Yes/No - Detection of keys/regions in config |
| **Size (KB)** | Repository size |
| **Last Updated** | Last commit date |
| **Is Archived** | Yes/No - If repo is archived |
| **Migration Complexity** | Low / Medium / High |
| **Migration Notes** | Specific migration considerations |
### Sample Output
```
Repository Name | Application Type | Complexity | AWS SDK
================================================================================
batch-processor | Windows Service | High | Yes (DynamoDB, S3)
rest-api | REST API (ASP.NET) | Medium | Yes (RDS, SQS)
utility-lib | DLL/Library | Low | No
legacy-soap | SOAP API (ASMX) | High | No
```
---
## Detection Logic
### Application Type Detection
The analyzer detects types by checking:
1. **Windows Service**
- `WindowsService` or `TopShelf` in `.csproj`
- Windows service framework references
2. **REST API**
- `AspNetCore` or `AspNetFramework` in project files
- Presence of `Controllers` folder
- API attributes in code
3. **SOAP API**
- `.asmx` files present
- WCF (Windows Communication Foundation) references
- SOAP service namespaces
4. **DLL / Class Library**
- `ClassLibrary` project type in `.csproj`
- No executable output (`Exe`)
5. **Console Application**
- `Exe` output type
- `Program.cs` entry point
### AWS & Config Detection
The analyzer now performs a **Deep Scan** of configuration files:
1. **AWS SDKs**: Scans `.csproj` and `packages.config`.
2. **Environment Config**: Scans `appsettings.json`, `web.config`, and `app.config`.
3. **Sensitive Patterns**: Detects AWS Access Key IDs (AKIA/ASIA) and hardcoded AWS region endpoints.

### Complexity Scoring & GCP Mapping
| Factor | Points | Target Mapping |
|--------|--------|----------------|
| .NET Framework (Legacy) | +2 | GCE / App Engine Flex |
| .NET Core / 5+ (Modern) | +0 | Cloud Run (Recommended) |
| Windows Service | +3 | GCE with Custom Image |
| AWS SDK Usage | +2 | SDK Shim or Refactoring needed |
| High Complexity (6+) | - | Detailed assessment required |
---
## Troubleshooting
### Issue: "Authentication failed"
**Solution:** Verify your GitHub token is valid and has repo access
```bash
curl -H "Authorization: token ghp_xxx" https://api.github.com/user
```
### Issue: "Rate limit exceeded"
**Solution:** GitHub API has 60 requests/hour for unauthenticated, 5,000 for authenticated
- Default: Adds 1 second delay between repos
- For many repos: Implement exponential backoff (see advanced section)
### Issue: ".csproj not found" warnings
**Solution:** Some repos may not have `.csproj` at root
- Check subdirectories in `get_repo_files()` function
- Update code to recursively search
### Issue: "openpyxl not installed"
**Solution:**
```bash
pip install openpyxl
# Falls back to CSV if not available
```
---
## Advanced Usage
### Custom Filtering
Add filters to focus on specific repos:
```python
# Filter only repos with AWS usage
HIGH_RISK_REPOS = [r for r in results if r['aws_sdk_detected']]
# Filter by complexity
COMPLEX_MIGRATIONS = [r for r in results if r['migration_complexity'] == 'High']
# Filter by application type
WINDOWS_SERVICES = [r for r in results if 'Windows Service' in r['application_type']]
```
### Parallel Processing
For faster analysis with many repos:
```python
from concurrent.futures import ThreadPoolExecutor
def analyze_parallel(analyzer, repos):
with ThreadPoolExecutor(max_workers=5) as executor:
futures = [executor.submit(analyzer.analyze_repo, *repo.split("/")) for repo in repos]
return [f.result() for f in futures]
```
### Custom Classification
Extend the `detect_app_type()` method:
```python
def detect_app_type(self, repo_owner, repo_name, csproj_content=""):
types = []
# ... existing checks ...
# Custom: Check for specific frameworks
if "Hangfire" in csproj_content:
types.append("Background Job Processor")
if "MassTransit" in csproj_content:
types.append("Message Bus Service")
return "; ".join(types) if types else "Unknown"
```
---
## Best Practices
1. **Start small**: Test with 5-10 repos first
2. **Schedule regularly**: Run weekly to catch new repos
3. **Review results**: Manually verify classification for edge cases
4. **Track changes**: Keep version history of Excel exports for comparison
5. **Document assumptions**: Note any repos that required manual adjustment
---
## Next Steps After Inventory
Once you have the inventory:
1. **Prioritize**: Sort by complexity and business criticality
2. **Team assignment**: Assign repos to migration teams
3. **Detailed assessment**: Deep-dive on High complexity items
4. **Migration planning**: Create sprints based on complexity groups
5. **GCP mapping**: Match AWS services to GCP equivalents:
- AWS Lambda → Cloud Functions / Cloud Run
- RDS → Cloud SQL
- S3 → Cloud Storage
- DynamoDB → Firestore / Cloud Datastore
- SQS/SNS → Pub/Sub