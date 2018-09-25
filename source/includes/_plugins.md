# Edge Wallet Plugins

Edge Wallet plugins are a way to include additional functionality into Edge
Wallet on iOS and Android. It is currently used to buy Bitcoin, Ethereum and
Litecoin via Simplex.

The Wallet Plugin API provides a subset of the entire Edge SDK which allows the
plugin to access the currently logged in user's wallet to request sends and
receives.

These pages serve as an introduction on writing your own plugins. You only need
basic web development skills to build your own plugin. If you can write HTML,
CSS and Javascript, you can easily write a plugin and run it on Android and iOS.

## How it works

Edge plugins are just single page HTML files, with all the resources "compiled"
in. That means that the Javascript libraries, stylesheets and whatever else are
all included in one monolithic HTML file.

The code is loaded by the Edge Wallet into a sandboxed webview. A bridge is
created between the webview and the native-app via the
[`edge-libplugin`][libplugin] package.  Using the [`edge-libplugin`][libplugin]
package, plugins can easily:

- Read and write encrypted data
- Request Sends
- Generate receive addresses for coins and tokens

## Creating a plugin

```javascript
git clone https://github.com/EdgeApp/edge-plugin-skeleton
cd edge-plugin-skeleton
npm install
```

To get started you can clone the skeleton project. This already includes
[`edge-libplugin`][libplugin] as a dependency. If you look in the `package.json`
you'll also find the command `edge`. This command is responsible for taking
your code and creating the monolithic HTML file which the native app will load.

Note that each plugin is expected to have a `manifest.json` which will tell the
react native app where and how to display the plugin.

## Load plugin into Edge

First you'll have to have the Edge Wallet building locally. Please follow the
instructions in the Edge Wallet [README][edge-readme].

Once you are able to build Edge Wallet, you can make your plugin a dependency
and add it to the `plugins` array at the bottom of the `package.json` like so:

```
    "devDependencies": {
        ...
        "edge-plugin-simplex": "https://github.com/Airbitz/edge-plugin-simplex.git#c1b81e1",
        "my-edge-plugin": "https://github.com/MyUsername/my-edge-plugin.git"
        ...
    }

...
    "plugins": {
        "buysell": [
            "edge-plugin-simplex",
            "my-edge-plugin"
        ],
        "spend": []
```

At this point you can build Edge Wallet, and use your plugin. If you need to
debug your plugin, you can use Safari to debug iOS and
[Chrome][chrome-debugging] to debug on Android.

## Submit Your Plugin

To submit your plugin for inclusion in Edge Wallet, submit a pull request for
the changes to the `package.json` for `edge-react-gui` repository.

# Edge Plugin API Reference

## PluginWallet object

```javascript
{
    "id": "wallet-id",
    "name": "My Wallet",
    "type": "wallet:bitcoin",
    "currencyCode": "ETH",
    "fiatCurrencyCode": "USD",
}
```

The `PluginWallet` object represents a wallet from the user's Edge account.

| Property         | Type     | Description                 |
|------------------|----------|-----------------------------|
| id               | `String` | Wallet ID                   |
| name             | `String` | Wallet text name            |
| currencyCode     | `String` | Wallet currency code number |
| fiatCurrencyCode | `String` | Wallet currency code number |

## PluginAddress object

```javascript
{
    "encodeUri": "bitcoin://...",
    "address": {
        "publicAddress": "...",
        "segwitAddress": "...",
        "legacyAddress": "..."
    }
}
```

The `PluginAddress` object contains receive addresses for an Edge wallet.

| Property           | Type       | Description                   |
| ------------------ | ---------- | ----------------------------- |
| encodedUri         | `String`   | Receive URI                   |
| address            |            |                               |
| publicAddress      | `String`   | Public address                |
| segwitAddress      | `String`   | Segwit address if supported   |
| legacyAddress      | `String`   | Legacy address if supported   |

## Core

### wallets

```javascript
core.wallets().then(wallets => {
})
```

Returns a promise with a list of the wallets for this account. The success type
is an array of `PluginWallet` objects.

### getAddress

| Param        | Type     | Description                             |
| ---          | ---      | ---                                     |
| walletId     | `String` | Id of the wallet to generate an address |
| currencyCode | `String` | Currency code of wallet                 |

```javascript
core.getAddress('wallet-id', 'BTC').then(publicAddress => {
})
```

Returns a promise with a `PluginAddress` object for the specified wallet.

### writeData

```javascript
let data = JSON.stringify({
  "name": "Jenny",
  "phone": "800-867-5309",
  "age:" 42 
})
await core.writeData("userInfo", data)
```

Securely persist data into the Edge core under this user's account. Only the
current plugin will have access to that data. Data is fully encrypted and
synchronized between devices that the user logs into.

| Param | Type     | Description                      | 
| ---   | ---      | ---                              | 
| key   | `String` | Key string to reference the data | 
| data  | `String` | Value to store                   | 

### readData

```javascript
let data = await core.readData("userInfo")
console.log(data)
```

Read back data from the Edge core under this user's account. Only the current
plugin will have access to that data. Data is fully encrypted and synchronized
between devices that the user logs into.

| Param    | Type     | Description                                   | 
| ---      | ---      | ---                                           | 
| key      | `String` | Key string to reference the data              | 

| Response | Type     | Description                                   | 
| ---      | ---      | ---                                           | 
| data     | `String` | Data stored from a previous call to writeData | 

### clearData

```javascript
core.clearData()
```

Clear all data in the Edge core, for the current plugin.

### debugLevel

```javascript
core.debugLevel (level, text)

// Example
core.debugLevel (0, "Something happened, I don't want to talk about it")
```

Log messages to the Edge core at a particular level.

| Param | Type      | Description                                 | 
| ---   | ---       | ---                                         | 
| level | `integer` | ERROR = 0, WARNING = 1, INFO = 2, DEBUG = 3 | 
| text  | `String`  | Message to log                              | 


## Config

### get

```javascript
config.get(key)

// Example
let value = await config.get("API_KEY")
```

Fetch a configuration value. These are set in the plugins manifest file.  This
is useful to pass in parameters such as API keys and other app secrets.

| Param    | Type     | Description                      | 
| ---      | ---      | ---                              | 
| key      | `String` | Key string to reference the data | 

| Response | Type     | Description                      | 
| ---      | ---      | ---                              | 
| value    | `String` | Data passed in from native app   | 

## UI

### title

```javascript
ui.title(title)

// Example
ui.title("Bob's Awesome Burger");
```

Set the title of the current view. This updates the native apps titlebar.

| Param | Type     | Description     | 
| ---   | ---      | ---             | 
| title | `String` | Title on navbar | 

### showAlert

```javascript
ui.showAlert(title, message)

// Example
ui.showAlert("Creating account", "Please wait...");
```

Launches a native alert dialog. Alert will automatically fade when tapped or
after ~6 seconds unless the showSpinner option is given.

| Param       | Type      | Description                                                                    | 
| ---         | ---       | ---                                                                            | 
| title       | `String`  | Title of alert                                                                 | 
| message     | `String`  | Body of alert                                                                  | 

### hideAlert

```javascript
ui.hideAlert()
```

Hides any currently displaying alert from showAlert

[libplugin]: https://github.com/EdgeApp/edge-libplugin
[edge-readme]: https://github.com/EdgeApp/edge-react-gui
[chrome-debugging]: https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews
