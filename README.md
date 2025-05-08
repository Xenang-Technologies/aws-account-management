# aws-account-manage


# Directory Structure

* /environment-accounts
  * These are boilerplate files for creating a new aws account for a project environment
* /account-configuration
  * These files are for configuring the main AWS account. They contain cloudformation templates that are applied on commits to main.

# Managing users
IAM users and groups are configured via Cloudformation in this file, `account-configuration/main-account.yaml`. We have two groups that users can be added to: `dev` and `admin`. After a user is added to this file for the first time, an admin user will need to log into the AWS console to generate some creds for the user.

# Current AWS Accounts

* Main AWS Account
  * email: admin+aws.main@iknite.studio
  * account id: 842675981834
* Omala
  * Dev account
    * email: admin+aws.omala.dev@iknite.studio
    * account id: 022498999470
* Cliqkets
  * Dev account (also the artifacts account)
    * email: admin+aws.cliqkets.dev@iknite.studio
    * account id: 908027382967
* Itamba (all environments and resources)
  * account id: 208552340011
  * email: -
* VsorPay
  * Artifact Acocunt
    * email: admin+aws.vsorpay.artifact@iknite.studio
    * account id: 685873929242
  * Dev B Account
    * email: admin+aws.vsorpay.devb@iknite.studio
    * account id: 403395453307
    * domain: vpayb.site
  * Dev Account
    * email: admin+aws.vsorpay.dev@iknite.studio
    * account id: 557690596244
  * Testing Account
    * email: admin+aws.vsorpay.testing@iknite.studio
    * account id: 597088027552
  * Testing 2 Account
    * email: admin+aws.vsorpay.testing@iknite.studio
    * account id: 153870713210
* OSSP
  * Dev Account (also the artifacts account)
    * email: admin+aws.ossp.dev@iknite.studio
    * account id: 557690618326
  * Dev2 Account
    * email: admin+aws.ossp.dev2@iknite.studio
    * account id: 179359595775
  * Testing Account
    * email: kimboh@iknite.studio
    * account id: 717244548772
  * Artifact Environment
    * email: admin+aws.ossp.artifact@iknite.studio
    * account id: 420136252572


## Logging into the AWS dashboard
To login to the AWS dashboard for a particular project eenvironment do the following:
1) Login into the Main AWS account with your IAM credentials (if you don't have any IAM credentails, you can request them)
  * Use this url to login: https://842675981834.signin.aws.amazon.com/console
1) click on the menu in the top right corner of the AWS console (it should say `username @ 8426-75981834`)
1) Click on "Switch rolw" at the bottom of the menu.
1) A modal will open asking which role you wish to assume. Fill the form with the followin info and click "Switch Role"
  * Account ID: Find the account id for the environment you wish to connect to from the list above.
  * For the IAM role name type `dev`
  * Finally give a memorable display name to the role (something like "Omala Dev Environment (dev role)")


# AWS Account Types
We use the following terminology when referring to the various AWS accounts that we create for managing our projects. Main Account, Environment Account, Artifact Account. Below we describe the use of each account.

## Main Account
* We will have exactly one account of this type.
* Hosts all IAM users (dev accounts)
  * This is the only AWS account in which devs (and other users) will have IAM accounts
* Maintains a dev and admin IAM group (used to assume role into other accounts)
* Bootstrapping:
  * Create root account using the email address `admin+aws.main@iknite.studio`
  * Manually apply the bootstrap-aws-account.yaml cloudformation template
* A Cloudformation template will be applied on commits to main. This should create the groups and the IAM users

## Artifact Account
* Hosts all build artifacts for a project (ECR, s3)
* Hosts all "image-tag" data in systems manager
* Note: This account will just be one of the environment accounts for a project.
* Bootstrapping:
  * No additional bootstrapping needed as this account will also be an environment account



## Environment Accounts
* Maintains 3 roles that can be assumed
  * dev - gives access to devs for monitoring and
    * grant assume role to mainAccount::devs
  * admin -
    * Grant assume role to mainAccount::admins
  * deploy - automated deployments
    * Grant assume role to - GitHub repo OICD provider
* Bootstrapping: [see detailed directions here](environment-accounts/README.md)
