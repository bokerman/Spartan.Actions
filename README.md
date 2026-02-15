# Spartan.Actions - Keep-Warm & Health Monitoring

This repository contains GitHub Actions workflows that keep Azure Container Apps warm and monitor their health.

## ?? Features

- **Scheduled Keep-Warm Pings**: Prevents cold starts by pinging health endpoints every 10 minutes
- **Health Monitoring**: Tracks response codes and response times
- **Email Alerts**: Sends email notifications when health checks fail
- **Status Tracking**: Differentiates between healthy (200), warning (503/timeout), and unhealthy states
- **GitHub Summaries**: Creates detailed summaries for each workflow run

## ?? Workflow: Keep-Warm

**File:** `.github/workflows/keep-warm.yml`

**Schedule:** Every 10 minutes (24/7)

**Environments Monitored:**
- PRODUCTION (`spartangym-api-prod`)
- TEST (`spartangym-api-test`)

## ?? Configuration Required

### 1. Repository Variables

Set up the following variables in **Settings ? Secrets and variables ? Actions ? Variables**:

| Variable Name | Description | Example |
|--------------|-------------|---------|
| `PROD_API_URL` | Production API base URL (without protocol) | `spartangym-api-prod.nicegrass-12345678.canadacentral.azurecontainerapps.io` |
| `TEST_API_URL` | Test API base URL (without protocol) | `spartangym-api-test.nicegrass-12345678.canadacentral.azurecontainerapps.io` |

### 2. Repository Secrets

Set up the following secrets in **Settings ? Secrets and variables ? Actions ? Secrets**:

| Secret Name | Description | How to Get |
|------------|-------------|------------|
| `BREVO_SMTP_USERNAME` | Brevo SMTP login (email or key) | From Brevo Dashboard ? SMTP & API ? SMTP settings |
| `BREVO_SMTP_PASSWORD` | Brevo SMTP password/key | From Brevo Dashboard ? SMTP & API ? SMTP settings |
| `ALERT_EMAIL` | Email address to receive alerts | Your admin email (e.g., `bojan.ruzic@gmail.com`) |

### 3. Brevo SMTP Setup

1. Log in to [Brevo](https://app.brevo.com)
2. Go to **SMTP & API** ? **SMTP** tab
3. Note your SMTP credentials:
   - **Server:** `smtp-relay.brevo.com`
   - **Port:** `587`
   - **Login:** Your email or SMTP key
   - **Password:** Your SMTP password or API key
4. Add these credentials as GitHub secrets (see above)

## ?? Alert Behavior

### Production Alerts (High Priority)
- **Trigger:** HTTP status code is NOT 200, 503, or timeout
- **Subject:** `?? SpartanGym PROD Health Check Failed`
- **Priority:** High
- **Action:** Immediate investigation required

### Test Alerts (Normal Priority)
- **Trigger:** HTTP status code is NOT 200, 503, or timeout
- **Subject:** `?? SpartanGym TEST Health Check Failed`
- **Priority:** Normal
- **Action:** Investigate during business hours

### Status Classification

| Status | HTTP Code | Behavior | Alert? |
|--------|-----------|----------|--------|
| ? Healthy | 200 | Normal operation | No |
| ?? Warning | 503, 000 | Cold start or temporary issue | No |
| ? Unhealthy | Other codes | Unexpected error | Yes |

**Note:** 503 (Service Unavailable) and 000 (timeout/connection error) are treated as warnings because they often occur during cold starts when the container is waking up. These are expected and do not trigger alerts.

## ?? Monitoring Dashboard

Each workflow run creates a **GitHub Summary** showing:
- Health status for each environment
- Response codes and response times
- Timestamps
- Direct links to troubleshooting resources

**Access:** Go to **Actions** ? Select a workflow run ? View **Summary** tab

## ??? Troubleshooting

### Email Alerts Not Working

1. **Verify Secrets:**
   ```bash
   # Check if secrets are configured (they should show as ****)
   Settings ? Secrets and variables ? Actions ? Secrets
   ```

2. **Test Brevo SMTP:**
   - Log in to Brevo and check SMTP credentials
   - Verify daily email limit (300/day on free tier)
   - Check Brevo logs for failed sends

3. **Check Workflow Logs:**
   - Go to Actions ? Select a failed run
   - Expand "Send Email Alert" step
   - Review error messages

### Health Checks Failing

1. **Verify URLs:**
   - Check `PROD_API_URL` and `TEST_API_URL` variables
   - Ensure URLs do NOT include `https://` prefix
   - Test manually: `curl https://<URL>/spartan/health`

2. **Check Azure Container Apps:**
   - Verify apps are running in Azure Portal
   - Check if apps scaled to zero
   - Review Application Insights for errors

3. **Review Response Codes:**
   - 503: Container waking up (normal for cold starts)
   - 000: Connection timeout or network issue
   - 404: Health endpoint not found (check URL)
   - 500: Server error (check logs)

## ?? Important: Repository Activity Requirement

**GitHub disables scheduled workflows after 60 days of repository inactivity!**

To prevent workflow disruption:
- Commit to this repository at least once every 50 days
- Set up a calendar reminder
- See [Maintenance Strategy](#maintenance-strategy) below

### Maintenance Strategy

**Option 1: Manual Commits (Recommended)**
- Update README.md or documentation every 45-50 days
- Bump version numbers
- Add monitoring notes

**Option 2: Automated Maintenance Commit**
Create a workflow that auto-commits every 50 days (coming soon)

### Monitoring Repository Activity

Check last commit date:
```bash
git log -1 --format="%ai"
```

If approaching 50 days, create a maintenance commit:
```bash
git commit --allow-empty -m "chore: maintenance commit to keep workflows active"
git push
```

## ?? Cost Implications

- **GitHub Actions:** FREE (workflows use ~5 minutes/day, well within 2,000 min/month free tier)
- **Brevo Emails:** FREE (alerts use ~1-5 emails/day, within 300/day limit)
- **Azure Container Apps:** Keeps apps warm, prevents cold start delays
- **Estimated Savings:** $5-10 CAD/month by avoiding frequent cold starts

## ?? Related Resources

- **Azure Portal:** [Container Apps Dashboard](https://portal.azure.com)
- **Brevo Dashboard:** [Email Monitoring](https://app.brevo.com)
- **Application Insights:** Check logs for detailed error traces
- **Documentation:** See `_DevNotes/Maintenance/Monitoring.md` in Spartan.Backend repo

## ?? Action Items

- [x] Set up keep-warm workflow
- [x] Add health monitoring and alerting
- [ ] Configure Brevo SMTP secrets in GitHub
- [ ] Configure alert email address
- [ ] Test email alerts (trigger a manual failure)
- [ ] Set up calendar reminder for 50-day maintenance commits
- [ ] Monitor workflow execution logs weekly

---

**Last Updated:** February 2025  
**Maintained By:** DevOps Team  
**Contact:** Bojan Ruzic (bojan.ruzic@gmail.com)
