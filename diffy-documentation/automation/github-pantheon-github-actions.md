# GitHub Pantheon GitHub Actions

Pantheon Multidev allows you to spin environments on demand for your branches. So we could use those for creating environments per pull request and test them visually with Diffy.&#x20;

{% hint style="info" %}
Please bear in mind that Pantheon limits you with the number of environments you can create
{% endhint %}

Pantheon has its own repository [example-drops-8-composer](https://github.com/pantheon-systems/example-drops-8-composer) with the example setup of the CI to build the environments. In our example, we start from scratch with a clean Pantheon's project.

The basic workflow is:

1. Pull request created in GitHub
2. Branch copied to Pantheon git repo
3. Trigger creation of the multidev environment for the new branch
4. Run all post-deployment jobs (clear caches, import configuration, updatedb)
5. Trigger visual regression testing
6. Post results back to GitHub as a check

Our repo with all the configuration set is [https://github.com/DiffyWebsite/diffy-multidev/tree/master](https://github.com/DiffyWebsite/diffy-multidev/tree/master)

There is a [great article by Tyler Fahey](https://medium.com/swlh/pantheon-and-github-actions-automated-deployments-via-github-actions-c245aa954797) going deep into setting up similar configuration.

### GitHub Actions secrets

DIFFY\_API\_KEY to be generated from [https://app.diffy.website/#/keys](https://app.diffy.website/#/keys)

DIFFY\_PROJECT\_ID it is your project id. You can find it under project settings, General tab

PANTHEON\_MACHINE\_TOKEN to be generated in your Pantheon account under [Personal Settings / Machine Tokens](https://dashboard.pantheon.io/personal-settings/machine-tokens)

KNOWN\_HOSTS an empty variable required by `shimataro/ssh-key-action@v2` step to populate ssh key to container

PANTHEON\_REPO URL to Pantheon's git repo (example `ssh://codeserver.dev.51b73-XXX-b1f@codeserver.dev.51b-XXX-37cefb1f.drush.in:2222/~/repository.git`)

PANTHEON\_SITE\_NAME machine name of your Pantheon's site. You can find in the list of all sites under your account.

PANTHEON\_SSH\_KEY private ssh key to connect to Pantheon so we can push the code to Pantheon's git repo. See section below.

SSH\_CONFIG value is below. Used for connecting to Pantheon's git repo.

```
Host *.drush.in
  StrictHostKeyChecking no
```



### PANTHEON\_SSH\_KEY key for git push

You can generate a key pair with

```
ssh-keygen -m PEM -t rsa -b 4096 -f /tmp/new_key_for_ci -N ''
```

Copy the public key (`/tmp/new_key_for_ci.pub`) and add it to your [Pantheon's SSH keys](https://dashboard.pantheon.io/personal-settings/ssh-keys).&#x20;

Copy the private key (`/tmp/new_key_for_ci`) and add it as PANTHEON\_SSH\_KEY secret in GitHub Actions.



### Diffy configuration

In order for Diffy to post results back to GitHub as a check you need to add github repo to Diffy's Project settings under Notifications / GitHub and authenticate Diffy's GitHub check by visiting [https://github.com/apps/diffy-testing](https://github.com/apps/diffy-testing)



### Full code

Pay attention that we check if we didn't reach the limit of number of multidev environments.

```
name: Test PRs MultiDev and Diffy

on:
  pull_request:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.PANTHEON_SSH_KEY }}
          config: ${{ secrets.SSH_CONFIG }}
          known_hosts: ${{ secrets.KNOWN_HOSTS }}
      - name: Install Terminus
        uses: pantheon-systems/terminus-github-actions@main
        with:
          pantheon-machine-token: ${{ secrets.PANTHEON_MACHINE_TOKEN }}
      - name: Deploy to MultiDev
        env:
          pantheon_repo: '${{ secrets.PANTHEON_REPO }}'
          pantheon_site_name: '${{ secrets.PANTHEON_SITE_NAME }}'
          pr_number: ${{ github.event.number }}
        run: |
          PR_BRANCH="pr-$pr_number"
          git remote add pantheon $pantheon_repo
          git fetch pantheon
          git push --force pantheon HEAD:refs/heads/$PR_BRANCH
          
          # Check if current branch environment exists. If so -- delete it first.
          MULTIDEV_EXISTS=$(terminus env:list "${pantheon_site_name}" --format=list | grep "${PR_BRANCH}" | wc -l)
          if (( "${MULTIDEV_EXISTS}" > 0 )); then
            terminus multidev:delete "${pantheon_site_name}"."${PR_BRANCH}" --yes
          else
            # Check if we allowed to create new multidev and if not -- delete old ones.
            # See https://github.com/pantheon-systems/pantheon_advanced_page_cache/blob/2.x/.circleci/scripts/ensure-available-multidev.sh
            MAX_CDE_COUNT="$(terminus site:info "${pantheon_site_name}" --field='Max Multidevs')"
            echo "Max Multidev Count: ${MAX_CDE_COUNT}"
          
            DOMAINS="$(terminus env:list "${pantheon_site_name}" --format=string --fields=ID,Created)"

            # Filter out dev, test, live as they don't count agains the Max Multidev count.
            CDE_DOMAINS=$(echo "$DOMAINS" | grep -vE '\b(dev|test|live)\b')
          
            # Count current environments
            CDE_COUNT="$(echo "$CDE_DOMAINS" | wc -l)"
            # remove whitespace to make the arithmetic work
            CDE_COUNT="${CDE_COUNT//[[:blank:]]/}"
          
            NUMBER_OF_CDES_REQUIRED=1
            echo "There are currently ${CDE_COUNT}/${MAX_CDE_COUNT} multidevs. I need ${NUMBER_OF_CDES_REQUIRED}."
  
            POTENTIAL_CDE_COUNT=$((CDE_COUNT + NUMBER_OF_CDES_REQUIRED))
            if (( "${POTENTIAL_CDE_COUNT}" <= "${MAX_CDE_COUNT}" )); then
              echo "There are enough multidevs."
            else
              NUMBER_OF_CDES_TO_DELETE=$((POTENTIAL_CDE_COUNT - MAX_CDE_COUNT))
              echo "There are not enough multidevs, deleting the oldest ${NUMBER_OF_CDES_TO_DELETE} environment(s)."
            
              # Filter out any multidev environments that should never be deleted.
              # This is seperate from filtering out dev/test/live above as they still count towards 'Max Multidev'.
              # Then, sort the list by the timestamps
              SORTED_DOMAINS=$(echo "$CDE_DOMAINS" | grep -vE '\b(drupal10)\b' | sort -n -k2)
            
              # Delete as many multidevs as we need to make room for testing.
              for (( i = 1; i<=NUMBER_OF_CDES_TO_DELETE; i++ )); do
                ENV_TO_REMOVE="$(echo "$SORTED_DOMAINS" | head -n "$i" | tail -n 1 | cut -f1)"
                echo "Removing '${ENV_TO_REMOVE}'."
                terminus multidev:delete --delete-branch "${pantheon_site_name}.${ENV_TO_REMOVE}" --yes
              done
            fi
          fi
          echo "Creating multidev ${PR_BRANCH}"
          terminus multidev:create $pantheon_site_name.live $PR_BRANCH

      - name: Cache rebuild, import configuration
        env:
          pantheon_site_name: '${{ secrets.PANTHEON_SITE_NAME }}'
          pr_number: ${{ github.event.number }}
        run: |
          PR_BRANCH="pr-$pr_number"
          terminus remote:drush $pantheon_site_name.$PR_BRANCH cr
          terminus remote:drush $pantheon_site_name.$PR_BRANCH config:import -y
      - name: Trigger Diffy testing
        env:
          diffy_api_key: '${{ secrets.DIFFY_API_KEY }}'
          diffy_project_id: '${{ secrets.DIFFY_PROJECT_ID }}'
          pantheon_site_name: '${{ secrets.PANTHEON_SITE_NAME }}'
          pr_number: ${{ github.event.number }}
        run: |
          PR_BRANCH="pr-$pr_number"
          MULTIDEV_SITE_URL="https://$PR_BRANCH-$pantheon_site_name.pantheonsite.io/"
          # Download Diffy-CLI.
          wget https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar

          # Authenticate.
          php diffy.phar auth:login $diffy_api_key

          echo ${{ github.event.pull_request.head.sha }}

          # Compare with commit sha so Diffy's github check posts the results. 
          php diffy.phar project:compare $diffy_project_id dev custom --env2Url="${MULTIDEV_SITE_URL}" --commit-sha="${{ github.event.pull_request.head.sha }}"
```
