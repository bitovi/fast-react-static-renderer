<div align="center">
  <h1>Fast React Static Renderer</h1>
  <strong>A framework to build scalable and fast ecommerce platforms with thousands of products.</strong>
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

## Set up AWS account


[Sign up](https://portal.aws.amazon.com/billing/signup#/start/email) for an AWS account and get programmatic credentials

### Get credentials

The account you create when you first sign up is called the `Root user`. It is not recommended using that for day to day activities. Instead, you should create a new user.

To do that: 
- sign in to the AWS console and navigate to the IAM service.

- Under Access management click on Users and when on the Users page press the Add users button.

- Give the user a name and select the `Access key - Programmatic access` checkbox.
In the next page, give the user the necessary permissions and then complete the user creation.

At the end you will be able to download the user's credentials which we will need later:

`Access Key ID` and `Secret access key`

Make sure to store and protect these values, the secret access key cannot be retrieved again.



## Set up Contentful
- [Sign up](https://www.contentful.com/sign-up/)
- Get Credentials:
  - Go to settings > API keys
  - Click on Add API key
  - Copy the `Space ID` and the `Content Delivery API - access token`, you will need those later to connect the NextJS app to Contentful
- Enter your data into Contentful

## Add Contentful Keys to AWS Secrets

- Open the [Secrets Manager Console](https://us-east-1.console.aws.amazon.com/secretsmanager/landing?region=us-east-1)
- Click on Store a new secret
- For secret type, choose Other type of secret
- Enter the key/value pairs corresponding to your API keys
  - ContentfulAccessToken
  - ContentfulSpaceID
- Click Next and then give the secrets a name
- Finally review the information and Store the secrets

## Configure the operations repository

The operation repo is in charge of setting up the infrastructure and deploying our app. Everything is in place, we just need to plug in our credentials and change some variables.

First, add your credentials:

- Clone or fork the [operation repository](https://github.com/bitovi/fast-react-static-renderer-operations)
- Add your AWS credentials to the Github actions secrets
  - Click on the Settings in your operations repo
  - In the left panel, click on Secrets and then Actions
  - Press on the New repository secret button
  - In the name field, put `AWS_ACCESS_KEY_ID` and inside the value, the access key id you had from earlier
  - Similarly, the second secret is named `AWS_SECRET_ACCESS_KEY` with the corresponding value.

    More info about Gihub secrets [here](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

Notice that the project is divided into multiple folders. `build-dev`, `dev` and `global-tools` correspond to different environments that we configure with terraform. The `.github/workflow` contains github actions that we use to deploy each environment. 

We use terraform to create and provision AWS resources. To use this for your project, you need to change the resources names to what you need. Here's a list of every variable that needs to be changed. We repeat the same process for each environment folder, starting with `global-tools`:


  - A. Update `global-tools` to deploy resources that are shared across environments.
  
    i. Navigate to the GitHub Action for this environment at .github/workflows/deploy-global-tools.yaml and update the following values

      1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

      2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `global-tools`

    ii. Navigate to the Terraform backend configuration at global-tools/terraform/backend.tf and update the following value

      1. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

         This needs to match TF_STATE_BUCKET from above (refer to i.2.)

    iii. Navigate to the Terraform variables at global-tools/terraform/variables.tf and update the following default values.

    1. `bucket_name` - Used to store the app source in a zipped archive to use during the build process

    2. `domain_name` - Apex (root) domain name that will be used for DNS [default: fast-react-static-renderer.com ]



  - B. Update `dev` to deploy resources that will be used for the Development environment.

    i. Navigate to the GitHub Action for this environment at .github/workflows/deploy-dev.yaml and update the following values

    1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

    2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `dev`

    ii. Navigate to the Terraform backend configuration at dev/terraform/backend.tf and update the following value

    1. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

        This needs to match TF_STATE_BUCKET from above (refer to i.2.)

    iii. Navigate to the Terraform variables at dev/terraform/variables.tf and update the following default values.

    1. `bucket_name` - Used to store the app source in a zipped archive to use during the build process

    2. `domain_name` - Apex (root) domain name that will be used for DNS [default: fast-react-static-renderer.com ]

    3. `subdomain_name` - sub-domain that will be used to serve the site [default: dev.fast-react-static-renderer.com ]

    4. `hosted_zone_id` - ID of the hosted zone created by global-tools (can be found in the terraform output)



  - C. Update `build-dev` which triggers the build and deploy process for the `dev` environment.

    i. Navigate to the GitHub Action for this environment at .github/workflows/deploy-build-dev.yaml and update the following values

    1. `AWS_DEFAULT_REGION` - AWS Region where your resources will be deployed

    2. `TF_STATE_BUCKET` - This bucket is used to store Terraform state for `build-dev`

    ii. Navigate to the Terraform backend configuration at build-dev/terraform/backend.tf and update the following value

    1. `backend.s3.bucket` - This bucket is used to store the Terraform state for global tools

        This needs to match TF_STATE_BUCKET from above (refer to i.2.)

    iii. Navigate to the Terraform variables at build-dev/terraform/terraform.tfvars and update the following default values.

    1. `secret_arn_contentful_access_token` - ARN of the AWS Secret Manager Secret created earlier.

    2. `secret_arn_contentful_space_id` - ARN of the AWS Secret Manager Secret created earlier.

    3. `aws_region` - AWS Region where your resources will be deployed.

    4. `availability_zones` - Availability zones within the AWS region specified above where ECS can deploy containers.

    5. `image_registry_url` - URL for Fast React Static Renderer Build Image.

    6. `s3_bucket_contents` - Bucket that stores the app artifacts (refer to A.iii.1.)
   
    7. `publish_s3_bucket` - Bucket for static hosting (refer to B.iii.1.).

    8. `cloudfront_distribution_id` - ID of the CloudFront distribution created by global-tools (can be found in the terraform output).







