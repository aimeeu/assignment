---
title: Procedure
weight: 3
description: This guide shows how to use the Armory CD-as-a-Service GitHub Action.
---

## What the Armory CD-as-a-Service GitHub Action does

This GitHub Action enables deploying your app based on a specific GitHub trigger, such as a push to the main branch of your repo. You can configure the action to return immediately or wait for a final deployment state before exiting.

You can find the Action in the [GitHub Action Marketplace](https://github.com/marketplace/actions/armory-continuous-deployment-as-a-service).

## {{% heading "prereq" %}}

1. If you are new to using GitHub Actions, see GitHub's [Quickstart for GitHub Actions](https://docs.github.com/en/actions/quickstart) guide for information about setting up GitHub Actions.
1. You have a CD-as-a-Service credential with the `Deployments Full Access` role.
1. You have GitHub [secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) for the credential's Client ID and Client Secret so you don't expose them in plain text in your GitHub workflow file. You reference these secrets when you configure the CD-as-a-Service GitHub Action.
1. You have created your deployment config file and merged it into your repo.

## Configure the action

In your repo's `.github/workflows` directory, create a file with the following contents:

```yaml
name:

on:
  push: # What triggers a deployment. For example, `push`.
    branches:
      - BRANCH_NAME # What branch triggers a deployment. For example, `main`.

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
        uses: armory/cli-deploy-action@main
        with:
          clientId: "GITHUB_SECRET_NAME_FOR_CLIENT_ID" 
          clientSecret:  "GITHUB_SECRET_NAME_FOR_CLIENT_SECRET"
          path-to-file: "PATH_TO_DEPLOYMENT_FILE"
          waitForDeployment: true | false
```

* `clientId`: GitHub secret that you created for your CD-as-a-Service Client ID. For example, if you named your secret **CDAAS_CLIENT_ID**, the value for `clientId` would be `"${{ secrets.CDAAS_CLIENT_ID }}"`.
* `clientSecret`: GitHub secret that you created for your CD-as-a-Service Client ID. For example, if you named your secret **CDAAS_CLIENT_SECRET**, the value for `clientSecret` would be `"${{ secrets.CDAAS_CLIENT_SECRET }}"`.
* `path-to-file`: Relative path to your deployment file. The path you provide for the `path-to-file` parameter is relative to where your GitHub Action YAML is stored (`.github/workflows`).

   For example, if your repo looks like this:

   ```bash
   .github
   --workflows
   ---cdaas-deploy.yaml
   deployments
   --deployment.yaml
   --manifests
   ---sample-app.yaml
   ```

   Then `path-to-file` would be `"/deployments/deployment.yaml"`.

* `waitForDeployment`: (Optional); Default: false; this blocks the GitHub Action from completing until the deployment has transitioned to its final state (FAILED, SUCCEEDED, CANCELLED).

The Action prints out the Deployment ID, a link to the Deployments UI, and optionally the deployment's final state. It also returns that information in output parameters that you can use elsewhere in your workflow:

* `DEPLOYMENT_ID`: This is the unique deployment identifier, which you can use to query the status of the deployment in UI.
* `LINK`: This is the link to the UI, where you can check the state of the workflow and advance it to the next stages if you have manual judgments.
* `RUN_RESULT`: If you configured 'waitForDeployment=true', this variable contains the final state of the deployment (FAILED, SUCCEEDED, CANCELLED).

You could, as a simplistic example, add a job step that prints out the values of the output parameters:

```yaml
steps:
...
   - name: Print Armory CD-as-a-Service Deployment Output
     id: output
     run: echo -e 'DeploymentID ${{steps.deploy.outputs.DEPLOYMENT_ID}}\nLink ${{steps.deploy.outputs.LINK}}\n${{steps.deploy.outputs.RUN_RESULT}}'
```

See GitHub's [Using workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) content for more information.

## Example configuration

For this scenario:

1. You created secrets called `CDAAS_CLIENT_ID` AND `CDAAS_CLIENT_SECRET`.
1. Your `deployment.yaml` file is in the `deployments` directory of your repo.
1. You want to deploy when a pull request is merged to the `main` branch.
1. You want the Armory GitHub Action to run until it receives a final deployment status from CD-as-a-Service.
1. You want to print the output from the Armory GitHub Action in a separate job step.

Your workflow file contents looks like this:

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

      - name: Print deploy output
        id: output
        run: echo -e 'DeploymentID ${{steps.deploy.outputs.DEPLOYMENT_ID}}\nLink ${{steps.deploy.outputs.LINK}}\n${{steps.deploy.outputs.RUN_RESULT}}'
```

## Trigger a deployment

After you have created your deployment file and configured your workflow, you can trigger a CD-as-a-Service deployment based on the trigger you defined in your workflow.

You can monitor your deployment's progress in the GitHub UI or in the CD-as-a-Service UI. Be sure to you know how to access a GitHub Action [workflow run log](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/about-monitoring-and-troubleshooting) before you begin.

1. **GitHub workflow run log**: Use `waitForDeployment: true` in your job and watch the Action output in the workflow run log.

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

1. **CD-as-a-Service Deployments UI**: Obtain the **Deployments** UI direct link from the Action output.

   When you configure `waitForDeployment: false`, the Action immediately prints out the Deployment ID and a link to the **Deployments** UI and then exits. Output is similar to:

   ```bash
   Deployment ID: 065e9c2c-5e3e-4e6a-a591-bdd756a497c2
   See the deployment status UI: https://console.cloud.armory.io/deployments/pipeline/065e9c2c-5e3e-4e6a-a591-bdd756a497c2?environmentId=82431eae-1244-4855-81bd-9a4bc165f90b
   ```

>Note: if you configured a manual approval in your strategy, you must use the CD-as-a-Service Deployments UI to issue that approval.
