# Cloudflare Email Routing

> **📨 GitHub-driven self-service email forwarding system.**

[![Cloudflare](https://img.shields.io/badge/Email%20Routing-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://developers.cloudflare.com/email-routing/)
[![GitOps](https://img.shields.io/badge/GitOps-Automation-326CE5?logo=git&logoColor=white)](https://www.gitops.tech/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Self Service](https://img.shields.io/badge/Self%20Service-GitHub%20Issues-181717?logo=github&logoColor=white)](https://docs.github.com/en/issues)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

This project provides a secure and automated way for users to request new email addresses with forwarding rules, while giving administrators full control over approvals and deployment. The entire workflow is driven by GitHub Issues and powered by GitHub Actions and Cloudflare Email Routing API.

## ✨ Features

- **🎫 GitHub-Native Workflow:** Users request forwards using a simple GitHub Issue template.
- **🔒 Secure Admin Approval:** Designated administrators approve requests by adding a label.
- **🚀 Automated Deployment:** Approved requests are automatically deployed to Cloudflare via the API.
- **🤖 Bot-Driven Automation:** A GitHub Actions bot handles validation, verification, and notifications.
- **🔍 Collision Detection:** Prevents duplicate or conflicting forwarding rules.
- **👤 Ownership Tracking:** Users can only manage the forwarding rules they create.
- **📋 Full Audit Trail:** Every action is logged in GitHub Issues and Git history for complete transparency.
- **⏰ Stale Request Cleanup:** Automatically closes abandoned requests to keep the queue clean.

## 🚀 Getting Started

This repository is a template. To set up your own self-service email forwarding system, follow the instructions below.

### Prerequisites

- A **Cloudflare Account** with a domain and Email Routing enabled.
- A **GitHub Account** with a repository created from this template.

### For Administrators: Setup & Configuration

To get the system running, please follow the comprehensive **[Administrator Setup Guide](docs/ADMIN_GUIDE.md)**. This guide covers:

- Setting up Cloudflare API credentials.
- Creating the required GitHub secrets, variables, and labels.
- Defining administrator teams and permissions.
- Preparing the repository for user requests.

### For Users: Requesting Email Forwards

Once the system is configured, users can request and manage email forwards by following the **[User Guide](docs/USER_GUIDE.md)**.

## 📄 License

**[MIT](https://wei.mit-license.org)**

## 👨‍💻 Author

**Made with ❤️ by [@wei](https://github.com/wei)**
