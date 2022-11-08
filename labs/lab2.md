# Codespaces Lab 2 - CI/CD with Actions

In this lab, we will be using GitHub Actions to build and deploy our application to Azure.

## Exercise 1 - Creating your first Actions workflow to build the code

In this first section, you'll create a build using GitHub Actions.

1. Inside of the repo you have been using for the lab, click on the the `Actions` link in the toolbar.
2. You’ll see several popular workflow starters for us to start from. Actions is suggesting that we use the `.NET` workflow since we have .NET code.
3. Under the `.NET` starter template, click the `Configure` button.
4. In the Actions editor, change the default `.yml` filename from `dotnet.yml` to `readingtime6.yml`.
    <details>

    > **Note**: The filename is important because it will be used to identify the workflow in the `.github/workflows` folder in your repository. You can call the file whatever you want, but all GitHub Actions workflows must live in the `.github/workflows` directory
    </details>
5. Take a look at your action workflow. You can see by default, the `on:` keyword will run the workflow for any push to `main` and pull request to `main` (lines `6-10`).
    <details>

    > **Note**: You can see other events that trigger workflows by following the [link](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).
    >
    > A common example of another event type that we might want to add is the [workflow_dispatch](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch) event - this allows us to trigger the workflow manually from the Actions tab.
    </details>
