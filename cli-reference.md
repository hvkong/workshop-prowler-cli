# Prowler CLI Reference

Quick reference for Prowler CLI arguments. Run `prowler -h` or `prowler <provider> -h` for full help text.

## Basic Usage

```bash
prowler <provider> [options]
```

If no provider is specified, `aws` is used by default.

```bash
# These are equivalent
prowler
prowler aws
```

---

## Providers

| Provider | Description |
|----------|-------------|
| `aws` | Amazon Web Services (default) |
| `azure` | Microsoft Azure |
| `gcp` | Google Cloud Platform |
| `kubernetes` | Kubernetes clusters |
| `m365` | Microsoft 365 |
| `github` | GitHub |
| `googleworkspace` | Google Workspace |
| `okta` | Okta |
| `cloudflare` | Cloudflare |
| `oraclecloud` | Oracle Cloud Infrastructure |
| `openstack` | OpenStack |
| `alibabacloud` | Alibaba Cloud |
| `iac` | Infrastructure as Code (Terraform, CloudFormation) |
| `llm` | LLM Provider (Beta) |
| `image` | Container Image scanning |
| `nhn` | NHN Provider |
| `mongodbatlas` | MongoDB Atlas |
| `scaleway` | Scaleway |
| `stackit` | StackIT |
| `vercel` | Vercel |
| `dashboard` | Launch local dashboard |

---

## Scan Scope

Control what gets scanned. These flags are mutually exclusive (pick one):

| Flag | Short | Description |
|------|-------|-------------|
| `--check <check_id ...>` | `-c` | Run specific checks by ID |
| `--checks-file <path>` | `-C` | JSON file listing checks to run |
| `--service <name ...>` | `-s` | Run all checks for specific services |
| `--compliance <framework>` | | Run checks for a compliance framework (e.g., `cis_3.0_aws`) |
| `--category <name ...>` | | Run checks in specific categories |
| `--resource-group <name ...>` | | Run checks for specific resource groups |

These can be combined with scan scope:

| Flag | Short | Description |
|------|-------|-------------|
| `--severity <level ...>` | | Filter by severity: `critical`, `high`, `medium`, `low`, `informational` |
| `--checks-folder <path>` | `-x` | Load custom checks from an external directory |

### Examples

```bash
# Scan only S3 and IAM services
prowler aws --services s3 iam

# Only critical and high severity findings
prowler aws --severity critical high

# Run CIS 3.0 benchmark
prowler aws --compliance cis_3.0_aws

# Specific checks only
prowler aws --check s3_bucket_public_access iam_root_mfa_enabled

# Combine service filter with severity
prowler aws --services ec2 --severity critical high
```

---

## Exclude Checks/Services

| Flag | Short | Description |
|------|-------|-------------|
| `--excluded-check <check_id ...>` | `-e` | Skip specific checks |
| `--excluded-checks-file <path>` | | JSON file listing checks to skip |
| `--excluded-service <name ...>` | | Skip entire services |

### Examples

```bash
# Skip specific noisy checks
prowler aws --excluded-check s3_bucket_versioning_enabled

# Skip an entire service
prowler aws --excluded-service cloudwatch
```

---

## Output

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--output-formats <format ...>` | `-M` | `csv json-ocsf html` | Output file formats |
| `--output-directory <path>` | `-o` | `./output` | Directory for output files |
| `--output-filename <name>` | `-F` | auto-generated | Custom filename (no extension) |
| `--status <status ...>` | | all | Filter output by status: `PASS`, `FAIL`, `MANUAL` |
| `--verbose` | | off | Show all check results in terminal |
| `--no-banner` | `-b` | off | Hide the Prowler banner |
| `--no-color` | | off | Disable color codes in terminal |
| `--unix-timestamp` | | off | Use unix timestamps instead of ISO format |
| `--ignore-exit-code-3` | `-z` | off | Don't exit with code 3 on failed checks |

Available output formats: `csv`, `json-ocsf`, `html`, `json-asff` (AWS only), `json-sarif` (IaC only)

### Examples

```bash
# Output only CSV to a custom directory
prowler aws --output-formats csv --output-directory ~/reports

