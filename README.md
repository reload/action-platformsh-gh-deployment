# Creating a GitHub deployment if PlatformSH has deployed

This action creates a GitHub deployment when PlatformSH has deployed a
commit.

## Usage

```yaml
---
name: "GH Deployment from PSH status"

on:
  push:
    branches: [main]
  status:

permissions:
  pull-requests: read
  deployments: write
  statuses: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-24.04
    if: github.event_name == 'push' || (github.event.context == 'platformsh' && github.event.state != 'pending')
    steps:
      - uses: actions/checkout@v4
      - run: echo "Triggered by ${{ github.event_name }}"

      - uses: reload/action-platformsh-gh-deployment@main
        with:
          PLATFORMSH_ID: ${{ vars.PSH_PROJECT_ID }}
          PLATFORMSH_KEY: ${{ secrets.DAIS_PLATFORMSH_KEY }}
          GH_DEPLOYMENT_TOKEN: ${{ github.token }}
          PSH_ACTIVITY_TYPES: "push,domain.create,domain.update"
          BRANCH_NAME: ${{ github.event_name == 'status' && github.event.branches[0].name || github.ref_name }}
          USE_PULL_REQUESTS: ${{ github.event_name == 'status' && 1 || 0 }}
```

## Inputs

- `PLATFORMSH_KEY`: API key for connecting to Platform.sh.
- `PLATFORMSH_ID`: ID for the Platform.sh project.
- `GH_DEPLOYMENT_TOKEN`: GitHub token, for
  deployment. E.g. github.token.
- `PSH_DEPLOY_STATUS_PATH`: The location of the deploy-status file on
  the website.
- `PSH_ACTIVITY_TYPES`: Comma-seperated types of deployments to wait
  for. Default: push - See types here:
  <https://docs.platform.sh/integrations/activity/reference.html#environment-activity-types>.
- `PSH_DETECTION_WAIT`: How long should we maximum wait for PSH to
  detect the push? If the branch doesnt exist in the PlatformSH GIT
  Remote, this is how long the action will take. Actually inactive
  environments get detected instantly. Default: 30 seconds.
- `BRANCH_NAME`: Branch, which we will use to look.
- `USE_PULL_REQUESTS`: If true, we will look up PSH environments based
  on PRs, rather than just the branch name. Default: 0.

## Outputs

- `url`: The ready-to-connect URL.
- `deployment_id` : The GH Deployment ID.
- `deployment_status_id`: The GH Deployment status ID.
