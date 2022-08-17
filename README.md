<div align="center">
  <h1>Fast React Static Renderer</h1>
  <strong>A framework to build scalable and fast ecommerce platforms with thousands of pages.</strong>
</div>

<br />
<br />

## Introduction

This project consists of 3 repositories.

- [fast-react-static-renderer-app](https://github.com/bitovi/fast-react-static-renderer-app) \
  A Next.js ecommerce app connected with Contentful

- [fast-react-static-renderer-operations](https://github.com/bitovi/fast-react-static-renderer-operations) \
  Infrastructure as code with Terraform in charge of deploying the app

- [fast-react-static-renderer-build-image](https://github.com/bitovi/fast-react-static-renderer-build-image) \
  The docker image to run the builds

Follow the guide below to setup the project.

## Need help or have questions?

This project is supported by [Bitovi, a React consultancy](https://www.bitovi.com/frontend-javascript-consulting/react-consulting). You can get help or ask questions on our:

- [Slack Community](https://www.bitovi.com/community/slack)
- [Twitter](https://twitter.com/bitovi)

Or, you can hire us for training, consulting, or development. [Set up a free consultation.](https://www.bitovi.com/frontend-javascript-consulting/react-consulting)

## Prerequisites

### Set up AWS account

[Sign up for an AWS account](https://portal.aws.amazon.com/billing/signup#/start/email) and get programmatic credentials

#### Get credentials

The account you create when you first sign up is called the `Root user`. It is not recommended to use that for day to day activities. Instead, you should create a new user.

To do that:
- Sign in to the AWS console and navigate to the IAM service.

- Under Access management click on Users and when on the Users page press the Add users button.

- Give the user a name and select the `Access key - Programmatic access` checkbox.
In the next page, give the user the necessary permissions and then complete the user creation.

At the end you will be able to download the user's credentials which we will need later:

`Access Key ID` and `Secret access key`

Make sure to store and protect these values, the secret access key cannot be retrieved again.

### Set up Contentful
- [Sign up](https://www.contentful.com/sign-up/)
- Get Credentials:
  - Go to settings > API keys
  - Click on Add API key
  - Copy the `Space ID` and the `Content Delivery API - access token`, you will need those later to connect the NextJS app to Contentful
- Enter your data into Contentful

### (Optional) Have a valid domain name

If you want the application deployed with a custom domain name, you should have a certified domain name before the next steps.

If you don't have a domain name, please ignore the following variables in the guide:
- `domain_name`
- `subdomain_name`
- `hosted_zone_id`

And make sure to follow the [No Custom domain setup](#no-custom-domain-setup) step below.



**Once you have the all prerequisites, you are ready to start the guide.**

## Add Contentful Keys to AWS Secrets

- Open the [Secrets Manager Console](https://us-east-1.console.aws.amazon.com/secretsmanager/landing?region=us-east-1)
- Click on `Store a new secret`
- For secret type, choose `Other type of secret`
- Enter the key/value pairs corresponding to your API keys
  - ContentfulAccessToken
  - ContentfulSpaceID
- Click Next and then give the secrets a name
- Finally review the information and Store the secrets

## Configure the operations repository

The operation repo is in charge of setting up the infrastructure and deploying our app. Everything is in place, we just need to plug in our credentials and change some variables.

### Add your credentials:

- Clone or fork the [operation repository](https://github.com/bitovi/fast-react-static-renderer-operations)
- Add your AWS credentials to the Github actions secrets
  - Click on the Settings in your operations repo
  - In the left panel, click on Secrets and then Actions
  - Press on the New repository secret button
  - In the name field, put `AWS_ACCESS_KEY_ID` and inside the value, the access key id you had from earlier
  - Similarly, the second secret is named `AWS_SECRET_ACCESS_KEY` with the corresponding value.

    More info about Gihub secrets [in the github documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### No Custom domain setup:

**IMPORTANT** If you chose to move forward without a custom domain name, you need to do the following before you start:

- Navigate to the GitHub Action for global tools at `.github/workflows/deploy-global-tools.yaml` and **remove** the line starting with `TFVARS_AUTO_JSON`

- Remove the same line for the dev GitHub Action at `.github/workflows/deploy-dev.yaml`

### General Setup

Notice that the project is divided into multiple folders. `build-dev`, `dev` and `global-tools` correspond to different environments that we configure with terraform. The `.github/workflow` contains github actions that we use to deploy each environment.

We use terraform to create and provision AWS resources. To use this for your project, you need to change the resources names to what you need. Below is a comprehensive list of every variable that needs to be changed. You can refer to the existing values to get an indication of what you can name your variables.

We repeat the same process for each environment folder, starting with `global-tools` and one folder at a time:

  - A. Update `global-tools` to deploy resources that are shared across environments.

    1. Navigate to the GitHub Action for this environment at .github/workflows/deploy-global-tools.yaml and update the following values

       1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

       2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `global-tools`

    2. Navigate to the Terraform backend configuration at global-tools/terraform/backend.tf and update the following value

       1. `backend.s3.region` - AWS Region where your resources will be deployed
       2. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

            This needs to match TF_STATE_BUCKET from above (refer to i.b.)

    3. Navigate to the Terraform variables at global-tools/terraform/variables.tf and update the following default values.

       1. `bucket_name` - Used to store the app source in a zipped archive to use during the build process

       2. `domain_name` (Optional - if you have a domain name) - Apex (root) domain name that will be used for DNS [default: fast-react-static-renderer.com ]

       3. `hosting_bucket_name` - Used to store the statically generated files from the build

  - B. Update `dev` to deploy resources that will be used for the Development environment.

    1. Navigate to the GitHub Action for this environment at .github/workflows/deploy-dev.yaml and update the following values

       1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

       2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `dev`

    2. Navigate to the Terraform backend configuration at dev/terraform/backend.tf and update the following value

       1. `backend.s3.region` - AWS Region where your resources will be deployed
       2. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

            This needs to match TF_STATE_BUCKET from above (refer to i.b.)

    3. Navigate to the Terraform variables at dev/terraform/variables.tf and update the following default values.

       1. `bucket_name` - Used to store the statically generated files from the build (same as A.iii.3)

       2. `domain_name` (Optional - if you have a domain name) - Apex (root) domain name that will be used for DNS [default: fast-react-static-renderer.com ]

       3. `subdomain_name` (Optional - if you have a domain name) - Domain name for CloudFront, will be the primary domain for end users to access the application. [default: dev.fast-react-static-renderer.com ]

       4. `hosted_zone_id` (Optional - if you have a domain name) - ID of the hosted zone created by global-tools (can be found in the AWS console. Navigate to Route 53 - hosted zones and copy the hosted zone ID )

    4. Change `aws.region` in `dev/tarraform/us-east-1`

  - C. Update `build-dev` which triggers the build and deploy process for the `dev` environment.

    1. Navigate to the GitHub Action for this environment at .github/workflows/deploy-build-dev.yaml and update the following values

       1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

       2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `build-dev`

    2. Navigate to the Terraform backend configuration at build-dev/terraform/backend.tf and update the following value

       1. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

            This needs to match TF_STATE_BUCKET from above (refer to i.b.)

    3. Navigate to the Terraform variables at build-dev/terraform/terraform.tfvars and update the following default values.

       1. `secret_arn_contentful_access_token` - ARN of the AWS Secret Manager Secret created earlier.

       2. `secret_arn_contentful_space_id` - ARN of the AWS Secret Manager Secret created earlier.

       3. `aws_region` - AWS Region where your resources will be deployed.

       4. `availability_zones` - Availability zones within the AWS region specified above where ECS can deploy containers.

       5. `image_registry_url` - URL for Fast React Static Renderer Build Image.

       6. `s3_bucket_contents` - Bucket that stores the app artifacts (refer to A.iii.1.)

       7. `publish_s3_bucket` - Bucket for static hosting (refer to B.iii.1.).

       8.  `cloudfront_distribution_id` - ID of the CloudFront distribution created by global-tools (can be found in the AWS console. Navigate to the Cloudfront service - click on distributions - copy the ID for dev).

    4. Change region if needed in `provider.tf`

- D. Update the `deploy-build-dev-trigger.yaml` workflow file in `.github/workflows/deploy-build-dev-trigger.yaml` by changing the following values.
    - `AWS_DEFAULT_REGION`
    - `TF_STATE_BUCKET`: should be the same as in B.i.2

- E. (Optional) Change the repository name tags in `iam-ci-user.tf`, `route53-zone.tf` and `s3-packages.tf`


## Configure React App

- Clone or fork the [React app repository](https://github.com/bitovi/fast-react-static-renderer-app)
- Add your AWS credentials to the Github actions secrets (same as above)
- Add a Github personal access token as an action secret called `OPERATIONS_REPO_TOKEN`.
  To generate the token, follow the steps [in the Github documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- Edit the `publish-and-deploy.yaml` workflow file:
  - Change `aws-region` if needed
  - Put your artifacts s3 bucket name in `PUBLISH_CONTENTS_S3_BUCKET`

After commiting these changes, the `publish and deploy` worklow will be triggered in the react app and that will trigger the `Deploy build-dev` workflow in the operations repo. That will take care of building and deploying your App. After a few minutes, navigate to your domain name. Your app should be live!.