# Custom filename
prowler aws --output-filename my-scan-2024

# Show only failures
prowler aws --status FAIL

# Verbose terminal output
prowler aws --verbose
```

---

## Dashboard

Launch an interactive web dashboard to explore scan results.

```bash
prowler dashboard
```

Opens at `http://localhost:11666`. Reads CSV files from the `output/` directory relative to where you run the command.

> **Note:** The dashboard does not accept CLI flags for host or port. It always binds to `127.0.0.1:11666`.

---

## Logging

| Flag | Default | Description |
|------|---------|-------------|
| `--log-level <LEVEL>` | `CRITICAL` | Log verbosity: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `--log-file <path>` | none | Write logs to a file |
| `--only-logs` | off | Print only logs to stdout (implies `--no-banner`) |

---

## Configuration

| Flag | Description |
|------|-------------|
| `--config-file <path>` | Custom Prowler configuration file |
| `--fixer-config <path>` | Custom fixer configuration file |
| `--mutelist-file <path>` | YAML mutelist to suppress findings (also supports DynamoDB/S3/Lambda for AWS) |
| `--custom-checks-metadata-file <path>` | Override check metadata (severity, descriptions) |

---

## List / Discovery

These flags print information and exit — they don't run a scan.

| Flag | Short | Description |
|------|-------|-------------|
| `--list-checks` | `-l` | List all available checks |
| `--list-checks-json` | | List checks in JSON format |
| `--list-services` | | List services covered by the provider |
| `--list-compliance` | | List available compliance frameworks |
| `--list-compliance-requirements <framework>` | | List requirements for a specific framework |
| `--list-categories` | | List available check categories |
| `--list-resource-groups` | | List available resource groups |
| `--list-fixer` | | List available auto-remediation fixers |

### Examples

```bash
# See all AWS checks
prowler aws --list-checks

# What compliance frameworks are available?
prowler aws --list-compliance

# What does CIS 3.0 check?
prowler aws --list-compliance-requirements cis_3.0_aws

# List all services prowler covers for GCP
prowler gcp --list-services
```

---

## 3rd Party Integrations

| Flag | Short | Description |
|------|-------|-------------|
| `--shodan` | `-N` | Check public IPs against Shodan (uses `SHODAN_API_KEY` env var) |
| `--slack` | | Send summary to Slack (requires `SLACK_API_TOKEN` and `SLACK_CHANNEL_NAME` env vars) |
| `--push-to-cloud` | | Send findings to Prowler Cloud (requires `PROWLER_CLOUD_API_KEY` env var) |

---

## AWS-Specific Flags

These only apply when using `prowler aws`.

### Authentication

| Flag | Short | Description |
|------|-------|-------------|
| `--profile <name>` | `-p` | AWS CLI profile to use |
| `--role <arn>` | `-R` | IAM role ARN to assume |
| `--role-session-name <name>` | | Custom session name for assumed role |
| `--session-duration <seconds>` | `-T` | STS session duration (default: 3600) |
| `--external-id <id>` | `-I` | External ID for cross-account role assumption |
| `--mfa` | | Prompt for MFA TOTP code |

### Regions

| Flag | Description |
|------|-------------|
| `--region <region ...>` | Scan only specific regions |
| `--excluded-region <region ...>` | Skip specific regions |

### Organizations

| Flag | Short | Description |
|------|-------|-------------|
| `--organizations-role <arn>` | `-O` | Role ARN to assume in each org account |

### Security Hub

| Flag | Short | Description |
|------|-------|-------------|
| `--security-hub` | `-S` | Send findings to AWS Security Hub |
| `--skip-sh-update` | | Don't update previous Security Hub findings |
| `--send-sh-only-fails` | | Only send FAIL findings to Security Hub |

### S3 Output

| Flag | Short | Description |
|------|-------|-------------|
| `--output-bucket <name>` | `-B` | Upload output to S3 bucket (assumes role) |
| `--output-bucket-no-assume <name>` | `-D` | Upload to S3 without assuming a role |

### Resource Filtering

