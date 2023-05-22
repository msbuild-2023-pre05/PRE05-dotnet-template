# Lab 4 - GitHub Actions Basics

In this lab you'll get to know the basics of working with GitHub Actions.

## Exercise 1 - Adding a Workflow file

In this first exercise, you'll create a simple workflow using GitHub Actions.

1.	Using a web browser, access your lab repo that you have been using today.

1.	Click the **Actions** link on the toolbar.

    > You’ll see several popular workflows.

1. While you can start from scratch, click the **Configure** button for **Simple workflow**.

    > If you don't see it at the top, scroll/search for it.

1. Change the name from **blank.yml** to **simple.yml**.

    > Inside the editor you'll see lots starter YML to get you going. 

1. Now, inside the editor, change **line 3** from `name: CI` to `name: Simple`.

1. If you look at the block of code from line 5 to line 14, you'll see three different ways for the system to start executing the workflow (see below). You're going to comment out lines **8** to **11**. 

    ``` YML  
    # Controls when the workflow will run
    on:
    # Triggers the workflow on push or pull request events but only for the "main" branch
    push:
        branches: [ "main" ]
    pull_request:
        branches: [ "main" ]

    # Allows you to run this workflow manually from the Actions tab
    workflow_dispatch:
    ```

    > By doing this you'll be able to iterate on the workflow file and only run workflows when you're ready. Once it's good, you can uncomment / adjust the other items.

1. Now, take a moment to read the comments (they being with a `#` mark) and 'code' that explain what the workflow file will do when run.

1. When ready, click the **Commit changes ...** button.

1. Fell free to add an extended description. For simplicity, you'll commit directly to Main so just click the **Commit changes** button when ready.

1.	Click the **Actions** link on the toolbar.

1. In the left bar, click **Simple** under *All workflows*.

1. Notice how a bar appears stating "This workflow has a workflow_dispatch event trigger." with a button to the far right.

1. Click the **Run workflow** button to see what options exist. With the *main* branch selected, click the **Run workflow** button.

    > After a couple of seconds, your workflow will show up in the list.

1. Pick you workfllow to watch it run by clicking the title hyperlink.

1. Examine the workflow run screen until it completes. Look at the build log? Do you see all the messages? Do you notice anything interesting?

1. Return to the **Actions** home page. 

1. Notice the fiter options above the **Run workflow** button. This makes it easy to find a specfic run by filtering by *Event*, *Status*, *Branch*, and/or *Actor*. Give it a try.

1.  When ready, move on to the next exercise.
   

## Exercise 2 - More stuff 

In this exercise, you'll modify the workflow to use jobs and some other fun stuff.

1. Next  

2. More

3. More

4. More

5.  More

6.  More


