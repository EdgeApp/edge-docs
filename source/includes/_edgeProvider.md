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

The following Javascript Flow types describes the functions available in the `window.edgeProvider` object.

```javascript
const type EdgeProvider = {
  // Set the currency wallet to interact with. This will show a wallet selector modal
  // for the user to pick a wallet within their list of wallets that match `currencyCodes`
  // Returns the currencyCode chosen by the user
  async setCurrentWallet: (currencyCodes: Array<string>) => string

  // Get an address from the user's wallet
  async getReceiveAddress: (options: EdgeGetReceiveAddressOptions) => EdgeReceiveAddress,

  // Request that the user spend to an address or multiple addresses
  async requestSpend: (options: EdgeRequestSpendOptions) => EdgeTransaction,

  // Request that the user spend to a URI
  async requestSpendUri: (options: EdgeRequestSpendUriOptions) => EdgeTransaction,

  // Write data to user's account. This data is encrypted and persisted in their Edge
  // account and transferred between devices
  async writeData: (data: EdgeWriteDataObject) => void,

  // Read data back from the user's account. This can only access data written by this same plugin
  async readData: (key: string) => string,

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
  // Specify the currencyCdoe to spend to this URI. Required for spending tokens
  currencyCode: string,
  metadata?: EdgeMetadata,
  networkFeeOption?: 'low' | 'standard' | 'high',
  spendTargets?: Array<EdgeSpendTarget>,

  // If true, do not allow the user to change the amount to spend
  lockInputs?: boolean,

  // Do not broadcast transaction
  signOnly?: boolean,

  // Additional identifier such as a payment ID for Monero or destination tag for Ripple/XRP
  uniqueIdentifier?: string,
}

const type EdgeRequestSpendOptions = {
  // Specify the currencyCdoe to spend to this URI. Required for spending tokens
  currencyCode: string,

  // BIP21 style string to parse and spend to.
  // ie. "bitcoin:39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX?amount=0.123&label=Bitrefill&message=GiftCards"
  spendUri: string,

  // Do not broadcast transaction
  signOnly?: boolean
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

const type EdgeWriteDataObject = {
  key: string,
  value: string
}
```

### Examples


#### Create and retrieve an authToken

A typical use case for the EdgeProvider is for a website to generate a unique authToken for the user
and store it in their encrypted wallet. On subsequent logins, the website can attempt to read back the encrypted token and use it to authenticate the user.

```javascript
// Check if running inside Edge
if (typeof window.edgeProvider === 'Object') {
  let authToken
  try {
    authToken = await window.edgeProvider.readData('myAuthToken')
  } catch (e) {
    console.log(e)
  }

  if (!authToken) {
    // Generate a random value to store in the wallet
    authToken = window.randomBytes(32).toString('hex')
    await window.edgeProvider.writeData({
      myAuthToken: authToken
    })
  }
  // Send authToken to website to authenticate user or create a new account for them
}
```

#### Get an address from wallet

```javascript
// Check if running inside Edge
if (typeof window.edgeProvider === 'Object') {
  try {
    const currencyCode = await window.edgeProvider.setCurrentWallet(['BCH', 'ETH', 'BTC'])
    const edgeReceiveAddress = await window.edgeProvider.getReceiveAddress({
      metadata: {
        name: 'Wyre',
        category: 'Exchange:Buy ETH',
        notes: 'Purchased 2 ETH for $400 USD on 2019-01-01. Order ID 1234567890abcd'
      }
    })

    // Use the address
    console.log(edgeReceiveAddress.publicAddress)
  } catch (e) {
    console.log(e)
  }
```

#### Request wallet to spend to an address

```javascript
// Check if running inside Edge
if (typeof window.edgeProvider === 'Object') {
  try {
    const currencyCode = await window.edgeProvider.setCurrentWallet(['BTC'])
    const edgeTransaction = await window.edgeProvider.requestSpend({
      currencyCode,
      spendTargets: [
        {
          publicAddress: '39LPRaWgum1tPBsxToeydvYF9bbNAUdBZX',
          nativeAmount: '123456789'
        }
      ],
      metadata: {
        name: 'Bitrefill',
        category: 'Expense:Gift Cards',
        notes: 'Purchase $200 Whole Foods gift card. Order ID 1234567890abcd'
      }
    })

    // Get the txid of transaction
    console.log(edgeTransaction.txid)
  } catch (e) {
    console.log(e)
  }
```
