# Before getting started
This bootstrapping process assumes deployments are managed with GitHub Actions. Before you set up the first AWS account (environment) for a project, create the project's GitHub repository under the `Xenang-Technologies` organization. The CloudFormation template used to bootstrap accounts gives that repository's GitHub Actions access to the `deploy` role.

New accounts are created from the main account (124920735437), which is the management account of our AWS Organization.

# Bootstrapping a new Environment Account
1. Create a new Gmail address for the account. Use a name that identifies the project and environment, e.g. `<project>.<environment>.aws@gmail.com`. Save its password and set up its 2-step verification in the shared Bitwarden account.
1. In the main account, go to AWS Organizations → AWS accounts → Add an AWS account → Create an AWS account:
   * AWS account name: `<project>-<environment>`
   * Email address of the account's owner: the Gmail address from step 1
   * IAM role name: leave the default, `OrganizationAccountAccessRole`

   Wait for the account to be created, note its account id, and move it into the project's organizational unit if there is one.
1. Set up the account's root user. Accounts created through Organizations have no root password, so on the AWS sign-in page choose "Root user", enter the account's Gmail address and use "Forgot password" to set one. Save the password in the shared Bitwarden account, then sign in and add MFA, also using the shared Bitwarden account for the TOTP codes.
1. Switch into the new account: from the main account console, use "Switch role" with the new account id and the role `OrganizationAccountAccessRole`. This role is only assumable by main-account principals with administrator access (e.g. the main account root user); alternatively, sign in as the new account's root user.
1. In the new account, change the region to `us-east-1` and create a CloudFormation stack from [bootstrap-aws-account.yaml](bootstrap-aws-account.yaml):
   * Stack name: `bootstrap-aws-account`
   * `RepositoryName`: the name of the project repository that deploys into this account, without the organization (e.g. `my-project`)
   * `GitHubOrgUser`: `Xenang-Technologies` (default)
   * `MainAccountId`: `124920735437` (default)
   * `OIDCAudience`: `sts.amazonaws.com` (default)

   Acknowledge that the stack creates IAM resources and create it.
1. Update `cd/environments.yaml` in the project repo to include this new environment and set its account id.
1. Add the new account id and its Gmail address to the [README.md](../README.md#current-aws-accounts) in this repo.
1. [Activate access to the Billing and Cost Management console](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/control-access-billing.html#ControllingAccessWebsite-Activate).

The bootstrap template creates these resources in the account:
* A GitHub OIDC provider and the `deploy` role, which GitHub Actions in the project repository can assume. It has permissions to deploy the entire environment into the account. Both the classic (`repo:<org>/<repo>:...`) and immutable (`repo:<org>@<id>/<repo>@<id>:...`) OIDC subject formats are accepted.
* The `dev` and `admin` roles, which users in the main account can assume.

# Post bootstrapping setup
After performing the above process, users in the main account's `devs` and `admins` groups can switch to the `dev` role in the account, and GitHub Actions can assume the `deploy` role.

When bootstrapping a new project, configure the artifact account after you've bootstrapped the first environment:
1. Copy the contents of the [example](example) directory into your project repo.
1. Update `cd/artifact-account/config.yaml`: set `aws_account_id` to the artifact account's id and list the project's environment account ids under `environment_account_ids`.
1. Update `cd/artifact-account/cloudformation-template.yaml`. This CloudFormation template provisions the resources that store a project's build artifacts. The example template creates an ECR repository and an S3 bucket.
1. Commit to `main`. The `.github/workflows/artifact-account.yaml` workflow deploys the template to the artifact account.
1. Request SES production access if the project sends email via SES. See the template below.


### AWS SES production access request template
Note: This template is a work in progress. Please update it if you get feedback from AWS when requesting production access.
> Hello,
>
> Thank you for your response.
> 
> We are requesting production access to Amazon SES for the sole purpose of sending One-Time Passwords (OTPs) as part of our multifactor authentication (MFA) workflow. These OTPs are only sent to test users in a staging environment—not to real customers—and are triggered during login attempts.
> 
> Here are further details about our email-sending process:
> 
>     Frequency: Emails are sent only when a test user initiates a login and explicitly requests to receive the OTP via email. The sending volume is low and depends on internal development and QA activity.
> 
>     Recipient List Maintenance: All recipient email addresses belong to internal team members or authorized testers. There are no mailing lists involved.
> 
>     Bounce and Complaint Handling: We monitor bounce and complaint metrics through Amazon SNS notifications integrated with SES. We act promptly on any issues to ensure healthy sending practices.
> 
>     Email Content: The email is a simple plaintext message containing the OTP and its expiration time. Below is an example:
> 
>     Your one-time password is: 123456
>     This code will expire in 5 minutes.
> 
> We have already verified our sending domain and follow best practices for email authentication (SPF, DKIM, etc.).

