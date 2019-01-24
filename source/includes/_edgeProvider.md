# Edge Wallet Provider

Edge Wallet Provider is a method for external webpages to access the functionality
of a logged in user's Edge account while running inside a webview of Edge Wallet on 
iOS and Android. It's intended use is to provide the wallet additional functionality
such as buying and selling cryptocurrency by automating the sending and receiving
of funds from exchanges.

## How it works

Edge Wallet can launch an embedded browser for a website and provide a globally
accessible `window.edgeProvider` object to the website. Various functions are
provided by this object allowing the website to access the user wallet.

### Examples

The following examples illustrate the 3 basic use cases of the `edgeProvider` API for integration
with an exchange service

#### Check if edgeProvider object exists

Website should check first for the existence of the `window.edgeProvider` object to determine
the availability of the Edge Provider API

```javascript
// Check if running inside Edge
if (typeof window.edgeProvider === 'object') {
  // Continue with using window.edgeProvider
}
```

#### Create and retrieve an authToken

A typical use case for the EdgeProvider is for a website to generate a unique authentication
token for a user and store it in their encrypted wallet. The creation is done at account creation
time instead of asking the user for a password. Upon subsequent logins, the website can attempt
to read back the encrypted username and auth token and use it to authenticate the user
instead of asking for a username and password.

```javascript
let authToken
let username
try {
  const results = await window.edgeProvider.readData('username', 'authToken')
  authToken = results.authToken
  username = results.username
} catch (e) {
  console.log(e)
}

if (username && authToken) {
  // Send username and authToken to website to authenticate user
} else {
  // Website should launch into account creation mode. At account creation, website
  // should create a unique authentication token for the user as an alternative to
  // a password. Once created, save the username and auth token using
  // saveAccountCredentials function below.
}

const saveAccountCredentials = (username, authToken) => {
  await window.edgeProvider.writeData({
    username,
    authToken
  })
}
```

#### Get an address from wallet

```javascript
const currencyCode = await window.edgeProvider.chooseCurrentWallet(['BCH', 'ETH', 'BTC'])

// Next line assumes the user chose an ETH wallet
const edgeReceiveAddress = await window.edgeProvider.getReceiveAddress({
  metadata: { // Optional metadata to tag incoming transactions with
    name: 'Wyre',
    category: 'Exchange:Buy ETH',
    notes: 'Purchased 2 ETH for $400 USD on 2019-01-01. Order ID 1234567890abcd'
  }
})

// Use the address
console.log(edgeReceiveAddress.publicAddress)
```

#### Request wallet to spend to an address

```javascript
const currencyCode = await window.edgeProvider.chooseCurrentWallet(['BTC'])
const edgeTransaction = await window.edgeProvider.requestSpend(
  [{
  publicAddress: '39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX',
  nativeAmount: '123456789'
  }],
  {
    metadata: { // Optional metadata to tag this transaction with
      name: 'Bitrefill',
      category: 'Expense:Gift Cards',
      notes: 'Purchase $200 Whole Foods gift card. Order ID 1234567890abcd'
    }
  }
)

// Get the txid of transaction
console.log(edgeTransaction.txid)
```

#### Request wallet to spend to a URI

```javascript
const currencyCode = await window.edgeProvider.chooseCurrentWallet(['BTC'])
const edgeTransaction = await window.edgeProvider.requestSpendUri(
  'bitcoin:39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX?amount=1.23456789',
  {
    metadata: { // Optional metadata to tag this transaction with
      name: 'Bitrefill',
      category: 'Expense:Gift Cards',
      notes: 'Purchase $200 Whole Foods gift card. Order ID 1234567890abcd'
    }
  }  
)

// Get the txid of transaction
console.log(edgeTransaction.txid)
```

The following Javascript Flow types describes the functions available in the `window.edgeProvider` object.

```javascript
const type EdgeProvider = {
  // Set the currency wallet to interact with. This will show a wallet selector modal
  // for the user to pick a wallet within their list of wallets that match `currencyCodes`
  // Returns the currencyCode chosen by the user
  async chooseCurrentWallet: (currencyCodes: Array<string>) => string

  // Get an address from the user's wallet
  async getReceiveAddress: (options: EdgeGetReceiveAddressOptions) => EdgeReceiveAddress,

  // Request that the user spend to an address or multiple addresses
  async requestSpend: (spendTargets: Array<EdgeSpendTarget>, options?: EdgeRequestSpendOptions) => EdgeTransaction,

  // Request that the user spend to a URI
  async requestSpendUri: (uri: string, options?: EdgeRequestSpendOptions) => EdgeTransaction,

  // Write data to user's account. This data is encrypted and persisted in their Edge
  // account and transferred between devices
  async writeData: (data: { [key: string]: string }) => void,

  // Read data back from the user's account. This can only access data written by this same plugin
  // 'keys' is an array of strings with keys to lookup.
  // Returns an object with a map of key value pairs from the keys passed in
  async readData: (keys: Array<string>) => Object,

  // Sign a message using a public address from the current wallet
  async signMessage: (options: EdgeSignMessageOptions) => EdgeSignedMessage,
}

// ******************************************************
// Flow types for all method parameters and return values
// ******************************************************
const type EdgeReceiveAddress = {
  publicAddress: string,
  segwitAddress?: string,
  legacyAddress?: string
}

const type EdgeGetReceiveAddressOptions = {
  // Metadata to tag these addresses with for when funds arrive at the address
  metadata?: EdgeMetadata
}

const type EdgeSpendTarget = {
  // Public address of destination
  publicAddress: string,

  // Amount in the smallest unit of the currency (ie. satoshis)
  nativeAmount: string
}

const type EdgeMetadata = {
  name?: string,
  category?: string,
  notes?: string,
}

const type EdgeRequestSpendOptions = {
  // Specify the currencyCode to spend to this URI. Required for spending tokens
  currencyCode?: string,

  // This overrides any parameters specified in a URI such as label or message
  metadata?: EdgeMetadata,
  networkFeeOption?: 'low' | 'standard' | 'high',

  // If true, do not allow the user to change the amount to spend
  lockInputs?: boolean,

  // Do not broadcast transaction
  signOnly?: boolean,

  // Additional identifier such as a payment ID for Monero or destination tag for Ripple/XRP
  // This overrides any parameters specified in a URI
  uniqueIdentifier?: string,
}

const type EdgeSignMessageOptions = {
  // Public address from the current wallet to use to sign the message
  publicAddress: string,

  // Hex encoded message to sign
  messageHex: string
}

const type EdgeSignedMessage = {
  // Public key used to sign message
  publicKey: string,

  // Hex encoded signature
  signedMessage: string
}

```

