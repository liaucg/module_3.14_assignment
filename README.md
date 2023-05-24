# Module 3.14 Assignment
3.14 Continuous Deployment - Package Vulnerability Scan

## Objective
The objective of this assignment is to set up a continuous integration and continuous deployment (CI/CD) pipeline for a Node.js application that is deployed on a serverless platform with Package Vulnerability Scan in placed!.

## Step 1: Create a code repository in GitHub
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/3564226b-5580-4e78-99b6-b3b92d2c3a46)

## Step 2: Clone the code repository into local machine
```
$ git clone git@github.com:liaucg/module_3.14_assignment.git
```

## Step 3: Create index.js file
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/d70cd8b2-0e71-4dba-8658-3345f7d1126d)

[index.js](index.js)
```js
module.exports.handler = async (event) => {
    return {
      statusCode: 200,
      body: JSON.stringify(
        {
          message: "Go Serverless v3.0! Your function executed successfully!",
          input: event,
        },
        null,
        2
      ),
    };
  };
```

## Step 4: Create serverless.yml
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/05449a83-64bc-4250-96a8-2987156a1005)

[serverless.yml](serverless.yml)
```yml
service: liau-module-3-14-assignment
frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs18.x
  region: ap-southeast-1

functions:
  api:
    handler: index.handler
    events:
      - httpApi:
          path: /
          method: get

plugins:
  - serverless-offline
```

## Step 5: Deploy and verify that the serverless application is working
Install dependencies with:
```
$ npm install
```

and then deploy with:
```
$ serverless deploy
```

Following is the output from deploy command:
```
Running "serverless" from node_modules

Deploying liau-module-3-14-assignment to stage dev (ap-southeast-1)

✔ Service deployed to stack liau-module-3-14-assignment-dev (119s)

endpoint: GET - https://a9jq2b1243.execute-api.ap-southeast-1.amazonaws.com/
functions:
  api: liau-module-3-14-assignment-dev-api (53 MB)
```

After succesful deployment the created serverless application can be invoked with curl command:
```
curl https://a9jq2b1243.execute-api.ap-southeast-1.amazonaws.com/
```

which resulted in the following response:
```json
{
  "message": "Go Serverless v3.0! Your function executed successfully!",
  "input": {
    "version": "2.0",
    "routeKey": "GET /",
    "rawPath": "/",
    "rawQueryString": "",
    "headers": {
      "accept": "*/*",
      "content-length": "0",
      "host": "a9jq2b1243.execute-api.ap-southeast-1.amazonaws.com",
      "user-agent": "curl/7.81.0",
      "x-amzn-trace-id": "Root=1-646e3131-7c33a9d472537f212918c69f",
      "x-forwarded-for": "118.200.182.45",
      "x-forwarded-port": "443",
      "x-forwarded-proto": "https"
    },
    "requestContext": {
      "accountId": "255945442255",
      "apiId": "a9jq2b1243",
      "domainName": "a9jq2b1243.execute-api.ap-southeast-1.amazonaws.com",
      "domainPrefix": "a9jq2b1243",
      "http": {
        "method": "GET",
        "path": "/",
        "protocol": "HTTP/1.1",
        "sourceIp": "118.200.182.45",
        "userAgent": "curl/7.81.0"
      },
      "requestId": "FbyfvhUUSQ0EMxg=",
      "routeKey": "GET /",
      "stage": "$default",
      "time": "24/May/2023:15:45:53 +0000",
      "timeEpoch": 1684943153292
    },
    "isBase64Encoded": false
  }
}
```

## Step 6: Create CI/CD pipeline with GitHub Actions
Create main.yml in .github/workflows folder
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/91c791bb-b7aa-42fc-8e10-54a1378c094b)

