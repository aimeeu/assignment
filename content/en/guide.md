---
title: Procedure
weight: 3
description: This guide shows how to use the Armory CD-as-a-Service GitHub Action.
---

## What the Armory CD-as-a-Service GitHub Action does

The CD-as-a-Service GitHub Action deploys your app based on a specific GitHub trigger, such as a push to the main branch of your repository. You can configure your workflow to return immediately or wait for a final deployment state before exiting. Get the latest version from the [GitHub Action Marketplace](https://github.com/marketplace/actions/armory-continuous-deployment-as-a-service).

## {{% heading "prereq" %}}

* Create your deployment config file and merged it into your app's repository.
* Create a CD-as-a-Service credential with the `Deployments Full Access` role.
* Create GitHub [secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) for your CD-as-a-Service credential's Client ID and Client Secret. You reference these secrets when you configure the CD-as-a-Service GitHub Action.

## Configure the action

In your repository's `.github/workflows` directory, create a file with the following contents:

```yaml
name:

on:
  push: # What triggers a deployment. For example, `push`.
    branches:
      - BRANCH_NAME

permissions: # Needed for trigger node on deployment graph
  contents: read
  pull-requests: read
  statuses: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Armory CD-as-a-Service Deployment
        id: deploy
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }} # Needed for trigger node on deployment graph; GitHub generates this
        uses: armory/cli-deploy-action@v1.0.0
        with:
          clientId: CLIENT_ID
          clientSecret: CLIENT_SECRET
          path-to-file: PATH_TO_FILE
          waitForDeployment: WAIT_FOR_DEPLOYMENT
```

Replace the following:

* _`BRANCH_NAME`_: The branch, for example `main`.
* _`CLIENT_ID`_: (Required) The name of the GitHub secret that you created for your CD-as-a-Service Client ID. For example, if you name your secret **CDAAS_CLIENT_ID**, the value for `clientId` is `"${{ secrets.CDAAS_CLIENT_ID }}"`.
* _`CLIENT_SECRET`_: (Required) The name of the GitHub secret that you created for your CD-as-a-Service Client ID. For example, if you name your secret **CDAAS_CLIENT_SECRET**, the value for `clientSecret` is `"${{ secrets.CDAAS_CLIENT_SECRET }}"`.
* _`PATH_TO_FILE`_: (Required) Relative path to your deployment file. The path you provide is relative to where you store your GitHub workflow file in `.github/workflows`.

   For example, if your repository looks like this:

   ```bash
   .github
   --workflows
   ---cdaas-deploy.yaml
   deployments
   --deployment.yaml
   --manifests
   ---sample-app.yaml
   ```

   Then _`PATH_TO_FILE`_ is `"/deployments/deployment.yaml"`.

* _`WAIT_FOR_DEPLOYMENT`_: (Optional)(Default: false) This blocks the GitHub Action from completing until the deployment has transitioned to its final state.

The CD-as-a-Service GitHub Action displays the Deployment ID, a link to the Deployments UI, and optionally the deployment's final state. It also returns the following output parameters that you can use elsewhere in your workflow:

* `DEPLOYMENT_ID`: This is the unique deployment identifier, which you can use to query the status of the deployment in UI.
* `LINK`: This is the link to the UI, where you can verify the state of the workflow and advance it to the next stages if you have manual judgments.
* `RUN_RESULT`: If you configured 'waitForDeployment=true', this variable contains the final state of the deployment (FAILED, SUCCEEDED, CANCELLED).

You could, for example, add a job step that displays the values of the output parameters:

```yaml
steps:
...
   - name: Print Armory CD-as-a-Service Deployment Output
     id: output
     run: echo -e 'DeploymentID ${{steps.deploy.outputs.DEPLOYMENT_ID}}\nLink ${{steps.deploy.outputs.LINK}}\n${{steps.deploy.outputs.RUN_RESULT}}'
```

See GitHub's [Using workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) content for more information on reusing variables.

## Example configuration

For this scenario:

* You have GitHub secrets called `CDAAS_CLIENT_ID` AND `CDAAS_CLIENT_SECRET`.
* You have a `deployment.yaml` file in the `deployments` directory of your repository.
* You want to deploy when a pull request is merged to the `main` branch.
* You want the CD-as-a-Service GitHub Action to run until it receives a final deployment status from CD-as-a-Service.
* You want to display the output in a separate job step.

Your workflow file contents look like this:

```yaml
name: Deploy to CD-as-a-Service

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pull-requests: read
  statuses: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Deploy app
        id: deploy
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        uses: armory/cli-deploy-action@main
        with:
          clientId: ${{ secrets.CDAAS_CLIENT_ID }}
          clientSecret: ${{ secrets.CDAAS_CLIENT_SECRET }}
          path-to-file: "/deployments/deployment.yaml"
          waitForDeployment: true

      - name: Display deploy output
        id: output
        run: echo -e 'DeploymentID ${{steps.deploy.outputs.DEPLOYMENT_ID}}\nLink ${{steps.deploy.outputs.LINK}}\n${{steps.deploy.outputs.RUN_RESULT}}'
```

## Trigger a deployment

Trigger a CD-as-a-Service deployment based on the trigger you defined in your workflow. Monitor your deployment's progress using one of the following methods:

* **GitHub workflow run log**: Use `waitForDeployment: true` in your job and watch the output in the workflow run log.

   Output is similar to:

   ```bash
   Waiting for deployment to complete. Status UI: https://console.cloud.armory.io/deployments/pipeline/f4e1fbfe-641f-4613-aff3-0699698d5aed?environmentId=82431eae-1244-4855-81bd-9a4bc165f90b
   .
   Deployment status changed: RUNNING
   .....
   Deployment status changed: PAUSED
   ..
   Deployment status changed: RUNNING
   ...
   Deployment status changed: PAUSED
   ..
   Deployment status changed: RUNNING
   Deployment ID: f4e1fbfe-641f-4613-aff3-0699698d5aed
   .....
   Deployment status changed: SUCCEEDED
   Deployment f4e1fbfe-641f-4613-aff3-0699698d5aed completed with status: SUCCEEDED
   See the deployment status UI: https://console.cloud.armory.io/deployments/pipeline/f4e1fbfe-641f-4613-aff3-0699698d5aed?environmentId=82431eae-1244-4855-81bd-9a4bc165f90b
   ```

* **CD-as-a-Service Deployments UI**: Obtain the **Deployments** UI direct link from the workflow output.

   When you configure `waitForDeployment: false`, the Action immediately displays the Deployment ID and a link to the **Deployments** UI before exiting. Output is similar to:

   ```bash
   Deployment ID: 065e9c2c-5e3e-4e6a-a591-bdd756a497c2
   See the deployment status UI: https://console.cloud.armory.io/deployments/pipeline/065e9c2c-5e3e-4e6a-a591-bdd756a497c2?environmentId=82431eae-1244-4855-81bd-9a4bc165f90b
   ```

>If you configured a manual approval in your strategy, you must use the CD-as-a-Service Deployments UI to issue that approval.
