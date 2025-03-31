# Spectrum Street Epistemology Infrastructure

Infrastructure as Code for sse project.

## Setup

The `developer` role and [AWS SAM CLI](https://aws.amazon.com/serverless/sam/) are required to deploy this project.

### CloudFormation

Push your code or execute the `deploy` script to deploy the infrastructure to development. This project automatically deploys to production when a merge to `master` is made via a pull request.

```bash
npm run deploy
```

### AWS Credentials

To run locally, [AWS CLI](https://aws.amazon.com/cli/) is required in order to assume a role with permission to update resources. Install AWS CLI with:

```brew
brew install awscli
```

If file `~/.aws/credentials` does not exist, create it and add a default profile:

```toml
[default]
aws_access_key_id=<YOUR_ACCESS_KEY_ID>
aws_secret_access_key=<YOUR_SECRET_ACCESS_KEY>
region=us-east-2
```

If necessary, generate a [new access key ID and secret access key](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys).

Add a `developer` profile to the same credentials file:

```toml
[developer]
role_arn=arn:aws:iam::<AWS_ACCOUNT_ID>:role/developer
source_profile=default
mfa_serial=<YOUR_MFA_ARN>
region=us-east-2
```

If necessary, retreive the ARN of the primary MFA device attached to the default profile:

```bash
aws iam list-mfa-devices --query 'MFADevices[].SerialNumber' --output text
```

## Additional Documentation

- [AWS CLI](https://aws.amazon.com/cli/)

- [AWS SAM CLI](https://aws.amazon.com/serverless/sam/)

- [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

- [AWS credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)
