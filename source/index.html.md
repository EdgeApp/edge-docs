---
title: AirbitzCore API/SDK Reference

language_tabs:
  - javascript: Javascript
  - objective_c: Objective C
  - java: Java/Android

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---


# Airbitz Plugin API


Airbitz plugins are a way to include additional functionality into the Airbitz Bitcoin Wallet. They are currently used to buy and sell bitcoin through Glidera, and purchase discounted Starbucks and Target cards through Foldapp.

The plugin API provides a subset of the entire Airbitz SDK which allows the plugin to access the currently logged in user's wallet to request sends and receives.

These pages serve as an introduction on writing your own plugins. You only need basic web development skills to build your own plugin. If you can write HTML, CSS and Javascript, you can easily write a plugin and run it on Android and iOS.

# Creating a plugin

Airbitz plugins are just single page HTML files, with all the resources compiled in. That means that the javascript libraries, stylesheets and whatever else are all included in one monolithic HTML file.

The `airbitz-plugins` repo has a build system to help ease the creation of the files. Simply creating a new directory under the `plugins` directory will be treated as a new plugin. The build system knows to how to compile the javascript, HTML and CSS to work with Airbitz.

## Dependencies

### Get the repos

Clone the repos [airbitz-plugins](https://github.com/Airbitz/airbitz-plugins), [airbitz-ios-gui](https://github.com/Airbitz/airbitz-ios-gui), and [airbitz-android-gui](https://github.com/Airbitz/airbitz-android-gui) to the same level directory.

### NPM

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
npm install -g gulp
cd airbitz-plugins
npm install
```

First off, lets make sure you have the dependencies installed.

### Create New Plugin

To create a new plugin, you can copy the blank plugin and begin coding.

`cp -a blank yourpluginname`

### Plugin Files

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
plugins/yourpluginname/index.html
plugins/yourpluginname/css/style.css
plugins/yourpluginname/js/script.js
plugins/yourpluginname/vendors/jquery-2.1.3.min.js
plugins/yourpluginname/vendors/qrcode.min.js
```

And here is a list of the files in the new plugin.


## index.html

Let’s take a look at the `index.html`. The `index.html` is essentially your main function into the plugin. That means, you need to declare all of your dependencies here, such as CSS, fonts or javascript files. You can view the full source of the sample file [here.](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/blank/index.html). Let’s examine the file piece by piece.

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
<meta name=viewport content="initial-scale=1, maximum-scale=1.0, user-scalable=no">
<!-- include your stylesheet -->
<link type="text/css" rel="stylesheet" href="css/style.css"  media="screen,projection"/>
```

First up is the `<head>` tag. The head can contain whatever you want, in this case its just a reference to our `style.css` but other references or javascript files can be included. The build system will automatically inline them into the final build.


### `<body>` tag

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
<div id="container">
  <h2>Wallet Name</h2>
  <div id="walletName">Loading...</div>
  <h2>Address</h2>
  <div id="address">Loading...</div>
  <div id="qrcode"></div>
</div>
```

Next up is the `<body>` tag. To keep things simple you can add your UI directly inside of the HTML. Some of our the other plugins use a more sophisticated framework like angular. Here is our UI.






### Other Dependencies

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
<!-- Include the core javascript bindings -->
<script src="js/abc.js" type="text/javascript"></script>

<!-- Include some other libraries -->
<script src="vendors/jquery-2.1.3.min.js" type="text/javascript"></script>
<script src="vendors/qrcode.min.js" type="text/javascript"></script>

<!-- Include your javascript code -->
<script src="js/script.js" type="text/javascript"></script>
```

At the bottom of the `<body>` tag, we can include other dependencies such as all our javascript libraries. Its important to include `abc.js` which will give you access to the Airbitz wallet functionality.


## script.js

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```
```javascript
$(function() {
  Airbitz.ui.title('Blank Plugin');
  qrcode = new QRCode(document.getElementById("qrcode"), {
    text: '',
    width: 128,
    height: 128,
  });
  // If the user changes the wallet, we want to know about it
  Airbitz.core.walletSelectedChangeListener(function(wallet) {
    Airbitz.ui.showAlert("Wallet Changed", "Wallet Changed to " + wallet.name + ".");
    updateUi(wallet);
  });
  // After loading, lets fetch the currently selected wallet
  Airbitz.core.getSelectedWallet({
      success: updateUi,
      error: function() {
          Airbitz.ui.showAlert("Wallet Error", "Unable to load wallet!");
      }
  });
});
```

The next important part of your plugin is the javascript. As we can see from the `index.html`, we are using `abc.js`, `jquery-2.1.3.min.js`, `qrcode.min.js` and finally `script.js`. `script.js` is the plugin's code that implements the core business logic and application functionality

The `script.js` calls into the Airbitz core in a few ways. First it calls `Airbitz.ui.title` to change the page title. Next is sets up a wallet listener using `Airbitz.core.walletSelectedChangeListener `, so when the user changes their selected wallet, our code knows about it. Lastly, it requests the current wallet using `Airbitz.ui.getSelectedWallet`. You can view the sample code here.

### updateUI()

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
function updateUi(wallet) {
  $('#walletName').text(wallet.name);
  Airbitz.core.receiveRequestCreate(wallet, {
    label: "Blank App Request",
    category: "Income:Plugin",
    notes: "Income generated from a plugin",
    amountSatoshi: 0,
    amountFiat: 0,
    success: function(data) {
      var address = data["address"];
      $('#address').text(address);
      qrcode.clear();
      qrcode.makeCode('bitcoin:' + address);
    },
    error: function() {
      $('#address').text('');
      Airbitz.ui.showAlert("Wallet Error", "Unable to load request!");
    }
  });
}
```

Lastly we define the `updateUi` function. When the plugin loads or when the user changes their selected wallet, this function is called. We update the wallet name in the UI, and call into the Airbitz core library to create a receive request. This returns an address to us, but stores the meta with the address, so when bitcoin is received, the transactions meta-data will automatically be tag with the same information.

### Existing Plugins

For more ideas you can check out our existing plugins with [Foldapp](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/foldapp) and [Glidera](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/glidera).

The [Glidera](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/glidera) plugin is build on top of angular, while the foldapp plugin is using a jQuery and handlebars.

# Add Your Plugin to Airbitz

In order to see your plugin in Airbitz, you must modify the Native app to include the plugin. Those instructions are slightly different for Android vs iOS.

## Android

### Edit `mkplugin`

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
gulp glidera-android
gulp foldapp-android
gulp yourpluginname-android
cp build/android/glidera/index.html ${CURRENT_DIR}/Airbitz/airbitz/src/main/assets/glidera.html
cp build/android/foldapp/index.html ${CURRENT_DIR}/Airbitz/airbitz/src/main/assets/foldapp.html
cp build/android/yourpluginname/index.html ${CURRENT_DIR}/Airbitz/airbitz/src/main/assets/foldapp.html
```
##

Follow the README instructions in `airbitz-android-gui` to build the app.

To add your plugin to Android, first modify the `mkplugin` script to include your new plugin.

Look for the lines that begin with `gulp ...` and add a line of the format

`gulp [yourpluginname]-android`

Then find the lines that begin with `cp ...` and add a line of the format

`cp build/android/yourpluginname/index.html ${CURRENT_DIR}/Airbitz/airbitz/src/main/assets/yourpluginname.html`

### Edit `PluginFramework.java`

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
plugin = new Plugin();
plugin.pluginId = "com.yourplugincompany.yourplugin";
plugin.sourceFile = "file:///android\_asset/yourplugin.html";
plugin.name = "YourPlugin"
// These are the various environment settings to supply
plugin.env.put("SANDBOX", String.valueOf(api.isTestNet()));
mPlugins.add(plugin);
mPluginsGrouped.get(BUYSELL).add(plugin);
```

Now we need to modify the native code. In your favorite editor open `java/com/airbitz/plugins/PluginFramework.java` and look for the `class PluginList`. In the constructor, add your plugin with a configuration like the following.

### Build and Run the Airbitz app

Now that all the code is in place we can build the plugin and the app and run our plugin.

`./gradlew buildAirbitzPlugins installDevelopDebug`

Last, launch the app, login, navigate to Buy/Sell and launch your plugin.


## iOS

### Edit `mkplugin`

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
gulp glidera-ios
gulp foldapp-ios
gulp yourpluginname-ios
cp build/ios/glidera/index.html ${CURRENT_DIR}/Airbitz/Resources/plugins/glidera.html
cp build/ios/foldapp/index.html ${CURRENT_DIR}/Airbitz/Resources/plugins/foldapp.html
cp build/ios/yourpluginname/index.html ${CURRENT_DIR}/Airbitz/Resources/plugins/yourpluginname.html
```

Follow the README instructions in `airbitz-ios-gui` to build the app.

To add your plugin to iOS, first modify the `mkplugin` script to include your new plugin.

Look for the lines that begin with `gulp ...` and add a line of the format

`gulp [yourpluginname]-ios`

Then find the lines that begin with `cp ...` and add a line of the format

`cp build/ios/[yourpluginname]/index.html ${CURRENT_DIR}/Airbitz/Resources/plugins/yourpluginname.html`


### Edit `Plugins.m`

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
plugin = [[Plugin alloc] init];
plugin.pluginId = @"com.yourcompany.yourpluginname";
plugin.sourceFile = @"yourpluginname";
plugin.sourceExtension = @"html";
plugin.name = @"YourPluginName";
plugin.env = @{
    @"SANDBOX": (isTestnet ? @"true" : @"false"),
};
[buySellPlugins addObject:plugin];
```

In Xcode (or your favorite editor) open `Airbitz/Plugins/Plugins.m`. Add your plugin to the buySellPlugins array like the following.

### Build and Run the Airbitz app

Now you can run `./mkplugin`. Then build and run the code in Xcode. You can then launch the app, login, navigate to Buy/Sell and launch your plugin.

# Submit Your Plugin

To submit your plugin for inclusion in Airbitz, submit a pull request for the changes to the `airbitz-plugins` repo. In addition, submit pull requests for the iOS file `Plugins.m` and the Android file `PluginFramework.java`.

# Plugin API Reference

## ABCWallet object

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
{
    "id": "wallet-id",
    "name": "My Wallet",
    "currency": "USD",
    "currencyCode": 840,
    "balance": 1000000,
}
```

The ABCWallet object represents a single BIP32 HD wallet in the user's Airbitz account. Users can switch which wallet they currently use for requests and sends. Developers should check and use the current ABCWallet object using getSelectedWallet or setWalletChangeListener.

| Property | Type | Description |
| --- | --- | --- |
| id | <code>String</code> | Wallet ID |
| name | <code>String</code> | Wallet text name |
| currency | <code>String</code> | Wallet 3 letter currency code |
| currencyCode | <code>Number</code> | Wallet ISO currency code number |
| balance | <code>Number</code> | Wallet balance in satoshis |


## getSelectedWallet

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.getSelectedWallet(callback)

// Example

Airbitz.core.getSelectedWallet(function (response){
  console.log("Wallet name: " + response['name'])
  console.log("Wallet balance in satoshis: " + response['balance'])  
});
```

Returns the user’s currently selected wallet as the first parameter of the callback.


| Response | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | ABCWallet object |

## setWalletChangeListener

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.setWalletChangeListener(callback)

// Example

Airbitz.core.setWalletChangeListener(function (response){
  console.log("Wallet name: " + response['name'])
  console.log("Wallet balance in satoshis: " + response['balance'])  
});
```

Callback is called when wallet is changed AND every time the plugin is initialized.


| Response | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | ABCWallet object |

## createReceiveRequest


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.createReceiveRequest(wallet, options)

// Example

Airbitz.core.createReceiveRequest(wallet, {
    label: "Roger Mark",
    category: "Income:Consulting",
    notes: "Web development project from 2016-04",
    amountSatoshi: 12345234,
    success: function(response) {
      console.log("Got address " + response['address'])
    },
    error: function() {
      console.log("Error getting address")
    }
});
```
    
> Response
 
```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```
```javascript
{
    "address": "1PfLSCgMZdzHRKsQDSya6Pin3ugqLKri3n"
}
```

Create a receive request from the provided wallet. Returns an object with a bitcoin address.

| Param | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | Wallet to create a receive request/address from |
| options | <code>Object</code> | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| address | <code>String</code> | Bitcoin public address of request |

| Options | Type | Description |
| --- | --- | --- |
| label | <code>String</code> | (Optional) Transaction 'Payee/Payer' name |
| category | <code>String</code> | (Optional) Transaction category such as "Expense:Rent" |
| notes | <code>String</code> | (Optional) Transaction misc notes field |
| amountSatoshi | <code>Number</code> | (Optional) Amount, in satoshis, for this request |
| success | <code>Function</code> | Success callback |
| error | <code>Function</code> | Error callback |

## finalizeReceiveRequest

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```
```javascript
Airbitz.core.finalizeReceiveRequest(ABCWallet, address)

// Example

Airbitz.core.walletSelected({
  success: function(wallet) {
    Airbitz.core.createReceiveRequest(wallet, {
        label: "Roger Mark",
        category: "Income:Consulting",
        notes: "Web development project from 2016-04",
        amountSatoshi: 12345234,
        success: function(response) {
          console.log("Got address " + response['address'])
          Airbitz.core.finalizeReceiveRequest(wallet, data["address"]);
        },
        error: function() {
          console.log("Error getting address")
        }
    });
});
```


| Param | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | Wallet to finalize receive request/address from |
| address | <code>String</code> | Address to finalize |

Finalizing a request marks the address as used and it will not be used for future requests. The optional metadata given in 'options' such as 'label', 'category', and 'notes' will also be written for this address. This is useful so that when a future payment comes in, the metadata can be auto-populated in the user's transaction details.

## createSpendRequest


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.createSpendRequest(wallet, address, amount, options)

// Example

Airbitz.core.createSpendRequest(wallet, "", {
    label: "Roger Mark",
    category: "Income:Consulting",
    notes: "Web development project from 2016-04",
    success: function(response) {
      if (response.back) {
        console.log("User pressed back button. Funds not sent")
      } else {
        console.log("Bitcoin sent")
      }
    },
    error: function() {
      console.log("Error sending funds")
    }
});
```
    
> Response
 
```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```
```javascript
{
    "back": false
}
```

Request that the user spends. This takes the user to the native spend confirmation screen so they can confirm the spend.

The optional metadata given in 'options' such as 'label', 'category', and 'notes' will also be written for this transaction. 

| Param | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | Wallet to create a receive request/address from |
| address | <code>String</code> | Bitcoin address or BIP21 URI |
| options | <code>Object</code> | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| back | <code>Boolean</code> | True if user pressed back button and did not authorize the spend |

| Options | Type | Description |
| --- | --- | --- |
| label | <code>String</code> | (Optional) Transaction 'Payee/Payer' name |
| category | <code>String</code> | (Optional) Transaction category such as "Expense:Rent" |
| notes | <code>String</code> | (Optional) Transaction misc notes field |
| success | <code>Function</code> | Success callback |
| error | <code>Function</code> | Error callback |


## createSpendRequest2


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.createSpendRequest2(wallet, address, amount, address2, amount2, options)

// Example

Airbitz.core.createSpendRequest2(wallet, "", {
    label: "Roger Mark",
    category: "Income:Consulting",
    notes: "Web development project from 2016-04",
    success: function(response) {
      if (response.back) {
        console.log("User pressed back button. Funds not sent")
      } else {
        console.log("Bitcoin sent")
      }
    },
    error: function() {
      console.log("Error sending funds")
    }
});
```
    
> Response
 
```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```
```javascript
{
    "back": false
}
```

Request that the user spends to two different addresses with two different amounts. This takes the user to the native spend confirmation screen so they can confirm the spend. The amount2 value will be added to the 'fee' amount shown on the Send Confirmation screen presented to the user.

| Param | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | Wallet to create a receive request/address from |
| address | <code>String</code> | Bitcoin address or BIP21 URI |
| options | <code>Object</code> | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| back | <code>Boolean</code> | True if user pressed back button and did not authorize the spend |

| Options | Type | Description |
| --- | --- | --- |
| label | <code>String</code> | (Optional) Transaction 'Payee/Payer' name |
| category | <code>String</code> | (Optional) Transaction category such as "Expense:Rent" |
| notes | <code>String</code> | (Optional) Transaction misc notes field |
| success | <code>Function</code> | Success callback |
| error | <code>Function</code> | Error callback |

