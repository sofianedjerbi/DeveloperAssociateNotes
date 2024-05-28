# Cognito

## Cognito User Pools Precisions

It returns a **JWT** after login.  
You can **block an user if their credentials are compromised** elsewhere.  
**CUP integrates with API Gateway and APL**.

### Lambda Trigger

Synchronously use **Lambda functions to respond to authentication and authorization events**.  
We can **modify the token or the message** sent to the user.  
For pre/post and

### Hosted UI

**Cognito provides an UI** that you can add to your app to handle sign-up and sign-in workflows.  
Integrates with social logins, **custom logo and custom css**.  
To use a custom domain, you have to create an **ACM certificate in us-east-1**.

### Adaptive Authentication

**Block suspicious sign-ins or require MFA** if the login appears suspicious.  
Based on location, ip, device... Integration with CloudWatch logs.

### Decode an ID Token (JWT)

1. Decode base64
2. Json contains metadata

## ALB Authenticate users

You can use:

- **Identity provider**, OIDC compliant
- **Cognito user pools** for socials / corporate identities

HTTPS is needed.

## Cognito Identity Pools Precisions

For **users outside AWS environment** that need access.  
Supports **public providers, user pool, OIDC, guest access**.

It **exchanges 3rd party token with temporary AWS credentials**

### Roles

We can **define default roles** for authenticated / guest users.  
We can define rules to **choose the role based on the user ID**.  
**Policy Variables** allow to partition users access.

`cognito-identity.amazonaws.com:sub` is the user id, you can use it in bucket paths to restrict usage.


