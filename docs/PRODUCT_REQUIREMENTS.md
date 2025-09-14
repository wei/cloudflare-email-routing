# Product Requirements Document: GitHub-Driven Cloudflare Email Routing Management

## 1. Introduction & Executive Summary

This document outlines the requirements for a self-service system to manage Cloudflare Email Routing rules through a GitHub-native workflow. Currently, creating or removing email forwards requires direct administrative access, creating a bottleneck and operational overhead. This feature will empower users to manage their own email forwarding rules via GitHub Issues, while maintaining strict administrative oversight, security, and auditability. The entire process, from request to deployment, will be automated using GitHub Actions and direct Cloudflare API integration, ensuring that changes are tracked, approved, and applied systematically.

## 2. The Problem

Managing Cloudflare Email Routing rules is an infrastructure-level task, typically handled by a small group of administrators. This centralized model presents several challenges:

- **Bottlenecks & Delays:** Non-administrative users (e.g., developers, marketing team members) must file a ticket and wait for an admin to manually configure rules. This process can be slow and inefficient.
- **High Administrative Overhead:** Admins spend valuable time on routine, repetitive tasks instead of focusing on higher-impact infrastructure work.
- **Lack of User Empowerment:** Users have no direct way to request, verify, or remove the email forwards they need for their projects.
- **Limited Auditability:** While configuration changes can be tracked, the context behind a request—who asked for it, why, and the approval record—is often scattered across different systems.
- **Security Risks:** Granting more users access to infrastructure management tools is not a viable or secure scaling strategy.

We need a solution that decouples the _request_ for an email forward from the _implementation_ of the infrastructure change, creating a secure, auditable, and self-service workflow.

## 3. User Personas

- **Priya, the Developer:** Needs to create a temporary email address like `project-x-alerts@example.com` to receive notifications from a new service she is building. She wants to set this up quickly without needing to understand infrastructure details or wait for an admin.
- **Mark, the Marketer:** Wants to set up `webinar-september@example.com` to forward inquiries to his team's shared inbox for an upcoming campaign. He needs a simple, form-based way to make the request.
- **Alex, the Infrastructure Admin:** Is responsible for maintaining email routing configuration. Alex wants to delegate the routine task of managing email forwards while ensuring all changes are validated, approved, and auditable. Alex is the final gatekeeper for all changes.

## 4. Goals & Non-Goals

### Goals

- **Provide a simple UX** for requesting new email forwards through **GitHub Issues**.
- **Automate Destination Verification:** The workflow must integrate with Cloudflare's mandatory destination email verification process, automatically skipping verification for already verified addresses.
- **Enforce Collision Detection:** The system must prevent the creation of duplicate or conflicting forwarding rules.
- **Maintain Administrative Control:** An admin must explicitly approve all new forwarding rules before they are applied.
- **Automate API Deployment:** Automatically commit approved changes to a dedicated rules file and trigger a Cloudflare API deployment job.
- **Establish Clear Ownership:** Track the GitHub user who requested each rule to enable self-service removal.
- **Provide a User-Driven Removal Flow:** Allow users to request the deletion of forwarding rules they own.
- **Ensure Full Auditability:** Maintain a complete, transparent history of every request, verification, approval, and removal within GitHub Issues and the Git repository.

### Non-Goals

- Managing any Cloudflare settings (DNS, Zones, etc.) beyond Email Routing rules.
- Building a user interface or dashboard outside of the native GitHub ecosystem (Issues, Actions, PRs).
- Circumventing Cloudflare's destination verification process. The user must still click the verification link sent to their email.
- Supporting complex routing logic (e.g., multiple destinations for one source, catch-all rules). The initial scope is a 1-to-1 forward.

## 5. User Stories

| ID  | As a...               | I want to...                                                                                                                                       | So that I can...                                                         | Acceptance Criteria                                                                                                                                                                                                                                                                           |
| --- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Priya (Developer)** | Open a GitHub Issue using a template to request a new email forward from `source@example.com` to `dest@personal.com`.                              | Quickly set up an email for service alerts without waiting for an admin. | - A clear, intuitive issue template exists for "New Forward Request".<br>- The bot acknowledges the issue and checks for collisions.<br>- The bot either instructs me to check my email for a verification link OR automatically proceeds to approval if the destination is already verified. |
| 2   | **Priya (Developer)** | Receive a verification email from Cloudflare and confirm I've completed the step by commenting `/verify` on my issue (if verification was needed). | Prove I own the destination address and move my request forward.         | - If verification is needed: After I click the link, commenting `/verify` makes the bot check with Cloudflare's API.<br>- If already verified: This step is automatically skipped.<br>- The bot updates the issue label to `pending-approval`.                                                |
| 3   | **Alex (Admin)**      | Review a verified request and approve it by adding an `approved` label to the GitHub Issue.                                                        | Securely sanction the change and trigger the automated deployment.       | - Adding the `approved` label triggers a GitHub Action.<br>- The Action commits the new rule to the configuration.<br>- A separate, reusable workflow is triggered to apply changes via Cloudflare API.<br>- The issue is automatically closed with a success comment.                        |
| 4   | **Mark (Marketer)**   | Be notified with a clear error message if I request a source email that is already in use.                                                         | Understand why my request failed and try again with a different address. | - If `webinar-september@example.com` already exists, the bot comments with an error.<br>- The bot automatically closes the issue to prevent stale requests.                                                                                                                                   |
| 5   | **Priya (Developer)** | Open a different GitHub Issue using a "Remove Forward" template to delete a rule I previously created.                                             | Decommission an email forward that is no longer needed for my project.   | - The removal issue template asks for the source address to be removed.<br>- The bot checks the registry file to confirm I am the owner.<br>- If ownership is confirmed, the rule is removed via API and the issue is closed.                                                                 |
| 6   | **Alex (Admin)**      | View the history of any email forward—who requested it, who approved it, and when it was added or removed.                                         | Maintain a clear audit trail for security and compliance.                | - All actions are recorded as comments and label changes in the GitHub Issue.<br>- The central registry file provides a single source of truth for ownership.                                                                                                                                 |

