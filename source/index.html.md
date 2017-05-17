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
                         ABCAccountCallbacks,
                         callback)

// Example
function abcAccountRemotePasswordChange () {}
function abcAccountLoggedOut () {}
function abcAccountOTPRequired () {}
function abcAccountOTPSkew () {}
function abcAccountAccountChanged () {}

const abcCallbacks = {
   abcAccountRemotePasswordChange,
   abcAccountLoggedOut,
   abcAccountOTPRequired,
   abcAccountOTPSkew,
   abcAccountAccountChanged
}

abcContext.createAccount("myUsername",
                         "myNot5oGoodPassw0rd",
                         "2946",
                         abcCallbacks,
                         function (error, account) {
    if (error) {
      // Error creating account
    } else {
      abcAccount = account;
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
| callbacks | [`ABCAccountCallbacks`](#abcaccountcallbacks) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

### getLocalAccount

```javascript
abcContext.getLocalAccount(username, callbacks, callback)

// Example
abcContext.getLocalAccount("JoeHomey", callbacks,
                           function (error, account) {
    if (error) {
      // Failed to get account.
    } else {
      // Yay. Got local account info
      console.log("Account name = " + account.username)
    }
})
```

Get local account details for a previously logged in account. This returns an ABCAccount object with a `null` `dataStore` object but with a functioning `localDataStore` object. This is effectively getting the non-encrypted account data which can be accessed without the user logging into the device with a password, PIN, or fingerpint. Any [ABCWallet](#abcwallet) objects in the account will also have `null` `dataStore` objects but with functioning `localDataStore` objects. This is commonly used for background processing the accounts/wallets on a device to do querying of cryptocurrency transactions while the user is not logged in.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| callbacks | [`ABCAccountCallbacks`](#abcaccountcallbacks) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

### loginWithPassword

```javascript
abcContext.loginWithPassword(username, password, otp, callbacks, callback)

// Example
abcContext.loginWithPassword("JoeHomey", "My0KPa55WoRd@Airb1t5", null, callbacks,
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
| otp | `string` | (Optional) OTP key retrieved from getOTPLocalKey |
| callbacks | [`ABCAccountCallbacks`](#abcaccountcallbacks) | (Javascript) Callback event routines |
| delegate | [`ABCAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`ABCError`](#abcerror) | Error object. `null` if no error |
| account | [`ABCAccount`](#abcaccount) | Initialized account |

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
| callbacks | [`ABCAccountCallbacks`](#abcaccountcallbacks) | (Javascript) Callback event routines |
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

### parseUri

```javascript
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

console.log(abcParsedUri.publicAddress) // -> 1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs
console.log(abcParsedUri.amountSatoshi) // -> 123456700
console.log(abcParsedUri.paymentProtocolURL) // -> https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X
```

| Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`ABCParsedUri`](#abcparseduri) | Object with parsed parameters |

Parses a URI extracting various elements into an [ABCParsedUri](#abcparseduri) object

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

| Wallet Type | Key Name | Description |
| --- | --- | --- |
| `wallet:repo:bitcoin` | `bitcoinKey` | HD BIP32 bitcoin wallet with base16 private key |
| `wallet:repo:ethereum` | `ethereumKey` | Single address ethereum wallet with base16 private key
| `wallet:repo:bitcoin-bip44` | `bitcoinKey-BIP44` | HD BIP44 bitcoin wallet with 24 word mnemonic master private key |
| `wallet:repo:bitcoin-bip44-multisig` | `bitcoinKey-BIP44` | HD BIP44 bitcoin wallet with 24 word mnemonic master private key and upto 15 of 15 multisig support |
| `wallet:repo:com.mydomain.myapp.myDataStoreType` | `null` | Generic data store for your app |

Please contact developer@airbitz.co for the addition of new wallet types.

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

## ABCAccountCallbacks

Callback routines that notify application when various changes have occurred in the account. This is only utilized for Javascript. For ObjC, see [ABCAccountDelegate](#abcaccountdelegate).

### Object Properties

| Property | Type | Description |
| --- | --- | --- |
| abcAccountRemotePasswordChange | `Function` | Account password has been changed by a remote device. |
| abcAccountLoggedOut | `Function` | Account has been logged out |
| abcAccountOTPRequired | `Function` | Account has OTP enabled and this device does not have the correct OTP token |
| abcAccountOTPSkew | `Function` | Account has OTP enabled and this device has a time skew with the server |
| abcAccountAccountChanged | `Function` | Account dataStore has changed |

## ABCBitIDSignature

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| address | `String` | Public address of private key used to sign |
| signature | `String` | Public address of private key used to sign |

## ABCKeyInfo

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The globally-unique key ID. A 256-bit base64-encoded integer. |
| type | `String` | The type of resource these keys unlock. |
| keys | `Object` | The contents of this object depend on the key type. See the documentation for the key types below. |

### Storage Keys

Many different key types include access to Git repo for storage. In all cases, the `keys` property will have the following members:

| Property | Type | Description |
| --- | --- | --- |
| dataKey | `String` | The data encryption key. A 256-bit base64-encoded integer. |
| syncKey | `String` | The git repo identity. A 160-bit base64-encoded integer. |

In the future, Airbitz may introduce a `readKey`, which provides only read-only access to the sync server. This would be something like `sha256(syncKey)`. In that case, the `keys` object would contain one, the other, or both of `syncKey` and `readKey`.

### wallet:bitcoin

A BIP-32 Bitcoin wallet. The key derivation follows the same scheme specified in the original BIP 32 spec.

| Property | Type | Description |
| --- | --- | --- |
| bitcoinKey | `String` | The master entropy according to the BIP 32 spec. A 256-bit base64-encoded integer. |
| dataKey | `String` | See [Storage Keys](#storage-keys). |
| syncKey | `String` | See [Storage Keys](#storage-keys). |

### wallet:ethereum

An Ethereum wallet.

| Property | Type | Description |
| --- | --- | --- |
| ethereumKey | `String` | A hex-encoded 256-bit Ethereum private key. |
| dataKey | `String` | See [Storage Keys](#storage-keys). |
| syncKey | `String` | See [Storage Keys](#storage-keys). |

### account:&lt;appId&gt;

A git repo for an app. See the [Storage Keys](#storage-keys) section for the contents.

## ABCDataStore

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

# Wallet API

Once the user has logged in and retrieved their keys, there are serveral things that may take place with those keys, including:

* Accessing encrypted storage
* Sending and receiving digital currencies
* Signing BitId requests

The Airbitz SDK includes several helper classes which perform these capabilities on your behalf, using a set of provided keys.

## ABCStorageWallet

This wallet type handles encrypted and synced data. This is the base class for several other wallet types, including `ABCCurrencyWallet`. You can also use it directly if your application requires multiple encrypted and backed-up data stores. Bear in mind that each account also has its own data store, so you may not need this class unless you are doing something involving key sharing.

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
| keys | `String` | An [`ABCKeyInfo`](#abckeyinfo) structure from the account. |
| opts.account | `Object` | The [`ABCAccount`](#abcaccount) object that these keys belong to. |
| opts.onDataChange | `Function` | Called when the data changes as part of a sync operation. |

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The `id` property of the account-level [ABCKeyInfo](#abckeyinfo) |
| type | `String` | The `type` property of the account-level [ABCKeyInfo](#abckeyinfo) |
| keys | `Object` | The `keys` property of the account-level [ABCKeyInfo](#abckeyinfo) |
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
| keys | `ABCKeyInfo` | The key info obtained from the account |
| opts.account | `ABCAccount` | The account object this wallet belongs to. |
| opts.plugin | `ABCCurrencyPlugin` |  Object that exposes the [ABCCurrencyPlugin](#abccurrencyplugin) functions |
| opts.callbacks | `ABCCurrencyPlugin` | `Object` | Object with callback functions |

(Javascript) Returns a `Promise` for an ABCCurrencyWallet type.

| Callback name | Type | Description |
| --- | --- | --- |
| onAddressesChecked(progressRatio) | `Function` | Wallet has been fully updated with latest transactions from the network |
| onBalanceChanged(balance) | `Function` | Wallet balance has changed due to transactions already detected from other devices, from new transactions, or from dropped transactions |
| onTransactionsChanged(transactionArray) | `Function` | Array of new ABCTransaction objects. These are new funds that the GUI should show as a notificatino to the user |
| onNewTransactions(transactionArray) | `Function` | Array of ABCTransaction objects. These transactions are either previously recognized funds from a new device that have now synced to this device, or updates to previously seen transactions such as a change in the block number this transaction was confirmed in. |
| onBlockHeightChanged(height) | `Function` | Blockchain height changed. This is unused for sub wallets |

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

abcWallet.tx.setupContract(abcSetupContractOptions)
```

Setup the script/contract for this wallet. This is used to setup basic multisig wallets under currencies like bitcoin and ethereum.

### getBalance

```javascript
abcWallet.tx.getBalance(currencyCode)

// Example
const balance = abcWallet.tx.getBalance("BTC")
```

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Selects which currency to return a balance for. If `null`, returns the balance of the primary currency of the wallet's blockchain. ie. "BTC" or "ETH" |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | `Int` | Wallet balance in smallest unit of currency. ie Satoshis |

Gets the current balance of the wallet denominated in the smallest unit of the currency. ie. Unit of satoshis for the currency bitcoin.

### getTransactions

```javascript
abcWallet.tx.getTransactions(options, callback)

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

const abcTransactions = abcWallet.tx.getTransactions(options, function(error, abcTransactions) {
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

### getBlockHeight

```javascript
abcWallet.tx.getBlockHeight()

// Example
const height = abcWallet.tx.getBlockHeight()
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
abcWallet.tx.getReceiveAddress(options, callback)

// Example to simply return an unused address
const abcReceiveAddress = abcWallet.tx.getReceiveAddress(null, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE"
const abcReceiveAddress = abcWallet.tx.getReceiveAddress({'addressTag': 'QRCODE'}, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE" and for the currency "REP"
const options = {
  'addressTag': 'QRCODE',
  'currencyCode': 'REP'
}
const abcReceiveAddress = abcWallet.tx.getReceiveAddress(options, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to get an ABCRequestAddress object for BTC currency and previous publicAddress '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
const options = {
  currencyCode: "BTC",
  publicAddress: '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
}

const abcReceiveAddress = abcWallet.tx.getReceiveAddress(options, , function (error, abcReceiveAddress) {
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
abcWallet.tx.saveReceiveAddress(abcReceiveAddress, callback)

// Example
abcWallet.tx.getReceiveAddress(null, function (error) {
  if (!error) {
    // Success
    console.log("My bitcoin address: " + abcReceiveAddress.publicAddress)
    abcReceiveAddress.amountSatoshi = 150000000 // 1.5 BTC
    abcReceiveAddress.metadata.payeeName = "Johnny Be Good"
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
abcWallet.tx.lockReceiveAddress(callback)

// Example
abcWallet.tx.lockReceiveAddress(function (error) {
  if (!error) {
    // Success
  }
})
```

Locks this `receiveAddress` so that any future calls to [ABCCurrencyWallet.getReceiveAddress](#getreceiveaddress), without the `publicAddress` specified, will no longer return this address. Funds can still be received on this address. Use [ABCCurrencyWallet.getReceiveAddress](#getreceiveaddress) with `publicAddress` specified to get back this object in the future.

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
    payeeName: 'Tyra CPA',
    category: 'Expense:Professional Services'
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
      amountSatoshi: 210000000 // 2.1 BTC
    },
    {
      currencyCode: 'LTBCOIN'
      publicAddress: '1FSxyn9AbBMwGusKAFqvyS63763tM8KiA2',
      amountSatoshi: 120 // 120 LTBCOIN
    }
  ]
}

const abcTransaction = abcWallet.tx.makeSpend(abcSpendInfo)
abcWallet.tx.signBroadcastAndSave(abcTransaction, function(error, abcTransaction) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcWallet.tx.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, spendTarget) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      payeeName: 'Tyra CPA',
      category: 'Expense:Professional Services'
    },
    spendTargets: [ spendTarget ]
  }

  const abcTransaction = abcWallet.tx.makeSpend(abcSpendInfo)
  abcWallet.tx.signBroadcastAndSave(abcTransaction, function(error, abcTransaction) {
    if (!error) {
      // Success, transaction sent
      console.log("Sent transaction with ID = " + abcTransaction.txid)
    }
  })
}

// Example wallet to wallet transfer
const walletIds = abcAccount.listWalletIds()
const srcWallet = abcAccount.getWallet(walletId[0]) // Add check for null and correct wallet type
const destWallet = abcAccount.getWallet(walletId[1]) // Add check for null and correct wallet type

const abcSpendInfo = {
  networkFeeOption: 'high',
  metadata: {
    payeeName: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund'
  },
  spendTargets: [
    {
      destWallet,
      amountSatoshi: 210000000 // 2.1 BTC
    }
  ]
}

const abcTransaction = srcWallet.tx.makeSpend(abcSpendInfo)
srcWallet.tx.signBroadcastAndSave(abcTransaction, function(error, abcTransaction) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcSpendInfo | [`ABCSpendInfo`](#abcspendinfo) | [ABCSpendInfo](#abcspendinfo) object with various parameters for a spend operation including output addresses, amounts, or payment protocol payment objects (BIP70) |

| Return Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`ABCTransaction`](#abctransaction) | Unsigned [ABCTransaction](#abctransaction) object |

Creates an unsigned [ABCTransaction](#abctransaction) object which can be then be signed and broadcast to the network. Complete the spend by calling [ABCTransaction.signBroadcastAndSave](#signbroadcastandsave). Estimated fees can be determined by reading back [ABCTransaction.networkFee](#abctransaction)

### signTx

```javascript
abcWallet.tx.signTx(abcTransaction, callback)

// Example
abcWallet.tx.signTx(abcTransaction, function(error) {
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
abcWallet.tx.broadcastTx(abcTransaction, callback)

// Example
abcWallet.tx.broadcastTx(abcTransaction, function(error) {
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
abcWallet.tx.saveTx(abcTransaction, callback)

// Example
abcWallet.tx.saveTx(abcTransaction, function(error) {
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

Saves transaction to local cache. This will cause the transaction to show in calls to [ABCWallet.tx.getTransactions](#gettransactions).

### signBroadcastAndSave

```javascript
abcWallet.tx.signBroadcastAndSave(abcTransaction, callback)

// Example
abcWallet.tx.signBroadcastAndSave(abcTransaction, function(error) {
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

abcWallet.tx.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, paymentProtocolInfo) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      payeeName: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  const abcTransaction = abcWallet.tx.makeSpend(abcSpendInfo)
  abcWallet.tx.signBroadcastAndSave(abcTransaction, function(error, abcTransaction) {
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
    payeeName: 'Tyra CPA',
    category: 'Expense:Professional Services',
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
    }
  ]
}

abcWallet.tx.getMaxSpendable(abcSpendInfo, function(error, maxAmountSatoshi) {
  if (error === null) {
    console.log(maxAmountSatoshi)
  }
})
```

Get the maximum amount spendable from this wallet given the parameters of an [ABCSpendInfo](#abcspendinfo) object. The [ABCSpendInfo.spendTargets](#abcspendtarget) amountSatoshi values are ignored when calculating the max spendable amount. This only ever returns the max spendable of the primary currency of the wallet. Any meta-tokens of the wallet will always have a max spendable equal to the number of meta-tokens in the wallet determined by [getBalance](#getbalance).

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
    payeeName: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund',
  },
  spendTargets: [
    {
      destWallet,
      amountSatoshi: 210000000, // 2.1 BTC
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
| customNetworkFee | `Int` | Amount of per byte network fee if `networkFeeOption` is set to `custom`. Should be specified as smallest denomination of currency. ie Satoshis |
| metadata | [`ABCMetadata`](#abcMetadata) | [ABCMetadata](#abcMetadata) object. Outgoing transaction will have the specified metadata copied to the [ABCTransaction](#abctransaction) object |

Parameter object used for creating an [ABCSpend](#abcspend) object.

## ABCSpendTarget

```javascript
// Example spend target with a public addresses
const spendTarget =
  {
    currencyCode: 'BTC',
    publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
    amountSatoshi: 210000000 // 2.1 BTC
  }

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcWallet.tx.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, paymentProtocolInfo) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      payeeName: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  const abcSpend = abcWallet.tx.makeSpend(abcSpendInfo)
  abcSpend.signBroadcastAndSave(function(error, abcTransaction) {
    if (!error) {
      // Success, transaction sent
      console.log("Sent transaction with ID = " + abcTransaction.txid)
    }
  })
}
```

Parameters

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token type for the destination receiving address or wallet. If not specified, uses the primary currency of this wallet |
| publicAddress | `String` | Public address in the format of the current wallet's currency. This requires the `amountSatoshi` field to be set. Must not set both `publicAddress` and `destWallet` |
| amountSatoshi | `Int` | Amount to send in the smallest denomination of the source wallet's currency. ie Satoshis |
| destWallet | [`ABCWallet`](#abcwallet) | Destination wallet to transfer funds to. Must also set `amountSatoshi`. Must not set both `publicAddress` and `destWallet` |
| destMetadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) object with which will tag the transaction in the destination wallet. Must only be used when `destWallet` is set. |


## ABCParsedUri

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Public address from URI |
| privateKey | `String` | Private key |
| bitIDURI | `String` | Full BitID URI |
| bitIDDomain | `String` | Domain name portion of BitID URI |
| bitIDCallbackURI | `String` | BitID Callback URI |
| paymentProtocolURL | `String` | BIP70 Payment Request URL |
| amountSatoshi | `Int` | Amount in the currency's smallest denomination (satoshis)
| metadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) object with info extracted from URI |
| returnURI | `String` | URI to send user after URI/payment has been processed |
| bitidPaymentAddress | `Bool` | True if BitID URI is requesting a payment address (experimental)|
| bitidKYCProvider | `Bool` | True if BitID URI would like to provide KYC token (experimental) |
| bitidKYCRequest | `Bool` | True if BitID URI is requesting a KYC token (experimental) |


Object contains various parts of a parsed URI depending on the source URI. Any of the properties may be `null`.

## ABCPaymentProtocolInfo

Object provides basic UI displayable info about a BIP70 payment request. Also includes an [ABCSpendTarget](#abcspendtarget) for use in an [ABCSpendInfo](#abcspendinfo) to send the transaction.

| Property | Type | Description |
| --- | --- | --- |
| domain | `String` | DNS name of originator of request |
| amountSatoshi | `Int` | Amount of request |
| memo | `String` | Memo field returned by merchant |
| merchant | `String` | Name of merchat (may be blank) |
| abcSpendTarget | [`ABCSpendTarget`](#abcspendtarget) | [ABCSpendTarget](#abcspendtarget) object that can be used in an [ABCSpendInfo](#abcspendinfo) |

## ABCReceiveAddress

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Raw public address in native format of wallet currency type. (ie. base58 for bitcoin, base16 for ethereum) |
| amountSatoshi | `Int` | Amount of request denominated in the smallest unit of this wallet's currency (ie. bitcoin satoshis) |
| metadata | `ABCMetadata` | [ABCMetadata](#abcmetadata) object corresponding to this address. Any transactions receiving funds into this address will automatically have this metadata in the [ABCTransaction](#abctransaction) object.

## ABCMetadata

Non-blockchain transaction meta data associated to an [ABCTransaction](#abctransaction) or to an [ABCReceiveAddress](#abcreceiveaddress)

| Property | Type | Description |
| --- | --- | --- |
| payeeName | `String` | Name of external recipient or sender of funds |
| category | `String` | Transaction category of format "Expense:Food & Dining". Category must be of the form [Category]:[Sub Category] where category is one of "Income", "Expense", "Transfer", or "Exchange". Income refers to incoming funds such as payroll or business sales. Expense is the purchase of goods or services. Transfer is a transfer of funds to/from another wallet or exchange account owned by the user. Exchange is the change of funds from one type of currency to another. If ABCWallet.tx is of type bitcoin, an incoming transaction from the purchase of bitcoin with USD should be categorized as "Exchange:Buy Bitcoin". |
| notes | `String` | Misc notes field |
| amountFiat | `Float` | Amount of transaction in the wallet's fiat currency at the time of the transaction |
| bizId | `Int` | Unique bizId associated to a business listing in the Airbitz Business Directory |
| miscJson | `String` | Generic JSON string that can be used for additional meta data |

## ABCTransaction

Object represents a signed or unsigned transaction that may or may not be broadcast to the blockchain.

| Property | Type | Description |
| --- | --- | --- |
| wallet | [`ABCCurrencyWallet`](#abccurrencywallet) | [ABCCurrencyWallet](#abccurrencywallet) this transaction is from |
| metadata | [`ABCMetadata`](#abcmetadata) | [ABCMetadata](#abcmetadata) of this transaction |
| txid | `String` | Transaction ID as represented by the wallet's crypto currency. For bitcoin this is base16. This parameter is `null` until [signTx](#signtx) is called. |
| date | `Date` | Date that transaction was broadcast, detected, or confirmed on the blockchain. If the tx detection date is after the confirmation time, then this is the confirmation time. `null` if transaction has not been broadcast |
| blockHeight | `Int` | Block number that included this transaction |
| amountSatoshi | `Int` | Amount of transaction in denomination of smallest unit of currency. Incoming funds are positive numbers. Outgoing funds are negative. Outgoing amounts should include the providerFee and networkFee. In the case of a transaction that both spends from the current wallet to the current wallet, the amount should be the net effect on the balance of the wallet. |
| providerFee | `Int` | Additional app provider fees in denomination of smallest unit of currency (ie. Satoshis) |
| networkFee | `Int` | Fee paid to network (mining fee) in denomination of smallest unit of currency (ie. Satoshis) |
| runningBalance | `Int` | Running balance of entire wallet as of this transaction |
| signedTx |`Array` | Buffer of signed transaction data with signature. `null` if transaction is unsigned |
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

## ABCExchangeRateLib

Exchange rate info is provided to AirbitzCore via exchange source plugins that provide the following API. Plugins are not expected to spin up background tasks but to instead initiate a server connection immediately upon receiving a call to [getCurrencyPairs](#getcurrencypairs).

### getName

Returns the name of the exchange rate source in a human friendly String. ie ("Bitstamp")

```javascript
name = getName()
```

| Return | Type | Description |
| --- | --- | --- |
| name | `String` | Name of the exchange rate source |

### getCurrencyPairs

```javascript
getCurrencyPairs(pairs, callback)

// Example
const pairs =
[
  { source: "BTC", dest: "USD" },
  { source: "BTC", dest: "EUR" },
  { source: "LTC", dest: "GBP" }
]

getCurrencyPairs(pairs, function(returnPairs) {
  console.log(returnPairs)
})

For a source that cannot convert LTC, the output could be =>
"
[
  { source: "BTC", dest: "USD", value: 890.23 },
  { source: "BTC", dest: "EUR", value: 870.12 },
]
"
```

Requests the exchange rate from an array of currency pairs. Function should return an array of currency pairs that it is able to provide along with the current exchange rate.

# Currency Plugin API

Cryptocurrency functionality for [`ABCCurrencyWallet`](#abccurrencywallet) is provided by currency plugins that follow the Currency Plugin API. These libraries can be easily added to Airbitz by providing the following library API for import into [`makeCurrencyWallet`](#makeCurrencyWallet). This function will call [`currencyPlugin.makeEngine`](#makeengine) to initialize the library with a set of callbacks.

To add additional currency functionality, create a library that exposes an API that follows the ABCCurrencyPlugin template below.

[Disklet](https://www.npmjs.com/package/disklet) folders will be passed into [`currencyPlugin.makeEngine`](#makeengine). The `walletLocalFolder` allows the plugin to store wallet specific data such as a blockchain cache. If the implementation chooses to hold a global cache of data, use the `accountLocalFolder` object. Neither local folder is encrypted, revision controlled, or backed up.

The repo `airbitz-currency-bitcoin` exposes this API for bitcoin.

## ABCCurrencyPlugin

This prototype class provides all the necessary API to support a cryptocurrency in Airbitz. Each primary blockchain based currency needs a separate `ABCCurrencyPlugin`. A single `ABCCurrencyPlugin` much implement support for any meta-tokens supported by that blockchain. For example, the `ABCCurrencyPlugin` that supports bitcoin might also support Counterparty and Colored Coin. An `ABCCurrencyPlugin` that supports Ethereum might also support its ERC20 tokens.

The higher level [`makeCurrencyWallet`](#makecurrencywallet) will initialize the library as outlined below. Your application code must `require` the plugin and initialize it appropriately.

### Include and initialize the library

```javascript
import { makeBitcoinPlugin } from 'airbitz-currency-bitcoin'

const bitcoinPlugin = makeBitcoinPlugin({
  io: yourPlatformSpecificIo
})
```

Each plugin should export a `make<Name>Plugin` or similarly-named function, which initializes the plugin with any needed options. This will typically include the [platform-specific IO resources](#platform-specific-io) via an `io` property. If a plugin supports muliple chains, custom servers, or other weird currency-specific  things, this would also be the place to provide those options. Please see the plugin's documentation for any additional options.

### getInfo

```javascript
const details = currencyPlugin.getInfo()

console.log(details)
"
{
  currencyCode: 'BTC',
  denominations: [
    {
      name: 'bits',
      multiplier: 100,
      symbol: 'ƀ'
    },
    {
      name: 'mBTC',
      multiplier: 100000,
      symbol: 'mɃ'
    },
    {
      name: 'BTC',
      multiplier: 100000000,
      symbol: 'Ƀ'
    }
  ],
  symbolImage: 'qq/2iuhfiu1/3iufhlq249r8yq34tiuhq4giuhaiwughiuaergih/rg',
  metaTokens: [
    {
      currencyCode: 'XCP',
      denominations: [
        {
          name: 'XCP',
          multiplier: 1
        }
      ],
      symbolImage: 'fe/3fthfiu1/3iufhlq249r8yq34tiuhqggiuhaiwughiuaergih/ef'
    },
    {
      currencyCode: 'TATIANACOIN',
      denominations: [
        {
          name: 'TATIANACOIN',
          multiplier: 1
        }
      ],
      symbolImage: 'qe/3fthfi2fg1/3iufhlq249r8yq34tiuhqggiuhaiwughiuaergih/ef'
    }
  ]
}
"
```

| Return Param | Type | Description |
| --- | --- | --- |
| details | `Object` | Details of supported currency |

The `details` object includes the following params:

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | The 3 character code for the currency |
| denominations | `Array` | An array of Objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol (optional) |
| metaTokens | `Array` | Array of objects describing the supported metatokens |

The `denominations` object includes the following params:

| Param | Type | Description |
| --- | --- | --- |
| name | `String` | The human readable string to describe the denomination. |
| multiplier | `Int` | The value to multiply the smallest unit of currency to get to the denomination. |
| symbol | `String` | The human readable 1-3 character symbol of the currency. ie. "Ƀ" |
| font | `String` | (Optional) The font required to display the symbol specified above. If not given, will use the default system font. |

The `metaTokens` array includes the following params:

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | The human readable string to describe the denomination. |
| denominations | `Array` | An array of Objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol (optional) |

Get details of the crypto currency supported by this library

### createMasterKeys

```javascript
// Example
var masterKeys = currencyPlugin.createMasterKeys('bitcoin-bip44')
```

Creates a new randomly generated master private key and master public key for the wallet type specified.

### makeEngine

```javascript
// Example
function transactionsChanged(abcTransactions) {
  // your_callback_here
}

const callbacks = {
  transactionsChanged,
  blockHeightChanged,
  addressesChecked
}
const options = {
  callbacks,
  walletFolder,
  walletLocalFolder
}

currencyPlugin.makeEngine(keyInfo, options, function(error, abcTxEngine) {
  if (error === null) {
    // Success
  }
})

```

| Param | Type | Description |
| --- | --- | --- |
| keyInfo | [`ABCKeyInfo`](#abckeyinfo) | The keys to the wallet. This may include just the pubic key (no private key) in read-only scenarios. |
| options | `Object` | Options for [`currencyPlugin.makeEngine`](#makeengine) |
| callback | `Callback` | (Javascript) Callback function |

| Options | Type | Description |
| --- | --- | --- |
| callbacks | [`ABCTxLibCallbacks`](#abctxlibcallbacks) | Various callbacks when wallet is updated. |
| walletFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for synced and encrypted data. May not be present in certain read-only scenarios. |
| walletLocalFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for non-encrypted, device-only data. |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |
| abcTxEngine | [`ABCTxEngine`](#abctxengine) | [ABCTxEngine](#abctxengine) object |

[`currencyPlugin.makeEngine`](#makeengine) initializes the library effectively creating a cryptocurrency wallet within the [ABCWallet](#abcwallet) object. The plugin should spin up any background tasks necessary to begin querying the blockchain and field any requests for transactions. [`currencyPlugin.makeEngine`](#makeengine) will be called once for every wallet of the same or different currency.

Any persistent global information that the plugin needs to keep should be kept in the `accountDataStore` for encrypted, backed-up data, and in the `accountLocalDataStore` for unencrypted, device specific data. Both the dataStores are persisted to disk and survive app shutdown or reboots.

Any persistent wallet-specific information that the plugin needs to keep should be kept in the `walletFolder` for encrypted, backed-up data, and in the `walletLocalFolder` for unencrypted, device specific data. Both the dataStores are persisted to disk and survive app shutdown or reboots.

The plugin implementation should use the [Disklet](https://www.npmjs.com/package/disklet) API for accessing the above data stores.

Any in-memory values needed by the plugin should simply be added to the ABCTxEngine object as dynamically added parameters to the object.

It is recommended the master public keys be kept in the `walletLocalFolder`  so they can be accessed for querying the blockchain while not logged in. Local blockchain cache information can be stored in either the `walletLocalFolder` or the `accountLocalDataStore` depending on whether the implementation chooses to hold a global blockchain cache or per wallet information.

It is NOT recommended to use the `walletFolder` or `accountDataStore` for blockchain cache information as these data stores are encrypted, backed up, and revision controlled and are therefore more heavy weight.

## ABCTxEngine

An ABCTxEngine is created by the plugin and returned to Airbitz Core. It must contain the following methods. Any additional in-memory parameters needed should be dynamically placed into this object before returning it back to Airbitz Core.

### startEngine

Starts the TxEngine and causes background processes to start querying for blockchain data. This must be called prior to using any other methods of [ABCTxEngine](#abctxengine)

```javascript
abcTxEngine.startEngine()
```

### killEngine

```javascript
abcTxEngine.killEngine()
```

Terminate background processes for querying network

### getBlockHeight

```javascript
// Example
var blockHeight = abcTxEngine.getBlockHeight()
console.log(blockHeight)
"455487"
```

Retrieve the current block height from the network

### enableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

abcTxEngine.enableTokens(tokens, function(error) {
  if (error === null) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to enable in this wallet |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |

Enable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library should begin checking the blockchain for the specified tokens and triggering the callbacks specified in [`currencyPlugin.makeEngine`](#makeengine).

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

abcTxEngine.getTransactions(options, function(error, transactions) {
  if (error === null) {
    console.log(transactions[0].txid) // => "1209befa09ab3efc039abf09490ac34fe09abc938"
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options for getTransactions. If `null`, return all transactions |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`ABCError`](#abcerror) | [ABCError](#abcerror) object |
| transactions | `Array` | Array of [ABCTransaction](#abctransaction) objects |

Returns an array of transactions matching the options specified. The plugin must fill in the following [ABCTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `amountSatoshi`

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
abcTxEngine.makeSpend(abcSpendInfo, function(error, abcTransaction) {
  // your_callback_here
})
```

Given an [ABCSpendInfo](#abcspendinfo) object, returns an unsigned [ABCTransaction](#abctransaction) object. [ABCTransaction](#abctransaction).signedTx should be NULL. `makeSpend` does not need to touch the metadata parameter in the `abcSpendInfo`. `makeSpend` only needs to support the [ABCSpendTarget](#abcspendtarget) parameters `currencyCode`, `publicAddress`, and `amountSatoshi`.

### signTx

```javascript
abcTxEngine.signTx(abcTransaction, function(error) {
  // your_callback_here
})
```

This routine will set [ABCTransaction](#abctransaction).signedTx to an Array of bytes corresponding to the complete signed transaction. Takes an unsigned [ABCTransaction](#abctransaction) object and signs it.

### broadcastTx

```javascript
abcTxEngine.broadcastTx(abcTransaction, function(error) {
  // your_callback_here
})
```

Takes a signed [ABCTransaction](#abctransaction) and broadcasts it to the blockchain network.

### saveTx

```javascript
abcTxEngine.saveTx(abcTransaction, function(error) {
  // your_callback_here
})
```

Saves an already signed [ABCTransaction](#abctransaction) object to the local cache so that funds are considered spent by the wallet. Any future calls to [getTransactions](#gettransactions), [getBalance](#getbalance), or [getNumTransactions](#getNumTransactions) should reflect the outcome of this saved transaction. This routine should also trigger the callback [transactionsChanged](#transactionschanged).

## ABCTxLibCallbacks

### transactionsChanged

```javascript
transactionsChanged(abcTransactions)
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransactions | `Array` | Array of [ABCTransaction](#abctransaction) objects which are new or have changed. Changes may include the block height when this transaction was confirmed |

Callback fires when the plugin detects new or updated transactions from the blockchain network. The plugin must fill in the following [ABCTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `amountSatoshi`

### blockHeightChanged

```javascript
blockHeightChanged(blockHeight)
```

| Param | Type | Description |
| --- | --- | --- |
| blockHeight | `Int` | New block height value |

Callback fires when the plugin detects a blockheight change for the supported currency.

### addressesChecked

```javascript
addressesChecked(progressRatio)
```
| Param | Type | Description |
| --- | --- | --- |
| progressRatio | `Number` | 0 to 1 value indicating how far along the core is in checking all the wallet addresses for new funds. This is only meaningful after the inital call to [startEngine](#startEngine) |

Callback fires when the plugin detects a blockheight change for the supported currency.

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
