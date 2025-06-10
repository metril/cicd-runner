# GitLab Runner Service Workflows

This repository contains GitHub Actions workflows that provision temporary GitLab runners across different operating systems (macOS, Ubuntu, and Windows) with Tailscale network connectivity.

## Overview

These workflows create ephemeral GitLab runners that can execute jobs from your GitLab instance while connected to your private network via Tailscale. The runners are automatically cleaned up after job completion.

## Workflows

### 1. GitLab Runner Service - MacOS (`gitlab-runner-macos.yaml`)
- **Runner OS**: `ubuntu-latest` (GitHub Actions runner)
- **Target Environment**: macOS via Docker
- **Tailscale Version**: 1.62.0

### 2. GitLab Runner Service - Ubuntu (`gitlab-runner-ubuntu.yaml`)
- **Runner OS**: `ubuntu-latest`
- **Target Environment**: Ubuntu via Docker
- **Tailscale Version**: 1.66.3

### 3. GitLab Runner Service - Windows (`gitlab-runner-windows.yaml`)
- **Runner OS**: `windows-latest`
- **Target Environment**: Native Windows
- **Setup**: Manual installation of Tailscale and GitLab Runner

## Prerequisites

Before using these workflows, ensure you have:

1. **GitLab Instance**: A GitLab server with admin access
2. **Tailscale Account**: With OAuth credentials and ACL tags configured
3. **GitHub Repository**: With Actions enabled
4. **Required Secrets**: Configure the following in your GitHub repository

## Required Inputs

All workflows require the following inputs when manually triggered:

| Parameter | Description | Required |
|-----------|-------------|----------|
| `tailscale_oauth_client_id` | Tailscale OAuth Client ID | Yes |
| `tailscale_oauth_client_secret` | Tailscale OAuth Client Secret | Yes |
| `tailscale_tags` | ACL tags for Tailscale device authorization | Yes |
| `runner_authentication_token` | GitLab runner registration token | Yes |
| `gitlab_url` | Your GitLab instance URL (e.g., https://gitlab.example.com) | Yes |
| `docker_image` | Docker image to use for the runner | Yes |
| `delete_runner` | Whether to delete the runner after completion | Yes (default: false) |

## Setup Instructions

### 1. Configure Tailscale OAuth

1. Go to your [Tailscale Admin Console](https://login.tailscale.com/admin/)
2. Navigate to Settings → OAuth clients
3. Create a new OAuth client with appropriate permissions
4. Note the Client ID and Client Secret

### 2. Get GitLab Runner Token

1. In your GitLab instance, go to Admin Area → CI/CD → Runners
2. Click "New instance runner"
3. Copy the authentication token

### 3. Configure GitHub Secrets (Optional)

While inputs are provided manually, you may want to store sensitive values as repository secrets:

```
TAILSCALE_OAUTH_CLIENT_ID
TAILSCALE_OAUTH_CLIENT_SECRET
GITLAB_RUNNER_TOKEN
```

## Usage

### Manual Trigger

1. Go to your GitHub repository
2. Click on "Actions" tab
3. Select the desired workflow (macOS, Ubuntu, or Windows)
4. Click "Run workflow"
5. Fill in the required parameters
6. Click "Run workflow"

### Workflow Behavior

#### macOS and Ubuntu Workflows
- Use the `metril/gitlab-runner-action` custom action
- Automatically handle runner registration, execution, and cleanup
- Connect via Tailscale for secure network access

#### Windows Workflow
- Manually installs Tailscale via Chocolatey
- Downloads and configures GitLab Runner directly
- Monitors job completion via Windows Event Log
- Performs cleanup operations

## Features

### Security
- **Input Masking**: Sensitive inputs are automatically masked in logs
- **Ephemeral Tailscale**: Connections are temporary and auto-expire
- **Automatic Cleanup**: Runners are unregistered after job completion

### Network Connectivity
- **Tailscale Integration**: Secure access to private networks
- **DNS Resolution**: Accepts DNS settings from Tailscale
- **Route Advertisement**: Can access private subnets

### Flexibility
- **Multiple OS Support**: macOS, Ubuntu, and Windows
- **Docker Integration**: Configurable Docker images for containerized jobs
- **Custom Naming**: Runners are named using GitHub run IDs for uniqueness

## Troubleshooting

### Common Issues

**Runner Registration Fails**
- Verify GitLab URL is accessible
- Check authentication token validity
- Ensure GitLab instance allows new runners

**Tailscale Connection Issues**
- Verify OAuth credentials
- Check ACL tags are properly configured
- Ensure Tailscale account has sufficient permissions

**Windows-Specific Issues**
- PowerShell execution policy may need adjustment
- Windows Defender might block GitLab Runner executable
- Event log monitoring may fail if GitLab Runner service doesn't start

### Debugging

Enable debug logging by adding this step to any workflow:

```yaml
- name: Debug Information
  run: |
    echo "Runner ID: ${{ github.run_id }}"
    echo "GitLab URL: ${{ inputs.gitlab_url }}"
    # Add other debug information as needed
```

## Customization

### Extending Workflows

You can customize these workflows by:

1. **Adding Pre/Post Steps**: Insert additional setup or cleanup steps
2. **Modifying Docker Images**: Use different base images for specific requirements
3. **Adjusting Tailscale Settings**: Modify network configuration as needed
4. **Adding Notifications**: Include Slack, Teams, or email notifications

### Example Customization

```yaml
- name: Custom Setup
  run: |
    # Install additional tools
    # Configure environment variables
    # Set up application dependencies
```

## Security Considerations

1. **Token Rotation**: Regularly rotate GitLab runner tokens
2. **ACL Configuration**: Use restrictive Tailscale ACL rules
3. **Input Validation**: Validate all user inputs
4. **Network Isolation**: Consider using dedicated Tailscale tailnets
5. **Audit Logging**: Monitor runner usage and access patterns
