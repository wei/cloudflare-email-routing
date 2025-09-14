# Self-Service Email Forwarding for `example.com`

Welcome! This repository provides an automated, self-service system for managing email forwarding rules, powered by Cloudflare Email Routing. You can request new email forwards or remove ones you own, all through GitHub Issues.

## Table of Contents

- [Requesting a New Email Forward](#requesting-a-new-email-forward)
- [Removing an Existing Email Forward](#removing-an-existing-email-forward)
- [How It Works (The Lifecycle of a Request)](#how-it-works-the-lifecycle-of-a-request)

## Requesting a New Email Forward

[![New Forward Request](https://img.shields.io/badge/New_Forward_Request-Open_Issue-brightgreen?style=for-the-badge&logo=github)](../../issues/new?template=new_forward_request.yml)

Follow these steps to create a new forward (e.g., `new-name@example.com` -> `your-email@gmail.com`).

### Step 1: Open a New Issue

1. Navigate to the **Issues** tab in this repository.
2. Click the **New issue** button.
3. Find the "New Forward Request" template and click **Get started**.

### Step 2: Fill Out the Form

- **Source Email:** Enter the new email address you want to create (e.g., `new-name@example.com`). This address must not already exist.
- **Destination Email:** Enter the existing, real email address where you want to receive the forwarded emails (e.g., `your-email@gmail.com`).

Submit the issue once you've filled it out. A bot will immediately acknowledge your request.

### Step 3: Verify Your Destination Email (If Needed)

For security, Cloudflare requires you to prove you own the destination address. **If your destination email is already verified in Cloudflare, this step will be automatically skipped and your request will proceed directly to admin approval.**

If verification is needed:

1. **Check Your Inbox:** Look for a verification email from Cloudflare in the inbox of your destination address.
2. **Click the Verification Link:** Open the email and click the confirmation link.
3. **Confirm on GitHub:** Return to the GitHub issue you created and add a new comment containing only the text `/verify`.

The bot will then confirm the verification and automatically update the issue status to `pending-approval`.

### Step 4: Admin Approval

An administrator will be notified. Once they review and approve your request by adding the `approved` label, the workflow will automatically apply the change. The bot will comment on and close the issue once the forward is active.

## Removing an Existing Email Forward

[![Remove Forward Request](https://img.shields.io/badge/Remove_Forward_Request-Open_Issue-red?style=for-the-badge&logo=github)](../../issues/new?template=remove_forward_request.yml)

You can only remove forwards that you are the original owner of.

1. Navigate to the **Issues** tab and click **New issue**.
2. Find the "Remove Forward Request" template and click **Get started**.
3. Enter the full **Source Email** that you wish to delete.
4. Submit the issue.

The bot will check the registry to confirm you are the owner. If confirmed, the forward will be removed automatically, and the issue will be closed. If you are not the owner, the bot will post an error and close the issue.

## How It Works (The Lifecycle of a Request)

1. **Issue Opened:** You open an issue. The bot automatically checks for conflicts and asks you to verify your destination (unless already verified).
2. **`pending-verification`:** The request is paused, waiting for you to click the link from Cloudflare and comment `/verify`. This label is automatically applied when verification is needed.
3. **`pending-approval`:** You have verified your email, or verification was automatically skipped. The bot automatically transitions to this status. The request is now in the queue for an administrator to review.
4. **`approved`:** An admin has manually approved your request by adding this label. An automated job commits the change to the `email_forwards.json` file and triggers a Cloudflare TypeScript SDK deployment.
5. **Issue Closed:** The forward is now active, and the issue is automatically closed by the bot.
