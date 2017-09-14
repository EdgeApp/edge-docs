---
title: AirbitzCore API/SDK Reference

language_tabs:
  - javascript: Javascript
  - objective_c: Objective C
  - java: Java/Android

toc_footers:
  - <a href='https://developer.airbitz.co'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---
# Introduction

AirbitzCore (ABC) is a Javascript/ObjC/Java client-side blockchain and Edge Security SDK providing auto-encrypted and auto-backed up accounts and wallets with zero-knowledge security and privacy. All blockchain/bitcoin private and public keys are fully encrypted by the users' credentials before being backed up on to peer to peer servers.

ABC allows developers to apply client-side data security, encrypted such that only the end-user can access the data. The Airbitz ABCAccount object allows developers to store arbitrary Edge-Secured data on the user’s account which is automatically encrypted, automatically backed up, and automatically synchronized between the user’s authenticated devices.

To get started, you’ll first need an API key. Get one at our [developer portal.](https://developer.airbitz.co)

# Airbitz Login System

The Airbitz login system provides a way to backup and retrieve encrypted private keys. Users can login in using various methods:

* Password
* PIN
* Recovery questions
* Fingerprint
* Barcode scan

A single account (username and password) can log into multiple applications, each with its own keys. The application id (or `appId`) determines which keys are visible through this API. The keys provide access to various resources, including:

* Airbitz git repos for account settings
* Airbitz git repos for wallet metadata
* Public blockchains, like Bitcoin
* BitId identities

Airbitz provides a separate wallet API for working with these keys once they've been retrieved by the login system.

## Install the SDK

See the following Github repos for your various development environments. Installation instructions are in the README.md files

[Javascript](https://github.com/Airbitz/airbitz-core-js)

[Objective-C](https://github.com/Airbitz/airbitz-core-objc)

[Java/Android](https://github.com/Airbitz/airbitz-core-java)

```objc
#import "ABCContext.h"
```

## Platform-specific IO

(Javascript only) The Airbitz login system runs in many different environments, including the web, Node.js and React Native. To handle the differences between these platforms, the login system talks to the outside world through an `io` object. Before intializing the login API, you need an `io` object for your particular platform.

### makeFakeIos

```javascript
const abc = require('airbitz-core-js')

const [io1, io2] = abc.makeFakeIos(2)
const context1 = abc.makeContext({ io: io1 })
const context2 = abc.makeContext({ io: io2 })
```

Creates an array of virtual IO objects with a simulated server and storage. This is useful for unit testing. All objects share the same virtual server, which makes it possible to simulate multi-phone login scenarios.

| Param | Type | Description |
| --- | --- | --- |
| count | `number` | Number of `io` objects to create

### makeBrowserIo

```javascript
const abc = require('airbitz-core-js')

const io = abc.makeBrowserIo()
const context = abc.makeContext({ io })
```

Gathers the various browser API's into an IO object. This allows Airbitz to work on the web.

### airbitz-io-node-js

```javascript
const abc = require('airbitz-core-js')
const abcNode = require('airbitz-io-node-js')

const io = abcNode.makeNodeIo('/home/username/.config/appname')
const context = abc.makeContext({ io })
```

Gathers various Node.js API's into an IO object.

| Param | Type | Description |
| --- | --- | --- |
| path | `string` | The filesystem path to save files to

### react-native-airbitz-io

```javascript
const abc = require('airbitz-core-js')
const abcReact = require('react-native-airbitz-io')

abcReact.makeReactNativeIo().then(io => {
  const context = abc.makeContext({ io })
})
```

Gathers various React Native API's into an IO object. This is an asynchronous function, and returns a `Promise`.

## ABCContext

Starting point of Airbitz Core SDK. Used for operations that do not require a logged in ABCAccount

### makeContext

```javascript
const abc = require('airbitz-core-js')

const abcContext = abc.makeContext({
  apiKey: '178a732cacebed37...',
  appId: 'com.yourname.yourapp',
  io: myPlatformSpecificIo
})
```

```objc
+(ABCContext *) makeABCContext:(NSString *)abcAPIKey type:(NSString *)type hbits:(NSString *)hbitsKey

ABCContext *abcContext = [ABCContext makeABCContext:@"your-api-key-here" type:@"account:repo:com.mydomain.myapp" hbits:null];
```

Initialize and create an ABCContext object. Required for functionality of ABC SDK.

| Option | Type | Description |
| --- | --- | --- |
| apiKey | `string` | Get an API Key from <https://developer.airbitz.co> |
| appId | `string` | Type of account that this application will be accessing. This should be in reverse-domain format, like `"com.domain.app"`. The `appId` determines which keys are available to your application, so a login performed by `"com.domain.app1"` will receive different keys from `"com.domain.app2"`. |
| io | `object` | (Optional) Platform-specifc io resources. Defaults to browser IO if not provided. |

| Return Param | Type | Description |
| --- | --- | --- |
| context | [`ABCContext`](#abccontext) | Initialized context |

### createAccount

```javascript
abcContext.createAccount(username,
                         password,
                         pin,
                         ABCAccountOptions,
                         callback)

// Example
const callbacks = {
  onDataChanged () {},
  onKeyListChanged () {},
  onLoggedOut () {},
  onOTPRequired () {},
  onOTPSkew () {},
  onRemotePasswordChange () {}
}

abcContext.createAccount(
  "myUsername",
  "myNot5oGoodPassw0rd",
  "2946",
  { callbacks },
  function (error, account) {
    if (error) {
      // Error creating account
    } else {
      abcAccount = account;
    }
  }
})
```

```objc
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
| username | `string` | Account username |
| password | `string` | Account password |
| pin | `string` | Account PIN for fast re-login |
| options | [`ABCAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

### getLocalAccount

```javascript
abcContext.getLocalAccount(username, callbacks, options)

// Example
abcContext.getLocalAccount("JoeHomey", null,
                           function (error, account) {
    if (error) {
      // Failed to get account.
    } else {
      // Yay. Got local account info
      console.log("Account name = " + account.username)
    }
})
```

(Proposal)

Get local account details for a previously logged in account. This returns an ABCAccount object with a `null` `dataStore` object but with a functioning `localDataStore` object. This is effectively getting the non-encrypted account data which can be accessed without the user logging into the device with a password, PIN, or fingerpint.

Any [ABCWallet](#abcwallet) objects in the account will also have `null` `dataStore` objects but with functioning `localDataStore` objects. This is commonly used for background processing the accounts/wallets on a device to do querying of cryptocurrency transactions while the user is not logged in.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| options | [`ABCAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

### loginWithPassword

```javascript
abcContext.loginWithPassword(username, password, options, callback)

// Example
abcContext.loginWithPassword(
  "JoeHomey",
  "My0KPa55WoRd@Airb1t5",
  { callbacks },
  function (error, account) {
    if (error) {
      if (error.code === ABCConditionCodeInvalidOTP) {
        console.log("otpResetToken: " + error.otpResetToken)
      } else {
        // login failed.
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

Login to an Airbitz account with a full password. May optionally send 'otp' key which is required for any accounts that have OTP enabled using [ABCAccount.enableOTP](#enableotp). OTP key can be retrieved from a device that has account logged in and OTP enabled using getOTPLocalKey.

If routine returns with error.code == ABCConditionCodeInvalidOTP, then the account has OTP enabled and needs the OTP key specified in parameter 'otp'. ABCError object may have properties otpResetToken and otpResetDate set which allow the user to call requestOTPReset to disable OTP.

This routine allows caller to receive back an error.otpResetToken which is used with requestOTPReset to remove OTP from the specified account.

The otpResetToken is only returned if the caller has provided the correct username and password but the account had OTP enabled. error.otpResetDate is the date when the account OTP will be disabled if a prior OTP reset was successfully requested. The reset date is set by default to 7 days from when a reset was initially requested.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| password | `string` | Account password |
| options | [`ABCAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines & OTP key |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

### loginWithPIN

```javascript
abcContext.loginWithPIN(username, pin, callbacks, opts)

// Example
abcContext.loginWithPIN(
  "JoeHomey",
  "2847",
  { callbacks },
  function (error, account) {
    if (error) {
      // Error
    } else {
      // Yay. logged in
    }
})
```

```objc
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
| username | `string` | Account username |
| pin | `string` | Account PIN for fast re-login |
| options | [`ABCAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |


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

```objc
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
| username | `string` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| hasPassword | `Boolean` | True if account has a password |


### deleteLocalAccount

```javascript
abcContext.deleteLocalAccount(username, callback)

// Example
abcContext.deleteLocalAccount("JoeHomey", function (error) {
    if (error) {
      // Error
    } else {
      // Success deleting account from local device
    }
})
```

```objc
- (ABCError *) deleteLocalAccount:(NSString *) username;

// Example
ABCError *error;

ABCError *error = [abcContext deleteLocalAccount:@"myUsername"];
```

Deletes named account from local device. Account is recoverable if it contains a password. Use accountHasPassword to determine if account has a password. Recommend warning user before executing deleteLocalAccount if accountHasPassword returns FALSE.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| hasPassword | `Boolean` | True if account has a password |

### pinLoginEnabled

```javascript
abcContext.pinLoginEnabled(username, callback)

// Example
abcContext.pinLoginEnabled(username,function (error, enabled) {
    if (!error) {
      console.log("PIN Login enabled state: " + enabled)
    }
})
```

```objc
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example
ABCError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| enabled | `Boolean` | True if PIN login is enabled |

### getRecovery2Key

```javascript
recovery2Key = getRecovery2Key(username)
```

Retrieves the recovery key for a particular user. The user should email this key to themselves.

If this returns an error, the user does not have recovery set up on this device. The user needs to click the link in their email if they want to do a recovery login.

### fetchRecovery2Questions

```javascript
const questions = await fetchRecovery2Questions(recovery2Key, username)

// Result:
[
  'What is your name?',
  'What is your quest?',
  'What is your favorite color?'
]
```

Retrieves the recovery questions a user has configured for their account.

### loginWithRecovery2

```javascript
const answers = [
  'Sir Lancelot of Camelot',
  'To seek the Holy Grail',
  'Blue'
]

const account = await loginWithRecovery2(
  recovery2Key,
  username,
  answers,
  { callbacks }
)
```

Performs a recovery login.

### listRecoveryQuestionChoices

```javascript
const questions = await listRecoveryQuestionChoices()
```

Fetches a list of possible recovery questions from the server.

TODO: How does this work with localization? Why not build the question list into the app itself?

### listUsernames

```javascript
abcContext.listUsernames(callback)

// Example
abcContext.listUsernames(function (error, usernames) {
    if (!error) {
      console.log("username 0: " + usernames[0])
    }
})
```

```objc
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
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| usernames | `Array` | Array |

### usernameAvailable

```javascript
abcContext.usernameAvailable(username, callback)

// Example
abcContext.usernameAvailable(username, function (error, available) {
    if (!error && available) {
      console.log("username available = " + available)
    }
})
```

```objc
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example
ABCError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| enabled | `Boolean` | True if PIN login is enabled |

### requestOTPReset

```javascript
abcContext.requestOTPReset(username, otpResetToken, callback)

// Example
abcContext.requestOTPReset("JoeHomey", "ZR2G9d8TYW8DH6", function (error) {
    if (!error) {
      // Yay
    }
})
```

```objc
- (void)requestOTPReset:(NSString *)username
                  token:(NSString *)otpResetToken
               callback:(void (^)(ABCError *error)) callback;

// Example
[abc requestOTPReset:@"JoeHomey" token:@"ZR2G9d8TYW8DH6" callback:^(ABCError *error)
{
    if (!error)
        // Yay. Now to wait until the timeout expires to login again.
}];
```

Launches an OTP reset timer on the server, which will disable the OTP authentication requirement on the account `username` when timeout expires. The expiration timeout is set when OTP is first enabled using [ABCAccount.enableOTP](#enableotp).

To obtain an otpResetToken, attempt a login into the OTP protected account using loginWithPassword with the correct username/password. The login should fail with error.code == ABCConditionCodeInvalidOTP. The OTP token will be in the error.otpResetToken property of the ABCError object.


| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| otpResetToken | `String` | Reset token from loginWithPassword |
| callback | `Callback` | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |

### getCurrencyPlugins

```javascript
const plugins = await abcContext.getCurrencyPlugins()

// Example value:
[
  { currencyInfo: { ... }, ... }, // AbcCurrencyPlugin
  { currencyInfo: { ... }, ... } // AbcCurrencyPlugin
]
```

Retrieves an array of [`AbcCurrencyPlugin`](#abccurrencyplugin) objects extracted from the plugin list passed in at context creation time. These can be used for parsing URI's, retrieving currency information, and other currency-related tasks.

## ABCAccount

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |

### logout

```javascript
abcAccount.logout(function() {
  // Hooray. I'm out
})
```

```objc
[abcAccount logout];
```

Logout the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| account | [`ABCAccount`](#abcaccount) | Account object |
| callback | `Callback` | (Javascript) Callback function |

### currencyWallets

```javascript
abcAccount.currencyWallets

// Example value:
{
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=': { ... }, // AbcCurrencyWallet
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY=': { ... } // AbcCurrencyWallet
}
```

A key-value map. The keys are wallet id's, and the values are instances of [`AbcCurrencyWallet`](#abccurrencywallet). The account will automatically load and manage any wallet types that have corresponding plugins.

### activeWalletIds

```javascript
abcAccount.activeWalletIds

// Example value:
[
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=',
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY='
]
```

An array of active wallet ID's, in their proper sort order.  This list only includes wallets are not deleted or archived and have a corresponding currency plugin.

### archivedWalletIds

```javascript
abcAccount.archivedWalletIds

// Example value:
[
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=',
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY='
]
```

An array of archived wallet ID's, in their proper sort order. This list only includes wallets are not deleted and have a corresponding currency plugin.

### allKeys

```javascript
keys = abcAccount.allKeys
```

This read-only property contains a `Array` of all keys visible to the account, including keys that may belong to other applications (i.e. child logins).

### changePassword

```javascript
abcAccount.changePassword(password, callback)

// Example
abcAccount.changePassword(password, function(error) {
  if (error) {
    // Oh no
  } else {
    // Yay!
  }
})
```

```objc
- (void)changePassword:(NSString *)password
              callback:(void (^)(ABCError *error)) callback;

// Example
[abcAccount changePassword:password callback:^(ABCError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new password set
    }
}];
```

Change the password of the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| password | `String` | Password string |
| callback | `Callback` | Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |

### changePIN

```javascript
abcAccount.changePIN(pin, callback)

// Example
abcAccount.changePIN(pin, function(error) {
  if (error) {
    // Oh no
  } else {
    // Yay!
  }
})
```

```objc
- (void)changePIN:(NSString *)pin
         callback:(void (^)(ABCError *error)) callback;

// Example
[abcAccount changePIN:pin callback:^(ABCError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new PIN set
    }
}];
```

Change the PIN of the currently logged in ABCAccount

| Param | Type | Description |
| --- | --- | --- |
| pin | `String` | 4 digit PIN string|
| callback | `Callback` | (Javascript/ObjC) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |

### checkPassword

```javascript
abcAccount.checkPassword(password, callback)

// Example
abcAccount.checkPassword(password, function (error, passwordCorrect) {
    if (!error) {
      if (passwordCorrect) {
        // Yay
      }
    }
})
```

```objc
- (BOOL)checkPassword:(NSString *)password;

// Example
BOOL passwordCorrect = [abcAccount checkPassword:password];
```

Checks if password is the correct password for this account

| Param | Type | Description |
| --- | --- | --- |
| password | `string` | Account password |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| passwordCorrect | `Boolean` | True if password is correct |

### enablePINLogin

```javascript
abcAccount.enablePINLogin(enable, callback)

// Example
abcAccount.enablePINLogin(enable, function (error) {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objc
- (NSError *) enablePINLogin:(BOOL)enable;

// Example
ABCError *error = [abcAccount enablePINLogin:enable];
```

Enable or disable PIN login on this account. Set enable = YES to allow PIN login. Enabling PIN login creates a local account decryption key that is split with one have in local device storage and the other half on Airbitz servers. When using loginWithPIN the PIN is sent to Airbitz servers to authenticate the user. If the PIN is correct, the second half of the decryption key is sent back to the device. Combined with the locally saved key, the two are then used to decrypt the local account thereby loggin in the user.

| Param | Type | Description |
| --- | --- | --- |
| enable | `Boolean` | Set true to enable PIN login. False to disable |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |


### setupOTPKey

```javascript
abcAccount.setupOTPKey(key, callback)

// Example
abcAccount.setupOTPKey(key, function (error) {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objc
- (ABCError *) setupOTPKey:(NSString *)key;

// Example
ABCError *error = [abcAccount setupOTPKey:key];
```

Associates an OTP key with the currently logged in account. An OTP key can be retrieved from a previously logged in account using otpLocalKeyGet. The account must have had OTP enabled by using otpEnable().

This method is primarily used to add an OTP key to an account & device that was successfully logged in prior to OTP being enabled. A second device may have enabled OTP which would cause OTP warnings on the first device upon login.

ie.
Device A creates account "bob".
Device B login to account "bob".
Device B enables OTP.

Device A can still login but will get OTP token notification warnings and will be unable to make any account changes such as changing password/PIN or creating new wallets.
Device B can call getOTPLocalKey and Device A can add the OTP key using setupOTPKey


| Param | Type | Description |
| --- | --- | --- |
| key | `String` | OTP key |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |


### getOTPLocalKey

```javascript
abcAccount.getOTPLocalKey(callback)

// Example
abcAccount.getOTPLocalKey(key, function (error, key) {
    if (!error) {
      // Yay. Success
      console.log("My key is: " + key)
    }
})
```

```objc
- (NSError *) getOTPLocalKey:(BOOL)enable;

// Example
ABCError *error;
NSString *key = [abcAccount getOTPLocalKey:&error];
```

Gets the locally saved OTP key for the current user

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| key | `String` | OTP key |


### getOTPDetails

```javascript
abcAccount.getOTPDetails(callback)

// Example
abcAccount.getOTPDetails(function (error, enabled, timeout) {
    // Do something with 'enabled' or 'timeout'
})
```

```objc
- (ABCError *)getOTPDetails:(bool *)enabled
                   timeout:(long *)timeout;

// Example
BOOL enabled = NO;
long timeout = 0;

ABCError *error = [abcAccount getOTPDetails:&enabled
                                    timeout:&timeout];
```

Reads the OTP configuration from the server. Gets information on whether OTP is enabled for the current account, and how long a reset request will take. An OTP reset is a request to disable OTP made through the method ABCContext.requestOTPReset.

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| enabled | `Boolean` | True if OTP is enabled on this accout |
| timeout | `Number` | Number seconds required after a reset is requested before OTP is disabled (iOS only. Android returns 0) |


### enableOTP

```javascript
abcAccount.enableOTP(timeout, callback)

// Example
abcAccount.enableOTP(timeout, function (error) {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objc
- (ABCError *)enableOTP:(long)timeout;

// Example
#define OTP_RESET_DELAY (60 * 60 * 24 * 7) // 7 days

ABCError *error;
error = [abcAccount enableOTP:OTP_RESET_DELAY];
```

Sets up OTP authentication on the server for currently logged in user. This will generate a new token if the username doesn't already have one.

| Param | Type | Description |
| --- | --- | --- |
| timeout | `Number` | Number seconds required after a reset is requested before OTP is disabled |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |


### disableOTP

```javascript
abcAccount.disableOTP(callback)

// Example
abcAccount.disableOTP(function (error) {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objc
- (ABCError *)disableOTP;

// Example
ABCError *error = [abcAccount disableOTP];
```

Removes the OTP authentication requirement from the server for the currently logged in user. Also removes local key from device

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

### cancelOTPResetRequest

```javascript
abcAccount.cancelOTPResetRequest(callback)

// Example
abcAccount.cancelOTPResetRequest(function (error) {
    if (error) {
      // Failed
    } else {
      // Yay. Success
    }
})
```

```objc
- (ABCError *)cancelOTPResetRequest;

// Example
ABCError *error = [abcAccount cancelOTPResetRequest];
```

Removes the OTP reset request from the server for the currently logged in user. When a user logs in on a new device for an account with OTP enabled, the login will fail with ABCConditionCodeInvalidOTP. A reset request can then be made using ABCContext.requestOTPReset. cancelOTPResetRequest allows a logged in user to cancel that request and prevent that device from logging in.

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

### signBitIDRequest

```javascript
abcAccount.signBitIDRequest(uri, message, callback)

// Example
abcAccount.signBitIDRequest("bitid://airbitz.co/developer?nonce=12345",
                            "Hello World", function (error, abcSignature) {
  if (!error) {
    console.log("Public address of signature = " + abcSignature.address)
    console.log("Signature = " + abcSignature.signature)
  }
})
```

```objc
- (ABCError *)enableOTP:(long)timeout;

// Example
ABCBitIDSignature *abcSignature;
abcSignature = [abcAccount signBitIDRequest:@"bitid://airbitz.co/developer?nonce=12345"
                                    message:@"Hello World"];
if (!error) {
  NSLog(@"Public address of signature = %@", abcSignature.address)
  NSLog(@"Signature = %@", abcSignature.signature)
}
```

Sign an arbitrary message with a BitID URI. The URI determines the key derivation used to sign the message.

| Param | Type | Description |
| --- | --- | --- |
| url | `String` | URI used to derive the private key to do the signature |
| message | `String` | Message to sign |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| abcSignature | [`ABCBitIDSignature`](#ABCBitIDSignature) | BitID Signature Object |

### createWallet

```javascript
abcAccount.createWallet(walletType, abcWalletKeys)

// Example
const abcWalletKeys = {
  ethereumKey: new Buffer(secureRandom(32)).toString('hex')
}


abcAccount.createWallet("wallet:repo:ethereum",
                        abcWalletKeys,
                        function (error, id) { /* your callback */ })
```

Create a new [ABCWallet](#abcwallet) object and add it to the current account. Each wallet object represents key storage for a specific cryptocurrency type or other misc functionality such as BitID or general data storage. Once a wallet is created, the wallet keys cannot be modified. ABCWallet objects may be shared between ABCAccount objects of the same or different users given permission by the user.

| Param | Type | Description |
| --- | --- | --- |
| walletType | `String` | Wallet type corresponding to the one of the following below |
| callback | `Callback` | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| id | `String` | Strings of wallet ID |

Please seee the [ABCWalletInfo](#abcwalletinfo) documentation for the different wallet types Airbitz understands.

### listWalletIds

```javascript
const walletIds = abcAccount.listWalletIds()
```

Get an array list of wallet IDs in the current account

| Return Param | Type | Description |
| --- | --- | --- |
| walletIds | `Array` | Array of strings of wallet IDs |

### getWallet

```javascript
abcAccount.getWallet(walletId)

// Example
const abcWallet = abcAccount.getWallet(walletId)
```

Get an [ABCWallet](#abcwallet) object given a `walletId`

| Param | Type | Description |
| --- | --- | --- |
| walletId | `String` | Wallet ID from listWallets. `null` if no matching wallet |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | [`ABCWallet`](#abcwallet) | [ABCWallet](#abcwallet) object |

### getFirstWallet

```javascript
abcAccount.getFirstWallet(walletType)

// Example
const abcWallet = abcAccount.getFirstWallet('wallet:repo:ethereum')
```

Get the first [ABCWallet](#abcwallet) object of type `walletType`

| Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| walletType | `String` | Wallet type |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | [`ABCWallet`](#abcwallet) | (Javascript) Error object. `null` if no error |

### changeWalletStates

```javascript
const walletStates = {
  "wallet-id-1": {
    archived: false,
    deleted: false,
    sortIndex: 0
  },
  "wallet-id-2": {
    archived: true,
    sortIndex: 2
  }
}

await account.changeWalletStates(keyStates)
```

Keys (which represent wallets) can have associated metadata. This method makes it possible to change the metadata for one or more wallets.

| Param | Type | Description |
| --- | --- | --- |
| walletStates | `Object` | An object mapping from wallet ID's to the new metadata. All properties are optional, and unspecified properties will remain unchanged. |
| callback | `Callback` | (Javascript) Callback function |

The metadata structure looks like this:

| Property | Type | Description |
| --- | --- | --- |
| archived | `boolean` | Marks the wallet as "archived". Archived wallets are still visible, but do not regularly poll the network. |
| deleted | `boolean` | Marks the wallet as "deleted". Deleted wallets are completely invisible to the user. |
| sortIndex | `number` | Used to place the wallets in a particular order in the user interface. Lower indices should sort first. |

### shareWallet (proposal)

```javascript
abcAccount.shareWallet(abcWallet, abcShareWalletOptions, function(error, shareWalletToken) {
  if (error == null) {
    // Send 'shareWalletToken' to other user
  }
})

// On other device
abcAccount.importWallet(shareWalletToken, function(error, abcWallet) {
  if (error == null) {
    // Success, wallet imported
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcWallet | [`ABCWallet`](#abcwallet) | Wallet to share |
| callback | `Callback` | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| shareWalletToken | `String` | Token to use with [importWallet](#importWallet) |

Provides a key that can given to another user/app to share a wallet with that account.

### importWallet (proposal)

```javascript
abcAccount.shareWallet(abcWallet, abcShareWalletOptions, function(error, shareWalletToken) {
  if (error == null) {
    // Send 'shareWalletToken' to other user
  }
})

// On other device
abcAccount.importWallet(shareWalletToken, function(error, abcWallet) {
  if (error == null) {
    // Success, wallet imported
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| shareWalletToken | `String` | Token to use with [importWallet](#importwallet) |
| callback | `Callback` | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| shareWalletToken | `String` | Token from [shareWallet](#sharewallet) |

Provides a key that can given to another user/app to share a wallet with that account.

## ABCAccountOptions

```javascript
const options = {
  otp: 'PMLCAKJ2IUDQQCIB',

  callbacks: {
    onDataChanged () {
      // Reload any files we care about...
    },
    onLoggedOut () {
      // Return to the login screen...
    }
    // etc...
  }
}

const account = await context.loginWithPassword('user', 'pass', options)
```

Callback routines that notify application when various changes have occurred in the account. This is only utilized for Javascript. For ObjC, see [ABCAccountDelegate](#abcaccountdelegate).

| Property | Type | Description |
| --- | --- | --- |
| otp | `string` | The OTP generation seed. This is only needed when logging into a 2fa-protected account for the first time on a new device. |
| callbacks | `object` | A collection of callback functions for listening to changes in the account state. |

### Callbacks

| Property | Type | Description |
| --- | --- | --- |
| onDataChanged() | `Function` | The account's synced data has changed in any way. If the GUI is storing settings or other information in the account repo, it should refresh those. |
| onKeyListChanged() | `Function` | The user has changed the key list in some way, including adding, deleting, archiving, or sorting. The GUI should refresh its wallets list. |
| onLoggedOut() | `Function` | Account has been logged out. Not used. |
| onOTPRequired() | `Function` | Another device has enabled OTP, and this device does not have the correct OTP token. The GUI should notify the user and give them an opportunity to scan the OTP barcode if they desire. |
| onOTPSkew() | `Function` | This device's clock is out-of-sync with the Airbitz servers. The user should be notified to fix their clock. Otherwise, if the skew gets worse, they may not be able to log in again. |
| onRemotePasswordChange() | `Function` | The account password has been changed by a remote device. The GUI should notify the user and give them an opportunity to log in again, which will activate the change on this device as well. Otherwise, this device will continue to have the old password. |

## ABCBitIDSignature

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| address | `String` | Public address of private key used to sign |
| signature | `String` | Public address of private key used to sign |

## ABCWalletInfo

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin",
  "keys": {
    "bitcoinKey": "K83+b3/fuR4wzIymH3SltCAafJwgW54E/gQh91nmRSo=",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

An `ABCWalletInfo` contains the keys needed to access a wallet or other resource. The Airbitz login system exists to store these wallet keys in an encrypted and backed-up manner.

The Airbitz SDK includes full send & receive capability for a variety of blockchains. If you use these features, you won't need to deal with these wallet keys directly.

Otherwise, if your application does its own blockchain access, keeping your wallet keys in this format will ensure that the Airbitz Wallet application can seamlessly interoperate with the keys your app creates.

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The globally-unique wallet ID. A 256-bit base64-encoded integer. |
| type | `String` | The type of wallet these keys unlock. |
| keys | `Object` | The contents of this object depend on the wallet type. See the documentation for the wallet types below. |

Every wallet has a globally-unique `id`. The simplest approach is to pick a random number for this `id`, using the same entropy source that would be used for private keys.

Some supported wallet types are documented in the sections below:

* [Storage Wallets](#storage-wallets)
* [Account Repos](#account-repos)
* [wallet:bitcoin](#wallet-bitcoin)
* [wallet:bitcoin-bip44](#wallet-bitcoin-bip44)
* [wallet:ethereum](#wallet-ethereum)

### Storage Wallets

Many different wallets include access to a Git repo for storage. In these cases, the `keys` property will have the following members:

| Property | Type | Description |
| --- | --- | --- |
| dataKey | `String` | The data encryption key. A 256-bit base64-encoded integer. |
| syncKey | `String` | The git repo identity. A 160-bit base64-encoded integer. |

For pure storage wallets, the wallet type should be a reverse domain-name starting with `storage:`, like `storage:com.yourdomain.yourtype`.

In the future, Airbitz may introduce a `readKey`, which provides only read-only access to the sync server. This would be something like `sha256(syncKey)`. In that case, the `keys` object would contain one, the other, or both of `syncKey` and `readKey`.

### Account Repos

```javascript
{
  "type": "account-repo:co.airbitz.wallet",
  "id": "91MD0VDrSH6i+Sq6oM+uzIT6CEJlmz3cG3VYDRu+s1Y=",
  "keys": {
    "dataKey": "u0dnSMP+/Qz/CfDpeqloT1HOSlrqM59lIOeMG2Yz44g=",
    "syncKey": "otzd6UewIXAQv+FHOPYVPWzJjZ8="
  }
}
```

| Property | Type | Description |
| --- | --- | --- |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

Account repos have `type` set to `'account-repo:' + appId`. They have storage keys and nothing else; everything interesting is stored in Git. Each app has the freedom to decide what goes in their own account repo.

### wallet:bitcoin

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin",
  "keys": {
    "bitcoinKey": "K83+b3/fuR4wzIymH3SltCAafJwgW54E/gQh91nmRSo=",
    "bitcoinXpub": "xpub6Bk1pSnncGei3JScQNzoZT4Ry6z4Zv4FJPKMNhGiHNmuw6o77oN4sPbmMYwPe2ED7KXxcFx8ZBVnEiDEnBoUHor3jzpYuhJUhLh9wR6Shsx",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

A BIP-32 Bitcoin wallet. The key derivation follows the same scheme specified in the original BIP 32 spec.

| Property | Type | Description |
| --- | --- | --- |
| bitcoinKey | `String` | The master entropy according to the BIP 32 spec. A 256-bit base64-encoded integer. |
| bitcoinXpub | `String` | Key `m/0` in `xpub` format, as defined by the BIP 32 spec |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable bitcoin wallet will have a `bitcoinKey`, while a read-only bitcoin wallet will just have `bitcoinXpub`. Individual private keys are derived using the BIP32 path m/0/0/n where `n` is the index of the private key to derive.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

### wallet:bitcoin-bip44

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin-bip44",
  "keys": {
    "bitcoinKey": "dawn subway charge into unhappy limb kind palace dwarf pear shoulder battery",
    "bitcoinXpub": "xpub6Bk1pSnncGei3JScQNzoZT4Ry6z4Zv4FJPKMNhGiHNmuw6o77oN4sPbmMYwPe2ED7KXxcFx8ZBVnEiDEnBoUHor3jzpYuhJUhLh9wR6Shsx",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

A BIP-44 Bitcoin wallet. The key derivation follows the same scheme specified in the original BIP 44 spec.

| Property | Type | Description |
| --- | --- | --- |
| bitcoinKey | `String` | 12-24 word phrase master private seed as specified in BIP 44 spec |
| bitcoinXpub | `String` | Key `m/0` in `xpub` format, as defined by the BIP 44 spec. |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable bitcoin wallet will have a `bitcoinKey`, while a read-only bitcoin wallet will just have `bitcoinXpub`.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

### wallet:ethereum

```javascript
{
  "type": "wallet:ethereum",
  "id": "lZvB9W1waiDHn52JRzUjfqAbyp1wyN5jreKbdyto4pI=",
  "keys": {
    "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89",
    "dataKey": "X/ovC9WM7EClpjMc/U+0HHQv/cmUlgHFM+drtuiF73Q=",
    "syncKey": "VvijidoeI07d3sYvgyGghwGJBpU="
  }
}
```

An Ethereum wallet.

| Property | Type | Description |
| --- | --- | --- |
| ethereumKey | `String` | A hex-encoded 256-bit Ethereum private key without leading `0x` |
| ethereumAddress | `String` | A hex-encoded Ethereum payment address WITH a leading `0x` |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable ethereum wallet will have an `ethereumKey`, while a read-only ethereum wallet will just have `ethereumAddress`.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

## ABCDataStore

This API is no longer used. Please see [ABCStorageWallet](#abcstoragewallet).

### writeData

```javascript
abcWallet.dataStore.writeData(folder, key, value, callback)

// Example
abcWallet.dataStore.writeData("userAddress",
                              "state",
                              "California",
                              function(error) {
  if (error === null) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| value | `String` | data value |
| callback | `Function` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

Writes a string of data to the specified dataStore folder using the given key. `folder` must not include the characters "/". The dataStore object at this point only provides a single depth folder for storing key/value pairs. Keys can be enumerated using the [ABCDataStore.listKeys](#listkeys) method. Data can read back using the [ABCDataStore.readData](#readdata) method.

### readData

```javascript
abcWallet.dataStore.readData(folder, key, callback)

// Example
abcWallet.dataStore.readData("userAddress",
                             "state",
                             function(error, value) {
  if (error === null) {
    console.log(value)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| value | `String` | data value |

Reads back a string of data from the specified dataStore folder using the given key. `folder` must not include the characters "/". The dataStore object at this point only provides a single depth folder for storing key/value pairs. Keys can be enumerated using the [ABCDataStore.listKeys](#listkeys) method.

### removeKey

```javascript
abcWallet.dataStore.removeKey(folder, key, callback)

// Example
abcWallet.dataStore.removeKey("userAddress",
                              "state",
                              function(error) {
  if (error === null) {
    // Success. Key/value pair removed
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

Removes the specified key/value pair from the dataStore.

### removeFolder

```javascript
abcWallet.dataStore.removeFolder(folder, callback)

// Example
abcWallet.dataStore.removeFolder("userAddress",
                                 function(error) {
  if (error === null) {
    // Success. Key/value pair removed
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

Removes the specified folder from the dataStore. All key/value pairs in the folder will also be removed.

### listKeys

```javascript
abcWallet.dataStore.listKeys(folder, callback)

// Example
abcWallet.dataStore.listKeys("userAddress",
                             function(error, keys) {
  if (error === null) {
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| keys | `Array` | Array of strings of keys in folder |

List the keys in the specified folder

## Errors

```javascript
import { error } from 'airbitz-core-js'

try {
  // Something went wrong!
  throw new error.NetworkError('Cannot reach the Ethereum network')

  // Later...
} catch (e) {
  if (e.name === error.NetworkError.name) {
    console.log('Check your network cable!')
  }
}
```

All Airbitz API's use one of the standard Javascript error-reporting mechanisms:

* Throwing an exception
* Rejecting a promise
* Passing an error to a callback (for Node.js style API's)

Regardless of the reporting mechanism, the returned object will always be an instance of the built-in `Error` object. To determine the exact type of error, use the error object's `name` property.

The `airbitz-core-js` defines a number of error names with special meaning, documented below. The library also exports constructor functions in the `error` namespace, which can be used to create instances of all these error types and to access their name constants in a consistent way.

### DustSpendError

Trying to send an uneconomically small amount of money.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "DustSpendError" |

### InsufficientFundsError

Trying to send more money than the wallet has available.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "InsufficientFundsError" |

### NetworkError

A server was unreachable.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "NetworkError" |

### ObsoleteApiError

The app is attempting to reach a server endpoint that no longer exists, which implies that the app needs to be upgraded.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "ObsoleteApiError" |

### OtpError

The user provided the correct login credentials, but did not provide a 2-factor token.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "OtpError" |
| resetToken | `String` | Use this token to request a 2-factor reset, if the user requests one. |
| resetDate | `Date` | If a 2-factor reset is pending, gives the reset time. |

### PasswordError

The user has entered an invalid password, PIN, or recovery answers.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "PasswordError" |
| wait | `Number` | Number of seconds the user must wait before trying again |

### UsernameError

The user has entered an invalid username, or the username already exists for account creation.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "UsernameError" |

# Wallet API

Once the user has logged in and retrieved their keys, there are serveral things that may take place with those keys, including:

* Accessing encrypted storage
* Sending and receiving digital currencies
* Signing BitId requests

The Airbitz SDK includes several helper classes which perform these capabilities on your behalf, using a set of provided keys.

## ABCStorageWallet

```javascript
// Save a JSON file:
const text = JSON.stringify(settingsJson)
abcStorageWallet.folder.file('settings.json').setText(text)

// Load a JSON file:
const text = await abcStorageWallet.folder.file('settings.json').getText()
const settingsJson = JSON.parse(text)

// Save a binary file:
const image = new Uint8Array(bytes)
abcStorageWallet.folder.file('avatar.png').setData(image)

// Save a text file in a folder:
abcStorageWallet.folder.folder('articles').file('blog.md').setText(article)

// Iterate over the files in a folder:
import { mapFiles } from 'disklet'

mapFiles(abcStorageWallet.folder.folder('articles'), async file => {
  console.log(await file.getText())
})
```

This wallet type handles encrypted and synced data. This is the base class for several other wallet types, including `ABCCurrencyWallet`. You can also use it directly if your application requires multiple encrypted and backed-up data stores. Bear in mind that each account also has its own data store, so you may not need this class unless you are doing something involving key sharing.

See the [Disklet](https://www.npmjs.com/package/disklet) documentation for information on the filesystem API.

### makeStorageWallet

```javascript
const abc = require('airbitz-core-js')

// Select some keys, using any approach you find useful:
const storageWalletKeys = abcAccount.allKeys.find(keyInfo => {
  return keyInfo.type === 'account:myApp'
})

// Access synced data:
abc
  .makeStorageWallet(storageWalletKeys, { account: abcAccount })
  .then(function (abcStorageWallet) {})
```

This function creates an instance of the storage wallet class. 

| Param | Type | Description |
| --- | --- | --- |
| keys | `String` | An [`ABCWalletInfo`](#abcwalletinfo) structure from the account. |
| opts.account | `Object` | The [`ABCAccount`](#abcaccount) object that these keys belong to. |
| opts.onDataChange | `Function` | Called when the data changes as part of a sync operation. |

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The `id` property of the account-level [ABCWalletInfo](#abcwalletinfo) |
| type | `String` | The `type` property of the account-level [ABCWalletInfo](#abcwalletinfo) |
| keys | `Object` | The `keys` property of the account-level [ABCWalletInfo](#abcwalletinfo) |
| name | `String` | Human readable name of wallet. May be `null` if the wallet has no name. |
| folder | `Folder` | A [`Disklet`](https://www.npmjs.com/package/disklet) folder. This datastore object is encrypted by default, backed up to the cloud, and synchronized with any device the user logs into. Data modifications are versioned and can be rolled back but this functionality is not yet exposed via API. dataStore object may `null` if parent [ABCAccount](#abcaccount) has not been logged into yet |
| localFolder | `Folder` | A [`Disklet`](https://www.npmjs.com/package/disklet) folder that only exists on the current device. This data is not encrypted nor backed up. Not to be used for sensitive data but rather as a local cache of network data. Data is not version controlled and has no rollback capability. Common use case will be for local device specific wallet settings, blockchain cache information, and public address cache for use when account/wallet has not yet been decrypted (background processing) |

### renameWallet

```javascript
abcStorageWallet.renameWallet(name, callback)

// Example
abcStorageWallet.renameWallet("My Wallet", function (error) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| name | `String` | New wallet name |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

Returns a list of transactions in the current wallet. Transactions are returned ordered from newest to oldest.

## ABCCurrencyWallet

Each ABCCurrencyWallet represents a single or HD cryptocurrency wallet tied to one specific blockchain such as bitcoin or ethereum. Various methods and fields in ABCCurrencyWallet are arrays or accept an index which selects which token system in the wallet is being referenced.

This class also includes all the methods and callbacks of [`ABCStorageWallet`](#abcstoragewallet).

### makeCurrencyWallet

```javascript
const abc = require('airbitz-core-js')
const abcBitcoin = require('airbitz-currency-bitcoin')

// This should probably happen once when the app first boots:
const bitcoinPlugin = abcBitcoin.makeBitcoinPlugin({
  io: yourPlatformSpecificIo
})

function onAddressesChecked (progressRatio) {}
function onBalanceChanged (balance) {}
function onTransactionsChanged (transactionArray) {}
function onNewTransactions (transactionArray) {}
function onBlockHeightChanged (height) {}

// Select some keys, using any approach you find useful:
const bitcoinWalletKeys = abcAccount.allKeys.find(
  keyInfo => keyInfo.type === 'wallet:bitcoin'
)

// Access synced data and transactions:
abc
  .makeCurrencyWallet(bitcoinWalletKeys, {
    account: abcAccount,
    plugin: bitcoinPlugin,
    callbacks: {
      onAddressesChecked,
      onBalanceChanged,
      onTransactionsChanged,
      onNewTransactions,
      onBlockHeightChanged
    }
  })
  .then(abcBitcoinWallet => {})
```

Creates a wallet capable of send and receive functionality. An [ABCCurrencyPlugin](#abccurrencyplugin) object must be passed in that exposes the entire [ABCCurrencyPlugin](#abccurrencyplugin) interface.

| Param | Type | Description |
| --- | --- | --- |
| keys | [`ABCWalletInfo`](#abcwalletinfo) | The wallet info obtained from the account |
| opts.account | `ABCAccount` | The account object this wallet belongs to. |
| opts.plugin | `ABCCurrencyPlugin` |  Object that exposes the [ABCCurrencyPlugin](#abccurrencyplugin) functions |
| opts.callbacks | `ABCCurrencyPlugin` | `Object` | Object with callback functions |

(Javascript) Returns a `Promise` for an ABCCurrencyWallet type.

| Callback name | Type | Description |
| --- | --- | --- |
| onAddressesChecked(progressRatio) | `Function` | Called as the engine checks addresses. If money has arrived while the app was shut down, this process will eventually detect it. The balance and history may be incomplete until `progressRatio` reaches 100%. |
| onBalanceChanged(balance) | `Function` | The spendable balance has changed, either from newly-detected transactions or from dropped transactions. |
| onBlockHeightChanged(height) | `Function` | Blockchain height changed. This is unused for sub wallets. The confirmation status of each transaction is `chainHeight - transactionHeight`, so this affects all confirmed transactions. |
| onNewTransactions(transactionArray) | `Function` | Array of ABCTransaction objects. These transactions are either previously-recognized funds that are being synced to this device for the first time, or updates to previously-seen transactions such as new metadata or confirmation status. |
| onTransactionsChanged(transactionArray) | `Function` | Array of never-before-seen ABCTransaction objects. These are new funds that the GUI should notify the user about. |
| onWalletNameChanged(name) | `Function` | The user has changed the wallet name, either on this device or on another device. |

This class also includes all callbacks of [`ABCStorageWallet`](#abcstoragewallet).

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| fiatCurrencyCode | `String` | 3 character fiat currency code |
| cryptoCurrencyCodes | `Array` | Array of Strings of crypto currency code. ie "BTC" |
| primaryCryptoCurrencyCode | `String` | String of primary crypto currency code. ie "BTC". Other codes in `cryptoCurrencyCodes` will reference meta-tokens of this currency's blockchain. |

### setupContract

(under construction)

```javascript
// Example
var abcSetupContractOptions = [
  { addMasterPublicKey: "xpub12fe094ab5d830ee9ac0102fbd223e" },
  { addMasterPublicKey: "xpubef92309571209798ab098ce0912fae" },
  { addMasterPublicKey: "xpuba39f948e0d0139bc99848cb10a0935" },
  { numRequiredKeys: 2 }
]

abcCurrencyWallet.setupContract(abcSetupContractOptions)
```

Setup the script/contract for this wallet. This is used to setup basic multisig wallets under currencies like bitcoin and ethereum.

### getBalance

```javascript
abcCurrencyWallet.getBalance(opts)

// Example
const balance = abcCurrencyWallet.getBalance({ curencyCode: "BTC" })
```
| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | May be `null` which will return the balance of the wallet's primary currency. |

| Option | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Selects which currency to return a balance for. |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | `Int` | Wallet balance in smallest unit of currency. ie Satoshis |

Gets the current balance of the wallet denominated in the smallest unit of the currency. ie. Unit of satoshis for the currency bitcoin.

### getTransactions

```javascript
abcCurrencyWallet.getTransactions(options, callback)

// Example
// Create query that looks in the first 100 transactions filtered to the past 2 days that have meta data matching the string "Mom". Then returns the first 10 of those transactions
var end = new Date();
var start = new Date();
start.setDate(end.getDate() - 1);

const options = {
  currencyCode: "BTC",
  startIndex: 0,
  numEntries: 100,
  startDate: start,
  endDate: end,
  searchString: 'Mom',
  returnIndex: 0,
  returnEntries: 10
}

const abcTransactions = abcCurrencyWallet.getTransactions(options, function(error, abcTransactions) {
  const abcTransaction = abcTransactions[0]
})
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | May be `null` which will return all transactions |

| Option Params | Type | Description |
| --- | --- | --- |
| startIndex | `Int` | Index into the full list of transactions. If unspecified, no transactions are filtered and search passes on to `startDate`. Index 0 refers to the most recent transaction. |
| startEntries | `Int` | Number of entries to return from start of index. `startIndex` must be specified |
| startDate | `Date` | Date object, in local time, when to start returning transactions. If unspecified, transactions are not filtered by date and search passes on to `searchString` |
| endDate | `Date` | Date object, in local time, when to stop returning transactions. Must be later than `startDate` and `startDate` must be specified |
| searchString | `String` | Include only transactions that have metadata that matches `searchString`. (Optional) |
| returnIndex | `Int` | Index into the filtered list of transactions. If unspecified, no transactions are filtered and current results are returned.  Index 0 refers to the most recent transaction. |
| returnEntries | `Int` | Number of entries to return from index of filtered transactions. `returnEntries` must be specified |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| transactions | `Array` | Array of [ABCTransaction](#abctransaction) objects |

Returns a list of transactions in the current wallet. Options allow pruning of the search to a subset of the transactions in addition to string filtering. Options are applied in the following order:

Note that all indices start with the most recent transaction and work downwards to oldest.

1. `startIndex` and `startEntries` specify the starting index and number of entries from the full, un-filtered list of transactions.
2. `startDate` and `endDate` are applied to filter all transactions to only within specified dates.
3. `searchString` is applied to only include string matching transactions
4. `startIndex` and `numEntries` specify the starting index and number of entries from the Array created from steps 1 through 3. They do NOT refer to the Array of all transactions.

Transactions are returned ordered from newest to oldest.

### saveTxMetadata

```javascript
abcCurrencyWallet.saveTxMetadata(txid, currencyCode, metadata)

// Example
const txid = someTransaction.txid
const metadata = {
  name: "City Tacos",
  category: "Expense:Food & Dining"
}

abcCurrencyWallet.saveTxMetadata(txid, "BTC", metadata).then(() => {
  console.log("Transaction successfully updated")
})
```

| Param | Type | Description |
| --- | --- | --- |
| txid | `String` | The txid of an [`ABCTransaction`](#abctransaction) object |
| currencyCode | `String` | Which crypto-currency this metadata applies to, for wallets with multiple token support. |
| metadata | [`ABCMetadata`](#abcmetadata) | The metadata to save with the transaction. |
| callback | `Callback` | Called when the save is completed. |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

Updates the metadata saved with a transaction.

### getBlockHeight

```javascript
abcCurrencyWallet.getBlockHeight()

// Example
const height = abcCurrencyWallet.getBlockHeight()
```

| Param | Type | Description |
| --- | --- | --- |
| void | `Void` | none |

| Return Param | Type | Description |
| --- | --- | --- |
| height | `Int` | Block height of current wallet |

Gets the current blockchain height of the wallet's cryptocurrency. This is expected to be the same value for all meta-tokens in this wallet and hence does not require a currencyCode index.

### getReceiveAddress

```javascript
abcCurrencyWallet.getReceiveAddress(options, callback)

// Example to simply return an unused address
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(null, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE"
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress({'addressTag': 'QRCODE'}, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE" and for the currency "REP"
const options = {
  'addressTag': 'QRCODE',
  'currencyCode': 'REP'
}
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(options, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to get an ABCRequestAddress object for BTC currency and previous publicAddress '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
const options = {
  currencyCode: "BTC",
  publicAddress: '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
}

const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(options, , function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| options |  `Object` | Options object. See below
| callback | `Callback` | (Javascript) Callback function |

| Options Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token to get an address for. If not specified, uses the primary currency of this wallet |
| publicAddress | `String` | (Optional) Get an ABCReceiveAddress object using a previously returned public address. If `publicAddress` is set, `addressTag` must be null or unspecified |
| addressTag | `String` | (Optional) Arbitrary tag for this specific address request type. Future calls to getReceiveAddress with the same tag will return the same address as the previous call with the same tag unless that address has received funds. Useful for specifying a "display" address which is shown on screen but never used for email or SMS requests. Calling [lockReceiveAddress](#lockreceiveaddress) will cause this address to no longer be returned regardless of whether it has received funds. If `addressTag` is set, `publicAddress` must be null or unspecified |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| abcReceiveAddress | `ABCReceiveAddress` | [ABCReceiveAddress](#abcreceiveaddress) |

Returns an [ABCReceiveAddress](#abcreceiveaddress) object. This routine is used to generate a new public address or return a previously request address object.  The `metadata` object

### saveReceiveAddress

```javascript
abcCurrencyWallet.saveReceiveAddress(abcReceiveAddress, callback)

// Example
abcCurrencyWallet.getReceiveAddress(null, function (error) {
  if (!error) {
    // Success
    console.log("My bitcoin address: " + abcReceiveAddress.publicAddress)
    abcReceiveAddress.nativeAmount = '150000000' // 1.5 BTC
    abcReceiveAddress.metadata.name = "Johnny Be Good"
    abcReceiveAddress.metadata.category = "Income:Rent"
    abcReceiveAddress.saveReceiveAddress(function(error) {
      // Meta data for this address has been saved
    })
  }
})
```

Updates the internal database of `metadata` corresponding to this `receiveAddress`. Any transactions that are received in this address are automatically tagged with the `metadata` from this `receiveAddress`.

### lockReceiveAddress

```javascript
abcCurrencyWallet.lockReceiveAddress(callback)

// Example
abcCurrencyWallet.lockReceiveAddress(function (error) {
  if (!error) {
    // Success
  }
})
```

Locks this `receiveAddress` so that any future calls to [ABCCurrencyWallet.getReceiveAddress](#getreceiveaddress), without the `publicAddress` specified, will no longer return this address. Funds can still be received on this address. Use [ABCCurrencyWallet.getReceiveAddress](#getreceiveaddress) with `publicAddress` specified to get back this object in the future.

### parseUri

```javascript
const abcParsedUri = wallet.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X&label=Mom")

console.log(abcParsedUri.publicAddress) // -> 1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs
console.log(abcParsedUri.nativeAmount) // -> '123456700'
console.log(abcParsedUri.metadata.name) // -> 'Mom'
console.log(abcParsedUri.paymentProtocolURL) // -> https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X
```

| Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`ABCParsedUri`](#abcparseduri) | Object with parsed parameters |

Parses a URI extracting various elements into an [ABCParsedUri](#abcparseduri) object

### encodeUri

```javascript
// Example
const abcParsedUri = {
  publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
  nativeAmount: '123456700',
  metadata: {
    name: 'Mom'
  },
  paymentProtocolURL: 'https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X'
}

const uri = wallet.encodeUri(abcParsedUri)

console.log(uri)
// ->  "bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&label=Mom&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

```

| Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`ABCParsedUri`](#abcparseduri) | Object with parsed parameters |

| Return Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI output |

Encodes a URI given a [ABCParsedUri](#abcparseduri) object


### makeAddressQrCode

```javascript
const qrCode = makeAddressQrCode(abcReceiveAddress)
```

| Return | Type | Description |
| --- | --- | --- |
| qrCode | `String` | Base64 encoded image of addressUri |

### makeAddressUri

```javascript
const uri = makeAddressUri(abcReceiveAddress)
```

| Return | Type | Description |
| --- | --- | --- |
| addressUri | `String` | BIP21 or equivalent URI that encodes public address and optionally requested amount, name of requestor, and category of requested funds |

### makeSpend

```javascript
// Example to spend to two bitcoin addresses
const abcSpendInfo = {
  networkFeeOption: 'high',
  metadata: {
    name: 'Tyra CPA',
    category: 'Expense:Professional Services'
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
      nativeAmount: '210000000' // 2.1 BTC
    },
    {
      currencyCode: 'LTBCOIN',
      publicAddress: '1FSxyn9AbBMwGusKAFqvyS63763tM8KiA2',
      nativeAmount: '120' // 120 LTBCOIN
    }
  ]
}

abcCurrencyWallet.makeSpend(abcSpendInfo).then(abcTransaction => {
  abcCurrencyWallet.signBroadcastAndSave(abcTransaction).then(() => {
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  })
})

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, spendTarget) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      name: 'Tyra CPA',
      category: 'Expense:Professional Services'
    },
    spendTargets: [ spendTarget ]
  }

abcCurrencyWallet.makeSpend(abcSpendInfo).then(abcTransaction => {
  abcCurrencyWallet.signBroadcastAndSave(abcTransaction).then(() => {
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  })
})

// Example wallet to wallet transfer
const walletIds = abcAccount.listWalletIds()
const srcWallet = abcAccount.getWallet(walletId[0]) // Add check for null and correct wallet type
const destWallet = abcAccount.getWallet(walletId[1]) // Add check for null and correct wallet type

const abcSpendInfo = {
  networkFeeOption: 'high',
  metadata: {
    name: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund'
  },
  spendTargets: [
    {
      destWallet,
      nativeAmount: '210000000' // 2.1 BTC
    }
  ]
}

abcCurrencyWallet.makeSpend(abcSpendInfo).then(abcTransaction => {
  abcCurrencyWallet.signBroadcastAndSave(abcTransaction).then(() => {
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  })
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcSpendInfo | [`ABCSpendInfo`](#abcspendinfo) | [ABCSpendInfo](#abcspendinfo) object with various parameters for a spend operation including output addresses, amounts, or payment protocol payment objects (BIP70) |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Unsigned [ABCTransaction](#abctransaction) object |

Creates an unsigned [ABCTransaction](#abctransaction) object which can be then be signed and broadcast to the network. Complete the spend by calling [ABCTransaction.signBroadcastAndSave](#signbroadcastandsave). Estimated fees can be determined by reading back [ABCTransaction.networkFee](#abctransaction)

May produce an [InsufficientFundsError](#insufficientfundserror) if the amount is too large, or a [DustSpendError](#dustspenderror) if the amount is too small.

### signTx

```javascript
abcCurrencyWallet.signTx(abcTransaction, callback)

// Example
abcCurrencyWallet.signTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction signed
    console.log("Signed transaction with txId = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Unsigned [ABCTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

Signs this [ABCTransaction](#abctransaction) object. Does not broadcast this to the blockchain or save it in the local transaction cache. [ABCTransaction](#abctransaction).txid is null until this routine is called.

Call [ABCTransaction.broadcastTx](#broadcasttx) followed by [ABCTransaction.saveTx](#savetx) to broadcast and save the transaction.

### broadcastTx

```javascript
abcCurrencyWallet.broadcastTx(abcTransaction, callback)

// Example
abcCurrencyWallet.broadcastTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Signed [ABCTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

Broadcasts transaction to the blockchain.

### saveTx

```javascript
abcCurrencyWallet.saveTx(abcTransaction, callback)

// Example
abcCurrencyWallet.saveTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction saved
    console.log("Saved transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Signed [ABCTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

Saves transaction to local cache. This will cause the transaction to show in calls to [abcCurrencyWallet.getTransactions](#gettransactions).

### signBroadcastAndSave

```javascript
abcCurrencyWallet.signBroadcastAndSave(abcTransaction, callback)

// Example
abcCurrencyWallet.signBroadcastAndSave(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Unsigned [ABCTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

Convenience routine to do `signTx`, `broadcastTx`, then `saveTx` in one call.

### getPaymentProtocolInfo

```javascript
// Example
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, paymentProtocolInfo) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      name: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  const abcTransaction = abcCurrencyWallet.makeSpend(abcSpendInfo)
  abcCurrencyWallet.signBroadcastAndSave(abcTransaction, function(error, abcTransaction) {
    if (!error) {
      // Success, transaction sent
      console.log("Sent transaction with ID = " + abcTransaction.txid)
    }
  })
}
```

| Param | Type | Description |
| --- | --- | --- |
| paymentProtocolURL | `String` | Payment protocol URL |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |
| abcPaymentProtocolInfo | [`ABCPaymentProtocolInfo`](#abcpaymentprotocolinfo) | [ABCPaymentProtocolInfo](#abcpaymentprotocolinfo) object |

Communicates over network with BIP70 payment request URL to get exact payment parameters. This returns a [ABCPaymentProtocolInfo](#abcpaymentprotocolinfo) which contains a custom, non-editable [ABCSpendTarget](#abcspendtarget) that can simply be placed into an [ABCSpendInfo](#abcspendinfo) object to send the transaction.


### getMaxSpendable

```javascript
// Example
const abcSpendInfo = {
  networkFeeOption: 'standard',
  metadata: {
    name: 'Tyra CPA',
    category: 'Expense:Professional Services',
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
    }
  ]
}

abcCurrencyWallet.getMaxSpendable(abcSpendInfo, function(error, maxNativeAmount) {
  if (error === null) {
    console.log(maxNativeAmount)
  }
})
```

Get the maximum amount spendable from this wallet given the parameters of an [ABCSpendInfo](#abcspendinfo) object. The [ABCSpendInfo.spendTargets](#abcspendtarget) `nativeAmount` values are ignored when calculating the max spendable amount. This only ever returns the max spendable of the primary currency of the wallet. Any meta-tokens of the wallet will always have a max spendable equal to the number of meta-tokens in the wallet determined by [getBalance](#getbalance).

### sweepPrivateKey

coming soon...

### exportTransactionsToCSV

coming soon...

### exportTransactionsToQBO

coming soon...

## ABCSpendInfo

```javascript
// Example spend info to transfer from wallet to wallet
const walletIds = abcAccount.listWalletIds()
const srcWallet = abcAccount.getWallet(walletId[0]) // Add check for null and correct wallet type
const destWallet = abcAccount.getWallet(walletId[1]) // Add check for null and correct wallet type

abcSpendInfo = {
  networkFeeOption: 'high',
  metadata:  {
    name: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund',
  },
  spendTargets: [
    {
      destWallet,
      nativeAmount: '210000000' // 2.1 BTC
    },
  ]
}
```

Parameters

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token to spend from. If not specified, uses the primary currency of this wallet |
| noUnconfirmed | `Boolean` | (Optional) If set to TRUE, this will not spend from any unconfirmed funds. Default is FALSE |
| spendTargets | `Array` | Array of [ABCSpendTarget](#abcspendtarget) objects |
| networkFeeOption | `String` | Adjusts network fee amount. Must be either "low", "standard", "high", or "custom". If unspecified, the default is "standard" |
| customNetworkFee | `String` | Amount of per byte network fee if `networkFeeOption` is set to `custom`. Should be specified as smallest denomination of currency, as a string (i.e. Satoshis or Wei) |
| metadata | [`ABCMetadata`](#abcMetadata) | [ABCMetadata](#abcMetadata) object. Outgoing transaction will have the specified metadata copied to the [ABCTransaction](#abctransaction) object |

Parameter object used for creating an [ABCSpend](#abcspend) object.

## ABCSpendTarget

```javascript
// Example spend target with a public addresses
const spendTarget =
  {
    currencyCode: 'BTC',
    publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
    nativeAmount: '210000000' // 2.1 BTC
  }

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, paymentProtocolInfo) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      name: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  abcCurrencyWallet.makeSpend(abcSpendInfo).then(abcTransaction => {
    abcCurrencyWallet.signBroadcastAndSave(abcTransaction).then(() => {
      console.log("Sent transaction with ID = " + abcTransaction.txid)
    })
  })
}
```

Parameters

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token type for the destination receiving address or wallet. If not specified, uses the primary currency of this wallet |
| publicAddress | `String` | Public address in the format of the current wallet's currency. This requires the `nativeAmount` field to be set. Must not set both `publicAddress` and `destWallet` |
| nativeAmount | `String` | Amount to send in the smallest denomination of the source wallet's currency, as a string. ie Satoshis or Wei |
| destWallet | [`ABCWallet`](#abcwallet) | Destination wallet to transfer funds to. Must also set `nativeAmount`. Must not set both `publicAddress` and `destWallet` |
| destMetadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) object with which will tag the transaction in the destination wallet. Must only be used when `destWallet` is set. |


## ABCParsedUri

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Public address from URI |
| privateKey | `String` | Private key |
| bitIDURI | `String` | Full BitID URI |
| bitIDDomain | `String` | Domain name portion of BitID URI |
| bitIDCallbackUri | `String` | BitID Callback URI |
| paymentProtocolUri | `String` | BIP70 Payment Request URL |
| nativeAmount | `String` | Amount in the currency's smallest denomination, as a string (i.e. Satoshis or Wei)
| metadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) object with info extracted from URI |
| returnUri | `String` | URI to send user after URI/payment has been processed |
| bitidPaymentAddress | `Bool` | True if BitID URI is requesting a payment address (experimental)|
| bitidKycProvider | `Bool` | True if BitID URI would like to provide KYC token (experimental) |
| bitidKycRequest | `Bool` | True if BitID URI is requesting a KYC token (experimental) |


Object contains various parts of a parsed URI depending on the source URI. Any of the properties may be `null`.

## ABCPaymentProtocolInfo

Object provides basic UI displayable info about a BIP70 payment request. Also includes an [ABCSpendTarget](#abcspendtarget) for use in an [ABCSpendInfo](#abcspendinfo) to send the transaction.

| Property | Type | Description |
| --- | --- | --- |
| domain | `String` | DNS name of originator of request |
| nativeAmount | `String` | Amount of request, as a string |
| memo | `String` | Memo field returned by merchant |
| merchant | `String` | Name of merchat (may be blank) |
| abcSpendTarget | [`ABCSpendTarget`](#abcspendtarget) | [ABCSpendTarget](#abcspendtarget) object that can be used in an [ABCSpendInfo](#abcspendinfo) |

## ABCReceiveAddress

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Raw public address in native format of wallet currency type. (ie. base58 for bitcoin, base16 for ethereum) |
| nativeAmount | `String` | Amount of request denominated in the smallest unit of this wallet's currency, as a string (i.e. Satoshis or Wei) |
| metadata | `ABCMetadata` | [ABCMetadata](#abcmetadata) object corresponding to this address. Any transactions receiving funds into this address will automatically have this metadata in the [ABCTransaction](#abctransaction) object.

## ABCMetadata

Non-blockchain transaction meta data associated to an [ABCTransaction](#abctransaction) or to an [ABCReceiveAddress](#abcreceiveaddress)

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | Name of external recipient or sender of funds |
| category | `String` | Transaction category of format "Expense:Food & Dining". Category must be of the form [Category]:[Sub Category] where category is one of "Income", "Expense", "Transfer", or "Exchange". Income refers to incoming funds such as payroll or business sales. Expense is the purchase of goods or services. Transfer is a transfer of funds to/from another wallet or exchange account owned by the user. Exchange is the change of funds from one type of currency to another. If abcCurrencyWallet is of type bitcoin, an incoming transaction from the purchase of bitcoin with USD should be categorized as "Exchange:Buy Bitcoin". |
| notes | `String` | Misc notes field |
| amountFiat | `Float` | Amount of transaction in the wallet's fiat currency at the time of the transaction |
| bizId | `Int` | Unique bizId associated to a business listing in the Airbitz Business Directory |
| miscJson | `String` | Generic JSON string that can be used for additional meta data |

## ABCTransaction

Object represents a signed or unsigned transaction that may or may not be broadcast to the blockchain.

| Property | Type | Description |
| --- | --- | --- |
| wallet | [`ABCCurrencyWallet`](#abccurrencywallet) | [ABCCurrencyWallet](#abccurrencywallet) this transaction is from |
| currencyCode | `String` | Which blockchain currency or token this transaction applies to. |
| metadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) of this transaction |
| txid | `String` | Transaction ID as represented by the wallet's crypto currency. For bitcoin this is base16. This parameter is `null` until [signTx](#signtx) is called. |
| date | `Date` | Date that transaction was broadcast, detected, or confirmed on the blockchain. If the tx detection date is after the confirmation time, then this is the confirmation time. `null` if transaction has not been broadcast |
| blockHeight | `Int` | Block number that included this transaction |
| nativeAmount | `String` | Amount of transaction in denomination of smallest unit of currency, as a string. Incoming funds are positive numbers. Outgoing funds are negative. Outgoing amounts should include the providerFee and networkFee. In the case of a transaction that both spends from the current wallet to the current wallet, the amount should be the net effect on the balance of the wallet. |
| providerFee | `String` | Additional app provider fees in denomination of smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| networkFee | `String` | Fee paid to network (mining fee) in denomination of smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| runningBalance | `String` | Running balance of entire wallet as of this transaction in the smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| signedTx |`Array` | Buffer of signed transaction data with signature. `null` if transaction is unsigned |
| ourReceiveAddresses | `Array<string>` | Contains a list of addresses that belong to this wallet and have received money in this transaction. |
| otherParams |`Object` | Crypto currency specific data |

`otherParams` has the following parameters for bitcoin wallets

| Property | Type | Description |
| --- | --- | --- |
| isReplaceByFee | `Bool` | True if this transaction is marked as RBF (BIP125) |
| isDoubleSpend | `Bool` | True if this transaction is found to be a double spend attempt |
| inputOutputList | `Array` | Array of transaction inputs and outputs |

## ABCExchangeRateCache

### addSource

```javascript
const sourceBitstamp = require('airbitz-core-js-bitcoin').exchangeRateSources.bitstamp
const sourceCoinbase = require('airbitz-core-js-bitcoin').exchangeRateSources.coinbase
const sourceBitcoinAverage = require('airbitz-core-js-bitcoin').exchangeRateSources.bitcoinaverage
const sourceBraveNewCoin   = require('airbitz-core-js-bitcoin').exchangeRateSources.bravenewcoin

const abcExchangeRateCache = new ABCExchangeRateCache()
abcExchangeRateCache.addSource(sourceBitstamp)
abcExchangeRateCache.addSource(sourceCoinbase)
abcExchangeRateCache.addSource(sourceBitcoinAverage)
abcExchangeRateCache.addSource(sourceBraveNewCoin)
```

Adds an exchange rate source. Initialize with object of type [ABCExchangeRateLib](#abcexchangeratelib). `airbitz-core-js-bitcoin` includes support for Bitstamp.com, Coinbase.com, BitcoinAverage.com, and BraveNewCoin.com

### convertCurrency

```javascript
destinationCurrencyAmount = convertCurrency(amount, sourceCurrency, destinationCurrency)

// Example
const priceofBitcoin = convertCurrency(1, "BTC", "USD")
```

| Param | Type | Description |
| --- | --- | --- |
| amount | `Float` | Amount of source currency to convert |
| sourceCurrency | `String` | 3 character currency code of source currency |
| destinationCurrency | `String` | 3 character currency code of destination currency |

| Return | Type | Description |
| --- | --- | --- |
| destinationAmount | `Float` | Amount of destination currency after conversion |

Converts one currency value into another using exchange rate cache. Returns 0 if the currency pair cannot be converted due to missing support from exchange rate sources or if sources cannot be reached.

# Core Plugin API

```javascript
import { coinbasePlugin } from 'airbitz-exchange-plugins'
import { ethereumPlugin } from 'airbitz-currency-ethereum'

const context = makeContext({
  plugins: [coinbasePlugin, ethereumPlugin]
})
```

The Airbitz core library accepts an array of plugins in its `createContext` function. These plugins can provide the following functionality:

* Send and receive new currencies
* Lookup exchange rates from new sources

All core plugins begin in a common format, `ABCCorePlugin`, which describes the type of the plugin and provides an initialization funtion. The initialization function returns either the `ABCCurrencyPlugin` or an `ABCExchangePlugin` object that implements the actual functionality.

## ABCCorePlugin

```javascript
export const myPlugin = {
  pluginType: 'exchange',

  async makePlugin(opts) {
    return new MyExchangePlugin(opts.io)
  }
}
```

This is a the basic wrapper type for all plugins.

### pluginType

Either `exchange` or `currency`

### makePlugin

Returns either an `ABCCurrencyPlugin` or `ABCExchangePlugin` object instance, depending on the `pluginType` property described above. This is an async method.

| Param | Type | Description |
| --- | --- | --- |
| opts | `Object` | Options for the plugin |

The options are as follows:

| Property | Type | Description |
| --- | --- | --- |
| io | `Object` | Platform-specific resources |

| Return | Type | Description |
| --- | --- | --- |
| plugin | `Promise<ABCCurrencyPlugin or ABCExchangePlugin>` | A promise that resolves to the inner plugin instance |

Note: The `io` object passed in the `options` structure contains a `folder` member. This is a [Disklet](https://www.npmjs.com/package/disklet) folder, and can be used to store app-wide information. A currency plugin might use this forlder to store generic blockchain information such as the last block height or supported tokens, for example.

## ABCCurrencyPlugin

```javascript
// SDK users should do this:
import { EthereumCurrencyPluginFactory } from 'airbitz-currency-ethereum'

const context = makeContext({
  plugins: [ EthereumCurrencyPluginFactory ]
})

// Inside the `makeContext` function, this is what happens:
const currencyPlugin = await ethereumPlugin.makePlugin({ io })
```

Cryptocurrency functionality for [`ABCCurrencyWallet`](#abccurrencywallet) is provided by currency plugins. These begin life as a generic [ABCCorePlugin](#abccoreplugin) that returns an `ABCCurrencyPlugin` instance from is `createPlugin` method.

The `ABCCurrencyPlugin` object provides all the functionality needed to integrate the currency into the user interface, manage keys, and handle URI's. It also provides the ability to create `ABCTxEngine` objects, which provide the send, receive, and transaction history functionality for individual wallets.

### pluginName

Simple unique string representing the currency supported by this plugin. Will be used for comparison of uniqueness
against other plugins.

ie.

```javascript
console.log(currencyPlugin.pluginName) // => "ethereum"
```

### currencyInfo

```javascript
console.log(currencyPlugin.currencyInfo)
"
{
  walletTypes: [
    'wallet:ethereum'
  ],
  currencyName: 'Ethereum',
  currencyCode: 'ETH', // The 3 character code for the currency
  explorerAddress: 'https://etherscan.io/address/%s',
  explorerTransaction: 'https://etherscan.io/tx/%s',
  denominations: [
    // An array of Objects of the possible denominations for this currency
    {
      name: 'ETH',
      multiplier: '1000000000000000000',
      symbol: 'Ξ'
    }
  ],
  defaultSettings: {
    etherscanApiServers: [
      'https://api.etherscan.io'
    ],
    superethServers: [
      'http://supereth1.airbitz.co:8080'
    ]
  },
  symbolImage: 'data:image/png;base64,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', // Base64 encoded png image of the currency symbol (optional)
  metaTokens: [
    // Array of objects describing the supported metatokens
    {
      currencyCode: 'REP',
      currencyName: 'Augur',
      denominations: [
        {
          name: 'REP',
          multiplier: '1000000000000000000'
        }
      ],
      contractAddress: '0xE94327D07Fc17907b4DB788E5aDf2ed424adDff6',
      symbolImage: 'data:image/png;base64,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'
    },
    {
      currencyCode: 'WINGS',
      currencyName: 'Wings',
      denominations: [
        {
          name: 'WINGS',
          multiplier: '1000000000000000000'
        }
      ],
      contractAddress: '0x667088b212ce3d06a1b553a7221E1fD19000d9aF'
    }
  ]
}
"
```

The `currencyInfo` property should be an object with the following properties describing the currency:

| Property | Type | Description |
| --- | --- | --- |
| currencyName | `String` | Human readable string of the currency name. ie "Ethereum" |
| currencyCode | `String` | The 3 character code for the currency |
| addressExplorer | `String` | Full URL of explorer to give info on a specific address. ie. `https://etherscan.io/address/%s` |
| transactionExplorer | `String` | Full URL of explorer to give info on a specific tx. ie. `https://etherscan.io/tx/%s` |
| denominations | `Array` | An array of AbcDenomination objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol (optional) |
| metaTokens | `Array` | Array of AbcMetaToken objects describing the supported metatokens |
| walletTypes | `Array` | Array of strings listing the wallet types that this currency can handle, such as `wallet:ethereum`. Please see the [ABCWalletInfo](#abcwalletinfo) documentation for information about these types. The `makeEngine`, `createPrivateKey`, and `derivePublicKey` methods can handle keys with these types. |
| defaultSettings | `object` | Default per-currency settings. This acts as a template for the settings that should be passed to [`makeEngine`](#makeengine) and [`updateSettings`](#updatesettings). Optional. |

The `AbcDenomination` object includes the following properties:

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | The human readable string to describe the denomination. |
| multiplier | `Int` | The value to multiply the smallest unit of currency to get to the denomination. |
| symbol | `String` | The human readable 1-3 character symbol of the currency. ie. "Ƀ" |
| font | `String` | (Optional) The font required to display the symbol specified above. If not given, will use the default system font. |

The `AbcMetaToken` array includes the following properties:

| Property | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | The human readable string to describe the denomination. |
| denominations | `Array` | An array of AbcDenomination objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol (optional) |

### parseUri

```javascript
const abcParsedUri = currencyPlugin.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X&label=Mom")

console.log(abcParsedUri.publicAddress) // -> 1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs
console.log(abcParsedUri.nativeAmount) // -> '123456700'
console.log(abcParsedUri.metadata.name) // -> 'Mom'
console.log(abcParsedUri.paymentProtocolURL) // -> https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X
```

| Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`ABCParsedUri`](#abcparseduri) | Object with parsed parameters |

Parses a URI extracting various elements into an [ABCParsedUri](#abcparseduri) object

### encodeUri

```javascript
// Example
const abcParsedUri = {
  publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
  nativeAmount: '123456700',
  metadata: {
    name: 'Mom'
  },
  paymentProtocolURL: 'https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X'
}

const uri = currencyPlugin.encodeUri(abcParsedUri)

console.log(uri)
// ->  "bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&label=Mom&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

```

| Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`ABCParsedUri`](#abcparseduri) | Object with parsed parameters |

| Return Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI output |

Encodes a URI given a [ABCParsedUri](#abcparseduri) object

### createPrivateKey

```javascript
// Example
const walletKey = currencyPlugin.createPrivateKey('wallet:ethereum')

// Output:
{
  "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89"
}
```

Creates a new random master private key. The returned object will be used as the `keys` member of an [`ABCWalletInfo`](#abcwalletinfo) structure. Please see [`ABCWalletInfo`](#abcwalletinfo) for documentation on the various key types Airbitz understands. If your currency is not documented in that section, please submit a pull request to add your format to [the documentation](https://github.com/Airbitz/airbitz-docs/tree/full-api-docs).

| Param | Type | Description |
| --- | --- | --- |
| type | `string` | The type of wallet to create. See [`ABCWalletInfo`](#abcwalletinfo) for valid wallet types. |

The core will treat the returned object as the `keys` member of an [`ABCWalletInfo`](#abcwalletinfo) object. The core will generate a random `id`, and will use the same `type` as passed to `createPrivateKey`. The core will also insert the `dataKey` and `syncKey`, so the currency plugin does not need to worry about those either.

### derivePublicKey

```javascript
// Example
const walletInfo = {
  "type": "wallet:ethereum",
  "id": "lZvB9W1waiDHn52JRzUjfqAbyp1wyN5jreKbdyto4pI=",
  "keys": {
    "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89"
  }
}
const readOnlyWalletKey = currencyPlugin.derivePublicKey(walletInfo)

// Output:
{
  "ethereumAddress": "0x9ba8075585b5fad5c0455894a667715374f1ed63"
}
```

Converts a spending-capable [`ABCWalletInfo`](#abcwalletinfo) structure into a receive-only wallet. The core will treat the return value as the `keys` member of a new [`ABCWalletInfo`](#abcwalletinfo) structure. This has multiple uses:

1. Saving a unencrypted wallet on the local device for offline balance checks.
2. Sharing a wallet with another user is a watch-only mode.

For Bitcoin, this means converting the private seed into an xpub-format key. For Ethereum, it means converting the private key into a payment address. Please see [`ABCWalletInfo`](#abcwalletinfo) for documentation on the various key types Airbitz understands. If your currency is not documented in that section, please submit a pull request to [the documentation](https://github.com/Airbitz/airbitz-docs/tree/full-api-docs).

| Param | Type | Description |
| --- | --- | --- |
| walletInfo | [`ABCWalletInfo`](#abcwalletinfo) | The private keys to the wallet. |

The core will treat the returned object as the `keys` member of a new [`ABCWalletInfo`](#abcwalletinfo) object. The core also manages the `dataKey` and `syncKey`, so the currency plugin does not need to worry about those either.

### makeEngine

```javascript
// Example
function onTransactionsChanged(txids) {
  // your_callback_here
}

const callbacks = {
  onAddressesChecked,
  onBalanceChanged,
  onBlockHeightChanged,
  onTransactionsChanged
}
const options = {
  callbacks,
  walletFolder,
  walletLocalFolder,
  optionalSettings
}

const abcTxEngine = await currencyPlugin.makeEngine(walletInfo, options)
```

This function creates an [`ABCTxEngine`](#abctxengine) object to send, receive, and list transactions for an individual wallet instance.

| Param | Type | Description |
| --- | --- | --- |
| walletInfo | [`ABCWalletInfo`](#abcwalletinfo) | The keys to the wallet. This may include just the pubic key (no private key) in read-only scenarios. See the [`ABCWalletInfo`](#abcwalletinfo) documentation for details. |
| options | `Object` | Options for [`currencyPlugin.makeEngine`](#makeengine) |

| Options | Type | Description |
| --- | --- | --- |
| callbacks | [`ABCTxLibCallbacks`](#abctxlibcallbacks) | Various callbacks when wallet is updated. |
| walletFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for synced and encrypted data. May not be present in certain read-only scenarios. |
| walletLocalFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for non-encrypted, device-only data. |
| optionalSettings | `object` | Per-currency settings. The plugin should provide a template for this in its [`currencyInfo`](#currencyinfo). Optional. |

| Return | Type | Description |
| --- | --- | --- |
| engine | `Promise<ABCTxEngine>` | A promise that will resolve to the requested engine, or an error. |

The fresh [`ABCTxEngine`](#abctxengine) instance should either load cached transactions from disk, if available, or start with a blank transaction list if this is its first time running for a particular wallet. This makes the [`ABCTxEngine`](#abctxengine) usable right away, creating a fast start-up experience.

The [`ABCTxEngine`](#abctxengine) instance should only start querying the blockchain for transactions after the core calls `startEngine`. There are cases where the user will want to view their cached transactions without hitting the network, such as with archived walles. In these cases, the core will never call `startEngine`.

The engine should store its per-wallet transaction cache inside the provided `walletLocalFolder`. This location is unencrypted, so the plugin must never cache private keys in here.

Multiple engines may be watching the same blockchain. If these engines would like to share common chain informaiton, such as the last block height or SPV headers, they can maintain a shared cache in the `opts.io.folder` folder passed to the [ABCCorePlugin](#abccoreplugin)'s `createPlugin` method. This is an unencrypted app-wide location.

The `walletFolder` location should never be used for normal currencies. This location is for encrypted and backed-up metadata. There are proposals such as payment channels that may require metadata in the future, but normal currencies don't need this.

All these locations are [Disklet](https://www.npmjs.com/package/disklet) folders. Please see the Disklet documentation for examples of how to access them, or see the sample code for [ABCStorageWallet](#abcstoragewallet).

## ABCTxEngine

An `ABCTxEngine` object sends, receives, and lists transactions for a single user wallet. The [ABCCurrencyPlugin](#abccurrencyplugin) object creates `ABCTxEngine` instances through its `makeEngine` method.

### updateSettings

```javascript
abcTxEngine.updateSettings(settings)
```

| Param | Type | Description |
| --- | --- | --- |
| settings | `object` | A settings object in the same format as `defaultSettings` provided by [`currencyInfo`](#currencyinfo). |

If the settings change while the engine is running (due to user action), provides the new settings to the plugin.

Returns void. This method is optional if the currency does not have settings.

### startEngine

```javascript
await abcTxEngine.startEngine()
```

Begins checking the blockchain for incoming transactions. Prior to this method, the engine should simply return whatever cached data it has (or nothing, if this is the first run).

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the engine has fully initialized. |

### killEngine

```javascript
await abcTxEngine.killEngine()
```

Stops checking the blockchain for new transactions, and flushes any caches to disk. It should be safe to shut down the app once this method completes.

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the engine has fully shut down. |

### enableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

abcTxEngine.enableTokens(tokens).catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to enable in this wallet |

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the tokens are ready to use. |

Enable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library should begin checking the blockchain for the specified tokens and triggering the callbacks specified in [`currencyPlugin.makeEngine`](#makeengine).

### getBlockHeight

```javascript
// Example
var blockHeight = abcTxEngine.getBlockHeight()
console.log(blockHeight)
"455487"
```

Retrieve the current block height from the network.

### getBalance

```javascript
// Example
const balance = abcTxEngine.getBalance(options)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options parameters below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | `Int` | Balance in the smallest unit of the currency |

Get the current balance of this wallet in the currency's smallest denomination (ie. satoshis)

### getNumTransactions

```javascript
// Example
const numTransactions = abcTxEngine.getNumTransactions(options)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options parameters below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return Param | Type | Description |
| --- | --- | --- |
| numTransactions | `Int` | Number of transactions in wallet |

Get the number of transactions in the wallet

### getTransactions

```javascript
const options = {
  startIndex: 5,
  numEnteries: 50
}

abcTxEngine.getTransactions(options)
  .then(transaction => {
    console.log(transactions[0].txid) // => "1209befa09ab3efc039abf09490ac34fe09abc938"
  })
  .catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options for getTransactions. If `null`, return all transactions |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| transactions | `Promise<Array<ABCTransaction>>` | A promise that resolves to an array of [ABCTransaction](#abctransaction) objects |

Returns an array of transactions matching the options specified. The plugin must fill in the following [ABCTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `nativeAmount`
* `ourReceiveAddresses`

The remaining fields are updated by Airbitz Core.

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |
| startIndex | `Int` | The starting index into the list of transactions. 0 specifies the newest transaction |
| numEntries | `Int` |  The number of entries to return. If there aren't enough transactions to return `numEntries`, then the plugin should return the maximum possible |

### getFreshAddress

```javascript
const address = abcTxEngine.getFreshAddress(options)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options object documented below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return | Type | Description |
| --- | --- | --- |
| address | `String` | Public address |

Returns an address that has never received funds

### addGapLimitAddresses

```javascript
const abcError = abcTxEngine.addGapLimitAddresses(addresses, options)
```

| Param | Type | Description |
| --- | --- | --- |
| addresses | `Array` | Array of Strings containing public addresses |
| options | `Object` | Options object documented below |

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified,

| Return | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

When implementing an HD wallet with multiple addresses, wallet implementations typically search for funds by going a limited number of addresses ahead of the last address that has funds received. This is usually about 10 addresses. `addGapLimitAddresses` allows ABC to specify to the plugin to treat the given addresses as if they had funds received and to forward their gap limit accordingly.

### isAddressUsed

```javascript
const isUsed = abcTxEngine.isAddressUsed(address, options)
```

| Param | Type | Description |
| --- | --- | --- |
| address | `String` | String of public address to query |
| options | `Object` | Options parameters below |

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified,

| Return | Type | Description |
| --- | --- | --- |
| isUsed | `Boolean` | True if address has ever received money |

### makeSpend

```javascript
abcTxEngine.makeSpend(abcSpendInfo)
  .then(abcTransaction => {
    // your logic here
  })
  .catch(handleError)
```

Given an [ABCSpendInfo](#abcspendinfo) object, returns an unsigned [ABCTransaction](#abctransaction) object. [ABCTransaction](#abctransaction).signedTx should be NULL. `makeSpend` does not need to touch the metadata parameter in the `abcSpendInfo`. `makeSpend` only needs to support the [ABCSpendTarget](#abcspendtarget) parameters `currencyCode`, `publicAddress`, and `nativeAmount`.

Should produce an [InsufficientFundsError](#insufficientfundserror) if the amount is too large, or a [DustSpendError](#dustspenderror) if the amount is too small.

### signTx

```javascript
abcTxEngine.signTx(abcTransaction)
  .then(()) => {
    // your logic here
  })
  .catch(handleError)
```

This routine will set [ABCTransaction](#abctransaction).signedTx to an Array of bytes corresponding to the complete signed transaction. Takes an unsigned [ABCTransaction](#abctransaction) object and signs it.

### broadcastTx

```javascript
abcTxEngine.broadcastTx(abcTransaction)
  .then(()) => {
    // your logic here
  })
  .catch(handleError)
```

Takes a signed [ABCTransaction](#abctransaction) and broadcasts it to the blockchain network.

### saveTx

```javascript
abcTxEngine.saveTx(abcTransaction)
  .then(()) => {
    // your logic here
  })
  .catch(handleError)
```

Saves an already signed [ABCTransaction](#abctransaction) object to the local cache so that funds are considered spent by the wallet. Any future calls to [getTransactions](#gettransactions), [getBalance](#getbalance), or [getNumTransactions](#getNumTransactions) should reflect the outcome of this saved transaction. This routine should also trigger the callback [transactionsChanged](#transactionschanged).

## ABCTxLibCallbacks

### onAddressesChecked

```javascript
onAddressesChecked(progressRatio)
```
| Param | Type | Description |
| --- | --- | --- |
| progressRatio | `Number` | 0 to 1 value indicating how far along the core is in checking all the wallet addresses for new funds. This is only meaningful after the inital call to [startEngine](#startEngine) |

Callback fires as the plugin makes progress synchronizing with the blockchain network.

### onBalanceChanged

```javascript
onBalanceChanged(currencyCode, balance)
```
| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `string` | The three-letter code for which currency balance has changed. |
| balance | `string` | The new spendable balance, in the smallest units the currency supports as a string (i.e. Satoshi or Wei). |

Callback fires when the plugin detects a balance change for the supported currency.

### onBlockHeightChanged

```javascript
onBlockHeightChanged(blockHeight)
```

| Param | Type | Description |
| --- | --- | --- |
| blockHeight | `Int` | New block height value |

Callback fires when the plugin detects a blockheight change for the supported currency.

### onTransactionsChanged

```javascript
onTransactionsChanged(abcTransactions)
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransactions | `Array` | Array of [ABCTransaction](#abctransaction) objects which are new or have changed. Changes may include the block height when this transaction was confirmed |

Callback fires when the plugin detects new or updated transactions from the blockchain network. The plugin must fill in the following [ABCTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `nativeAmount`
* `ourReceiveAddresses`

## ABCExchangePlugin

```javascript
// SDK users should do this:
import { coinbasePlugin } from 'airbitz-currency-ethereum'

const context = makeContext({
  plugins: [coinbasePlugin]
})

// Inside the `makeContext` function, this is what happens:
const exchangePlugin = await coinbasePlugin.makePlugin({ io })
```

These plugins fetch price information from various exchange-rate providers. They begin life as a generic [ABCCorePlugin](#abccoreplugin) that returns an `ABCExchangePlugin` instance from is `createPlugin` method.

The `ABCExchangePlugin` object provides information about the exchange, as well as the ability to fetch rates. Exchange plugins should not spin up background tasks, but should make a fresh server connection on every `fetchExchangeRates` call.

### exchangeInfo

This property should contain information about the exchange.

```javascript
console.log(exchangePlugin.info.exchangeName) // "Coinbase"
```

| Property | Type | Description |
| --- | --- | --- |
| exchangeName | `string` | Name of the exchange as a human-friendly string |

### fetchExchangeRates

```javascript
fetchExchangeRates(pairHints)

// Example
const pairHints = [
  { fromCurrency: 'BTC', toCurrency: 'iso:EUR' },
  { fromCurrency: 'ETH', toCurrency: 'iso:USD' }
]

const pairs = await exchangePlugin.fetchExchangeRates(pairHints)

// The returned pairs might look like this:
[
  { fromCurrency: 'BTC', toCurrency: 'iso:EUR', rate: 2997.75 },
  { fromCurrency: 'ETH', toCurrency: 'iso:USD', rate: 297.12 }
]
```

Asks the plugin to fetch all available currency pairs.

The passed-in currency pairs are a hint. Normal plugins should just return all available data, but in cases where that's too expensive (such as needing a separate HTTP request per pair), the plugin can use the passed-in list as a filter.

# Account Management UI

To ease the implementation of the Airbitz SDK in applications, Airbitz provides a base user interface capable of all the account creation, account login, password & PIN management, and password recovery. This functionality is currently only available in Javascript for HTML apps

The repo [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui) implements a UI layer on top of [airbitz-core-js](https://github.com/Airbitz/airbitz-core-js) to provide web applications the interface required to do all the accounts management in just a small handful of Javascript API calls. All UI operates in an overlay iframe on top of the current HTML view.

Install instructions are in [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui).

## Sample webpage

A sample webpage exists in [airbitz-core-js-sample](https://github.com/Airbitz/airbitz-core-js-sample) which makes a simple page with a Login & Register button on the top bar and uses [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui).

## Usage

### makeABCUIContext

```javascript
abcUiContext = abcui.makeABCUIContext({'apiKey': 'api-key-here',
                                       'accountType': 'account:repo:com.mydomain.myapp',
                                       'bundlePath': '/path-to-abcui/',
                                       'vendorName': 'My Awesome Project',
                                       'vendorImageUrl': 'https://mydomain.com/mylogo.png'})
```

Initializes the ABCUI library and returns an [ABCUIContext](#abcuicontext) object

| Param | Type | Description |
| --- | --- | --- |
| apiKey | `String` | API Key from <https://developer.airbitz.co> |
| accountType | `String` | App account type of form 'account:repo:com.mydomain.myapp' |
| bundlePath | `String` | Website path to the airbitz-core-js-ui repo |
| vendorName | `String` | Name of your app |
| vendorImageUrl | `String` | URL to image of your app's logo |

| Return Param | Type | Description |
| --- | --- | --- |
| abcUiContext | [`ABCUIContext`](#abcuicontext) | Airbitz account object |

## ABCUIContext

### openLoginWindow

```javascript
abcUiContext.openLoginWindow(function(error, account) {
  _account = account;
})
```

Create an overlay popup where a user can register a new account or login to a previously created account via password or PIN.

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Airbitz account object |

![Login UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.04-PM.png)

### openManageWindow

```javascript
abcUiContext.openManageWindow(_account, function(error) {
  // your_callback_here
})
```

Launch an account management window for changing password, PIN, and recovery questions

| Param | Type | Description |
| --- | --- | --- |
| account | [`ABCAccount`](#abcaccount) | Airbitz account object to modify |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | (Javascript) Error object. `null` if no error |

![Manage UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.26-PM.png)

# Wallet Plugin API

Airbitz plugins are a way to include additional functionality into the Airbitz Mobile Bitcoin Wallet on iOS and Android. They are currently used to buy and sell bitcoin through Glidera, and purchase discounted Starbucks and Target cards through Foldapp.

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

> Install node on Ubuntu

```javascript
apt-get install nodejs
```

> Or on a Mac

```javascript
brew install node
```

> Install node modules

```javascript
npm install -g gulp
cd airbitz-plugins
npm install
```

First off, lets make sure you have the dependencies installed.

### Create New Plugin

To create a new plugin, you can copy the blank plugin and begin coding.

```javascript
cd plugins
cp -a blank yourpluginname
```

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

Next up is the `<body>` tag. To keep things simple you can add your UI directly inside of the HTML. Some of our the other plugins use a more sophisticated framework like Angular. Here is our UI.

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
  Airbitz.core.setupWalletChangeListener(function(wallet) {
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

The next important part of your plugin is the javascript. As we can see from the `index.html`, this sample is using `abc.js`, `jquery-2.1.3.min.js`, `qrcode.min.js` and finally `script.js`. `abc.js` is the only required javascript file. `script.js` is the plugin's code that implements the core business logic and application functionality. You are welcome to use any other filename other than `script.js` but just need to include it in your `index.html` file.

The `script.js` calls into the Airbitz core in a few ways. First it calls `Airbitz.ui.title` to change the top header title in the Airbitz app. Next is sets up a wallet listener using `Airbitz.core.setupWalletChangeListener`, so when the user changes their selected wallet, our code knows about it. Lastly, it requests the current wallet using `Airbitz.ui.getSelectedWallet`. You can view the sample code here.

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
  Airbitz.core.createReceiveRequest(wallet, {
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

Lastly we define the `updateUi` function. When the plugin loads or when the user changes their selected wallet, this function is called. We update the wallet name in the UI, and call into the Airbitz core library to create a receive request. This returns an address to us, but stores some metadata with the address, so when bitcoin is received, the transaction's meta-data will automatically be tag with the same information.

### Existing Plugins

For more ideas you can check out our existing plugins with [Foldapp](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/foldapp) and [Glidera](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/glidera).

The [Glidera](https://github.com/Airbitz/airbitz-plugins/tree/master/plugins/glidera) plugin is build on top of angular, while the foldapp plugin is using a jQuery and handlebars.

# Add Your Plugin to Airbitz

In order to see your plugin in Airbitz, you must modify the Native app to include the plugin. The instructions are slightly different for Android vs iOS.

## Android

### Build the App

Follow the README instructions in `airbitz-android-gui` to make sure you can build the app.

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

Now that all the code is in place we can build the plugin, then the Airbitz app, and run your plugin.

`./gradlew buildAirbitzPlugins installDevelopDebug`

Last, launch the app, login, navigate to Buy/Sell and launch your plugin.

## iOS

### Build the App

Follow the README instructions in `airbitz-ios-gui` to make sure you can build the app.

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

Now you can run `./mkplugin`. This will pack all the HTML files and javascript into a monolithic yourpluginname.html and copy it from `airbitz-plugins` to `airbitz-ios-gui`.

In Xcode, navigate to Airbitz/Resources/Plugins. Drag the yourpluginname.html file from Finder in Airbitz/Resources/Plugins and into Xcode in the same folder path.

Then build and run the code in Xcode. You can then launch the app, login, navigate to Buy/Sell and launch your plugin.

# Submit Your Plugin

To submit your plugin for inclusion in Airbitz, submit a pull request for the changes to the `airbitz-plugins` repo. In addition, submit pull requests for either the iOS file `Plugins.m` or the Android file `PluginFramework.java`.

# Plugin API Reference

## ABCPluginWallet object

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
| id | `String` | Wallet ID |
| name | `String` | Wallet text name |
| currency | `String` | Wallet 3 letter currency code |
| currencyCode | `Number` | Wallet ISO currency code number |
| balance | `Number` | Wallet balance in satoshis |

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
| wallet | `ABCWallet` | ABCWallet object |

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
| wallet | `ABCWallet` | ABCWallet object |

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
| wallet | `ABCWallet` | Wallet to create a receive request/address from |
| options | `Object` | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| address | `String` | Bitcoin public address of request |

| Options | Type | Description |
| --- | --- | --- |
| label | `String` | (Optional) Transaction 'Payee/Payer' name |
| category | `String` | (Optional) Transaction category such as "Expense:Rent" |
| notes | `String` | (Optional) Transaction misc notes field |
| amountSatoshi | `Number` | (Optional) Amount, in satoshis, for this request |
| success | `Function` | Success callback |
| error | `Function` | Error callback |

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
| wallet | `ABCWallet` | Wallet to finalize receive request/address from |
| address | `String` | Address to finalize |

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
// Send 1.23 BTC to address 12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN
Airbitz.core.createSpendRequest(wallet, "12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN", 123000000, {
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
| wallet | `ABCWallet` | Wallet to create a receive request/address from |
| address | `String` | Bitcoin address or BIP21 URI |
| amount | `Int` | Amount to send in satoshis |
| options | `Object` | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| back | `Boolean` | True if user pressed back button and did not authorize the spend |

| Options | Type | Description |
| --- | --- | --- |
| label | `String` | (Optional) Transaction 'Payee/Payer' name |
| category | `String` | (Optional) Transaction category such as "Expense:Rent" |
| notes | `String` | (Optional) Transaction misc notes field |
| success | `Function` | Success callback |
| error | `Function` | Error callback |

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
Airbitz.core.createSpendRequest2(wallet,
                                 "12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN", 123000000,
                                 "1FSxyn9AbBMwGusKAFqvyS63763tM8KiA2", 312000000, {
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
| wallet | `ABCWallet` | Wallet to create a receive request/address from |
| address | `String` | Bitcoin address or BIP21 URI |
| amount | `Int` | Amount to send in satoshis |
| address2 | `String` | Bitcoin address or BIP21 URI |
| amount2 | `Int` | Amount to send in satoshis |
| options | `Object` | JS Object of options for receive request |

| Response | Type | Description |
| --- | --- | --- |
| back | `Boolean` | True if user pressed back button and did not authorize the spend |

| Options | Type | Description |
| --- | --- | --- |
| label | `String` | (Optional) Transaction 'Payee/Payer' name |
| category | `String` | (Optional) Transaction category such as "Expense:Rent" |
| notes | `String` | (Optional) Transaction misc notes field |
| success | `Function` | Success callback |
| error | `Function` | Error callback |

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
| key | `String` | Key string to reference the data |
| data | `String` | Value to store |

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
| key | `String` | Key string to reference the data |

| Response | Type | Description |
| --- | --- | --- |
| data | `String` | Data stored from a previous call to writeData |

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
| denomination | `String` | "BTC", "mBTC" or "bits" |


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
| amountSatoshi | `Number` | Number of satoshis to convert |
| withSymbol | `Boolean` | Set to true to display the denomination symbol "Ƀ, mɃ, ƀ" |

| Response | Type | Description |
| --- | --- | --- |
| amountString | `String` | Formatted denomination string |

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
| key | `String` | Key string to reference the data |

| Response | Type | Description |
| --- | --- | --- |
| value | `String` | Data passed in from native app |

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

// Example
Airbitz.ui.title("Bob's Awesome Plugin");
```

Set the title of the current view. This updates the native apps titlebar.

| Param | Type | Description |
| --- | --- | --- |
| title | `String` | Title on navbar |

### showAlert

```objc
// No content for this language. Select 'Javascript/HTML` above
```

```java
// No content for this language. Select 'Javascript/HTML` above
```

```javascript
Airbitz.ui.showAlert(title, message, options)

// Example
Airbitz.ui.showAlert("Account creation error", "Error creating account. Please try again later");

Airbitz.ui.showAlert("Creating account", "Please wait...", {"showSpinner": true});
```

Launches a native alert dialog. Alert will automatically fade when tapped or after ~6 seconds unless the showSpinner option is given.

| Param | Type | Description |
| --- | --- | --- |
| title | `String` | Title of alert |
| message | `String` | Body of alert  |
| options | `Object` | Optional parameters |

| Options | Type | Description |
| --- | --- | --- |
| showSpinner | `Boolean` | (Optional) Add spinner to alert. Alert will persist until hideAlert is called. |

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
