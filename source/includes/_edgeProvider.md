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

Use the following funciton to determine if the website is running inside the Edge application's webview:

```js
function isEdge() {
  return window.navigator.userAgent.indexOf("app.edge") >= 0;
}
```

Assuming this function returns true, the web page can use the following function to acquire the `edgeProvider` instance:

```js
/**
 * Calls the provided callback when the EdgeProvider is ready.
 */
function getEdgeProvider(callback) {
  if (window.edgeProvider != null) {
    callback(window.edgeProvider);
  } else {
    document.addEventListener("edgeProviderReady", function() {
      callback(window.edgeProvider);
    });
  }
}

// If you prefer a promise, just do this:
const edgePromise = new Promise(getEdgeProvider);
```

This code is necessary because the `edgeProvider` object takes some time to appear on the `window` object, so a small wait may be necessary.

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
  const results = await window.edgeProvider.readData(['username', 'authToken'])
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
const currencyCode = await window.edgeProvider.chooseCurrencyWallet(['BCH', 'ETH', 'BTC'])

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
Using nativeAmount:
```javascript
const currencyCode = await window.edgeProvider.chooseCurrencyWallet(['BTC'])
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
    }, 
    orderId: 'cWmFSzYKfRMGrN'
  }
)

// Get the txid of transaction
console.log(edgeTransaction.txid)
```
Using exchangeAmount:
```javascript
const currencyCode = await window.edgeProvider.chooseCurrencyWallet(['BTC'])
const edgeTransaction = await window.edgeProvider.requestSpend(
  [{
  publicAddress: '39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX',
  exchangeAmount: '0.002'
  }],
  {
    metadata: { // Optional metadata to tag this transaction with
      name: 'Bitrefill',
      category: 'Expense:Gift Cards',
      notes: 'Purchase $200 Whole Foods gift card. Order ID 1234567890abcd'
    }, 
    orderId: 'cWmFSzYKfRMGrN'
  }
)

// Get the txid of transaction
console.log(edgeTransaction.txid)
```



#### Request wallet to spend to a URI

```javascript
const currencyCode = await window.edgeProvider.chooseCurrencyWallet(['BTC'])
const edgeTransaction = await window.edgeProvider.requestSpendUri(
  'bitcoin:39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX?amount=1.23456789',
  {
    metadata: { // Optional metadata to tag this transaction with
      name: 'Bitrefill',
      category: 'Expense:Gift Cards',
      notes: 'Purchase $200 Whole Foods gift card. Order ID 1234567890abcd'
    },
    orderId: 'cWmFSzYKfRMGrN'
  }  
)

// Get the txid of transaction
console.log(edgeTransaction.txid)
```
#### Conversion Tracking

All conversions from Crypto to Fiat are tracked generically. However we need to keep track of purchases of crypto from fiat. Upon completion of a transaction please call the conversion endpoing 
```javascript
window.edgeProvider.trackConversion({currencyCode: 'iso:USD', exchangeAmount: 100, orderId: 'cWmFSzYKfRMGrN') // 100 USD 
```

#### Get Transaction History

Get a list of transactions for the currently selected wallet. User will have to confirm before transactions are returned. Method returns an EdgeGetTransactionsResult object. 
```javascript
const transactions = await window.edgeProvider.getTransactions() 
```

The following Javascript Flow types describes the functions available in the `window.edgeProvider` object.

```javascript
const type EdgeProvider = {
  // Set the currency wallet to interact with. This will show a wallet selector modal
  // for the user to pick a wallet within their list of wallets that match `currencyCodes`
  // Returns the currencyCode chosen by the user
  async chooseCurrencyWallet: (currencyCodes: Array<string>) => string

  // Get an address from the user's wallet
  async getReceiveAddress: (options: EdgeGetReceiveAddressOptions) => EdgeReceiveAddress,

  // Request that the user spend to an address or multiple addresses
  async requestSpend: (spendTargets: Array<EdgeProviderSpendTarget>, options?: EdgeRequestSpendOptions) => EdgeTransaction,

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

  // Track a buy conversion for analytics and referral program
  async trackConversion: (options: EdgeTrackConversionOptions) => void
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

const type EdgeProviderSpendTarget = {
  // Public address of destination
  publicAddress: string,

  // Amount in the smallest unit of the currency (ie. satoshis). 
  // If this is specified do not pass exchangeAmount
  nativeAmount?: string,
  
  // Amount specified in the denomination useed by exchange ie., BTC, ETH, BCH
  // If this is specified do not pass nativeAmount
  exchangeAmount?: string
}

const type EdgeMetadata = {
  name?: string,
  category?: string,
  notes?: string,
}

const type EdgeRequestSpendOptions = {
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

  // Unique orderID for this exchange transaction
  orderId?: string
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

const type EdgeTrackConversionOptions = {
  // Currency code of the conversion amount
  // I.e., "btc" or "iso:USD"
  currencyCode: string,

  // Amount of the conversion
  amount: number,

  // Unique orderID
  orderId: string
}

type EdgeMetadata = {
  name?: string,
  category?: string,
  amountFiat?: number
}

type EdgeTransaction = {
  // Amounts:
  currencyCode: string,
  nativeAmount: string,

  // Fees:
  networkFee: string,
  parentNetworkFee?: string,

  // Confirmation status:
  blockHeight: number,
  date: number,

  // Transaction info:
  txid: string,
  ourReceiveAddresses: string[],

  // Core:
  metadata?: EdgeMetadata
}

type EdgeGetTransactionsResult = {
  fiatCurrencyCode: string, // the fiat currency code of all transactions in the wallet. I.e. "iso:USD"
  balance: string, // the current balance of wallet in the native amount units. I.e. "satoshis"
  transactions: Array<EdgeTransaction>
}

```