## 6. Functional & Technical Requirements

### 6.1. Data & State Management

- **Forwarding Rules Registry (`email_forwards.json`):**
  - A single JSON file in the repository will serve as the source of truth for all forwarding rules.
  - The list of forwards must be kept sorted alphabetically by the `source` field to ensure consistent ordering and prevent merge conflicts.
  - The TypeScript SDK deployment script will parse this file to create the necessary Cloudflare email routing rules.
  - Each entry must contain:
    - `source`: The source email address (e.g., `hello@example.com`).
    - `destination`: The destination email address (e.g., `user@gmail.com`).
    - `owner`: The GitHub username of the requester (e.g., `priya-dev`).

  _Example `email_forwards.json`:_

  ```json
  {
    "_comment": "Email Forwards Registry - managed automatically by GitHub Actions",
    "forwards": [
      {
        "source": "project-x-alerts@example.com",
        "destination": "priya@personal.com",
        "owner": "priya-dev"
      },
      {
        "source": "webinar-september@example.com",
        "destination": "mark@team-inbox.com",
        "owner": "mark-mktg"
      }
    ]
  }
  ```

### 6.2. GitHub Issue Templates

- **`new_forward_request.md`:**
  - Fields for "Source Email" and "Destination Email".
  - Uses structured issue forms for easy parsing by the bot.
- **`remove_forward_request.md`:**
  - Field for "Source Email to Remove".
  - Checkbox confirming they understand the action is irreversible.

### 6.3. Required GitHub Repository Labels

The workflow depends on specific labels existing in the GitHub repository. Administrators must create these labels before deploying the system:

- `pending-verification`: Automatically applied by the workflow when a user has submitted a request and needs to verify their destination email address (only applied if verification is needed).
- `pending-approval`: Automatically applied by the workflow after successful verification or when verification is automatically skipped, indicating the request is waiting for admin approval.
- `approved`: **Manually applied by administrators** to trigger the automated deployment workflow.
- `new_forward_request`: Automatically applied to issues created with the new forward request template.
- `remove_forward_request`: Automatically applied to issues created with the remove forward request template.

**Label State Management:** The workflows automatically handle label transitions throughout the request lifecycle. The only manual label action required from administrators is applying the `approved` label to trigger deployment.

### 6.4. GitHub Actions Workflows

- **Concurrency Control:** All workflows that modify the registry file will use a concurrency group to ensure that only one instance runs at a time. This prevents race conditions where multiple workflows attempt to write to the `email_forwards.json` file simultaneously.

  ```yaml
  concurrency:
    group: email-forwards-registry
    cancel-in-progress: false
  ```

- **Workflow 1: Request Intake & Verification (`on: issues, types: [opened]`)**
  1. **Trigger:** New issue with `new_forward_request` label.
  2. **Parse Issue:** Extract `source` and `destination`.
  3. **Collision Check:** Read `email_forwards.json` and fail if `source` exists.
  4. **Create Destination Address:** Use the Cloudflare API to create the destination address.
  5. **Check Verification Status:** Parse the API response to check if the destination is already verified.
  6. **If Not Verified:** Add a comment: "Request received... A verification email has been sent... Please click the link... and then comment `/verify` here." Apply the `pending-verification` label.
  7. **If Already Verified:** Add a comment: "Request processed... Destination already verified... Awaiting admin approval." Apply the `pending-approval` label directly.

- **Workflow 2: Handle `/verify` Comment (`on: issue_comment, types: [created]`)**
  1. **Trigger:** Comment with `/verify` on an issue with the `pending-verification` label.
  2. **Check Verification Status:** Call Cloudflare API to confirm verification.
  3. **On Success:** Remove `pending-verification`, add `pending-approval`, and comment "✅... awaiting admin approval."
  4. **On Failure:** Comment: "Verification failed..."

