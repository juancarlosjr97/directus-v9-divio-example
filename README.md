# Directus v9 Installation using Divio

This guide is to create a Dockerise Directus v9 using Divio free plan with Continuous Deployment using Webhooks, and GitHub as hosting platform.

Prerequisites

- [Divio account](https://www.divio.com/)
- [Docker](https://docker.com/)
- [Python3](https://www.python.org/)
- [Divio CLI](https://docs.divio.com/en/latest/reference/divio-cli/) `pip install divio-cli`

The result of the guide is the following repository: https://github.com/juancarlosjr97/directus-v9-divio-example

### Part 1 - Project creation

1. Create a empty repository either private or public on GitHub, GitLab or Bitbucket
2. Create a master branch and add a README file. This branch will be the production environment
3. Create a staging branch. This branch will be the testing environment
4. Create a project on Divio

- Select as configuration `Build your own`
- Select as Repository Manager `Custom`
- Paste the repository url as SSH format. e.g. `git@github.com:juancarlosjr97/directus-v9-divio-example.git`
- Follow the instructions to enable the repository to be connected to Divio https://docs.divio.com/en/latest/how-to/resources-configure-git/#add-the-git-repository-url-to-the-control-panel

After the repository has been connected successfully it will add a `Dockerfile` and a `.gitignore` to the repository

5. Select Developer plan
6. Click on subscribe to complete the project creation

### Part 2 - Configure testing environment settings

1. Go to `Repository` on the menu
2. Change the `Test Server Branch` as `staging`

### Part 3 - Adding Webhooks

This guide is using `GitHub`, please use this link https://docs.divio.com/en/latest/how-to/resources-configure-git/#configure-a-webhook-for-the-git-repository to configure a webhook for other Git repositories

In order for the Control Panel to receive a signal when the repository is updated, you need to set up a webhook. This step is optional but strongly recommended for convenience.

1.  In the repository On GitHub, go to Settings > Webhooks > Add webhook
2.  Add the Webhook URL to the Payload URL field
3.  Leave the Content type as `application-x/www-form-urlencode`
4.  Add the Webhook Shared Secret to the Secret field
5.  Set Push events as the trigger for the webhook

### Part 4 - Adding Continuous Deployment

On this step, we are using GitHub actions to add Continuos Deployment to the repository.

1. Create a directory `.github` on the root of the project, and then create a subdirectory named `workflow`
2. Copy the `deploy.example.yml` and paste to the directory `workflow` with the name `deploy.yml`
3. Create the following secrets on your GitHub repository:

- DIVIO_LOGIN_TOKEN
  - Copy the `Access token` from this website https://control.divio.com/account/desktop-app/access-token/
- DIVIO_PROJECT_ID
  - Run `divio project list` and copy the `ID` of the project
- MASTER_ENVIRONMENT_NAME = `live`
- STAGING_ENVIRONMENT_NAME = `test`

For more information please visit: https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets

### Part 5 - Database creation

This part will create a AWS Redshift database with the free plan

1. On the Divio project, go to `Services`
2. Click on `Add Services`
3. Set as prefix `DATABASE`
4. Select `PostgreSQL database`
5. Select as environments `Test` and `Live`
6. Click on `Add new service` to finish the database setup
7. Click on the vertical ellipsis, and then to `provision` to finish the database setup

### Part 6 - Storage creation

This part will create a AWS S3 storage bucket with the free plan

1. On the Divio project, go to `Services`
2. Click on `Add Services`
3. Set as prefix `STORAGE`
4. Select `Object storage`
5. Select as environments `Test` and `Live`
6. Click on `Add new service`
7. Click on the vertical ellipsis, and then to `provision` to finish the storage setup

### Part 7 - Configure Directus v9 with Docker

1. Clone your repository for local development using divio `divio project setup ${PROJECT_NAME}` and cd into the project
2. Change to the staging branch `git checkout staging`
3. Copy the `Dockerfile.example` and replace the current `Dockerfile` on your local repository
4. Create a new `KEY` and `SECRET` environment variable

### Part 8 - Database configuration using environment variables

1. From the repository created using divio run the following command `divio project env-vars -s test --all --get "DATABASE_DATABASE_DSN"`. To see live environment variable, replace `test` with `live`. Read more here https://docs.divio.com/en/latest/background/configuration-environment-variables/

2. The environment variable is in the following form `schema://<user name>:<password>@<address>:<port>/<name>`. Read more here https://docs.divio.com/en/latest/how-to/interact-database/#connecting-to-a-database-manually

3. Split the given environment variable following the form presented on the step two as follow:

For example:

```
postgres://directusv9divioexample-test-8e68edcd52ca46d0a50365f56a9-d9e8d3a:dWU3XJ0aweNzt6aUDK32kA-BYX4FZ9M7GwIRUb2fgRQY55FzX1v6a4j80Dw9GtnMgiDsoJHXuR3tV6sl@appctl-black-sites-02.cs4nfpul9fcn.us-east-1.rds.amazonaws.com:5432/directusv9divioexample-test-8e68edcd52ca46d0a50365f56a9-c231b3a
```

```
DB_CLIENT="postgres"
DB_HOST="appctl-black-sites-02.cs4nfpul9fcn.us-east-1.rds.amazonaws.com"
DB_PORT="5432"
DB_DATABASE="directusv9divioexample-test-8e68edcd52ca46d0a50365f56a9-c231b3a"
DB_USER="directusv9divioexample-test-8e68edcd52ca46d0a50365f56a9-d9e8d3a"
DB_PASSWORD="dWU3XJ0aweNzt6aUDK32kA-BYX4FZ9M7GwIRUb2fgRQY55FzX1v6a4j80Dw9GtnMgiDsoJHXuR3tV6sl"
```

4. Add the environment variables to the `Test` environment. Repeat the same process for `Live` environment

### Part 9 - Storage configuration using environment variables

1. From the repository created using divio run the following command `divio project env-vars -s test --all --get "STORAGE_STORAGE_DSN"`. To see live environment variable, replace `test` with `live`. Read more here https://docs.divio.com/en/latest/background/configuration-environment-variables/.

2. The environment variable is in the following form `schema://<key>:<secret>@<bucket-name/endpoint/region>`. Read more here https://docs.divio.com/en/latest/how-to/interact-storage/#storage-access-details

3. Split the given environment variable following the form presented on the step two as follow:

For example:

```
s3://AKIA2L5WWB3LQV4PDSXY:pBgV%2FpdPY4MVFyCL66Zl%2BE8vUBYwVxIYkodC1seS@directusv9divioexample-test-8e68edcd52c-adfb0c6.divio-media.com.s3.amazonaws.com/?auth=s3v4&domain=directusv9divioexample-test-8e68edcd52c-adfb0c6.divio-media.com
```

- Check the encoding of the secret. The secret may contain some symbols encoded as hexadecimal values, and you will need to change them back before using them:

  - %2B must be changed to +
  - %2F must be changed to /

And for any other values beginning with % use [Conversion table](https://en.wikipedia.org/wiki/ASCII#Printable_characters)

- Check the storage region. Use as reference the following link https://docs.divio.com/en/latest/how-to/interact-storage/#storage-access-details

```
STORAGE_LOCATIONS="amazon"
STORAGE_AMAZON_DRIVER="s3"
STORAGE_AMAZON_PUBLIC_URL="s3.amazonaws.com"
STORAGE_AMAZON_KEY="AKIA2L5WWB3LQV4PDSXY"
STORAGE_AMAZON_SECRET="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"
STORAGE_AMAZON_ENDPOINT="s3.amazonaws.com"
STORAGE_AMAZON_BUCKET="directusv9divioexample-test-8e68edcd52c-adfb0c6.divio-media.com"
STORAGE_AMAZON_REGION="us-east-1"
```

4. Add the environment variables to the `Test` environment. Repeat the same process for `Live` environment

### Part 10 - Deployment

1. Commit the new `Dockerfile`
2. Visit the project on divio.com and open the logs
3. Search for `Creating administrator user` and copy the `Email` and `Password`
4. Visit your `Env URL`, login and change the administrator credentials

### Final Part - Have fun!

The final result is the following Directus instance:

- Test environment:
  - User:
  - Password:
- Live environment:

Feel free to test the final result!