[main.yml](.github/workflows/main.yml)
```yml
name: CICD for Serverless Application
run-name: ${{ github.actor }} is doing CICD for Serverless Application

on:
  push:
    branches:  [ main, "*"]

jobs:
  pre-deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ github.event_name }} event."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by GitHub!"
      - run: echo "🔎 The name of your branch is ${{ github.ref }} and your repository is ${{ github.repository }}."

  install-dependencies:
    runs-on: ubuntu-latest
    needs: pre-deploy
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Run Installation of Dependencies Commands
        run: npm install

  deploy:
    name: deploy
    runs-on: ubuntu-latest
    needs: install-dependencies
    strategy:
      matrix:
        node-version: [18.x]
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
    - run: npm ci
    - name: serverless deploy
      uses: serverless/github-action@v3.2
      with:
        args: deploy
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Workflow Syntax
**name**: The name of the workflow.

**on**: The type of event that can run the workflow. This workflow will only run when there is a git push to either the main or other branch.

**jobs**: A workflow consists of one or more jobs. Jobs run in parallel unless a ***needs*** keyword is used. Each job runs in a runner environment specified by ***runs-on***.

**steps**: A sequence of tasks to be carried out.

**uses**: Selects an action to run as part of a step in your job. An action is a reusable unit of code.

**with**: A map of input parameters.

**run**: Runs command line programs.

**env**: Set the environment variables.

## Step 7: Add AWS_ACCESS_KEY_ID and ASW_SECRET_ACCESS_KEY to GitHub Secrets
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/c96a13e3-13c5-4aec-8200-9da80b4e2285)

## Step 8: Push changes to GitHub to start the workflow
Commit changes locally and push it to GitHub. Navigate the repo on GitHub, click on the **Actions** tab to see the workflows.
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/4610cf33-f149-43f8-9323-3fc908dfb0e9)

![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/ff439593-84a9-4973-8211-ad12a088a86d)

## Step 9: Add a secret in AWS Secrets Manager
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/3536fcdc-0716-455c-9f7b-4842a44288c3)
![image](https://github.com/liaucg/module_3.13_assignment/assets/22501900/d5fb8a2f-7bd5-4f61-a2f3-abe3a48df829)

## Step 10: Retrieve the stored secret from AWS Secrets Manager as part of the CI/CD pipeline
Add a new job **retrieve-secret** to .github/workflows/main.yml to retrieve the stored secret *LIAU_SECRET_2* from AWS Secrets Manager and inject into environment variables
```yml
retrieve-secret:
  runs-on: ubuntu-latest
  needs: install-dependencies
  steps:
  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v1
    with:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      aws-region: ap-southeast-1

   - name: Read secrets from AWS Secrets Manager into environment variables
     uses: abhilash1in/aws-secrets-manager-action@v2.1.0
     with:
       secrets: LIAU_SECRET_2
       parse-json: false
```

The last step added will print the value of *LIAU_SECRET_2*
```yml
- name: Check if env variable is set after fetching secrets
  run: if [ -z ${LIAU_SECRET_2+x} ]; then echo "LIAU_SECRET_2 is unset"; else echo "LIAU_SECRET_2 is set to '$LIAU_SECRET_2'"; fi
```

**secrets**:
 - List of secret to be retrieved.
 - Examples:
   - To retrieve a single secret, use ```secrets: my_secret_1```
   - To retrieve multiple secrets, use:
     ```
     secrets: |
     my_secret_1
     my_secret_2
     ```
   - To retrieve "all secrets having names that contain dev" or "begin with app1/dev/", use:
     ```
     secrets: |
     *dev*
     app1/dev/*   
     ```
**parse-json**:
 - If parse-json: true and secret value is a valid stringified JSON object, it will be parsed and flattened. Each of the key value pairs in the flattened JSON object will become individual secrets. The original secret name will be used as a prefix.

## Step 10: Push changes to GitHub to start the workflow
Commit changes locally and push it to GitHub. Navigate the repo on GitHub, click on the **Actions** tab to see the workflows.
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/d7cf0e9f-9b28-4751-bb9a-0f28d641de12)
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/40bb09b5-f9cd-4ce7-882c-bd56b13fb879)

## Step 11: Add a new job **scan-dependencies** in .github/workflows/main.yml to scan for package vulnerability with *npm audit* command.
```yml
scan-dependencies:
  runs-on: ubuntu-latest
  needs: install-dependencies
  steps:
    - name: Run Scan Dependencies Commands
      run: npm audit
```

# Step 11: Push changes to GitHub to start the workflow
Commit changes locally and push it to GitHub. Navigate the repo on GitHub, click on the **Actions** tab to see the workflows.
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/7e821111-0f34-4c2a-a307-2dbc6c0909ec)
![image](https://github.com/liaucg/module_3.14_assignment/assets/22501900/b5b2a94e-e1b5-4d5f-ad3a-68ecc50135b8)