- **Workflow 3: Handle Approval & Deployment (`on: issues, types: [labeled]`)**
  1. **Trigger:** `approved` label is added to an issue with `pending-approval`.
  2. **Check Actor:** Verify the user is in the `ADMIN_USERS`.
  3. **Update Registry:** Read `email_forwards.json`, add the new rule, sort the list by `source`, and write the file back.
  4. **Commit & Push:** Commit the change with message `feat: Add email forward... via #${issue_id}`.
  5. **Trigger API Deployment:** Call the reusable `api_apply.yml` workflow.
  6. **Close Issue:** On success, comment "🚀 Approved and deployed!" and close the issue.

- **Workflow 4: Handle Removal Request (`on: issues, types: [opened]`)**
  1. **Trigger:** New issue with `remove_forward_request` label.
  2. **Parse Issue:** Extract `source` to be removed.
  3. **Ownership Check:** Read `email_forwards.json` and verify the issue author matches the `owner`. If not, comment with an error and close.
  4. **Update Registry:** Remove the entry from `email_forwards.json`. The list will remain sorted.
  5. **Commit & Push:** Commit the change with message `fix: Remove email forward... via #${issue_id}`.
  6. **Trigger API Deployment & Close:** Call API deployment workflow and close the issue on success.

### 6.4. Cloudflare API Integration

The system uses the Cloudflare TypeScript SDK via GitHub Actions to manage email routing resources. The email routing operations are handled directly within the workflow using the `wei/cloudflare-script` action based on the `email_forwards.json` registry file:

#### API Configuration

The TypeScript SDK requires the following environment variables to be set in GitHub Secrets:

- `CLOUDFLARE_API_TOKEN` - API token with Email Routing permissions
- `CLOUDFLARE_ZONE_ID` - Zone ID for email routing rules
- `CLOUDFLARE_ACCOUNT_ID` - Account ID for email routing addresses

#### API Operations

The system automatically handles all operations through GitHub Actions workflows:

```yaml
# Apply all email forwards from registry (via workflow)
uses: wei/cloudflare-script@master
env:
  CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
with:
  script: |
    // TypeScript code using Cloudflare SDK
    const forwards = await cloudflare.emailRouting.rules.list({
      zone_id: '${{ secrets.CLOUDFLARE_ZONE_ID }}'
    });
```

#### Dynamic Resource Management

The script automatically:

1. Reads the `email_forwards.json` registry file
2. Creates destination addresses if they don't exist
3. Creates routing rules for new forwards
4. Skips existing rules to avoid duplicates
5. Provides detailed feedback on operations performed

This ensures the Cloudflare configuration is always in sync with the registry file.

## 7. Non-Functional Requirements

- **Security:**
  - `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ZONE_ID`, and `CLOUDFLARE_ACCOUNT_ID` will be stored as encrypted GitHub secrets and used as environment variables by the GitHub Actions workflows.
  - These credentials are never stored as plain text but are provided securely through the CI/CD pipeline.
  - Approval logic must strictly check that the user adding the `approved` label is part of a predefined admin group.
  - Removal logic must strictly check for ownership.
- **Reliability:**
  - Workflows must be idempotent where possible.
  - Concurrency controls will prevent race conditions during file writes.
- **Auditability:**
  - State changes must be reflected by labels and bot comments in the issue.
  - Commit messages must reference the originating issue number.

## 8. Risks and Mitigations

| Risk                            | Likelihood | Impact | Mitigation Strategy                                                                                                                                                                                                                                       |
| ------------------------------- | ---------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cloudflare API Failure          | Low        | Medium | The Cloudflare API could be down or return an error. The GitHub Action will be configured to fail loudly, leaving the issue open for an admin to investigate. Implement retry logic for transient network errors.                                         |
| User Never Verifies Destination | High       | Low    | A user requests a forward but never clicks the verification link. The issue will remain in the `pending-verification` state. A separate, scheduled workflow will run weekly to ping users with stale requests and close them after 14 days of inactivity. |

## 9. Success Metrics

- **Reduction in Admin Toil:** 75% reduction in the number of hours admins spend per week manually managing email forwarding rules.
- **Adoption Rate:** >90% of all new forwarding rules are created via the self-service workflow within 3 months of launch.
- **Time-to-Fulfillment:** The average time from issue creation to deployment for a new forward is reduced from days to under 1 hour (contingent on admin approval time).
- **User Satisfaction:** Positive feedback from developers and other users who benefit from the new, faster process.

## 10. Future Work & Open Questions

- **Editing Existing Rules:** Should we support editing a destination address? This adds complexity but could be a valuable feature.
- **Bulk Imports:** A script could be developed to import existing manually-created rules into the `email_forwards.json` format.
- **Notifications:** Integrate with Slack to notify an `#infra-approvals` channel when an issue is moved to `pending-approval`.
- **Multiple Destinations:** Can we extend the model to support forwarding one source to multiple destinations?
