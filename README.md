# AWS SDK Go v2 Interface Generator Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-AWS%20Ifaces%20Action-blue?logo=github)](https://github.com/marketplace/actions/aws-sdk-go-v2-interface-generator)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A GitHub Action that auto-generates Go interfaces (and optionally mocks & tests) for [AWS SDK Go v2](https://github.com/aws/aws-sdk-go-v2) services using [nmccready/aws-sdk-go-v2-ifaces](https://github.com/nmccready/aws-sdk-go-v2-ifaces).

## Why?

AWS SDK Go v2 doesn't ship interfaces for service clients, making it hard to mock in tests. This action generates `IClient` interfaces for any (or all) AWS services so you can write testable Go code.

## Quick Start

```yaml
- uses: brickhouse-tech/aws-ifaces-action@v1
  with:
    services: 's3,dynamodb,sqs'
    commit-results: 'true'
```

## Usage

### Generate interfaces for specific services

```yaml
name: Generate AWS Interfaces
on:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: brickhouse-tech/aws-ifaces-action@v1
        with:
          services: 's3,dynamodb,sqs,lambda'
          output-dir: 'pkg/awsifaces/service'
          commit-results: 'true'
          commit-message: 'chore: regenerate AWS SDK interfaces'
```

### Generate all services and create a PR

```yaml
name: Update AWS Interfaces
on:
  schedule:
    - cron: '0 0 1 * *'  # Monthly
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4

      - uses: brickhouse-tech/aws-ifaces-action@v1
        id: generate
        with:
          create-pr: 'true'
          pr-title: 'chore: update AWS SDK Go v2 interfaces'
          generate-mocks: 'true'
```

### Generate and use in the same workflow

```yaml
- uses: brickhouse-tech/aws-ifaces-action@v1
  id: ifaces
  with:
    services: 's3'

- name: Run tests
  if: steps.ifaces.outputs.has-changes == 'true'
  run: go test ./...
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `services` | Comma-separated AWS service names (empty = all) | `''` (all) |
| `generate-mocks` | Also generate mocks via mockery | `'false'` |
| `generate-tests` | Also generate tests | `'false'` |
| `output-dir` | Output directory (relative to workspace) | `'service'` |
| `ifaces-version` | Version/ref of aws-sdk-go-v2-ifaces | `'main'` |
| `commit-results` | Commit generated files to the repo | `'false'` |
| `commit-message` | Commit message | `'chore: update auto-generated AWS SDK Go v2 interfaces'` |
| `create-pr` | Create a PR instead of direct commit | `'false'` |
| `pr-title` | PR title | `'chore: update auto-generated AWS SDK Go v2 interfaces'` |
| `pr-branch` | Branch name for PR | `'auto-update/aws-ifaces'` |
| `go-version` | Go version | `'1.24'` |
| `node-version` | Node.js version | `'20'` |

## Outputs

| Output | Description |
|--------|-------------|
| `generated-services` | Comma-separated list of services generated |
| `has-changes` | `'true'` if files were changed |

## How It Works

1. Clones [nmccready/aws-sdk-go-v2-ifaces](https://github.com/nmccready/aws-sdk-go-v2-ifaces)
2. Clones the latest AWS SDK Go v2 source
3. Extracts public `Client` method signatures from each service
4. Generates `IClient` interfaces in `{service}/{service}_iface/iface.go`
5. Optionally generates mocks (via [mockery](https://github.com/vektra/mockery)) and tests
6. Copies results to your repo and optionally commits or creates a PR

## License

MIT
