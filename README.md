# aws-sso-login-with-sdk-support
Simple shell script that you can use after logging in to the CLI via AWS SSO to store temporary role credentials in your ~/.aws/credentials file so that your AWS SDK commands also work locally


## How it works

The latest [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) supports [AWS SSO](https://aws.amazon.com/single-sign-on/), so that you can issue a `aws sso login` ([see docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)) command to log in to AWS SSO and be able to use AWS CLI commands locally with temporary credentials. No need to manage long-lived IAM users or access keys. Yayyyyyy!

However, `aws sso login` does not store your local credentials in the usual format or location in `~/.aws/credentials`, so if you try to use the AWS SDKs (or other tools) that require a profile and valid credentials in the `~./aws/credentials` file, the command will fail. 

The solution is to create an IAM role with the SDK permissions you need and give permission to your AWS SSO user to assume that role. Then, after logging in via `aws sso login`, you can use an AWS CLI `aws sts assume-role` command to retrieve temporary credentials for that role and write them to your `~/.aws/credentials` file. 

Below, is a simple shell script that you can run after `aws sso login` that does this for you.

There might be an easier way to do this, but this is what I landed on for now :)

## Usage

1. Install the AWS CLI v2
2. Set up AWS SSO, get things working :)
3. Install `jq` (https://stedolan.github.io/jq/) (needed by the shell script to parse response from **sts assume-role**)
4. From your local terminal, run `aws sso login`; complete the SSO login process to configure your local CLI for SSO
5. Run the shell script below to call `aws sts assume-role`, retrieve temporary access keys, and write them to your credentials file

## Assume role script

```
# get-aws-creds.sh

ROLE=$(aws sts assume-role \
  --role-arn arn:aws:iam::123456890:role/some_iam_role_to_assume \
  --role-session-name LaptopForMatWerber)
 
# Parse the JSON response from sts assume-role
ACCESS_KEY=$(echo $ROLE | jq -r '.Credentials.AccessKeyId')
SECRET_KEY=$(echo $ROLE | jq -r '.Credentials.SecretAccessKey')
TOKEN=$(echo $ROLE | jq -r '.Credentials.SessionToken')
 
# write the credentials to your ~/.aws/credentials file
cat >~/.aws/credentials <<EOL
[default]
aws_access_key_id = ${ACCESS_KEY}
aws_secret_access_key = ${SECRET_KEY}
aws_session_token = ${TOKEN}
```
