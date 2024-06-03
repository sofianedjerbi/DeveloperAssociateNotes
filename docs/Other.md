# Other Services

## AWS Private Certificate Authority

Allow you to create **private** certificates.  
Works with any service that have integration with ACM.

*Use-case: Encrypted TLS, Crypto signing code, Authenticate users etc.*

## AWS AppConfig

Configure, validate and deploy **dynamic configuration**.  
Like **dynamic environment variables**.
Supports rollback.

## CloudWatch Evidently

**Validate new features by serving them to a few % of users**.  
- **Launches:** Enable and disable features for a subset of users
- **A/B Testing:** Compare multiple versions of the same feature.
- **Overrides:** Pre-define a variation for specific users.

## Route 53 - Traffic Flow

An UI that **manage zones**. It is able to show the **geoproximity** map.  
It is saved into a **traffic policy**.

## Burst Balances

When you go under a certain quota in a service (CPU / IOPS), you gain **burst balances that allow you to go over this quota**.  

*Note: You lose your balance when creating new instances*
