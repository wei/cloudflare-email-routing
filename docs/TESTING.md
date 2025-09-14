# Testing Guide for Cloudflare Email Routing

This document provides comprehensive testing instructions for the email forwarding system powered by Cloudflare Email Routing.

## Prerequisites for Testing

1. **Repository Setup Complete**
   - All files present and configured
   - GitHub secrets added (CLOUDFLARE_API_TOKEN, CLOUDFLARE_ZONE_ID, CLOUDFLARE_ACCOUNT_ID)
   - Required labels created

2. **Cloudflare Configuration**
   - Email Routing enabled for your domain
   - API token with proper permissions
   - Test email addresses available for verification

## Test Scenarios

### Test 1: New Forward Request (Happy Path)

1. **Create Request**
   - Navigate to Issues → New Issue
   - Select "New Email Forward Request" template
   - Fill in test data:
     - Source: `test-forward@yourdomain.com`
     - Destination: `your-test-email@gmail.com`
   - Submit issue

2. **Expected Behavior (New Destination)**
   - Bot comment acknowledging request
   - `pending-verification` label applied
   - No collision detected
   - Cloudflare verification email sent

3. **Expected Behavior (Already Verified Destination)**
   - Bot comment acknowledging request and noting destination already verified
   - `pending-approval` label applied directly (skips verification)
   - No collision detected
   - No verification email needed

4. **Verify Email (If Needed)**
   - Check destination email inbox
   - Click Cloudflare verification link
   - Comment `/verify` on the issue

5. **Expected Behavior (After Verification)**
   - `pending-verification` label removed
   - `pending-approval` label added
   - Bot comment confirming verification

6. **Admin Approval**
   - As an authorized admin, add `approved` label
   - Check workflow runs in Actions tab

7. **Expected Behavior**
   - Registry file updated with new forward
   - API configuration applied successfully
   - Issue closed with success message
   - Email forward active in Cloudflare

### Test 2: Already Verified Destination

1. **Create Request with Previously Verified Email**
   - Use a destination email that has been verified in Cloudflare before
   - Create new forward request
   - Submit issue

2. **Expected Behavior**
   - Bot detects destination is already verified
   - Bot comment indicating automatic verification skip
   - `pending-approval` label applied directly
   - No verification email sent
   - No need for `/verify` comment

3. **Admin Approval**
   - As an authorized admin, add `approved` label

4. **Expected Behavior**
   - Registry file updated with new forward
   - API configuration applied successfully
   - Issue closed with success message

### Test 3: Collision Detection

1. **Create Duplicate Request**
   - Create another request with same source email
   - Submit issue

2. **Expected Behavior**
   - Bot detects collision
   - Error message posted
   - Issue automatically closed

### Test 4: Email Verification Failure

1. **Create Request Without Verification**
   - Create new forward request
   - Comment `/verify` without clicking Cloudflare link

2. **Expected Behavior**
   - Bot reports verification failed
   - Issue remains in `pending-verification` state

### Test 5: Unauthorized Approval

1. **Non-Admin Approval Attempt**
   - As a non-admin user, try adding `approved` label

2. **Expected Behavior**
   - Approval rejected
   - `approved` label removed
   - Error message posted

### Test 6: Forward Removal (Happy Path)

1. **Create Removal Request**
   - Navigate to Issues → New Issue
   - Select "Remove Email Forward" template
   - Specify source email you previously created
   - Submit issue

2. **Expected Behavior**
   - Bot verifies ownership
   - Registry updated (forward removed)
   - API configuration applied
   - Issue closed with success message

### Test 7: Unauthorized Removal

1. **Wrong Owner Removal**
   - As different user, try to remove someone else's forward
   - Submit removal request

2. **Expected Behavior**
   - Bot detects ownership mismatch
   - Error message with actual owner info
   - Issue automatically closed

### Test 8: Stale Request Cleanup

1. **Create Old Request**
   - Create a request but don't verify email
   - Wait for weekly cleanup or trigger manually

2. **Expected Behavior**
   - Requests older than 14 days get closed
   - Requests 7-14 days old get reminder
   - Appropriate bot messages posted

## Manual Testing Checklist

- [ ] All GitHub Actions workflows execute without errors
- [ ] Issue templates render correctly
- [ ] Bot comments are formatted properly and helpful
- [ ] Label transitions work as expected
- [ ] Registry file updates maintain correct format and sorting
- [ ] API configuration successfully updates Cloudflare
- [ ] Email forwarding works end-to-end (send test email)
- [ ] Error cases handle gracefully with clear messages
- [ ] Audit trail is complete in issues and git history

## Troubleshooting Common Issues

### Workflow Failures

1. **Permission Denied**
   - Check GITHUB_TOKEN permissions in workflow
   - Verify branch protection settings allow pushes

2. **Cloudflare API Errors**
   - Verify API token has correct permissions
   - Check zone_id and account_id are correct
   - Ensure Email Routing is enabled

### Bot Not Responding

1. **Check Triggers**
   - Verify issue labels match workflow conditions
   - Check workflow file syntax
   - Review GitHub Actions logs

2. **Parsing Issues**
   - Ensure issue body format matches expected structure
   - Check for special characters in email addresses

### Email Verification Problems

1. **Verification Emails Not Sent**
   - Check Cloudflare account limits
   - Verify destination email is valid
   - Check spam/junk folders

2. **Verification Status Not Detected**
   - Wait a few minutes after clicking link
   - Check Cloudflare dashboard manually
   - Verify API permissions include email routing

## Performance Testing

For larger deployments, test:

- [ ] Multiple concurrent requests
- [ ] Large registry files (100+ forwards)
- [ ] Network timeouts and retries
- [ ] Rate limiting behavior

## Security Testing

Verify:

- [ ] Only authorized users can approve
- [ ] Users can only remove their own forwards
- [ ] API credentials are never exposed in logs
- [ ] Email verification prevents unauthorized destinations
- [ ] Git commits maintain proper attribution

## Monitoring

Set up monitoring for:

- [ ] GitHub Actions workflow success/failure rates
- [ ] Issue response times
- [ ] API configuration success rates
- [ ] Email delivery success (if needed)

## Documentation Testing

Verify:

- [ ] README instructions are accurate
- [ ] Setup guide can be followed successfully
- [ ] User guide covers all common scenarios
- [ ] Admin guide includes all necessary steps
