# GitHub Pantheon CircleCI

Pantheon Multidev allows you to spin environments on demand for your branches. So we could use those for creating environments per pull request and test them visually with Diffy.&#x20;

{% hint style="info" %}
Please bear in mind that Pantheon limits you with the number of environments you can create
{% endhint %}

Pantheon has its own repository [example-drops-8-composer](https://github.com/pantheon-systems/example-drops-8-composer) with the example setup of the CI to build the environments. In our example, we start from scratch with a clean Pantheon's project.

The basic workflow is:

1. Pull request created in GitHub
2. Branch copied to Pantheon git repo (uses Pantheon's CircleCI orb)
3. Trigger creation of the multidev environment for the new branch (uses Pantheon's CircleCI orb)
4. Run all post-deployment jobs (clear caches, import configuration, updatedb)
5. Trigger visual regression testing
6. Post results back to GitHub as a check

Our repo with all the configuration set is [https://github.com/DiffyWebsite/diffy-multidev/tree/master](https://github.com/DiffyWebsite/diffy-multidev/tree/master)

### CircleCI variables

These can be set up in the settings of your CircleCI project under Environmental Variables.

DIFFY\_API\_KEY to be generated from [https://app.diffy.website/#/keys](https://app.diffy.website/#/keys)

DIFFY\_PROJECT\_ID it is your project id. You can find it under project settings, General tab

TERMINUS\_TOKEN to be generated in your Pantheon account under [Personal Settings / Machine Tokens](https://dashboard.pantheon.io/personal-settings/machine-tokens)

TERMINUS\_SITE machine name of your Pantheon's site. You can find in the list of all sites under your account.



### SSH key for git push

The only last thing you would need to do is to generate an ssh key and set in up in CircleCI and Pantheon so CircleCI can push the code to Pantheon's repo.

Here are steps from [instructions on Pantheon's CircleCI orb](https://github.com/pantheon-systems/circleci-orb):

```
ssh-keygen -m PEM -t rsa -b 4096 -f /tmp/new_key_for_ci -N ''
```

Copy the public key (`/tmp/new_key_for_ci.pub`) and add it to your [Pantheon's SSH keys](https://dashboard.pantheon.io/personal-settings/ssh-keys).&#x20;

Copy the private key (`/tmp/new_key_for_ci`) and add it to your CircleCI Project settings / Additional SSH Keys section. Set the hostname as `drush.in`

### Diffy configuration

In order for Diffy to post results back to GitHub as a check you need to add github repo to Diffy's Project settings under Notifications / GitHub and authenticate Diffy's GitHub check by visiting [https://github.com/apps/diffy-testing](https://github.com/apps/diffy-testing)

### Complete code example

```
version: 2.1
workflows:
  version: 2
  build_and_push:
    jobs:
      - pantheon/push:
          context:
            - drupal
          checkout: false
          terminus_clone_env: live
          clone_content: false
          env_create_max_time: "15m"
          pre-steps:
            - checkout
            - attach_workspace:
                at: .
      - afterbuild:
          requires:
            - pantheon/push
          context:
            - drupal
      - diffy_manual:
          requires:
            - afterbuild
          context:
            - drupal
          filters:
            branches:
              ignore:
                - master

orbs:
  # Pantheon Orb source and README: https://github.com/pantheon-systems/circleci-orb
  # More Pantehon Orb Docs https://circleci.com/developer/orbs/orb/pantheon-systems/pantheon
  pantheon: pantheon-systems/pantheon@0.6.0
jobs:
  afterbuild:
    docker:
      - image: quay.io/pantheon-public/build-tools-ci:6.x
    steps:
      - checkout
      - attach_workspace:
          at: /tmp/workspace

      - run: cp /tmp/workspace/bash_env.txt $BASH_ENV
      - run: source $BASH_ENV
      - run:
          name: Run config import and cache clear on multidev env
          command: |
            terminus -n auth:login --machine-token="$TERMINUS_TOKEN"
            TERMINUS_FULL_SITE="$TERMINUS_SITE.$TERMINUS_ENV"
            echo "$TERMINUS_FULL_SITE"
            terminus remote:drush "$TERMINUS_FULL_SITE" config:import -y
            terminus remote:drush "$TERMINUS_FULL_SITE" cr

  diffy_manual:
    docker:
      - image: cimg/php:8.1.21
    steps:
      - checkout
      - attach_workspace:
          at: /tmp/workspace

      - run: cp /tmp/workspace/bash_env.txt $BASH_ENV
      - run: source $BASH_ENV
      - run:
          name: Run compare job and post a github comment
          command: |
            if [ -z "$DIFFY_API_KEY" ]
            then
              echo "Diffy integration is not configured. Add DIFFY_API_KEY to CircleCI variables."
              exit 1;
            fi
            if [ -z "$DIFFY_PROJECT_ID" ]
            then
              echo "Diffy integration is not configured. Add DIFFY_PROJECT_ID to CircleCI variables."
              exit 1;
            fi

            # Ping the multidev environment to wake it from sleep
            echo -e "\nPinging the ${TERMINUS_ENV} multidev environment to wake it from sleep..."
            curl -I "$MULTIDEV_SITE_URL" >/dev/null
            # Ping the live environment to wake it from sleep
            echo -e "\nPinging the dev environment to wake it from sleep..."
            curl -I "$DEV_SITE_URL" >/dev/null

            # Download Diffy-CLI.
            wget https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar

            # Authenticate.
            php diffy.phar auth:login $DIFFY_API_KEY

            # Compare with commit sha so Diffy's github check posts the results. 
            LAST_GIT_COMMIT_HASH=$(git log -1 --pretty=%H)
            php diffy.phar project:compare $DIFFY_PROJECT_ID dev custom --env2Url="${MULTIDEV_SITE_URL}" --commit-sha="${LAST_GIT_COMMIT_HASH}"
```
