# Buildkite Pull Request Commands GitHub Action
 
A [GitHub Action](https://github.com/actions) for triggering a build on a [Buildkite](https://buildkite.com/) pipeline when a pull request comment contains a specific slash command. 

This action uses the [Buildkite Trigger Pipeline Action](https://github.com/buildkite/trigger-pipeline-action/) as a base to send the builds to Buildkite.

## Features
It supports two commands:
- `/bk trigger <pipeline-slug>`
- `/bk pipeline <file.yml>`

## Usage

Create a [Buildkite API Access Token](https://buildkite.com/docs/apis/rest-api#authentication) with `read_builds` & `write_builds` scope, and save it to your GitHub repository’s **Settings → Secrets** as `BUILDKITE_API_TOKEN`. Then you can configure your Actions workflow with your Buildkite `org-slug` and `pipeline-slug`.

Update your [Pipeline YAML Steps](https://buildkite.com/docs/tutorials/pipeline-upgrade#using-yaml-steps-for-new-pipelines) to include the following in the UI steps editor:
```yaml
env: 
  PIPELINE_FILE: "pipeline.yml" # you can change this value to whatever you usually name your pipeline files by default.

steps:
  - label: ":buildkite: :pipeline: Upload"
    command: buildkite-agent pipeline upload .buildkite/${PIPELINE_FILE}
```

Note: The above is required for `/bk pipeline <file.yml>` to work properly.

Create a [Pull Request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) on your repository and add a comment using one of the two commands:

- `/bk trigger <pipeline-slug>` will trigger a build the PR with it's details on a **new** pipeline based on whatever `pipeline-slug` is provided, and is valid in your Organization.
- `/bk pipeline <file.yml>` will trigger a build the PR with it's details on the **existing** pipeline (via the `buildkite_pipeline` input parameter in the workflow)  with a **new** yaml file specified in the command. The yaml file must exist either in the repo, or the PR.

## Configuration Options

### Configuration as Input Parameters

The following workflow listens to your PR comments (either created, or edited) and then creates a new Buildkite build based on the command chosen.

Note: `buildkite_pipeline` is optional, however if you intend to use the `/bk pipeline` slash command, it is required.

```yaml
name: Buildkite Pull Request Commands

on:
  issue_comment:
    types: [created, edited]

permissions:
  contents: read
  pull-requests: read

jobs:
  trigger-pipeline:
    runs-on: ubuntu-latest
    if: github.event.issue.pull_request != null
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Trigger Buildkite Pipeline
      uses: buildkite/trigger-pipeline-commands@v0.1.0
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        buildkite_api_token: ${{ secrets.BUILDKITE_API_TOKEN }}
        buildkite_org: your-org-slug
        buildkite_pipeline: your-pipeline-slug  # optional