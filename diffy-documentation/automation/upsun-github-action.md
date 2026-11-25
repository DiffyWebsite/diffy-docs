# Upsun GitHub Action

Upsun is a hosting platform that is very flexible in spinning up environments from git branches and pull requests.

Diffy can be integrated so that it visually compares each pull request with whatever environment you like. In our example, we compare it with production. You can watch a [video overview](https://www.loom.com/share/d4ecb80672bc4840ab0f785ca1c9b8a3) in the bottom of the page.

Complete steps to set up the integration are:

* Set up Upsun [Github integration](https://developer.upsun.com/docs/integrations/source/github)
* Create a project in Diffy, specify the production environment
* Under Project settings > Notifications > Github add URL of your GitHub repo and enable the Diffy GitHub app
* Create an [API key in Diffy](https://app.diffy.website/#/keys)
* Populate DIFFY\_API\_KEY and DIFFY\_PROJECT\_ID as [secrets in the GitHub repo](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)
* Create a GitHub action with the following code

```
name: "DiffyPullRequestTest"
run-name: "DiffyPullRequestTest"
on:
  pull_request:
    branches:
      - main

jobs:
  trigger-diffy-job:
    runs-on: ubuntu-latest
    steps:
      - name: 'Wait for psh and get target url'
        id: get-target-url
        uses: platformsh/gha-retrieve-psh-prenv-url@main
        with:
            github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: 'Install and Run Diffy'
        id: install-diffy-cli
        shell: bash
        env:
          diffy_api_key: '${{ secrets.DIFFY_API_KEY }}'
          diffy_project_id: '${{ secrets.DIFFY_PROJECT_ID }}'
        run: |
          wget -O /usr/local/bin/diffy https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar
          chmod a+x /usr/local/bin/diffy
          diffy auth:login $diffy_api_key
          diffy project:compare $diffy_project_id prod custom --env2Url="${{ steps.get-target-url.outputs.target_url }}" --commit-sha="${{ github.event.pull_request.head.sha }}"

```

Once all this is done and GitHub action is merged, you create a pull request with visual changes and see how Diffy gets triggered and posts results back to the pull request.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>Diffy GitHub check</p></figcaption></figure>

{% embed url="https://www.loom.com/share/d4ecb80672bc4840ab0f785ca1c9b8a3" %}
