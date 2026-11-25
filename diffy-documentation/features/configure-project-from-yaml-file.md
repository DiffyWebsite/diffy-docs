# Configure project from YAML file

You can keep your project settings in your repository and, in this way, have revisions of the settings. Another use case is sharing common settings across all your projects to ensure everything is consistent.

You can download YAML file from the Project Settings page.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Then after editing the configuration, you can run the Diffy CLI command

```
diffy project:update 22441 ./diffy-project-22441-diffy-monitoring.yaml
```

In this example, my project ID is 22441.

Here is a loom showing it live [https://www.loom.com/share/3eed8d4ac8c44b2a9d5bd29bbb36e7af](https://www.loom.com/share/3eed8d4ac8c44b2a9d5bd29bbb36e7af)
