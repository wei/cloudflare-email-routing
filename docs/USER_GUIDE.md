# Get your own `@example.com` email

This repository provides an automated, self-service system for managing email forwarding rules, powered by Cloudflare Email Routing. Request new email forwards or remove existing ones through GitHub Issues.

[![Cloudflare](https://img.shields.io/badge/Email%20Routing-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://developers.cloudflare.com/email-routing/)
[![GitOps](https://img.shields.io/badge/GitOps-Automation-326CE5?logo=git&logoColor=white)](https://www.gitops.tech/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Self Service](https://img.shields.io/badge/Self%20Service-GitHub%20Issues-181717?logo=github&logoColor=white)](https://docs.github.com/en/issues)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## Get Started

[![New Forward Request](https://img.shields.io/badge/New_Forward_Request-Open_Issue-brightgreen?style=for-the-badge&logo=github)](../../issues/new?template=new_forward_request.yml) [![Remove Forward Request](https://img.shields.io/badge/Remove_Forward_Request-Open_Issue-red?style=for-the-badge&logo=github)](../../issues/new?template=remove_forward_request.yml)

**To create a new forward:** Click "New Forward Request" and fill out the form.

**To remove an existing forward:** Click "Remove Forward Request" (you can only remove forwards you own).

## How It Works

1. **Open an issue** using one of the templates above
2. **Email verification** may be required for new forwards (if not already verified with Cloudflare)
3. **Admin approval** is required for new forwards
4. **Automated deployment** applies approved changes to Cloudflare Email Routing
