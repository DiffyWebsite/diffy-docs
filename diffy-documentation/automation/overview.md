---
description: >-
  In this section, we will provide examples of Diffy integrations with your
  Hosting/CI/CD systems
---

# 💡 Overview

You could have two levels of integrations:

## Trigger comparison post-deployment

Example: once you deploy to the Staging environment, the comparison is triggered to check it against Production.

This is when you do not have any CI/CD in place. So you manually push the code to the environment in the UI of your hosting platform (like Pantheon or Acquia) and get deployment started.



## Testing every pull request environment

If you build pull request environments we can set up Diffy to check these environments against staging (dev or production) and post results back to the pull request. This is the most complex but most rewarding type of integration as you will catch bugs as early as possible.



TL;DR (most basic trigger):

```
wget -O /usr/local/bin/diffy https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar
chmod a+x /usr/local/bin/diffy
diffy auth:login $DIFFY_API_KEY
diffy project:compare $DIFFY_PROJECT_ID prod custom --env2Url="${PR_SITE_URL}"
```



Some Diffy best practices on pull request testing:

1. Name your comparisons based on either a pull request or your dev ticket
2. Do not trigger comparisons on every commit but only selectively based on the commit message
3. Reuse screenshots from your baseline environment (production) to save time



Here are combinations of git provider / pull request environments provider and CI

|           | Pantheon Multidev                                                                             | Tugboat                               | Platform.sh                                   |
| --------- | --------------------------------------------------------------------------------------------- | ------------------------------------- | --------------------------------------------- |
| GitHub    | [CircleCI](github-pantheon-circleci.md) / [GitHub Actions](github-pantheon-github-actions.md) | [Tugboat - GitHub](github-tugboat.md) | [Platformsh - GitHub](upsun-github-action.md) |
| GitLab    |                                                                                               |                                       |                                               |
| Bitbucket |                                                                                               |                                       |                                               |



{% hint style="info" %}
If you do not see your setup here -- [let us know](mailto:info@diffy.website) and we are happy to add an example
{% endhint %}

