# Spartan.Actions - Troubleshooting Guide

## ?? Common Issues & Solutions

### Issue: "Sending has been rejected because the sender you used is not valid"

**Error Message:**
```
Sending has been rejected because the sender you used noreply@spartangym.com is not valid. 
Validate your sender or authenticate your domain
```

**Cause:**
Brevo requires the sender email address (`from` field) to be verified in your account.

**Solution:**

#### ? Current Fix Applied
The workflow now uses `bojan.ruzic@gmail.com` as the sender, which should already be verified in your Brevo account.

#### Verify Sender in Brevo
1. Log in to [Brevo](https://app.brevo.com)
2. Go to **Senders & IP** ? **Senders**
3. Check if `bojan.ruzic@gmail.com` is listed
4. Status should show as **Verified** (green checkmark)
5. If not verified:
   - Click **Add a sender**
   - Enter `bojan.ruzic@gmail.com`
   - Click verification link sent to your email
   - Wait for verification (usually instant)

#### Alternative: Use Domain Authentication
If you want to use `noreply@spartangym.com`:
1. Go to **Senders & IP** ? **Domains**
2. Click **Authenticate a domain**
3. Follow instructions to add DNS records for `spartangym.com`
4. Wait for DNS propagation (24-48 hours)
5. Update workflow `from` field back to `noreply@spartangym.com`

---

### Issue: "At least one of 'to', 'cc' or 'bcc' must be specified"

**Error Message:**
```
Error: At least one of 'to', 'cc' or 'bcc' must be specified
```

**Cause:**
The `ALERT_EMAIL` secret is not configured in GitHub repository secrets.

**Solution:**
1. Go to: https://github.com/bokerman/Spartan.Actions
2. Navigate to: **Settings** ? **Secrets and variables** ? **Actions** ? **Secrets**
3. Click **"New repository secret"**
4. Add:
   - Name: `ALERT_EMAIL`
   - Value: `bojan.ruzic@gmail.com`
5. Click **"Add secret"**

---

### Issue: SMTP Authentication Failed

**Error Message:**
```
Error: SMTP authentication failed
```

**Cause:**
Incorrect SMTP username or password in GitHub secrets.

**Solution:**

#### Option 1: Use Brevo API Key (Easiest)
Your API key from `.env` file works as SMTP password:
```
BREVO_SMTP_USERNAME: bojan.ruzic@gmail.com
BREVO_SMTP_PASSWORD: xkeysib-YOUR_BREVO_API_KEY_HERE
```

#### Option 2: Use Dedicated SMTP Credentials
1. Log in to [Brevo](https://app.brevo.com)
2. Go to **SMTP & API** ? **SMTP** tab
3. Look for **SMTP credentials** section
4. Copy the username and password shown
5. Update GitHub secrets with these values

---

### Issue: Health Check Returns 404

**Error Message:**
```
TEST returned unexpected status: 404
```

**Cause:**
The health endpoint URL is incorrect or the API is not deployed.

**Solution:**
1. **Verify Container App URL:**
   - Go to Azure Portal
   - Find your Container App (`spartangym-api-test` or `spartangym-api-prod`)
   - Copy the **Application URL**
   - Example: `https://spartangym-api-test.gentledune-2f88f2cc.canadacentral.azurecontainerapps.io`

2. **Update GitHub Variables:**
   - Go to: https://github.com/bokerman/Spartan.Actions
   - Navigate to: **Settings** ? **Secrets and variables** ? **Actions** ? **Variables**
   - Update:
     - `TEST_API_URL` (remove `https://` prefix)
     - `PROD_API_URL` (remove `https://` prefix)

3. **Verify Health Endpoint:**
   - Test manually: `curl https://<your-url>/spartan/health`
   - Should return HTTP 200
   - Check if API is deployed and running

---

### Issue: Workflow Runs But No Emails Sent

**Possible Causes:**
1. Health checks are returning 200 (healthy) or 503 (warning) - no alerts triggered
2. Email secrets are incorrect
3. Brevo sender not verified
4. Brevo daily limit exceeded (300 emails/day)

**Solution:**
1. **Check Workflow Logs:**
   - Go to Actions ? Select a run
   - Check if "Send Email Alert" step ran
   - Review any error messages

2. **Test with Manual Failure:**
   - Temporarily change `TEST_API_URL` to an invalid URL
   - Run workflow manually
   - Should trigger email alert
   - Restore correct URL after test

3. **Check Brevo Dashboard:**
   - Go to [Brevo Logs](https://app.brevo.com/log)
   - Look for sent emails
   - Check for errors or bounces

---

### Issue: Scheduled Workflow Not Running

**Error Message:**
(None - workflow just doesn't run automatically)

**Cause:**
GitHub disables scheduled workflows after 60 days of repository inactivity.

**Solution:**
1. **Check Workflow Status:**
   - Go to Actions tab
   - Look for warning: "This scheduled workflow is disabled..."
   - Click "Enable workflow" if disabled

2. **Prevent Future Disabling:**
   - Commit to repository every 45-50 days
   - Set up calendar reminder
   - See "Maintenance Strategy" in main README

3. **Create Maintenance Commit:**
   ```bash
   cd C:\Projects\SpartanGym\Spartan.Actions
   git commit --allow-empty -m "chore: maintenance commit to keep workflows active"
   git push
   ```

---

## ?? Debugging Tips

### View Workflow Logs
1. Go to: https://github.com/bokerman/Spartan.Actions/actions
2. Click on a workflow run
3. Expand each step to see detailed logs
4. Look for error messages in red

### Test Workflow Manually
1. Go to Actions tab
2. Select "Keep Warm - Scheduled Ping"
3. Click "Run workflow" ? "Run workflow"
4. Wait for completion
5. Check Summary tab for results

### Verify All Secrets/Variables
```
Required Secrets:
? ALERT_EMAIL
? BREVO_SMTP_USERNAME
? BREVO_SMTP_PASSWORD

Required Variables:
? PROD_API_URL
? TEST_API_URL
```

### Check Email Delivery
1. Go to [Brevo Logs](https://app.brevo.com/log)
2. Filter by date/time of workflow run
3. Look for sent emails
4. Check delivery status

---

## ?? Need Help?

**Repository:** https://github.com/bokerman/Spartan.Actions  
**Documentation:** See `README.md` for full setup guide  
**Contact:** Bojan Ruzic (bojan.ruzic@gmail.com)

---

**Last Updated:** February 2025

