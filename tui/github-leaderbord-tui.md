# GitHub Leaderboard TUI

Build a terminal dashboard that shows a ranked leaderboard of GitHub contributors across multiple repositories. Repositories are grouped by project (for example: web, mobile, infra), and users can switch views in the terminal.

## Project Goal

Create a tool that helps teams quickly see who is contributing across many repos without opening GitHub in a browser.

## Core Features

- Aggregate activity from multiple repositories.
- Track contributions per user.
- Support project groups (a group contains multiple repos).
- Apply a configurable scoring system.
- Auto-refresh data on a fixed interval.
- Show an interactive TUI leaderboard.
- Open a user detail view with contribution breakdown.

## Data Sources

Use GitHub API endpoints for:

- Commits
- Pull requests
- Issues

## High-Level Design

### 1) Data Collector

- Read config (GitHub token, org/user, groups, repos, refresh interval).
- Fetch contribution data from GitHub APIs.
- Cache recent responses to avoid rate-limit issues.

### 2) Aggregator and Scoring Engine

- Normalize records into one contribution model.
- Group by contributor.
- Calculate score using weighted rules (example: PR merged > commit > issue comment).
- Produce sorted leaderboard results by group and global view.

### 3) TUI Application

- Main screen: ranked contributors table.
- Filters: select group, date range, contribution type.
- Detail panel: selected user stats and repo-by-repo breakdown.
- Refresh indicator: show last updated time and loading state.

## Suggested Configuration

```yaml
github:
  token_env: GITHUB_TOKEN
  owner: your-org
  refresh_seconds: 300

groups:
  web:
    - web-app-frontend
    - web-app-backend
    - web-app-services
  mobile:
    - mobile-ios
    - mobile-android

scoring:
  commit: 1
  pull_request_opened: 3
  pull_request_merged: 5
  issue_opened: 2
  issue_commented: 1
```

## AI-Ready Build Tasks

1. Create a CLI entrypoint that loads config and starts the TUI.
2. Implement GitHub API client with token-based auth.
3. Add collectors for commits, pull requests, and issues.
4. Implement scoring and ranking logic.
5. Build TUI views (leaderboard, filters, user details).
6. Add periodic refresh loop and error handling.
7. Add tests for scoring and aggregation logic.

## Definition of Done

- Runs from terminal with one command.
- Reads configuration from YAML.
- Shows a ranked leaderboard for at least 2 groups.
- Supports manual refresh and auto refresh.
- Handles GitHub API errors gracefully.
- Includes basic unit tests for scoring rules.
