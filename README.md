# Deploy to Hostinger VPS GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Deploy%20to%20Hostinger%20VPS-purple)](https://github.com/marketplace/actions/deploy-on-hostinger-vps)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Deploy your Docker applications to Hostinger VPS with ease using GitHub Actions.

## Features

- 🚀 **Easy Deployment**: Deploy Docker applications with a simple workflow configuration
- 🔒 **Private Repository Support**: Works with both public and private GitHub repositories
- 🔧 **Environment Variables**: Pass environment variables to your Docker containers

## Prerequisites

- A Hostinger VPS with Docker installed
- Hostinger API key (available in your Hostinger dashboard)
- A `docker-compose.yml` file in your repository
- For private repositories: GitHub Personal Access Token with `repo` scope

## Usage

### Basic Usage (Public Repository)

```yaml
name: Deploy to Hostinger

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Deploy to Hostinger
        uses: hostinger/deploy-on-vps@v1
        with:
          api-key: ${{ secrets.HOSTINGER_API_KEY }}
          virtual-machine: ${{ vars.HOSTINGER_VM_ID }}
```

### Advanced Usage (Private Repository with Environment Variables)

```yaml
name: Deploy to Hostinger

on:
  push:
    branches: [ main, production ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Deploy to Hostinger
        uses: hostinger/deploy-on-vps@v1
        with:
          api-key: ${{ secrets.HOSTINGER_API_KEY }}
          virtual-machine: ${{ vars.HOSTINGER_VM_ID }}
          project-name: my-awesome-app
          personal-token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          docker-compose-path: docker/docker-compose.yml
          environment-variables: |
            NODE_ENV=production
            API_URL=https://api.example.com
            DATABASE_URL=${{ secrets.DATABASE_URL }}
```

## Inputs

| Input                   | Description                                                                | Required | Default                    |
|-------------------------|----------------------------------------------------------------------------|----------|----------------------------|
| `api-key`               | Hostinger API key for authentication                                       | Yes      | -                          |
| `virtual-machine`       | Virtual machine ID where the application will be deployed                  | Yes      | -                          |
| `project-name`          | Name of the project for identification                                     | No       | `${{ github.repository }}` |
| `environment-variables` | Environment variables (KEY=value format, newline separated)                | No       | -                          |
| `personal-token`        | GitHub Personal Access Token (required for private repositories)           | No       | -                          |
| `docker-compose-path`   | Path to docker-compose.yml file (will be detected automatically otherwise) | No       | -                          |

## Setting up Secrets

1. **Get your Hostinger API Key:**
    - Log in to your [Hostinger account](https://hpanel.hostinger.com/)
    - Navigate to API [Settings](https://hpanel.hostinger.com/profile/api)
    - Generate or copy your API key

2. **Get your Virtual Machine ID:**
    - Go to your VPS overview dashboard
    - Find the VM ID:
      - If your VM has default hostname, eg.: srv123456.hstgr.cloud, then the VM ID is 123456
      - If your VM has custom hostname, you can find the ID from the browser URL: https://hpanel.hostinger.com/vps/123456/overview

3. **Add secrets to GitHub:**
    - Go to your repository → Settings → Secrets and variables → Actions
    - Add the following secrets:
        - `HOSTINGER_API_KEY`: Your Hostinger API key
        - `PERSONAL_ACCESS_TOKEN`: (Optional) For private repositories only
    - Add the following variables:
      - `HOSTINGER_VM_ID`: Your Hostinger VM ID

## Environment Variables

You can pass environment variables in multiple formats:

```yaml
environment-variables: |
  NODE_ENV=production
  PORT=3000
  DEBUG=false
  API_KEY=${{ secrets.API_KEY }}
```

## Docker Compose File

Your repository should contain a `docker-compose.y(a)ml` / `compose.y(a)ml` file. Example:

```yaml
services:
  web:
    build: .
    ports:
      - "80:3000"
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - PORT=${PORT:-3000}
    restart: unless-stopped
```

## Examples

### Deploy on Tag Creation

```yaml
name: Deploy on Tag

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Deploy to Hostinger
        uses: hostinger/deploy-on-vps@v1
        with:
          api-key: ${{ secrets.HOSTINGER_API_KEY }}
          virtual-machine: ${{ vars.HOSTINGER_VM_ID }}
          project-name: my-app-${{ github.ref_name }}
```

### Multi-Environment Deployment

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [ main, staging ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Set environment
        id: env
        run: |
          if [ "${{ github.ref }}" == "refs/heads/main" ]; then
            echo "vm_id=${{ vars.PROD_VM_ID }}" >> $GITHUB_OUTPUT
            echo "env_vars=NODE_ENV=production" >> $GITHUB_OUTPUT
          else
            echo "vm_id=${{ vars.STAGING_VM_ID }}" >> $GITHUB_OUTPUT
            echo "env_vars=NODE_ENV=staging" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy to Hostinger
        uses: hostinger/deploy-on-vps@v1
        with:
          api-key: ${{ secrets.HOSTINGER_API_KEY }}
          virtual-machine: ${{ steps.env.outputs.vm_id }}
          environment-variables: ${{ steps.env.outputs.env_vars }}
```

## Support

For issues and feature requests, please [create an issue](https://github.com/hostinger/deploy-to-vps/issues) on GitHub.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
