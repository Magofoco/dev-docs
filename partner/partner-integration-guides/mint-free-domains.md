---
title: Free Domain Minting Guide | Unstoppable Domains Developer Portal
description: This guide shows how to mint free domains with your Partner account.
---

## Overview

Unstoppable Domains offers free domain minting to Partners based on specified criteria. Authorized Partners can use the following API endpoints: [Get Free Domains Suggestions](https://docs.unstoppabledomains.com/openapi/reference/#operation/GetDomainsSuggestionsFree), [Reserve Free Domain Name](https://docs.unstoppabledomains.com/openapi/reference/#operation/PostDomainReserve), and [Claim Free Domain](https://docs.unstoppabledomains.com/openapi/reference/#operation/PostOrders).

The diagram below illustrates the general process between a Partner and Unstoppable Domains during the process of minting a free domain.

<figure>

![Success flow for minting free domains with UD](/images/free-domain-minting-success-flow.png '#width=80%;')

<figcaption>Success flow for minting free domains with UD</figcaption>
</figure>

## Step 1: Retrieve Your Reseller ID and Secret API Token

<embed src="/snippets/_reseller-id-location.md" />

## Step 2: Setup Criteria for Free Domains

Partners work with the Unstoppable Domains team to establish the "allowed free TLDs" and "allowed free tiers" criteria for their account.

Then, the "allowed free TLDs" and "allowed free tiers" list is assigned to the Partner's account, and they can only mint a free domain if it matches the tier and has an appropriate domain ending.

The table below shows how the pricing tiers are structured at Unstoppable Domains. Most free domains will be a Tier 7 or Tier 8 domain with a combination of letters and numbers.

<figure>

![Pricing tiers for UD domains](/images/domain-pricing-tiers.png '#width=80%;')

<figcaption>Pricing tiers for UD domains</figcaption>
</figure>

:::info
Domains containing numerals in the name (i.e: tim1, monica95, etc) are discounted by up to 75% of the standard prices, and most free domains fall within this category.
:::

## Step 3: Prepare Your Order Security

Unstoppable Domains uses [Fingerprint](https://fingerprint.com/) to verify free domain orders and catch sophisticated fraudsters. See the [Fingerprint Docs](https://dev.fingerprint.com/docs) for integration guides and how to generate a Fingerprint Visitor ID for your users.

:::info
Fingerprint provides solutions to uniquely and accurately identify users through browser fingerprinting technologies. The `Visitor ID` generated by Fingerprint is a unique hash used to identify the device of your users.
:::

### Integrate Fingerprint Key Generation

Unstoppable Domains provided an API endpoint to fetch Fingerprint public keys, which are needed for generating the `Visitor ID`. Send a `POST` request to the [Get Fingerprint Public Key](https://docs.unstoppabledomains.com/openapi/reference/#operation/PostSecurityFingerprintjsKeys) endpoint. Here is the URL for our API environments:

Sandbox Environment:

```bash
https://api.ud-sandbox.com/api/v2/resellers/{PARTNER_RESELLERID}/security/fingerprintjs/keys
```

Production Environment:

```bash
https://unstoppabledomains.com/api/v2/resellers/{PARTNER_RESELLERID}/security/fingerprintjs/keys
```

:::info
The `PARTNER_RESELLERID` path parameter is the same one you retrieved from your partner account earlier.
:::

Here's a sample response from the [Get Fingerprint Public Key](https://docs.unstoppabledomains.com/openapi/reference/#operation/PostSecurityFingerprintjsKeys) endpoint:

```json
{
    "key": "qwerty12345"
}
```

:::info
The endpoint returns a different key when called to avoid rate limitations. Also, the generated Fingerprint `Visitor ID` will always be the same despite the different public keys being returned.
:::

### Integrate Fingerprint Subdomain

Unstoppable Domains has a dedicated subdomain for Fingerprint verification to improve integration and user identification. You need to set the `endpoint` property in your Fingerprint initialization to our subdomain.

Sandbox Environment:

```bash
https://fp.ud-sandbox.com
```

Production Environment:

```bash
https://fp.unstoppabledomains.com
```

Here's a sample code snippet:

```javascript
// Initialize the agent at application startup.
const fpPromise = import('https://fpcdn.io/v3/{FINGERPRINT_API_KEY}')
  .then(FingerprintJS => FingerprintJS.load({
    endpoint: 'https://fp.unstoppabledomains.com'
  }));
```

### Tag Your Fingerprint Request

Unstoppable Domains requires you to [tag your requests](https://dev.fingerprint.com/docs/quick-start-guide#tagging-your-requests) with the `linkedId` property and your `ResellerID` when generating the `Visitor ID` for your users. Here's a sample code snippet:

```javascript
// Get the visitor identifier when you need it.
fpPromise
  .then(fp => fp.get({linkedId: '{PARTNER_RESELLERID}'}))
  .then(result => console.log(result.visitorId));
```

The code snippet below shows how to completely integrate Fingerprint verification into your free domain flow:

```javascript
<script>
  // Initialize the agent at application startup.
  const fpPromise = import('https://fpcdn.io/v3/{FINGERPRINT_API_KEY}')
    .then(FingerprintJS => FingerprintJS.load({
      endpoint: 'https://fp.unstoppabledomains.com'
    }));

  // Get the visitor identifier when you need it.
  fpPromise
    .then(fp => fp.get({linkedId: '{PARTNER_RESELLERID}'}))
    .then(result => console.log(result.visitorId));
</script>
```

:::info
The `FINGERPRINT_API_KEY` placeholder in the code snippet above should be replaced with the public key gotten from the `Get Fingerprint Public Key` endpoint.
:::

Here's a sample response from the Fingerprint integration:

```json
{
    "requestId": "1669648408882.1kcr2m",
    "confidence": {
        "score": 0.999170944804056
    },
    "meta": {
        "version": "v1.1.971+673281e1"
    },
    "visitorFound": true,
    "visitorId": "qwerty12345"
}
```

:::info
The `confidence.score` field is a floating-point number between `0` and `1` that represents the probability of accurate identification. Unstoppable Domains will only accept a `Visitor ID` with a confidence score of at least 90% (0.9).
:::

## Step 4: Prepare Request Body

When making an order, the body must contain information about your domain order in JSON format with the following structure:

```javascript
{
  "payment": {
    "method": "free"
  },
  "security": [
    {
      "type": "fingerprintjs",
      "identifier": string // Fingerprint Visitor ID of the user minting the domain
    }
  ],
  "domains": [
    {
      "name": string, // domain name you are minting
      "ownerAddress": string, // wallet address to mint the domain to
      "email": string, // UD email address to link the domain to
      "phone": string, // user's phone number
      "resolution": object, // predefined records to mint the domain with
      "resellerIdentityKey": string // domain reservation ID
    }
  ]
}
```

* `payment`: A key-value dictionary with payment information about the order:
    * `method`: (string) The payment method the API should create. The value should be `"free"` for free domains.
* `security`: (array) An array with information about the order security:
    * `type`: The order security method. The value should be `"fingerprintjs"` for Fingerprint verification.
    * `identifier`: The Fingerprint Visitor ID of the user minting the domain.
* `domains`: (array) An array with information about the domains you want to purchase:
    * `name`: The domain name you want to purchase. This parameter is required.
    * `ownerAddress`: The wallet address the domain should be minted to. This parameter is required.
    * `email`: The email address the domain should be linked to after purchase. The user can mint the domain from their UD dashboard later. This parameter is required.
    * `phone`: The phone number of the user minting the free domain. This parameter is optional.
    * `resolution`: A key-value pair of resolution records to configure for the domain after minting. See the [Records Reference](/developer-toolkit/reference/records-reference.md) guide for supported key values. This parameter is optional.
    * `resellerIdentityKey`: The domain reservation ID. This parameter is required if you reserved the domain before minting.

:::info
You need to provide either the `email` or `phone` parameter in every order request. You can also provide both parameters in your request.
:::

## Step 5: Prepare Authorization Headers

<embed src="/snippets/_auth-headers-preparation.md" />

## Step 6: Use the Orders Endpoint

<embed src="/snippets/_orders-endpoint-usage.md" />

## Example

Here is an example request to mint a free domain with the following details:

| Field Name | Value |
| - | - |
| Security Type | Fingerprint |
| Fingerprint Visitor ID | qwerty12345 |
| Domain Name | partner-test-67687986466875.wallet |
| Customer Wallet Address | 0x6EC0DEeD30605Bcd19342f3c30201DB263291589 |
| Customer Email | sandbox-test@unstoppabledomains.com |
| Predefined Domain Records | {"crypto.ETH.address": "0x6EC0DEeD30605Bcd19342f3c30201DB263291589", "crypto.BTC.address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"} |

```bash Request
curl --location --request POST 'https://api.ud-sandbox.com/api/v2/resellers/{PARTNER_RESELLERID}/orders/' \
--header 'Authorization: Bearer {SECRET_API_TOKEN}' \
--header 'Content-Type: application/json' \
--data-raw '{
  "payment": {
    "method": "free"
  },
  "security": [
    {
      "type": "fingerprintjs",
      "identifier": "qwerty12345"
    }
  ],
  "domains": [
    {
      "name": "partner-test-67687986466875.wallet",
      "ownerAddress": "0x6EC0DEeD30605Bcd19342f3c30201DB263291589",
      "email": "sandbox-test@unstoppabledomains.com",
      "resolution": {
          "crypto.ETH.address": "0x6EC0DEeD30605Bcd19342f3c30201DB263291589",
          "crypto.BTC.address": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh"
      }
    }
  ]
}'
```

```json Response
{
    "orderNumber": "78023",
    "total": 0,
    "payment": {
        "method": "free"
    },
    "items": [
        {
            "domain": {
                "id": 966210,
                "name": "partner-test-67687986466875.wallet",
                "ownerAddress": null,
                "resolver": null,
                "resolution": {},
                "blockchain": "MATIC",
                "registryAddress": "0x2a93c52e7b6e7054870758e15a1446e769edfb93",
                "networkId": 80001,
                "freeToClaim": true,
                "node": "0xeb55b00b50af99d760b002c6d855532658c332d059dcd69eb167fec5ec97d0fa"
            },
            "mintingTransaction": {
                "id": 44958,
                "type": "MaticTx",
                "operation": "MintDomain",
                "statusGroup": "Pending",
                "hash": null,
                "blockchain": "MATIC",
                "blockExplorerUrl": null
            }
        }
    ]
}
```

:::success congratulations!
You have successfully minted a free domain with the Partner API.
:::

## Considerations

The following considerations apply to minting free domains:

* The Reseller ID will be allowed to provide specified domain endings for free.
* If the Reseller ID doesn't have an allowance to provide free domains, then they will not be permitted to mint free domains.
* The domain is available for sale (e.g., it's not restricted, protected, trademarked, claimed, etc.)
* The domain does not have a custom price set.
* It has 8+ characters, at least one letter, and one number.
* If a wallet or email already has a free domain, then a second free domain is not permitted.
* The Fingerprint Visitor ID provided must be generated within the past 30 seconds and have a confidence score of at least 90%.

<embed src="/snippets/_discord.md" />
