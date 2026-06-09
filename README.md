# Prowler CLI Workshop Guide

This guide walks you through installing Prowler CLI, running your first scan, exporting reports, and launching the dashboard.

Steps are provided for all three operating systems side by side. Follow the column that matches your machine.

## Prerequisites

| # | Linux (Amazon Linux / Ubuntu) | Windows | macOS |
|---|------|---------|-------|
| 1 | Python 3.12+ installed (`python3 --version`) | Python 3.12+ installed ([python.org](https://www.python.org/downloads/)) — check "Add to PATH" during install | Python 3.12+ installed via Homebrew: `brew install python@3.12` |
| 2 | `pip` available (`pip3 --version`) | `pip` available (`pip --version`) | `pip` available (`pip3 --version`) |
| 3 | AWS CLI v2 installed | AWS CLI v2 installed | AWS CLI v2 installed |

### AWS CLI Setup

The AWS CLI is required so Prowler can authenticate to your AWS account. If you don't have it installed yet, follow the official guide:

- [Installing the AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Once installed, log in using your browser-based console credentials:

```bash
aws login
```

This opens your browser and authenticates the CLI using the same sign-in method you use for the AWS Management Console. It generates short-lived credentials that rotate automatically — no long-term access keys needed.

- [aws login documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sign-in.html)

You will need an IAM user or role with at minimum the `SecurityAudit` and `ViewOnlyAccess` managed policies attached.

Verify your credentials are working:

```bash
aws sts get-caller-identity
```

If this returns your account ID and ARN, you're good to go.

---

## Step 1: Install Prowler

| Linux | Windows | macOS |
|-------|---------|-------|
| `pip3 install prowler` | `pip install prowler` | `pip3 install prowler` |
| Verify: `prowler --version` | Verify: `prowler --version` | Verify: `prowler --version` |

> **Note:** If you get a "command not found" error, ensure your Python scripts directory is in your PATH.
>
> - Linux/macOS: Add `export PATH="$HOME/.local/bin:$PATH"` to your shell profile
> - Windows: The installer usually handles this, but check `%APPDATA%\Python\Python312\Scripts`

---

## Step 2: Run Your First Scan

> **Important:** Prowler creates an `output/` folder in whatever directory you run the command from. Navigate to a familiar location first so you can easily find your results.

| Linux | Windows | macOS |
|-------|---------|-------|
| `cd ~/Documents` | `cd %USERPROFILE%\Documents` | `cd ~/Documents` |
| `prowler aws` | `prowler aws` | `prowler aws` |

This runs a full AWS scan using your configured credentials. It may take 20-45 minutes depending on your account size. Results will be saved to `~/Documents/output/` (or `Documents\output\` on Windows).

For a quicker workshop demo, scope the scan:

| Linux | Windows | macOS |
|-------|---------|-------|
| `prowler aws --services s3 iam --severity critical high` | `prowler aws --services s3 iam --severity critical high` | `prowler aws --services s3 iam --severity critical high` |

---

## Step 3: Review Output Files

Prowler writes output to an `output/` folder inside your current directory (i.e., `~/Documents/output/` since we ran it from Documents). The following formats are generated:

| Format | File | Purpose |
|--------|------|---------|
| CSV | `output/*.csv` | Machine-readable findings (semicolon-delimited) |
| JSON-OCSF | `output/*.ocsf.json` | Open Cybersecurity Schema Framework format |
| HTML | `output/*.html` | Human-readable report you can open in a browser |

| Linux | Windows | macOS |
|-------|---------|-------|
| `ls -la ~/Documents/output/` | `dir %USERPROFILE%\Documents\output\` | `ls -la ~/Documents/output/` |
| Open HTML: `xdg-open ~/Documents/output/*.html` | Open HTML: double-click the `.html` file in `Documents\output\` via Explorer | Open HTML: `open ~/Documents/output/*.html` |

---

## Step 4: Export to Specific Formats

You can control output formats explicitly:

| Linux | Windows | macOS |
|-------|---------|-------|
| `prowler aws --output-formats csv json-ocsf html` | `prowler aws --output-formats csv json-ocsf html` | `prowler aws --output-formats csv json-ocsf html` |

To send output to a specific directory (instead of the default `output/` folder in your current directory):

| Linux | Windows | macOS |
|-------|---------|-------|
| `prowler aws --output-directory ~/Documents/prowler-reports` | `prowler aws --output-directory %USERPROFILE%\Documents\prowler-reports` | `prowler aws --output-directory ~/Documents/prowler-reports` |

---

## Step 5: Launch the Dashboard

The Prowler CLI dashboard provides an interactive web UI to explore your scan results.

| Linux | Windows | macOS |
|-------|---------|-------|
| `prowler dashboard` | `prowler dashboard` | `prowler dashboard` |
| Open browser: `http://localhost:11666` | Open browser: `http://localhost:11666` | Open browser: `http://localhost:11666` |

The dashboard reads CSV files from the default `output/` folder relative to where you ran the scan. Since we ran from `~/Documents`, it will look for `~/Documents/output/`. Make sure you launch the dashboard from the same directory you ran the scan in.

---

## Step 5b: Dashboard Port Binding (EC2 / Remote Server Only)

| Linux (EC2 or remote) | Windows | macOS |
|------------------------|---------|-------|
| By default the dashboard binds to `127.0.0.1`, which means it only accepts connections from the same machine. If you're running on a remote server (e.g., EC2) and want to access the dashboard from your browser, you need to apply a workaround. See the **Remote Server Setup** section below. | Not applicable — when running locally on Windows, `localhost:11666` works in your browser since you're on the same machine. | Not applicable — when running locally on macOS, `localhost:11666` works in your browser since you're on the same machine. |

---

## Step 6: Explore the Dashboard

Once the dashboard is running and you've opened `http://localhost:11666` (or `http://<server-ip>:11666` for remote):

- **Overview page:** Summary of findings by severity, service, and region
- **Compliance page:** Results mapped to compliance frameworks (CIS, PCI-DSS, etc.)
- Use the navigation sidebar to switch between views

---

## Remote Server Setup (EC2 Workshop Environment)

If you're running this workshop on an EC2 instance and need attendees to access the dashboard from their browsers, follow these additional steps.

### EC2 Prerequisites

| Step | Details |
|------|---------|
| IAM Role | Attach `SecurityAudit` + `ViewOnlyAccess` + `AmazonSSMManagedInstanceCore` to the instance |
| Security Group | Open inbound TCP port `11666` (source: workshop network CIDR or `0.0.0.0/0`) |
| Instance Type | `t3.medium` minimum (2 vCPU, 4 GB RAM) |
| AMI | Amazon Linux 2023 |
| Public IP | Enable auto-assign public IP |

### Connect to the Instance

No SSH needed. Use Session Manager:

1. Go to **EC2 > Instances > Select instance > Connect**
2. Choose **Session Manager** tab
3. Click **Connect**

### Install Prowler

```bash
# Add local bin to PATH first to avoid pip warnings
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

# Install Prowler
sudo dnf install -y python3.12 python3.12-pip
pip3.12 install prowler
```

Verify:

```bash
prowler --version
```

### Run the Scan

```bash
cd /home/ssm-user/
mkdir Documents
prowler aws --services s3 iam --severity critical high
```

The instance's IAM role provides the AWS credentials automatically — no `aws configure` needed on the EC2 instance.

### Launch the Dashboard (with external access)

By default, `prowler dashboard` binds to `127.0.0.1` (localhost only), so external browsers cannot reach it. Since Prowler's CLI does not currently expose a `--host` flag for the dashboard, a workaround is needed.

Below are two options. Choose one.

#### Workaround A: Patch the Prowler dashboard binding (recommended)

This modifies the installed Prowler package to bind the dashboard to all network interfaces (`0.0.0.0`) instead of localhost only. It's a one-time change that persists for the life of the instance.

```bash
# Find and patch the dashboard config
DASHBOARD_INIT=$(python3.12 -c "import dashboard; import os; print(os.path.dirname(dashboard.__file__))")/__init__.py
sed -i 's/DASHBOARD_ARGS = {"debug"/DASHBOARD_ARGS = {"host": "0.0.0.0", "debug"/' "$DASHBOARD_INIT"
```

Then launch the dashboard normally:

```bash
cd /home/ssm-user/Documents
prowler dashboard
```

> **Note:** This is a workaround that modifies Prowler's installed code. It changes the dashboard to accept connections from any network interface rather than localhost only. This is safe for a workshop environment but should not be done on a shared or production system without understanding the security implications.

#### Workaround B: Use socat as a port forwarder

This leaves Prowler completely untouched. Instead, `socat` forwards external traffic on port 11666 to the dashboard's localhost binding.

```bash
# Install socat
sudo dnf install -y socat

# Start the port forwarder in the background
socat TCP-LISTEN:11666,fork,reuseaddr,bind=0.0.0.0 TCP:127.0.0.1:11666 &
```

Then launch the dashboard on a different port (since socat is already listening on 11666, or use the same port with socat forwarding):

```bash
cd /home/ssm-user/Documents
prowler dashboard
```

> **How this works:** Prowler dashboard starts on `127.0.0.1:11666` as usual. `socat` listens on `0.0.0.0:11666` and forwards incoming external connections to `127.0.0.1:11666`. Since both try to use port 11666, you need to start socat first — it will bind to the external interface while Prowler binds to localhost. This works because they bind to different interfaces.
>
> **Note:** If `socat` conflicts with Prowler on the same port, use a different external port:
> ```bash
> socat TCP-LISTEN:8080,fork,reuseaddr,bind=0.0.0.0 TCP:127.0.0.1:11666 &
> ```
> Then attendees access `http://<instance-ip>:8080` instead (and update your security group accordingly).

### Access the Dashboard

Open a browser and navigate to:

```
http://<instance-public-ip>:11666
```

You can find the public IP in the AWS Console under **EC2 > Instances > Select your instance > Public IPv4 address**.

### Download Report Files

Session Manager's browser terminal doesn't support file downloads directly. The simplest way to get the CSV/HTML output off the instance is to upload it to S3 and download it from the S3 console.

#### Add S3 permission to the IAM role

Add the following inline policy to the `ProwlerWorkshopRole`. Replace `YOUR-BUCKET-NAME` with the bucket attendees create during the workshop:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUploadToWorkshopBucket",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/reports/*"
    }
  ]
}
```

This only grants permission to upload objects under the `reports/` prefix in that one bucket — nothing else.

#### Upload the report

```bash
aws s3 cp /home/ssm-user/Documents/output/ s3://YOUR-BUCKET-NAME/reports/ --recursive
```

Attendees can then navigate to the S3 console, open the bucket, and download their files from there.

---

## Cleanup

| Linux (EC2) | Windows (local) | macOS (local) |
|-------------|-----------------|---------------|
| `Ctrl+C` to stop the dashboard | `Ctrl+C` in the terminal running the dashboard | `Ctrl+C` in the terminal running the dashboard |
| Terminate the EC2 instance | No cleanup needed | No cleanup needed |
| Delete the security group and IAM role | | |

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `prowler: command not found` | Ensure Python scripts directory is in PATH (see Step 1 note) |
| `aws sts get-caller-identity` fails | AWS CLI not configured — run `aws configure` or check your credentials file |
| Scan returns no findings | Verify AWS credentials have `SecurityAudit` and `ViewOnlyAccess` policies |
| Dashboard shows "no data" | Ensure CSV files exist in the `output/` directory and you launched the dashboard from the same directory |
| Dashboard not reachable on EC2 | Check security group has port 11666 open; verify the binding patch was applied (Workaround A) or socat is running (Workaround B) |
| Session Manager won't connect | Ensure IAM role includes `AmazonSSMManagedInstanceCore` and instance has internet access |
