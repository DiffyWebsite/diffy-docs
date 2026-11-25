# Astro GitHub Cloudflare

You can automate visual testing for your branch-based environments that were spun up by Cloudflare. So Diffy will compare those environments vs production and post results back to your pull request as a check.

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

For that, you need to:

* Create a project in Diffy, set up pages, breakpoints etc.
* in Diffy, under Project Settings > Notifications > GitHub add a link to the repo (i.e. `https://github.com/{username}/{repo}`) and install the Diffy GitHub app to the repo.
* generate an API key (`https://app.diffywebsite/#/keys`)
* populate the key and project ID in GitHub secret and variable&#x20;
* add the following GitHub workflow (`.github/workflows/visual-regression.yml`)

Here is a [video walkthrough](https://www.loom.com/share/4f989d3b4a2146d4ba6a31c6e72a70f8).

```
name: Visual Regression

on:
  check_run:
    types: [completed]

jobs:
  compare:
    name: Run Diffy Comparison
    # Only fire on successful Cloudflare Workers and Pages check runs.
    if: >-
      github.event.check_run.check_suite.conclusion == 'success' &&
      github.event.check_run.app.name == 'Cloudflare Workers and Pages'
    runs-on: ubuntu-latest
    timeout-minutes: 15
    permissions:
      contents: read

    steps:
      # Extract the preview URL from the check run summary and resolve the PR number.
      # Both are done here to avoid multiple API round-trips.
      - name: Get deployment URL and PR number
        id: info
        uses: actions/github-script@v9
        with:
          script: |
            const summary = context.payload.check_run.output.summary || '';
            const urlMatch = summary.match(/https:\/\/[a-z0-9-\.]+\.workers\.dev*/);
            if (!urlMatch) {
              core.setFailed('Could not find a *.workers.dev URL in the check run summary');
              return;
            }
            core.setOutput('deployment_url', urlMatch[0]);

            // console.log(JSON.stringify(context.payload.check_run.check_suite, null, 2));
            const branchName = context.payload.check_run.check_suite.head_branch || ''
            if (!branchName) {
              core.setFailed('Could not find a head branch name');
              return;
            }
            core.setOutput('branch_name', branchName);

      - name: Download Diffy CLI and trigger comparison job
        run: |
          wget -q -O /usr/local/bin/diffy \
            https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar
          chmod +x /usr/local/bin/diffy

          diffy auth:login "${DIFFY_API_KEY}"

          diffy project:compare "${DIFFY_PROJECT_ID}" prod custom \
            --env2Url="${{ steps.info.outputs.deployment_url }}" \
            --commit-sha="${{ github.event.check_run.head_sha }}" \
            --name="PR ${{ steps.info.outputs.branch_name }}"
        env:
          DIFFY_API_KEY: ${{ secrets.DIFFY_API_KEY }}
          DIFFY_PROJECT_ID: ${{ vars.DIFFY_PROJECT_ID }}

```

{% embed url="https://www.loom.com/share/4f989d3b4a2146d4ba6a31c6e72a70f8" %}
