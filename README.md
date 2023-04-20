# Serverless Architecture

![Serverless Architecture](./resources/serverless-architecture.png).

# startup-sample-project-aws-serverless-OIDC
Lambda serverless app meant to accelerate teams onboarding to the BC Gov SEA AWS space.
This repository use [Github OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services) to authenticate directly to AWS assuming an IAM role.

## Authentication architecture
![](resources/GitHub-OIDC_arch.png)

## Setup
- Fork this repo
- Enable github actions
#### Github Variables
The GitHub actions require the following github variables:
  - `LICENSEPLATE` is the 6 character alphanumeric string  associated with your project set e.g. `abc123` (Note: due to AWS constrains it is required the first characters is a letter)
  - `S3_BACKEND_NAME` is the name of the S3 Bucket name used to store the Terraform state.
  - `TERRAFORM_DEPLOY_ROLE_ARN` This is the ARN of IAM Role used to deploy resources through the Github action authenticate with the GitHub OpenID Connect. You also need to link that role to the correct IAM Policy.

    - To access the `TERRAFORM_DEPLOY_ROLE_ARN` you need to create it beforehand manually. To create it you need can use this example of thrust relationship :
    - The [minimal access policy can be found here](resources/deployement-policy.json) (Note: in the policy examples `<account-id>` has to be replaced by the AWS account id of the account where you are deploying the sample app)
    - The role requires to have the following trust relationship set up  (Note: as before `<account-id>` has to be replaced by the AWS account id of the account where you are deploying the sample app)
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<accound_id>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:<Github_organization>/<repo_name>:ref:refs/heads/<Your_branch>"
                },
                "ForAllValues:StringEquals": {
                    "token.actions.githubusercontent.com:iss": "https://token.actions.githubusercontent.com",
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
  ```

  - Once the app has been built, you should be able to log into AWS with your IDIR account (2FA may be implemented). Once logged in AWS search for Cloudfront and then click on Distributions (If you can not see it click the hamburger on the top left corner). The Distributions dashboard shows the Domain name url, you can use that url to open a browser session and load the app.


#### Pipeline
The github actions will trigger on a pull request creation and merge.
- Creating a pull request will run a `terraform plan` and outline everything that will be deployed into your AWS accounts, but will not create anything. Please review the plan to verify that all is correct.
- Merging into `main` will run a `terraform apply` and your AWS assets will be deployed in the `dev` accounts.

NOTE: make sure you are creating pull requests/ merging within your fork


#### Testing
For how to use the test associated to this project, please check the README file under `functional-tests`


## Testing Thanks

Thanks to BrowserStack for Testing Tool support via OpenSource Licensing ![BrowserStack](docs/resources/browserstack-logo-white-small.png)


