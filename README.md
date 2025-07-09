# Black Duck Polaris - GitHub Action Lab (Lab #2)

The goal of this lab is to provide hands on experience configuring a Polaris workflow in GitHub and viewing the results. As part of the lab, we will:
- execute a full scan, viewing the results in the Polaris UI
- break the build based on a policy defined in the Polaris UI
- review the code scanning findings in the GitHub Advanced Security tab
- introduce a vulnerable code change that adds a comment to the Pull Request

This repository contains everything you need to complete the lab except for the two prerequisites listed below.

## $\textsf{\color{#800080}{Prerequisites}}$
1. [signup](https://github.com/signup) for a free GitHub Account
2. [create](https://polaris.blackduck.com/developer/default/polaris-documentation/t_make-token) a Polaris Access Token 
> [!NOTE]  
> Please store your access token somewhere safe. We will use again in subsequent labs
> 

## CLONE REPOSITORY
![](https://img.shields.io/badge/steps-blueviolet?style=for-the-badge)
1. Clone this repository into your GitHub account. From the top GitHub Menu → [ + ] New → Import a Repository
   - enter https://github.com/bma-code/BD-Polaris-CTF2-GitHub-Lab-Java
   - enter repository name, First Initial + Last Initial-polaris-github-lab-java (ie. FL-polaris-github-lab-java
   - no username or access token is required to clone
   - leave as public (required for GHAS on free accounts)


## SETUP WORKFLOW
![](https://img.shields.io/badge/steps-blueviolet?style=for-the-badge)
1. Go to  GitHub → Project → Settings → Actions → General:
   - confirm that Actions permissions are set to "Allow all actions and reusuable workflows"
   - set Workflow permissions to "Read and write permissions"
   - click the box to "Allow GitHub Actions to create and approve pull requests"
   - click save

2. Go to GitHub → Project → Settings → Secrets and Variables → Actions and add the following variables:
 
   Under Secrets, add New Repository Secret
   - For the name add POLARIS_ACCESSTOKEN and for Secret* value add your Polaris access token

   Under Variables, add New Repository Variable  
   - For the name add POLARIS_SERVERURL and for the Value* add your Polaris server URL (i.e https://polaris.blackduck.com)

3. In your root folder, a coverity.yaml file has been added. Polaris can do buildless scans, but to get a higher fidelity scan this yaml file tells Polaris how to build the application and it can also be used to pass additional scan instructions. For this lab, simply verify it exists in your root folder. 

```
capture:
  build:
    clean-command: mvn -B clean
    build-command: mvn -B -DskipTests package
analyze:
  checkers:
    webapp-security:
      enabled: true
```

4. We are now going to create a new GitHub Action workflow. Below you will see a template that you can copy and paste and use as your workflow. This utilizes the Black Duck Security Scan GitHub Action.  

   The code below will works as is, but for our lab we need to make a change.  Under ### SCANNING: Optional fields, change <ins>**polaris_application_name**</ins> value to match the application name you created in the previous labs. Leave the project name as-is and this will add it to your application as a second project in Polaris.

   Go to, GitHub → Project → Actions → New Workflow → Setup a workflow yourself

```
# example workflow for Polaris scans using the Black Duck Action
# https://github.com/marketplace/actions/black-duck-security-scan
name: polaris
on:
  push:
    branches: [ main, master, develop, stage, release ]
  pull_request:
    branches: [ main, master, develop, stage, release ]
  workflow_dispatch:
jobs:
  build:
    runs-on: [ ubuntu-latest ]
    steps:
      - name: Checkout Source
        uses: actions/checkout@v3
      - name: Polaris Scan
        uses: blackduck-inc/black-duck-security-scan@v2.0.0
        with:
          ### SCANNING: Required fields
          polaris_server_url: ${{ vars.POLARIS_SERVERURL }}
          polaris_access_token: ${{ secrets.POLARIS_ACCESSTOKEN }}
          polaris_assessment_types: "SCA,SAST"
          
          ### SCANNING: Optional fields
          polaris_application_name: 
          polaris_project_name: ${{ github.event.repository.name }}
          
          ### PULL REQUEST COMMENTS: Uncomment below to enable
          polaris_prComment_enabled: true 
          github_token: ${{ secrets.GITHUB_TOKEN }} # Required when PR comments is enabled

          ### SARIF report parameters
          polaris_reports_sarif_create: true
          polaris_upload_sarif_report: true
          
          ### Signature scan
          #polaris_test_sca_type: "SCA-SIGNATURE"
```
## FULL  SCAN
![](https://img.shields.io/badge/steps-blueviolet?style=for-the-badge)
1. Monitor your workflow run and wait for scan to complete. _GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris_
   - Note that scan completes, and the workflow passes. This is because the default policy is notify on critical & high issues.
2. From the Polaris UI, [create a policy](https://polaris.blackduck.com/developer/default/polaris-documentation/t_post_scan_policies) that breaks the build and assign it to your project.
3. Rerun workflow, and once it completes, select _Summary_ in upper left to see policy enforcement and a failed workflow.
4. View findings in GitHub Advanced Security tab _GitHub → Project → Security → Code scanning


## PR SCAN
![](https://img.shields.io/badge/steps-blueviolet?style=for-the-badge)
1. Edit pom.xml GitHub → Project → Code → pom.xml → Edit pencil icon upper right
   - change log4j version from 2.14.1 to 2.15.0
3. Click on Commit Changes, select create a **new branch** and start a PR and click Propose Change
4. Review changes and click on _Create Pull Request_
5. Monitor workflow run GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris
6. Once workflow completes, navigate back to PR and see PR comment GitHub → Project → Pull requests


# Congratulations

You have now configured a Polaris workflow in GitHub and demonstrated all the current post-scan CI features. :clap: :trophy:

## ![](https://img.shields.io/badge/optional-CTF-blueviolet?style=for-the-badge)
Once you successfully run the scan, in the security > code-scanning tab from GitHub you will see a "Use of Hard-coded Credentials" finding. Locate the secret, and decrypt it. Add this to the first labs decrypted secret. 