| Flag | Description |
|------|-------------|
| `--resource-tag <Key=Value ...>` | Scan only resources with specific tags |
| `--resource-arn <arn ...>` | Scan only specific resources by ARN |

### Other

| Flag | Description |
|------|-------------|
| `--aws-retries-max-attempts <n>` | Max boto3 API retry attempts |
| `--scan-unused-services` | Include checks for unused services |
| `--quick-inventory` | Run a quick resource inventory (no security checks) |
| `--fixer` | Auto-remediate findings where fixers are available |

### Examples

```bash
# Use a specific profile
prowler aws --profile production

# Assume a role in another account
prowler aws --role arn:aws:iam::123456789012:role/ProwlerAudit

# Scan only us-east-1 and eu-west-1
prowler aws --region us-east-1 eu-west-1

# Send results to Security Hub
prowler aws --security-hub

# Upload output directly to S3
prowler aws --output-bucket my-prowler-reports

# Scan only resources tagged with Environment=production
prowler aws --resource-tag Environment=production

# Quick inventory of all resources
prowler aws --quick-inventory
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All checks passed |
| 2 | Execution error |
| 3 | One or more checks failed (suppressible with `-z`) |

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `SHODAN_API_KEY` | API key for Shodan integration |
| `SLACK_API_TOKEN` | Token for Slack integration |
| `SLACK_CHANNEL_NAME` | Slack channel for notifications |
| `PROWLER_CLOUD_API_KEY` | API key for Prowler Cloud integration |

---

## Cross-Account Scanning (AssumeRole)

Prowler can scan a different AWS account by assuming a role in that account. Your current credentials (local profile, EC2 instance role, etc.) are used to call `sts:AssumeRole` on the target.

### Prerequisites

1. **Target account** has a role (e.g., `ProwlerAuditRole`) with:
   - `SecurityAudit` and `ViewOnlyAccess` policies attached
   - A trust policy allowing your source account/role to assume it

2. **Source account** (where you run Prowler) has `sts:AssumeRole` permission on the target role ARN

### Trust policy on the target role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {}
    }
  ]
}
```

Replace `111111111111` with the source account ID (the account running Prowler).

### Usage

```bash
# Assume a role in another account
prowler aws --role arn:aws:iam::222222222222:role/ProwlerAuditRole

# With a custom session name
prowler aws --role arn:aws:iam::222222222222:role/ProwlerAuditRole --role-session-name prowler-workshop

# With an external ID (required by some trust policies)
prowler aws --role arn:aws:iam::222222222222:role/ProwlerAuditRole --external-id my-external-id

# With MFA (prompts for TOTP code)
prowler aws --role arn:aws:iam::222222222222:role/ProwlerAuditRole --mfa

# Custom session duration (seconds, default 3600)
prowler aws --role arn:aws:iam::222222222222:role/ProwlerAuditRole --session-duration 7200
```

### How it works

1. Prowler uses your current credentials to call `sts:AssumeRole`
2. AWS returns temporary credentials for the target role
3. Prowler uses those temporary credentials for all API calls during the scan
4. Results are tagged with the target account ID

### Multi-account with Organizations

For scanning all accounts in an AWS Organization:

```bash
prowler aws --organizations-role arn:aws:iam::222222222222:role/ProwlerAuditRole
```

This assumes the same role name exists in every member account and the management account credentials can assume it.

---

## Common Recipes

```bash
# Full AWS scan with all defaults
prowler aws

# Fast scan for a demo
prowler aws --services s3 iam --severity critical high

# CIS benchmark with verbose output
prowler aws --compliance cis_3.0_aws --verbose

# Multi-region scan excluding noisy checks
prowler aws --region us-east-1 eu-west-1 --excluded-service cloudwatch

# Azure scan
prowler azure

# GCP scan for specific project
prowler gcp --project-id my-project-id

# Kubernetes cluster scan
prowler kubernetes

# IaC scan on a Terraform directory
prowler iac --iac-framework terraform --iac-path ./infra

# Generate only CSV output to a specific folder
prowler aws --output-formats csv --output-directory /tmp/prowler-scan
```