6. On `line 4`, change the name to: `name: readingtime6 ci/cd`. This will be the name of the workflow that will show up in the Actions tab.
7. Now click the `Start commit` button.
8. Add a useful commit message, such as `initial commit of workflow`.
9. Click `Commit new file`.
10. Click the `Actions` link in the toolbar. You *might* need to refresh your browser before you see your first workflow. When you see it, click on the `workflow run name` (your commit message) to inspect the run.
11. You’ll notice the workflow failed. Click the on the red `build` job name to inspect.
12. Because there was a failure, the failed step should already be expanded. It is saying that it can't find the solution file during the `dotnet restore` step.
13. Let's fix this. In the left-hand side under `Run details`, click `View workflow file`.
14. Click on the `pencil` icon ![image](https://user-images.githubusercontent.com/19912012/200077089-ac040a00-ef78-4088-a30d-0f93903727fa.png) in the upper right-hand corner to edit the file.
15. Change `dotnet restore` line (`line 24`) (remember it's YAML; don’t mess up the whitespace!):
    ```yml
    run: dotnet restore ./src/ReadingTime6.sln
    ```
16. Change the `dotnet build` line (`line 26`) (remember it's YAML; don’t mess up the whitespace!):
    ```yml
    run: dotnet build ./src/ReadingTime6.sln --configuration Release --no-restore
    ```
17. Change the `dotnet test` line (`line 28`) as follows (have you guessed? Don't mess up the whitespace!)"

    ``` yml
    run: dotnet test ./src/ReadingTime6.Web.Tests.Unit/ReadingTime6.Web.Tests.Unit.csproj --configuration Release --no-build --verbosity normal --logger "junit;LogFilePath=../../results/tests.xml"
    ```

    > **Note**: that the above test line is ONE LONG LINE

18. Click the `Preview changes` button to see the diffs of what we have just changed.
19. Now click the `Start commit` button.
20. Add a message like: `Update with correct file paths`
21. Click `Commit changes`.
22. Click the `Actions` link on the toolbar.
23. A new job should appear. Click into it and watch it run. Once it’s done, you should have a green check and a completed workflow. View the log output if you're interested.

Excellent! You just created a CI workflow with GitHub Actions! :partying_face: :robot:

> **What have you learned?**
> - Using a [starter workflow](https://docs.github.com/en/actions/using-workflows/using-starter-workflows) to create a CI job in [GitHub Actions](https://docs.github.com/en/actions/quickstart)
> - How to [monitor and troubleshoot](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/about-monitoring-and-troubleshooting) a failed workflow run
> - How to edit a [workflow](https://docs.github.com/en/actions/using-workflows/about-workflows)

## Exercise 2 - Enhancing our CI job

You'll update your build workflow to make the test results appear a little nicer. Also, we need to to publish the build output so you can deploy your project to Azure.

1. Click the `Actions` link on the toolbar. On the left-hand side, click on the workflow name (ie: `readingtime6 ci/cd`) and then click on the blue file name (ie: `readingtime6.yml`).
2. Click the `pencil` button in the upper-right hand of the page to edit the file.
3. Add the following to the end of the file, under the tests step. Note that the `- name` section needs to be aligned with the previous `- name` above it:
    ```yml
        - name: Create test summary
          uses: test-summary/action@dist
          if: always()
          with:
            paths: results/*.xml
    
    ```

    > **Note**: It is easier if you place your cursor on a new line without any tabs or spaces and then paste the above code. It will automatically indent it for you.
4. Add the following after the `test summary` to run the `dotnet publish` command. We need the build output to deploy to Azure. Make sure the lines are aligned at the correct indentation level relative to their parent:
    ```yml
        - name: Publish with dotnet
          run: dotnet publish ./src/ReadingTime6.Web/ReadingTime6.Web.csproj --configuration Release --output ./output
    
    ```

    > This command creates a web deploy package that Azure App Service action uses to deploy the website.
5. Add the following after the `publish with dotnet` step to upload the build output to the workflow so our deployment job can use:
    ```yml
        - name: Upload build artifact
          uses: actions/upload-artifact@v3
          with:
            name: output
            path: './output'
    
    ```
6. Click the `Start commit` button. Add a message like: `Add test summary and publishing build output`.
7. Click `Commit changes` to commit to the main branch.
8. Click the `Actions` link on the toolbar.
9. Click on your new workflow run. Watch it run. Open the build log. Specifically examine the `Publish with dotnet` and `Upload build artifacts` step.
10. Once the job finishes, click the `⌂ Summary` link on the left-hand side of the page.
11. Scroll to the bottom of the summary page to view your 1) `artifacts` and 2) `test results`.

Pretty cool! We have everything we need to now deploy this to Azure. :rocket: :cloud:

> **What have you learned?**
> - How to use [job summaries](https://github.blog/2022-05-09-supercharging-github-actions-with-job-summaries/)
> - Using [workflow artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)

## Exercise 3: Infrastructure as Code Configuration using Azure Resource Manager (ARM)

To deploy your web app, you need some infrastructure. You’re going to add an [Azure Resource Manager (ARM) template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) which describes what you want to create in Azure.

> You're going to describe what you need from Azure using a JSON-based configuration file known as an Azure Resource Manager template or just *ARM template*. You can then have your workflow push the ARM template to Azure,  where the Azure fabric will create the infrastructure you need.

1. Click the `Code` tab on the repo toolbar.
2. Drill down to find the file `infra/azure.json`. The file is creating a single web app running on an Azure App Service *B1* configuration. Read more [here](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)):
3. There's nothing to change here, but just inspect the template to see what it's doing. Defining infrastructure as code is an industry best practice. Other options for defining infrastructure as code include [Terraform](https://www.terraform.io/), [CloudFormation](https://aws.amazon.com/cloudformation/), and [Pulumi](https://www.pulumi.com/).

> **What have you learned?**
> - An example of infrastructure as code
> - What an [ARM template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) is

## Exercise 4: Update Workflow to Deploy Infrastructure and App

Now you will update the workflow to take the output of your build and deploy your web app to Azure App Service.

1. Click the `Actions` link on the toolbar. On the left-hand side, click on the workflow name (ie: `readingtime6 ci/cd`) and then click on the blue file name (ie: `readingtime6.yml`).
2. At the end of the file, add a new `deploy-staging` job that depends on the `build` job:
    ```yml
      deploy-staging:
        runs-on: ubuntu-latest
        permissions:
          id-token: write
          contents: read
        needs: build

    ```
    > **Note**: It is easier if you place your cursor on a new line without any tabs or spaces and then paste the above code. It will automatically indent it for you.

    > **Note**: The `permissions` block is used for OIDC (aka, password-less deployments to the cloud!)
3. Next, we want to add an `environment` so that we can later define approvals, as well as more easily see what build is in what environment from the repo home page. 
    ```yml
        environment: 
          name: staging
          url: ${{ steps.deploy.outputs.webapp-url }}

    ```
4. Under `needs:`, add in the environment variables we need to deploy our app to Azure:
    ```yml
        env:
          AZURE_GROUP_NAME: rg-${{ github.event.repository.name }}-${{ github.event.repository.id }}
          AZURE_WEBAPP_NAME: ${{ github.event.repository.name }}-${{ github.event.repository.id }}
          AZURE_GROUP_NAME: rg-${{ github.event.repository.name }}-${{ github.event.repository.id }}
          AZURE_LOCATION: westus3
          AZURE_ENVIRONMENT: staging
    
    ```
    > **Note**: The `env` keyword needs to line up with the `runs-on`, `permissions`, `needs`, and `environment`.

5. Afterwards, add in the `steps:` keyword and the `actions/checkout` action (we need to check out the IaC code, but typically in a deployment you wouldn't do this):
    ```yml
        steps:
        - uses: actions/checkout@v3
    
    ```
6. After the checkout step, add in the action to download the build output:
    ```yml
        - uses: actions/download-artifact@v3
          with:
            name: output
            path: output
    
    ```
7. After the checkout step, add in the action to log into [Azure using OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure) (password-less deploy!):
    ```yml
        - name: Login to Azure with OIDC
          uses: azure/login@v1
          with:
            client-id: 'REPLACE-ME'
            tenant-id: 'REPLACE-ME'
            subscription-id: 'REPLACE-ME'
    
    ```
8. Replace the `REPLACE-ME` values with the following with the values in your **[issues](../../../issues/)**.
    > **Note**: You can get the OIDC login values (it's already-pre formatted so you can cut and paste) from your **[issues](../../../issues/)** in your lab repo. The title of the issue is `Data for your labs`, the issue includes a snippet you can just copy & paste (make sure it's indented though).
    > 
    > You can select the lines you pasted and use `tab` to indent them all together.
9. Next, add following step to provision the infrastructure with ARM:
    ```yml
        - name: Deploy infra
          run: |
            az deployment group create -n ghaction -g $AZURE_GROUP_NAME --template-file ./infra/azure.json --parameters webAppName=${{ env.AZURE_WEBAPP_NAME }} environment=${{ env.AZURE_ENVIRONMENT }} region=${{ env.AZURE_LOCATION }}
    
    ```
10. Finally, add in the action to deploy the web app, and a `run` command to write the url to the job summary:
    ```yml
        - name: 'Azure webapp deploy'
          id: deploy
          uses: azure/webapps-deploy@v2
          with:
            app-name: ghu22-${{ env.AZURE_WEBAPP_NAME }}-${{ env.AZURE_ENVIRONMENT }}
            package: './output' 

        - name: Write url to output summary
          id: write-url
          run: |
            echo '::notice title=Deployed App Url::${{ steps.deploy.outputs.webapp-url }}'
            echo '## :rocket: Your site is available [here](${{ steps.deploy.outputs.webapp-url }})' >> $GITHUB_STEP_SUMMARY
    
    ```
    > **Note**: We add an `id` to the `Azure webapp deploy` action so that we can use its output variable (`${{ steps.deploy.outputs.webapp-url }}`)

11. We want to `workflow_dispatch`: trigger under the `on:` section at the top of the workflow to ensure that we can [run the workflow manually](https://github.blog/changelog/2020-07-06-github-actions-manual-triggers-with-workflow_dispatch/) - lines 6-11 should look like this:
    ```yml
    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]
      workflow_dispatch:
    ```
12. If we lost you anywhere, you can take a look at the reference workflow:
    <details>

    ```yml
    # This workflow will build a .NET project
    # For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-net

    name: readingtime6 ci/cd

    on:
    push:
      branches: [ "main" ]
    pull_request:
      branches: [ "main" ]
    workflow_dispatch:

    permissions: 
      contents: read
      id-token: write

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - name: Setup .NET
            uses: actions/setup-dotnet@v3
            with:
              dotnet-version: 6.0.x
          - name: Restore dependencies
            run: dotnet restore ./src/ReadingTime6.sln
          - name: Build
            run: dotnet build ./src/ReadingTime6.sln --configuration Release --no-restore
          - name: Test
            run: dotnet test ./src/ReadingTime6.Web.Tests.Unit/ReadingTime6.Web.Tests.Unit.csproj --configuration Release --no-build --verbosity normal --logger "junit;LogFilePath=../../results/tests.xml"
          - name: Create test summary
            uses: test-summary/action@dist
            with:
              paths: results/*.xml
              if: always()
          - name: Publish with dotnet
            run: dotnet publish ./src/ReadingTime6.Web/ReadingTime6.Web.csproj --configuration Release --output ./output
          - name: Upload build artifact
            uses: actions/upload-artifact@v3
            with:
              name: output
              path: './output'
      deploy-staging:
        runs-on: ubuntu-latest
        needs: build
        environment: 
          name: staging
          url: ${{ steps.deploy.outputs.webapp-url }}
        env:
          AZURE_WEBAPP_NAME: ${{ github.event.repository.name }}-${{ github.event.repository.id }}
          AZURE_GROUP_NAME: rg-${{ github.event.repository.name }}-${{ github.event.repository.id }}
          AZURE_LOCATION: westus3
          AZURE_ENVIRONMENT: staging

        steps:
        - uses: actions/checkout@v3
        - uses: actions/download-artifact@v3
          with:
            name: output
            path: path
        - name: Login to Azure with OIDC
          uses: azure/login@v1
          with:
            client-id: '97671274-5105-4a1f-8062-25290d58f4e0'
            tenant-id: 'a6b90ce0-6fdc-4473-8a9a-673af0e1cdf7'
            subscription-id: '08a3cf8a-24d4-4e5b-9a27-c171509b1988'

        - name: Deploy web app
          run: |
            AZURE_GROUP_NAME=$(az group list | jq -r '.[].name')
            echo $AZURE_GROUP_NAME
            az deployment group create -n ghaction -g $AZURE_GROUP_NAME --template-file ./infra/azure.json --parameters webAppName=${{ env.AZURE_WEBAPP_NAME }} environment=${{ env.AZURE_ENVIRONMENT }} region=${{ env.AZURE_LOCATION }}
        - name: 'Azure webapp deploy'
          id: deploy
          uses: azure/webapps-deploy@v2
          with:
            app-name: ghu22-${{ env.AZURE_WEBAPP_NAME }}-${{ env.AZURE_ENVIRONMENT }}
            package: './output'
        - name: Write url to output summary
          run: |
            echo '::notice title=Deployed App Url::${{ steps.deploy.outputs.webapp-url }}'
            echo '## :rocket: Your site is available [here](${{ steps.deploy.outputs.webapp-url }})' >> $GITHUB_STEP_SUMMARY
    ```
    </details>
13. Now click the `Start commit` button. Add a message like: `Add support for deploying web app to Azure`
14. Click `Commit changes` to commit to the `main` branch.
15. Click the `Actions` link on the toolbar.
16. On the left-hand side, click on the workflow name (ie: `readingtime6 ci/cd`).
17. Click the most recent job run. You should see a `build` and `deploy-staging` job.
18. Inspect the logs as the job runs by clicking into a job that's in progress.
19. Once both jobs complete, Click the `⌂ Summary` link on the left-hand side of the page.
20. At the bottom of the page under `deploy-staging summary`, click the link to navigate to your website. 
21. Verify the website is running and is showing our site!

Huzzah! :sparkles: Access the site and admire your work! :octocat:

> **What have you learned?**
> - Deploying a [web app to Azure](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-azure/deploying-net-to-azure-app-service) using GitHub Actions

## Next Steps

- [Lab 3 - Optional - Prebuilds](./lab3-optional-prebuilds.md)

## Resources

