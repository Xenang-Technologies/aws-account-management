# Before getting started
This bootstrapping process assumes you wil be managing deployments with Github Actions. Before you setup the first AWS account (environment) for a project, make sure you have created the git repository where the deployments will be managed. The cloudformation used to bootstrap accounts grants access to Github Actions via the project's repository to the `deploy` role.


# Bootstrapping a new Environment Account
When creating a new AWS account there will be a few manual steps that will need to be performed for the initial setup.
1. When creating a new AWS account use the following email address: admin+aws.project-name.environment-id@iknite.studio. Note that the text after the + (plus) in the username will be ignored by Google Workspace and all emails will go to the admin@iknite.studio inbox.
1. Use a strong password and save this password in the shared bitwarden account.
1. Add MFA to the account. Use the Bitwarden shared account for the TOTP codes.
1. Run the bootstrap-aws-account.yaml Cloudformation template manually from the AWS console. (https://github.com/Iknite-Space/aws-account-management/tree/main/environment-accounts)
1. Update the environments.yaml in the project repo to include this new environment account and set the account number
1. Add the new account id to the README.md in this repo.
1. [Activate access to the Billing and Cost Management console](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html#ControllingAccessWebsite-Activate])

The bootstrap Cloudformation template should create a CI role and grant access to this role for Github Actions for the project repository. This CI role should have permissions to deploy the entire environment into the account.

# Post bootstrapping setup
After performing the above process, users in the main account will be able to assume the `dev` role into the account. Additionally Github Actions will be able to assume the deploy role.o
When bootstrapping a new project you'll want to configure the artifact account after you've bootstrapped the first environment. To do this, complete the following steps:
1. Copy the contents of the `example` directory into your project repo.
1. Update the `artifact-account-config.yaml` file with the AWS account id of the account your just created above.
1. Update the `cd/cloudformation/artifact-account.yaml`. This Cloudformation template provisions the required resources to store build artifacts for a project. The example Cloudformation template creates a ERC repository and a s3 bucket.
