# CloudFront

## Caching

Cache lives at CloudFront **Edge Locations**.  
It identity object with the **cache key**.

## Cache Key

An unique id for each object in the cache.  
By default, it is the **hostname + resource portion of the URL**.

*Ex: website.com/content/static.html*

## Cache Policy

Specify what you want to add to the cache key (cache precision):

- **HTTP Headers:** None - Whitelist
- **Cookies:** None - Whitelist - Include All Except - All
- **Query Strings:** None - Whitelist - Inclide All Except - All

All headers, cookies and query strings are **automatically included in origin requests**.

## Origin Request Policy

Speficy values to include in origin requests **without including them in the Cache Key**.

## Cache Invalidations

Used to **update content** by **refreshing the cache**.  
You can invalidate with a path: `/path/*`

## Cache Behaviors

Configure different settings for a **given URL path**.  
It allows to apply **Cache Policies to a subset of files**.  
You can also use behaviors to redirect to **multiple origin depending on the path**.

*Ex: Only accept a request when Signed Cookies are present.*

## CloudFront with external source

For custom (ALB / EC2) caching, you must **allow public access from CloudFront Edge Locations IPs**

## CloudFront Signed URL / Signed Cookies

**Restrict an URL to trusted visitors.**  
We attach a policy to the URL / Cookie with:

- URL Expiration
- IP Range access
- Trusted Signers *(Accounts that can create signed content)*

URL is for **individual files**, Cookie is used for **many files**.

*Ex: Distribute paid shared content to premium users*

## Origin Groups

An origin group contains **multiple origins**.  
A **failover** will occur if the primary origin don't respond with OK to CloudFront.  

*Ex: Multi-region failover S3 buckets to protect against region outrages*  
*Ex: Deliver static content from S3 and other content from an ALB*

## Field Level Encryption

Uses asymmetric encryption to **encrypt data at edge locations** with a public key.  
The webserver **uses the private key to decrypt the data**.  
It protect the data even if HTTPS is compromised.

## Real Time Logs

Send real-time **requests to Kinesis Data Streams**.  
Allow you to choose a **Sampling Rate** (percentage of requests to receive) and a **Cache Behavior**.


