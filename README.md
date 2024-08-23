# aws-account-manage


# Directory Structure

* /environment-accounts
  * These are boilerplate files for creating a new aws account for a project environment
* /account-configuration
  * These files are for configuring the main AWS account. They contain cloudformation templates that are applied on commits to main.


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
