# CloudFormation Deployment Guide

Deploy `cloudformation.yaml` via the AWS Console (CloudFormation > Create Stack > Upload template) or CLI.

## After Deployment

The instance runs a UserData script that installs Python 3.12, Prowler, and patches the dashboard binding. This takes 2-3 minutes after the instance launches.

### Check if setup is complete

```bash
ls /home/ssm-user/.prowler-setup-complete
```

If the file exists, setup is done. If not, it's still running or failed.

### Watch setup progress in real-time

```bash
sudo tail -f /var/log/cloud-init-output.log
```

### Check if setup is still running

```bash
ps aux | grep -i "pip\|dnf"
```

If only the `grep` process shows up, the script has finished.

## Troubleshooting

| Issue | Command |
|-------|---------|
| Prowler not found | `which prowler` — if empty, check the cloud-init log for errors |
| Setup failed | `sudo cat /var/log/cloud-init-output.log | tail -80` |
| Dashboard patch not applied | `python3.12 -c "import dashboard; print(dashboard.DASHBOARD_ARGS)"` — should show `host: 0.0.0.0` |
| Documents folder missing | `ls /home/ssm-user/Documents` — if missing, setup didn't complete |
| Instance has no public IP | Check subnet has "Auto-assign public IPv4" enabled |
| Can't reach dashboard | Verify security group has port 11666 open and `prowler dashboard` is running |
| cloud-init didn't run | `cloud-init status` — should say `done` |
