# Playwright integration

When you run your playwright tests, you can create a directory with all the screenshots. It is possible to upload those screenshots to Diffy so you can compare them.\
\
This resolves a few issues:

* No need to worry about where to store the screenshots if you run them in CI
* You have a better UI for comparing screenshots
* You keep the history



Here is how it can be done.

1.  Download and authenticate `diffy-cli`

    ```
    wget -O /usr/local/bin/diffy https://github.com/diffywebsite/diffy-cli/releases/latest/download/diffy.phar
    chmod a+x /usr/local/bin/diffy
    diffy auth:login $DIFFY_API_KEY
    ```
2. Run your playwright tests to create a set you will compare against
3.  Run a command to upload screenshots to Diffy (save the `id` of the set of screenshots)

    ```
    diffy screenshot:create-folder $PROJECT_ID /path/to/folder/with/screenshots
    ```
4.  Run playwright scripts on your dev environment, upload them to Diffy (same command as above), and compare these sets (you can start comparison via UI as well)

    ```
    diffy diff:create $PROJECT_ID $SCREENSHOT_ID1 $SCREENSHOT_ID2
    ```



{% embed url="https://www.loom.com/share/1e3dbcef9cb34b2d86522dbff756de6d" %}
