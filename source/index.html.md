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
# Introduction

AirbitzCore (ABC) is a Javascript/ObjC/Java client-side blockchain and Edge Security SDK providing auto-encrypted and auto-backed up accounts and wallets with zero-knowledge security and privacy. All blockchain/bitcoin private and public keys are fully encrypted by the users' credentials before being backed up on to peer to peer servers. 

ABC allows developers to apply client-side data security, encrypted such that only the end-user can access the data. ABCDataStore object in the Airbitz ABCAccount object allows developers to store arbitrary Edge-Secured data on the user’s account which is automatically encrypted, automatically backed up, and automatically synchronized between the user’s authenticated devices.

To get started, you’ll first need an API key. Get one at our [developer portal.](https://developer.airbitz.co)

# API Reference

## Install the SDK

See the following Github repos for your various development languages. Installation instructions are in the README.md files


[Javascript](https://github.com/Airbitz/airbitz-core-js)

[Objective-C](https://github.com/Airbitz/airbitz-core-objc)

[Java/Android](https://github.com/Airbitz/airbitz-core-java)

For Javascript using React Native, see instructions for [Objective C](https://github.com/Airbitz/airbitz-core-objc)


## Include and initialize the SDK

```javascript
var abc = require ('./abc.js')
```

```objective_c
#import "AirbitzCore.h"
```

Include the proper header files and/or libraries for your language.

<a name="ABCContext"></a>

## ABCContext

Starting point of Airbitz Core SDK. Used for operations that do not require a logged in ABCAccount

### makeABCContext

```javascript
abc.ABCContext.makeABCContext(apiKey, hbitsKey, callback)

// Example

var abcContext = null

abc.ABCContext.makeABCContext('your-api-key-here', null, function (error, context) {
  if (error) {
  } else {
    abcContext = context
  }
})
```

```objective_c
+(ABCContext *) makeABCContext:(NSString *)abcAPIKey hbits:(NSString *)hbitsKey

ABCContext *abcContext = [ABCContext makeABCContext:@"your-api-key-here" hbits:null];
```

Initialize and create an ABCContext object. Required for functionality of ABC SDK.

| Param | Type | Description |
| --- | --- | --- |
| apiKey | <code>string</code> | Get an API Key from https://developer.airbitz.co |
| hbitsKey | <code>string</code> | (Optional) Unique key used to encrypt private keys for use as implementation specific "gift cards" that are only redeemable by applications using this implementation.|
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | (Javascript) Error object. Null if no error |
| context | <code>[ABCContext](#ABCContext)</code> | Initialized context |



### createAccount

```javascript
abcContext.createAccount(username, 
                         password, 
                         pin, 
                         callbacks, 
                         callback)

// Example

abcContext.createAccount("myUsername", 
                         "myNot5oGoodPassw0rd", 
                         "2946", 
                         callbacks, 
                         (error, account) => {
    if (error) {
      reject(funcname)
    } else {
      abcAccount = account;
    }
```

```objective_c

-(void) createAccount:(NSString *) username
             password:(NSString *) password
                  pin:(NSString *) pin
             delegate:(ABCAccountDelegate) delegate
             callback:^(ABCError *error, ABCAccount *account)

// Example

ABCAccount *abcAccount;

[abcContext createAccount:@"myUsername"
                 password:@"myNot5oGoodPassw0rd"
                      pin:@"2946"
                 delegate:self
                 callback:^(ABCError *error, ABCAccount *account)
{
    if (error)
    {
        abcAccount = account;
    }
    else
    {
        // Yikes
    }
}];

```

Create and log into a new ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| password | <code>string</code> | Account password |
| pin | <code>string</code> | Account PIN for fast re-login |
| callbacks | <code>[ABCCallbacks](#ABCCallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#ABCAccountDelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|


| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#ABCAccount)</code> | Initialized account |



### loginWithPassword

```javascript
abcContext.loginWithPassword(username, password, otp, callbacks, callback)

// Example
abcContext.loginWithPassword("JoeHomey", "My0KPa55WoRd@Airb1t5", null, null, 
                             function (error, account) {
    if (error) {
      if (error.code === ABCConditionCodeInvalidOTP) {
        console.log("otpResetToken: " + error.otpResetToken)
      }
    } else {
      // Yay. logged in
      console.log("Account name = " + account.username)
    }
})
```

```objc

-(void) loginWithPassword:(NSString *) username
                 password:(NSString *) password
                 delegate:(ABCAccountDelegate) delegate
                 callback:^(ABCError *error, ABCAccount *account);

// Example

ABCAccount *abcAccount;

[abcContext loginWithPassword:@"myUsername"
                     password:@"myNot5oGoodPassw0rd"
                     delegate:self
                     callback:^(ABCError *error, ABCAccount *account)
{
    if (error)
    {       
      // Yikes
      if (error.code == ABCConditionCodeInvalidOTP) {
        NSLog(@"otpResetToken: %@", error.otpResetToken)
      }
    }
    else
    {
        NSLog(@"Account name = %@", account.username);
    }
}];

```
Login to an Airbitz account with a full password. May optionally send 'otp' key which is required for any accounts that have OTP enabled using ABCAccount.enableOTP. OTP key can be retrieved from a device that has account logged in and OTP enabled using getOTPLocalKey.

If routine returns with error.code == ABCConditionCodeInvalidOTP, then the account has OTP enabled and needs the OTP key specified in parameter 'otp'. ABCError object may have properties otpResetToken and otpResetDate set which allow the user to call requestOTPReset to disable OTP.

This routine allows caller to receive back an error.otpResetToken which is used with requestOTPReset to remove OTP from the specified account.

The otpResetToken is only returned if the caller has provided the correct username and password but the account had OTP enabled. error.otpResetDate is the date when the account OTP will be disabled if a prior OTP reset was successfully requested. The reset date is set by default to 7 days from when a reset was initially requested.

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| password | <code>string</code> | Account password |
| otp | <code>string</code> | (Optional) OTP key retrieved from getOTPLocalKey |
| callbacks | <code>[ABCCallbacks](#ABCCallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#ABCAccountDelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|


| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#ABCAccount)</code> | Initialized account |



### loginWithPIN

```javascript
abcContext.loginWithPIN(username, pin, callbacks, callback)

// Example
abcContext.loginWithPIN("JoeHomey", "2847", null, 
                        function (error, account) {
    if (error) {
      // Error
    } else {
      // Yay. logged in
    }
})
```

```objective_c
-(void) loginWithPIN:(NSString *) username
            password:(NSString *) password
            delegate:(ABCAccountDelegate) delegate
            callback:^(ABCError *error, ABCAccount *account);

// Example

ABCAccount *abcAccount;

[abcContext loginWithPIN:@"myUsername"
                     pin:@"4728"
                delegate:self
                callback:^(ABCError *error, ABCAccount *account)
{
    if (!error)
    {
        NSLog(@"Account name = %@", account.username);
    }
    else
    {
        // Yikes
    }
}];

```
Login to an Airbitz account with PIN. Used to sign into devices that have previously been logged into using a full username & password

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| pin | <code>string</code> | Account PIN for fast re-login |
| callbacks | <code>[ABCCallbacks](#ABCCallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#ABCAccountDelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#ABCAccount)</code> | Initialized account |


### accountHasPassword

```javascript
abcContext.accountHasPassword(username, callback)

// Example
abcContext.accountHasPassword("JoeHomey", function (error, hasPassword) {
    if (error) {
      // Error
    } else {
      if (hasPassword) {
        // This account has a password
      }
    }
})
```

```objective_c

-(BOOL) accountHasPassword:(NSString *) username
                     error:(ABCError *) error;

// Example

ABCError *error;

BOOL hasPassword = [abcContext accountHasPassword:@"myUsername"
                                            error:&error

```
Check if specified username has a password on the account or if it is a PIN-only account.

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| hasPassword | <code>Boolean</code> | True if account has a password |


### deleteLocalAccount

```javascript
abcContext.deleteLocalAccount(username, callback)

// Example
abcContext.deleteLocalAccount("JoeHomey", function (error) {
    if (error) {
      // Error
    } else {
    }
})
```

```objective_c

- (ABCError *) deleteLocalAccount:(NSString *) username;

// Example

ABCError *error;

ABCError *error = [abcContext accountHasPassword:@"myUsername"];

```
Deletes named account from local device. Account is recoverable if it contains a password. Use accountHasPassword to determine if account has a password. Recommend warning user before executing deleteLocalAccount if accountHasPassword returns FALSE.

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| hasPassword | <code>Boolean</code> | True if account has a password |

### pinLoginEnabled

```javascript
abcContext.pinLoginEnabled(username, callback)

// Example

abcContext.pinLoginEnabled(username,
                           (error, enabled) => {
    if (!error) {
      console.log("PIN Login enabled state: " + enabled)
    }
})
```

```objective_c
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example

ABCError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | <code>String</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| enabled | <code>Boolean</code> | True if PIN login is enabled |

### listUsernames

```javascript

abcContext.listUsernames(callback)

// Example

abcContext.listUsernames((error, usernames) => {
    if (!error) {
      console.log("username 0: " + usernames[0])
    }
})
```

```objective_c
- (NSArray *) listUsernames:(ABCError **) abcerror;

// Example

NSError *error;
NSArray *usernames = [abc listUsernames:&error];
if (!error) {
  NSLog(@"username 0: %@", usernames[0]);
}
```

Get a list of previously logged in usernames on this device

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| usernames | <code>Array</code> | Array |

### usernameAvailable

```javascript
abcContext.usernameAvailable(username, callback)

// Example

abcContext.usernameAvailable(username,
                             (error, available) => {
    if (!error) {
      console.log("username available = " + available)
    }
})
```

```objective_c
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example

ABCError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | <code>String</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| enabled | <code>Boolean</code> | True if PIN login is enabled |




<a name="ABCAccount"></a>
## ABCAccount

### Class Properties
| Property | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |



### logout

```objective_c
ABCError *error = [abcAccount logout];
if (error) {
    // Oh no
} else {
    // Hooray. I'm out
}

```

```javascript
abcAccount.logout(function(error) {
  if (error) {
    // Oh no
  } else {
    // Hooray. I'm out
  }
})
```

Logout the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| account | <code>[ABCAccount](#ABCAccount)</code> | Account object|
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |



### changePassword

```objective_c
ABCError *error = [abcAccount changePassword:password];
if (error) {
    // Oh no
} else {
    // Yay, new password
}

[abcAccount changePassword:password callback:^(ABCError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new password
    }
}];

```

```javascript
abcAccount.changePassword(password, function(error) {
  if (error) {
    // Oh no
  } else {
    // Yay!
  }
})
```

Logout the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| password | <code>String</code> | Password string|
| callback | <code>Callback</code> | Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |



### changePIN

```objective_c
NSError *error = [abcAccount changePIN:pin];
if (error) {
    // Oh no
} else {
    // Yay, new password
}

[abcAccount changePIN:pin callback:^(ABCError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new password
    }
}];
```

```javascript
abcAccount.changePIN(pin, function(error) {
  if (error) {
    // Oh no
  } else {
    // Yay!
  }
})
```

Change the PIN of the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| pin | <code>String</code> | 4 digit PIN string|
| callback | <code>Callback</code> | (Javascript/ObjC) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |



### checkPassword

```javascript
abcAccount.checkPassword(password,
                         (error, passwordCorrect) => {
    if (error) {
      reject(funcname)
    } else {
      abcAccount = account;
    }
})
```

```objective_c
BOOL passwordCorrect = [abcAccount checkPassword:password];
```

Check if a string is the correct password for the current account

| Param | Type | Description |
| --- | --- | --- |
| password | <code>string</code> | Account password |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | (Javascript) Error object. Null if no error |
| passwordCorrect | <code>Boolean</code> | True if password is correct |



### enablePINLogin

```javascript
abcAccount.enablePINLogin(enable,
                         (error) => {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objective_c
ABCError *error = [abcAccount enablePINLogin:enable];
```

Enable or disable PIN login on this account. Set enable = YES to allow PIN login. Enabling PIN login creates a local account decryption key that is split with one have in local device storage and the other half on Airbitz servers. When using loginWithPIN the PIN is sent to Airbitz servers to authenticate the user. If the PIN is correct, the second half of the decryption key is sent back to the device. Combined with the locally saved key, the two are then used to decrypt the local account thereby loggin in the user.

| Param | Type | Description |
| --- | --- | --- |
| enable | <code>Boolean</code> | Set true to enable PIN login. False to disable |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | (Javascript) Error object. Null if no error |



<a name="ABCWallet"></a>
## ABCWallet

<a name="ABCTransaction"></a>
## ABCTransaction





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

The ABCWallet object represents a single BIP32 HD wallet in the user's Airbitz account. Users can switch which wallet they currently use for requests and sends. Developers should check and use the current ABCWallet object using getSelectedWallet or setupWalletChangeListener.

| Property | Type | Description |
| --- | --- | --- |
| id | <code>String</code> | Wallet ID |
| name | <code>String</code> | Wallet text name |
| currency | <code>String</code> | Wallet 3 letter currency code |
| currencyCode | <code>Number</code> | Wallet ISO currency code number |
| balance | <code>Number</code> | Wallet balance in satoshis |


## Airbitz Core
### getSelectedWallet

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

## setupWalletChangeListener

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.setupWalletChangeListener(callback)

// Example

Airbitz.core.setupWalletChangeListener(function (response){
  console.log("Wallet name: " + response['name'])
  console.log("Wallet balance in satoshis: " + response['balance'])  
});
```

Callback is called when wallet is changed AND every time the plugin is initialized.


| Response | Type | Description |
| --- | --- | --- |
| wallet | <code>ABCWallet</code> | ABCWallet object |

### createReceiveRequest


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

### finalizeReceiveRequest

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

### createSpendRequest


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


### createSpendRequest2

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

### writeData


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.writeData(key, data)

// Example

Airbitz.core.writeData("userInfo", '{ "name": "Joe",
                                      "phone": "619-800-1234",
                                      "age:" 69 }')
```

Securely persist data into the Airbitz core under this user's account. Only the current plugin will have access to that data. Data is fully encrypted and synchronized between devices that the user logs into.

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | Key string to reference the data |
| data | <code>String</code> | Value to store |

### readData


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
data = Airbitz.core.readData(key)

// Example

Airbitz.core.writeData("userInfo", '{ "name": "Joe",
                                      "phone": "619-800-1234",
                                      "age:" 69 }')
data = Airbitz.core.readData("userInfo")

console.log(data) => { "name": "Joe",
                       "phone": "619-800-1234",
                       "age": 69 }
```    
Read back data from the Airbitz core under this user's account. Only the current plugin will have access to that data. Data is fully encrypted and synchronized between devices that the user logs into.

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | Key string to reference the data |

| Response | Type | Description |
| --- | --- | --- |
| data | <code>String</code> | Data stored from a previous call to writeData |

### clearData


```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.core.clearData()
```    

Clear all data in the Airbitz core, for the current plugin.

### getBtcDenomination

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
denom = Airbitz.core.getBtcDenomination()
```

Get the user’s currently selected BTC denomination. It can be BTC, mBTC or bits. Returns a denomination string.

| Response | Type | Description |
| --- | --- | --- |
| denomination | <code>String</code> | "BTC", "mBTC" or "bits" |


### formatSatoshi

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
amountString = Airbitz.core.formatSatoshi(amountSatoshi, withSymbol)

// Example
amountString = Airbitz.core.formatSatoshi(125834, true)

console.log(amountString) => "ƀ1258.34"
```

Formats satoshis to display to the user. This uses the user’s BTC denomination in their settings to format including the correct code and symbol.

| Param | Type | Description |
| --- | --- | --- |
| amountSatoshi | <code>Number</code> | Number of satoshis to convert |
| withSymbol | <code>Boolean</code> | Set to true to display the denomination symbol "Ƀ, mɃ, ƀ" |

| Response | Type | Description |
| --- | --- | --- |
| amountString | <code>String</code> | Formatted denomination string |


## Airbitz Config
### get

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
value = Airbitz.config.get(key)

// Example

value = Airbitz.config.get("API_KEY")
```

Fetch a configuration value. These are set in the native iOS/Android code, before the webview is loaded. Useful to pass in parameters such as API keys embedded in the native app.

| Param | Type | Description |
| --- | --- | --- |
| key | <code>String</code> | Key string to reference the data |

| Response | Type | Description |
| --- | --- | --- |
| value | <code>String</code> | Data passed in from native app |

## Airbitz UI

### title

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.ui.title(title)

// Examples

Airbitz.ui.title("Bob's Awesome Plugin");
```
Set the title of the current view. This updates the native apps titlebar.

| Param | Type | Description |
| --- | --- | --- |
| title | <code>String</code> | Title on navbar |


### showAlert

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.ui.showAlert(title, message, options)

// Examples

Airbitz.ui.showAlert("Account creation error", "Error creating account. Please try again later");

Airbitz.ui.showAlert("Creating account", "Please wait...", {"showSpinner": true});

```


Launches a native alert dialog. Alert will automatically fade when tapped or after ~6 seconds unless the showSpinner option is given.

| Param | Type | Description |
| --- | --- | --- |
| title | <code>String</code> | Title of alert |
| message | <code>String</code> | Body of alert  |
| options | <code>Object</code> | Optional parameters |

| Options | Type | Description |
| --- | --- | --- |
| showSpinner | <code>Boolean</code> | (Optional) Add spinner to alert. Alert will persist until hideAlert is called. |

### hideAlert

```objc
// No content for this language. Select 'Javascript/HTML` above
```
```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.ui.hideAlert()

```

Hides any currently displaying alert from showAlert

