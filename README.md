# aws-account-management

Configuration for Xenang Technologies' AWS accounts: the main account, which holds all IAM users and the AWS Organization, and the project environment accounts under it.

# Directory Structure

* /account-configuration
  * CloudFormation templates for configuring the main AWS account. They are applied by GitHub Actions on every commit to `main`.
* /environment-accounts
  * Boilerplate for bootstrapping a new AWS account for a project environment. See [environment-accounts/README.md](environment-accounts/README.md).

# Managing users
IAM users and groups are configured with CloudFormation in `account-configuration/main-account.yaml`. Users can be added to these groups:

* `admins` - can assume the `admin`, `deploy`, `ci` and `dev` roles in environment accounts
* `devs` - can assume the `dev`, `trainee` and `deploy` roles in environment accounts
* `finance` - can assume the `billing` role in environment accounts

To add a user, add an `AWS::IAM::User` resource to `main-account.yaml` and merge it to `main`. CI creates the user, but it can't sign in yet: an admin must then log into the main account's console, enable console access for the user (IAM → Users → user → Security credentials) and have them set up MFA.

# Current AWS Accounts
Record every account here when it is created, along with the email address used as its root email.

* Main AWS Account (AWS Organization management account)
  * email: xenangadmin@gmail.com
  * account id: 124920735437

* Pivot
  * Dev Account
    * email: xenangadmin+aws.pivot.dev@gmail.com
    * account id: 246152664877

## Logging into the AWS dashboard
To log into the AWS dashboard for a particular project environment:
1. Log into the main AWS account with your IAM credentials (if you don't have any, ask an admin).
   * Use this URL to log in: https://124920735437.signin.aws.amazon.com/console
1. Click the account menu in the top right corner of the AWS console (it should say `username @ 1249-2073-5437`).
1. Click "Switch role".
1. Fill in the form and click "Switch Role":
   * Account ID: the id of the environment account you want, from the list above.
   * IAM role name: `dev` (or `admin` if you are in the `admins` group).
   * Display name: something memorable, like "Pivot Dev (dev role)".


# AWS Account Types
We use these terms for the AWS accounts we create to manage our projects: Main Account, Environment Account and Artifact Account.

## Main Account
* There is exactly one account of this type.
* It is the management account of the AWS Organization. All environment accounts are created in, or invited into, this organization.
* Hosts all IAM users. This is the only AWS account in which devs (and other users) have IAM users.
* Maintains the `admins`, `devs` and `finance` IAM groups, which are used to assume roles into other accounts.
* A CloudFormation template (`account-configuration/main-account.yaml`) is applied on commits to `main`. It creates the groups and IAM users.
* Bootstrapping (already done for 124920735437; only needed if the main account is ever recreated):
  1. Create the root account with a `xenangadmin+aws.<name>@gmail.com` email address. Save the password and set up MFA in the shared Bitwarden account.
  1. Create an AWS Organization from the account.
  1. Manually apply `environment-accounts/bootstrap-aws-account.yaml` in `us-east-1`, with stack name `bootstrap-aws-account`, `RepositoryName=aws-account-management` and `MainAccountId` set to this account's id. This creates the `deploy` role that GitHub Actions uses to apply `main-account.yaml`.
  1. Update the main account id in `.github/workflows/main-account-config.yaml`, in the `MainAccountId` default of `bootstrap-aws-account.yaml`, and in this README.

## Artifact Account
* Hosts all build artifacts for a project (ECR, S3).
* Hosts all "image-tag" data in Systems Manager.
* Note: this account is just one of the environment accounts of a project.
* Bootstrapping:
  * No additional bootstrapping needed, as this account is also an environment account.

## Environment Accounts
* One account per project environment (dev, testing, production, ...), created in the AWS Organization.
* Maintains 3 roles that can be assumed:
  * `dev` - gives devs access for development and monitoring
    * Assumable from the main account (`devs` and `admins` groups)
  * `admin` - full administrator access
    * Assumable from the main account (`admins` group)
  * `deploy` - automated deployments
    * Assumable by GitHub Actions from the project's repository, via the account's GitHub OIDC provider
* Bootstrapping: [see detailed directions here](environment-accounts/README.md)
