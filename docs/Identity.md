# Identity

## Security Token Service

Allows to grant **limited and temporary access** to AWS resources.  
API:

- `AssumeRole`: Assume role within or cross accounts.
- `AssumeRoleWithSAML`: Return creds for users logged with SAML.
- `AssumeRoleWithWebIdentity`: Return creds for users logged with an IdP. Use **Identity Pools** instead.
- `GetSessionToken`: For MFA, from a user or AWS root user
- `GetFederationToken`: Obtain temporary creds for a federated user.
- `GetCallerIdentity`: Get info about the user making this call.
- `DecodeAuthorizationMessage`: Decode error message when an AWS API is denied.

We need to **define a role** and define **which principals can access the role**.  
Credentials are valid **from 15 min to 1h**.

With MFA, you need to use `GetSessionToken` and use `aws:MultiFactorAuthPresent: true` condition in the policy.

## IAM Authorization Model

**DENY** has priority over **ALLOW**. If nothing is defined, it's **DENY**. 

With **resource policies**, the **union** will be taken.

## Dynamic IAM policies

Dynamic policies **can use variables** such as `${aws:username}`.  
Example: Allow `/home/<user name>` for every user.

## Inline vs Managed Policies

AWS Managed:
- Maintained by AWS
- Good for power users
- Updated for new features

Customer Managed:
- Best practive, re-usable
- Version controlled + rollback

Inline:
- Strict one-to-one relationship between policy and principal
- Policy is deleted if you delete the principal

## Pass a role

You can **grant permissions** to user or services with `iam:PassRole`.  
You can view the role with `iam:GetRole`.  

Role can be passed according to **trust policies** that need to allow `sts:AssumeRole`.
