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

## Configure the operations repository

The operation repo is in charge of setting up the infrastructure and deploying our app. Everything is in place, all we need to do is plug in our credentials.

- Clone or fork the [operation repository](https://github.com/bitovi/fast-react-static-renderer-operations)
- Add your AWS credentials to the Github actions secrets
  - Click on the Settings in your operations repo
  - In the left panel, click on Secrets and then Actions
  - Press on the New repository secret button
  - In the name field, put `AWS_ACCESS_KEY_ID` and inside the value, the access key id you had from earlier
  - Similarly, the second secret is named `AWS_SECRET_ACCESS_KEY` with the corresponding value.

    More info about Gihub secrets [here](https://docs.github.com/en/actions/security-guides/encrypted-secrets)



