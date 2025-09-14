# Administrator's Setup Guide: Cloudflare Email Routing

This guide provides detailed instructions for administrators to fork, configure, and deploy this self-service email forwarding system powered by Cloudflare Email Routing.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Fork and Clone the Repository](#step-1-fork-and-clone-the-repository)
- [Step 2: Configure Cloudflare](#step-2-configure-cloudflare)
- [Step 3: Configure the GitHub Repository](#step-3-configure-the-github-repository)
- [Step 4: Final Deployment and Testing](#step-4-final-deployment-and-testing)
- [How the System Works](#how-the-system-works)

## Prerequisites

Before you begin, ensure you have the following:

- **An active Cloudflare account** with the domain you wish to manage.
- **A GitHub repository** where this system will live.

## Important Considerations

Before proceeding with the setup, please be aware of the following critical limitations and behaviors of Cloudflare Email Routing and this system:

- **Repository Visibility**: It is highly recommended to keep this repository **private**. The `email_forwards.json` file, which serves as the single source of truth for your Cloudflare Email Routing rules, will contain all source and destination email addresses. Exposing this file publicly could lead to privacy concerns and potential abuse.
- **Cloudflare Email Routing Rule Limit**: Cloudflare Email Routing has a default limit of **200 rules per zone**. If your domain requires more than 200 routing rules, you will need to contact Cloudflare support to request a limit increase as the system will not be able to create rules beyond this limit.
- **Existing Rules**: This system operates on a "GitOps" principle, where `email_forwards.json` is the single source of truth for your Cloudflare Email Routing rules. **Any existing Cloudflare Email Routing rules that are NOT managed by this system (i.e., not defined in `email_forwards.json`) may be removed or overwritten** when the `cloudflare-apply.yml` workflow runs. It is highly recommended to migrate all desired rules into `email_forwards.json` before deploying this system, or to ensure that this system is deployed to a zone with no pre-existing rules.
- **Rule Type Compatibility**: This system is designed to manage **forwarding rules** only. It is **not compatible** with Cloudflare Email Routing rules that use the "Drop" action or "Send to a Worker" action. Attempting to manage these rule types through `email_forwards.json` may lead to unexpected behavior or errors.

## Step 1: Fork and Clone the Repository

Start by creating a **private** repo from this template inside your own GitHub organization or account.

## Step 2: Configure Cloudflare

### Create a Cloudflare API Token

The GitHub Actions workflow needs an API token to interact with your Cloudflare account.

1. In your Cloudflare dashboard, go to **My Profile** > **API Tokens**.
2. Click **Create Token** and use the "Create Custom Token" option.
3. Give the token a descriptive name (e.g., `example.com Email Routing`).
4. Grant the following permissions:
   - **Zone** -> **Email Routing** -> **Edit**
   - **Account** -> **Email Routing Addresses** -> **Edit**
5. Under **Zone Resources**, select the specific zone you want this system to manage (e.g., `example.com`).
6. Create the token and **copy it immediately**. You will not be able to see it again.

## Step 3: Configure the GitHub Repository

### Add Repository Secrets

Navigate to your repository's **Settings** > **Secrets and variables** > **Actions** to add the following encrypted secrets:

- `CLOUDFLARE_API_TOKEN`: The API token you created in Step 2.
- `CLOUDFLARE_ZONE_ID`: The Zone ID of your Cloudflare domain, found on the "Overview" page in the Cloudflare dashboard.
- `CLOUDFLARE_ACCOUNT_ID`: The Account ID of your Cloudflare account, found on the same "Overview" page.

**Note:** The `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ZONE_ID`, and `CLOUDFLARE_ACCOUNT_ID` secrets will be used by the GitHub Actions workflows to authenticate with Cloudflare and manage email routing directly via the Cloudflare API. They are not passed as plain text but are made available as environment variables during workflow execution.

### Define Repository Variables

Create the following repository variables:

- `ADMIN_USERS`: A comma-delimited list of GitHub usernames who are authorized to approve requests (e.g., `user1,user2,user3`).
- `DOMAIN_NAME`: The base domain that all source email addresses must belong to (e.g., `example.com`). Subdomains of this domain are also allowed (e.g., `team.example.com`) but requires additional setup in Cloudflare. Requests with a source outside this domain are rejected.

The workflow checks against these variables before proceeding with an `approved` label. Approval can be granted by members of the specified team or any user in the list.

### Create Required Issue Labels

The workflow relies on specific labels to track the state of email forward requests.

Create the following labels in your repository's **Issues** section:

### Method 1: Using the GitHub CLI (`gh`) (Recommended)

```bash
gh label create "pending-verification" --color "ff9500" --description "Applied when waiting for email verification"
gh label create "pending-approval" --color "fbca04" --description "Applied when waiting for admin approval"
gh label create "approved" --color "0e8a16" --description "Manually applied by admins to trigger deployment"
gh label create "new_forward_request" --color "0052cc" --description "Applied to new forward requests"
gh label create "remove_forward_request" --color "d73a49" --description "Applied to removal requests"
```

### Method 2: Manual Creation

1. Go to your repository's **Issues** tab and click **Labels**.
2. Create these labels:
   - `pending-verification` (suggested color: orange `#ff9500`) - Automatically applied by workflow when waiting for user to verify their destination email
   - `pending-approval` (suggested color: yellow `#fbca04`) - Automatically applied by workflow when verification is complete and waiting for admin approval
   - `approved` (suggested color: green `#0e8a16`) - **Manually applied by admins** to trigger the deployment workflow
   - `new_forward_request` (suggested color: blue `#0052cc`) - Automatically applied to new forward requests by the issue template.
   - `remove_forward_request` (suggested color: red `#d73a49`) - Automatically applied to removal requests by the issue template.

**Note:** The workflows automatically manage label transitions. `new_forward_request` and `remove_forward_request` are added automatically when an issue is created. Admins only need to add the `approved` label to trigger deployment.

## Step 4: Prepare README

1. Overwrite `README.md` with contents of `docs/USER_GUIDE.md`. This will contain instructions users will see when they visit the repository.
2. Update the `docs/USER_GUIDE.md` file to reflect your specific domain.
3. Update new issue templates in `.github/ISSUE_TEMPLATE` to reflect your specific domain.

**To automate this process, run the following commands:**

```bash
# Update domain references in USER_GUIDE.md and issue templates (replace "your-domain.com" with your domain)
perl -i -pe 's/example\.com/your-domain\.com/g' docs/USER_GUIDE.md .github/ISSUE_TEMPLATE/*.yml

# Replace README.md with USER_GUIDE.md content
cp docs/USER_GUIDE.md README.md
```

## Step 5: Final Deployment and Testing

1. **Test GitHub Actions Setup:** Ensure all required secrets (CLOUDFLARE_API_TOKEN, CLOUDFLARE_ZONE_ID, CLOUDFLARE_ACCOUNT_ID) are properly configured in your repository settings.

2. **Create a Test Issue:** Use the system as a regular user would. Go to the **Issues** tab and open a "New Forward Request".

3. **Verify and Approve:** Follow the full workflow:
   - Check for the verification email.
   - Comment `/verify` on the issue.
   - As an admin (a user listed in `ADMIN_USERS`), add the `approved` label.

4. **Confirm the Result:**
   - Check that the GitHub Actions workflow runs successfully.
   - Verify that the `email_forwards.json` file in your repository was updated.
   - Confirm in your Cloudflare dashboard that the new email routing rule is active.

Your self-service email forwarding system is now fully configured and operational.

## How the System Works

### Architecture Overview

This system uses a **GitOps workflow** where infrastructure changes are triggered through GitHub Issues and managed via automated GitHub Actions:

1. **User Request** → GitHub Issue with structured template
2. **Validation** → GitHub Actions workflow checks for conflicts and triggers Cloudflare verification
3. **Approval** → Admin adds `approved` label to authorize deployment
4. **Deployment** → The `wei/cloudflare-script` action is triggered, which uses the Cloudflare TypeScript SDK to create/update Email Routing rules.
5. **Registry** → JSON file serves as single source of truth for all forwarding rules

### Key Components

- **`email_forwards.json`**: Central registry file containing all email forwarding rules
- **GitHub Actions Workflows**: Handle request intake, verification, and approval.
- **`wei/cloudflare-script` Action**: A generic GitHub Action that executes Cloudflare TypeScript SDK commands to manage Email Routing addresses and rules.
- **Cloudflare API**: Used for email verification and managing Email Routing rules

### Cloudflare Resources Used

The system manages several Cloudflare Email Routing resources via the `wei/cloudflare-script` action and the Cloudflare TypeScript SDK:

- **Email Routing Addresses**: Requires `account_id` (account-level resource)
  - Manages destination email addresses that need verification
  - Used when users request new forwarding rules

- **Email Routing Rules**: Requires `zone_id` (zone-level resource)
  - Defines the actual forwarding rules (source → destination mapping)
  - Created/updated when rules are approved

**Note**: The distinction between account-level and zone-level resources explains why both `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_ZONE_ID` are needed as secrets.

### Security Model

- **Encrypted Secrets**: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ZONE_ID`, `CLOUDFLARE_ACCOUNT_ID` stored as GitHub secrets
- **Environment Variables**: Secrets injected as environment variables during workflow execution
- **Admin Approval**: Manual approval required before any infrastructure changes
- **Audit Trail**: Complete history maintained in GitHub Issues and commit messages
