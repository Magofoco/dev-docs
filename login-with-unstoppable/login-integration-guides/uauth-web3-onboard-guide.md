---
title: Web3 Onboard Guide for Login with Unstoppable | UD Developer Portal
description: This integration guide is intended for a custom @uauth/js integration, with ethereum provider, using the web3 onboard library.
showNextButton: false
---

# Web3 Onboard Guide: Login with Unstoppable

:::warning BNC-onboard
`@uauth/web3-onboard` has been deprecated in favor of the `@web3-onboard/uauth` module and may no longer be supported. For new Web3-Onboard integrations, see the latest [Web3-Onboard Guide](/login-with-unstoppable/login-integration-guides/web3-onboard-guide.md) For integrations using `bnc-onboard`, see the [BNC Onboard Guide](/login-with-unstoppable/login-integration-guides/bnc-onboard-guide.md).
:::

This is the basic installation guide for the `web3-onboard` library and is best used for single page applications (SPAs). For more information about this library, please see the [associated github repo](https://github.com/unstoppabledomains/uauth/tree/main/packages/web3-onboard).

<figure>

![Web3 Onboard with Unstoppable Domains](/images/login-selection-web3-onboard.png '#width=70%')

<figcaption>Web3 Onboard with Unstoppable Domains</figcaption>
</figure>

## Step 1: Install the Libraries

Install with `yarn` or `npm`.

```sh yarn
yarn add @uauth/web3-onboard @uauth/js @web3-onboard/core
```

```sh npm
npm install --save @uauth/web3-onboard @uauth/js @web3-onboard/core
```

## Step 2: Configure the `web3-onboard` Library

```javascript
import Onboard from '@web3-onboard/core'
import injectedModule from '@web3-onboard/injected-wallets'
import UAuth from '@uauth/js'
import uauthBNCModule from '@uauth/web3-onboard'

const uauth = new UAuth({
  clientID: process.env.REACT_APP_CLIENT_ID!,
  redirectUri: process.env.REACT_APP_REDIRECT_URI!,
  fallbackIssuer: process.env.REACT_APP_FALLBACK_ISSUER!,
  scope: 'openid wallet',
})

const uauthBNCOptions = {
  uauth: uauth,
  walletconnect: {
    infuraId: process.env.REACT_APP_INFURA_ID!,
  },
}

const uauthModule = uauthBNCModule(uauthBNCOptions)

const onboard = Onboard({
    wallets: [injected, uauthModule],
    ...
  },
})
```

:::info
Because pop-ups are a more integration friendly approach, the `@uauth/web3-onboard` library now uses them by default. If you want the "old" redirect functionality, you need to set `shouldLoginWithRedirect: true` in your `uauthBNCOptions` and create a callback page.
:::

## Step 3: Login with Unstoppable

Once configured, the `web3-onboard` library can be used normally.

```javascript
async function handleLogin() {
  const connectedWallets = await onboard.connectWallet()
}

async function handleLogout() {
  uauth.logout()
  onboard.disconnectWallet({label: 'Unstoppable'})
}
```

<embed src="/snippets/_login-paths-next.md" />