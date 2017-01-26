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

ABC allows developers to apply client-side data security, encrypted such that only the end-user can access the data. [ABCDataStore](#abcdatastore) object in the Airbitz ABCAccount object allows developers to store arbitrary Edge-Secured data on the user’s account which is automatically encrypted, automatically backed up, and automatically synchronized between the user’s authenticated devices.

To get started, you’ll first need an API key. Get one at our [developer portal.](https://developer.airbitz.co)

# AirbitzCore API Reference

## Install the SDK

See the following Github repos for your various development environments. Installation instructions are in the README.md files

[Javascript](https://github.com/Airbitz/airbitz-core-js)

[Objective-C](https://github.com/Airbitz/airbitz-core-objc)

[Java/Android](https://github.com/Airbitz/airbitz-core-java)

[React Native/Javascript](https://github.com/Airbitz/airbitz-core-react-native)


## Include and initialize the SDK

```javascript
var abc = require ('airbitz-core-js')

// Or
<script src="abc.js"></script>

// React Native
var abc = require ('airbitz-core-react-native')

```

```objc
#import "ABCContext.h"
```

Include the proper header files and/or libraries for your language.

## ABCContext

Starting point of Airbitz Core SDK. Used for operations that do not require a logged in ABCAccount

### makeABCContext

```javascript
abc.makeABCContext(apiKey, type, hbitsKey, callback)
const abcContext = abc.makeABCContext(apiKey, type, hbitsKey)

// Example
const abcContext = abc.makeABCContext('your-api-key-here', 
                                      'account:repo:com.mydomain.myapp', 
                                      null)
                                      
// React Native version
const abcContext = abc.makeABCContextRN('your-api-key-here', 
                                        'account:repo:com.mydomain.myapp', 
                                         null)                                      
```

```objc
+(ABCContext *) makeABCContext:(NSString *)abcAPIKey type:(NSString *)type hbits:(NSString *)hbitsKey

ABCContext *abcContext = [ABCContext makeABCContext:@"your-api-key-here" type:@"account:repo:com.mydomain.myapp" hbits:null];
```

Initialize and create an ABCContext object. Required for functionality of ABC SDK.

| Param | Type | Description |
| --- | --- | --- |
| apiKey | <code>string</code> | Get an API Key from https://developer.airbitz.co |
| type | <code>string</code> | Type of account that this application will be accessing. Type is of the format "account:repo:com.domain.app". At this moment, all types must begin with "account:repo:" and developers should add their reverse domain and application afterwards. This 'type' is what allows a singleSignOn login to access the same account object for this particular application. ie. User's using Edge Login (SingleSignOn) in application type "account:repo:com.domain.app" will get a different account repository when logged into an app with type "account:repo:com.domain2.app". |
| hbitsKey | <code>string</code> | (Optional) Unique key used to encrypt private keys for use as implementation specific "gift cards" that are only redeemable by applications using this implementation.|
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| context | <code>[ABCContext](##abccontext)</code> | Initialized context |



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
| username | <code>string</code> | Account username |
| password | <code>string</code> | Account password |
| pin | <code>string</code> | Account PIN for fast re-login |
| callbacks | <code>[ABCAccountCallbacks](#abcaccountcallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#abcaccountdelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|


| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#abcaccount)</code> | Initialized account |

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

Get local account details for a previously logged in account. This returns an ABCAccount object with a NULL `dataStore` object but with a functioning `localDataStore` object. This is effectively getting the non-encrypted account data which can be accessed without the user logging into the device with a password, PIN, or fingerpint. Any [ABCWallet](#abcwallet) objects in the account will also have NULL `dataStore` objects but with functioning `localDataStore` objects. This is commonly used for background processing the accounts/wallets on a device to do querying of cryptocurrency transactions while the user is not logged in.

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| callbacks | <code>[ABCAccountCallbacks](#abcaccountcallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#abcaccountdelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|


| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#abcaccount)</code> | Initialized account |



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
| username | <code>string</code> | Account username |
| password | <code>string</code> | Account password |
| otp | <code>string</code> | (Optional) OTP key retrieved from getOTPLocalKey |
| callbacks | <code>[ABCAccountCallbacks](#abcaccountcallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#abcaccountdelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|


| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#abcaccount)</code> | Initialized account |



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
| username | <code>string</code> | Account username |
| pin | <code>string</code> | Account PIN for fast re-login |
| callbacks | <code>[ABCAccountCallbacks](#abcaccountcallbacks)</code> | (Javascript) Callback event routines |
| delegate | <code>[ABCAccountDelegate](#abcaccountdelegate)</code> | (ObjC) Callback event delegates |
| callback | <code>Callback</code> | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#abcaccount)</code> | Initialized account |


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
| username | <code>string</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| hasPassword | <code>Boolean</code> | True if account has a password |


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
| username | <code>string</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| hasPassword | <code>Boolean</code> | True if account has a password |

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
| username | <code>String</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| enabled | <code>Boolean</code> | True if PIN login is enabled |

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
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| usernames | <code>Array</code> | Array |

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
| username | <code>String</code> | Account username |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |
| enabled | <code>Boolean</code> | True if PIN login is enabled |


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
| username | <code>String</code> | Account username |
| otpResetToken | <code>String</code> | Reset token from loginWithPassword |
| callback | <code>Callback</code> | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |




## ABCAccount

### Class Properties
| Property | Type | Description |
| --- | --- | --- |
| username | <code>String</code> | Account username |


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
| account | <code>[ABCAccount](#abcaccount)</code> | Account object|
| callback | <code>Callback</code> | (Javascript) Callback function |


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
| password | <code>String</code> | Password string|
| callback | <code>Callback</code> | Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |



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
| pin | <code>String</code> | 4 digit PIN string|
| callback | <code>Callback</code> | (Javascript/ObjC) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | Error object. Null if no error |



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
| password | <code>string</code> | Account password |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| passwordCorrect | <code>Boolean</code> | True if password is correct |



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
| enable | <code>Boolean</code> | Set true to enable PIN login. False to disable |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |


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
| key | <code>String</code> | OTP key |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |


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
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| key | <code>String</code> | OTP key |


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
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| enabled | <code>Boolean</code> | True if OTP is enabled on this accout |
| timeout | <code>Number</code> | Number seconds required after a reset is requested before OTP is disabled (iOS only. Android returns 0) |


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
| timeout | <code>Number</code> | Number seconds required after a reset is requested before OTP is disabled |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |


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
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |


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
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |


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
| url | <code>String</code> | URI used to derive the private key to do the signature |
| message | <code>String</code> | Message to sign |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| abcSignature | <code>[ABCBitIDSignature](#ABCBitIDSignature)</code> | BitID Signature Object |

### createWallet

```javascript

abcAccount.createWallet(walletType, abcWalletKeys)

// Example

const abcWalletKeys = {
  ethereumKey: new Buffer(secureRandom(32)).toString('hex')
}


abcAccount.createWallet("wallet:repo:ethereum",
                        abcWalletKeys,
                        function (error, id) {
                        
}
```

Create a new [ABCWallet](#abcwallet) object and add it to the current account. Each wallet object represents key storage for a specific cryptocurrency type or other misc functionality such as BitID or general data storage. Once a wallet is created, the wallet keys cannot be modified. ABCWallet objects may be shared between ABCAccount objects of the same or different users given permission by the user.

| Param | Type | Description |
| --- | --- | --- |
| walletType | <code>String</code> | Wallet type corresponding to the one of the following below |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| id | <code>String</code> | Strings of wallet ID |

| Wallet Type | Key Name | Description |
| --- | --- | --- |
| <code>wallet:repo:bitcoin</code> | <code>bitcoinKey</code> | HD BIP32 bitcoin wallet with base16 private key |
| <code>wallet:repo:ethereum</code> | <code>ethereumKey</code> | Single address ethereum wallet with base16 private key 
| <code>wallet:repo:bitcoin-bip44</code> | <code>bitcoinKey-BIP44</code> | HD BIP44 bitcoin wallet with 24 word mnemonic master private key |
| <code>wallet:repo:bitcoin-bip44-multisig</code> | <code>bitcoinKey-BIP44</code> | HD BIP44 bitcoin wallet with 24 word mnemonic master private key and upto 15 of 15 multisig support |
| <code>wallet:repo:com.mydomain.myapp.myDataStoreType</code> | <code>NULL</code> | Generic data store for your app |


Please contact developer@airbitz.co for the addition of new wallet types.


### listWalletIds

```javascript
const walletIds = abcAccount.listWalletIds()
```


Get an array list of wallet IDs in the current account

| Return Param | Type | Description |
| --- | --- | --- |
| walletIds | <code>Array</code> | Array of strings of wallet IDs |


### getWallet

```javascript
abcAccount.getWallet(walletId)

// Example

const abcWallet = abcAccount.getWallet(walletId)
```

Get an [ABCWallet](#abcwallet) object given a `walletId`

| Param | Type | Description |
| --- | --- | --- |
| walletId | <code>String</code> | Wallet ID from listWallets. NULL if no matching wallet |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | <code>[ABCWallet](#abcwallet)</code> | [ABCWallet](#abcwallet) object |


### getFirstWallet

```javascript
abcAccount.getFirstWallet(walletType)

// Example
const abcWallet = abcAccount.getFirstWallet('wallet:repo:ethereum')
```

Get the first [ABCWallet](#abcwallet) object of type `walletType` 

| Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| walletType | <code>String</code> | Wallet type |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | <code>[ABCWallet](#abcwallet)</code> | (Javascript) Error object. Null if no error |


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
| abcWallet | <code>[ABCWallet](#abcwallet)</code> | Wallet to share |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| shareWalletToken | <code>String</code> | Token to use with [importWallet](#importWallet) |

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
| shareWalletToken | <code>String</code> | Token to use with [importWallet](#importwallet) |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| shareWalletToken | <code>String</code> | Token from [shareWallet](#sharewallet) |

Provides a key that can given to another user/app to share a wallet with that account.



## ABCAccountCallbacks 

Callback routines that notify application when various changes have occurred in the account. This is only utilized for Javascript. For ObjC, see [ABCAccountDelegate](#abcaccountdelegate).

### Object Properties

| Property | Type | Description |
| --- | --- | --- |
| abcAccountRemotePasswordChange | <code>Function</code> | Account password has been changed by a remote device. |
| abcAccountLoggedOut | <code>Function</code> | Account has been logged out |
| abcAccountOTPRequired | <code>Function</code> | Account has OTP enabled and this device does not have the correct OTP token |
| abcAccountOTPSkew | <code>Function</code> | Account has OTP enabled and this device has a time skew with the server |
| abcAccountAccountChanged | <code>Function</code> | Account dataStore has changed |


## ABCBitIDSignature

### Class Properties
| Property | Type | Description |
| --- | --- | --- |
| address | <code>String</code> | Public address of private key used to sign |
| signature | <code>String</code> | Public address of private key used to sign |

## ABCWallet

### Class Properties
| Property | Type | Description |
| --- | --- | --- |
| walletId | <code>String</code> | Unique ID for wallet |
| walletName | <code>String</code> | Human readable name of wallet |
| walletType | <code>String</code> | Type of wallet as specified in [createWallet](#createWallet)
| keys | <code>Object</code> | Object with keys placed at createWallet time |
| dataStore | <code>[ABCDataStore](#abcdatastore)</code> | [ABCDataStore](#abcdatastore) object. This datastore object is encrypted by default, backed up to the cloud, and synchronized with any device the user logs into. Data modifications are versioned and can be rolled back but this functionality is not yet exposed via API. dataStore object may NULL if parent [ABCAccount](#abcaccount) has not been logged into yet |
| localDataStore | <code>[ABCDataStore](#abcdatastore)</code> | [ABCDataStore](#abcdatastore) object that only exists on the current device. This data is not encrypted nor backed up. Not to be used for sensitive data but rather as a local cache of network data. Data is not version controlled and has no rollback capability. Common use case will be for local device specific wallet settings, blockchain cache information, and public address cache for use when account/wallet has not yet been decrypted (background processing) |
| tx | <code>[ABCWalletTx](#abcwallettx)</code> | Optional and may be null. Exposes transactional functionality for this wallet. Requires addTxFunctionality to be called |
 
### renameWallet

```javascript

abcWallet.renameWallet(name, callback)

// Example
abcWallet.renameWallet("My Wallet", function (error) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| name | <code>String</code> | New wallet name |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |

Returns a list of transactions in the current wallet. Transactions are returned ordered from newest to oldest.

### addTxFunctionality

```javascript

abcWallet.addTxFunctionality(ABCWalletTxLibrary)

// Example

var abcTxLibrary = require('airbitz-core-js-bitcoin`)

var success = abcWallet.addTxFunctionality(abcTxLibrary)
```

Adds send/receive transaction capability for a specific currency to a wallet. An [ABCWalletTxLibrary](#abcwallettxlibrary) object must be passed in that exposes the entire [ABCWalletTxLibrary](#abcwallettxlibrary) interface. Airbitz includes support for bitcoin transactions through the [`airbitz-core-js-bitcoin`](https://github.com/Airbitz/airbitz-core-js-bitcoin) repository. Once called, the ABCWallet object will expose the [ABCWalletTx](#abcwallettx) interface at [ABCWallet.tx](#abcwallettx).
 
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
| folder | <code>String</code> | Name of folder for data store |
| key | <code>String</code> | Name of data key |
| value | <code>String</code> | data value |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |

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
| folder | <code>String</code> | Name of folder for data store |
| key | <code>String</code> | Name of data key |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| value | <code>String</code> | data value |

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
| folder | <code>String</code> | Name of folder for data store |
| key | <code>String</code> | Name of data key |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |

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
| folder | <code>String</code> | Name of folder for data store |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |

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
| folder | <code>String</code> | Name of folder for data store |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| keys | <code>Array</code> | Array of strings of keys in folder |

List the keys in the specified folder

# ABC Currency Transactions

```javascript
var abcTxLibrary = require('airbitz-core-js-bitcoin`)

var success = abcWallet.addTxFunctionality(abcTxLibrary)
```

AirbitzCore can be extended to allow wallets to have transactional capabilities such as sending and receiving bitcoin and other digital currencies. To add transaction capabilities, import an [ABCWalletTxLibrary](#abcwallettxlibrary) object and call [ABCWallet.addTxFunctionality](#addtxfunctionality). This will add an ABCWalletTx object to the [ABCWallet](#abcwallet) which contains a majority of the transactional routines needed to send/receive funds.

## ABCWalletTx

| Property | Type | Description |
| --- | --- | --- |
| abcWallet | <code>[ABCWallet](#abcwallet)</code> | Parent [ABCWallet](#abcwallet) object |
| fiatCurrencyCode | <code>String</code> | 3 character fiat currency code |
| cryptoCurrencyCode | <code>String</code> | 3 character crypto currency code |
| subWallets | <code>Array</code> | Array of [ABCWalletTx](#abcwallettx) objects representing meta-token subwallets |

### setupCallbacks

```javascript
// Example
function abcWalletTxLoaded(abcWalletTx) { }
function abcWalletTxAddressesChecked(abcWalletTx) { }
function abcWalletTxBalanceChanged(abcWalletTx) { }
function abcWalletTxNewTransaction(abcTransaction) { }
function abcWalletTxBlockHeightChanged(abcWalletTx) { }

var callbacks = {
  abcWalletTxLoaded,
  abcWalletTxAddressesChecked,
  abcWalletTxBalanceChanged,
  abcWalletTxNewTransaction,
  abcWalletTxBlockHeightChanged
}

const abcError = abcWallet.tx.setupCallbacks(callbacks)
```

| Param | Type | Description |
| --- | --- | --- |
| callbacks | <code>Object</code> | Object with callback functions |

| Callback name | Type | Description |
| --- | --- | --- |
| abcWalletTxLoaded(ABCWalletTx) | <code>Function</code> | Wallet is loaded off disk and transactions can be accessed |
| abcWalletTxAddressesChecked(ABCWalletTx) | <code>Function</code> | Wallet has been fully updated with latest transactions from the network |
| abcWalletTxBalanceChanged(ABCTransaction) | <code>Function</code> | Wallet balance has changed due to transactions already detected from other devices. This may not be called for all transactions that change the balance but it will at least be called on the last updating transaction. Recommend that GUI be refreshed with all visible transactions when this is called.|
| abcWalletTxNewTransaction(ABCTransaction) | <code>Function</code> | New transaction detected. This may not be called for all transactions that change the balance but it will at least be called on the last updating transaction. Recommend that GUI be refreshed with all visible transactions when this is called.|
| abcWalletTxBlockHeightChanged(ABCWalletTx) | <code>Function</code> | Blockchain height changed. This is unused for sub wallets |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |

This is a necessary call before using the ABCWalletTx. `setupCallbacks` causes background tasks to start and notifications of new transactions to start. For testing, callbacks can be set to NULL.

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
abcWallet.tx.getBalance()

// Example

const balance = abcWallet.tx.getBalance()
```

| Param | Type | Description |
| --- | --- | --- |
| void | <code>Void</code> |  |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | <code>Int</code> | Wallet balance in smallest unit of currency |

Gets the current balance of the wallet denominated in the smallest unit of the currency. ie. Unit of satoshis for the currency bitcoin.

### getTransactions

```javascript
abcWallet.tx.getTransactions(options)

// Example

// Create query that looks in the first 100 transactions filtered to the past 2 days that have meta data matching the string "Mom". Then returns the first 10 of those transactions

var end = new Date();
var start = new Date();
start.setDate(end.getDate() - 1);

const options = {
  startIndex: 0,
  numEntries: 100,
  startDate: start,
  endDate: end,
  searchString: 'Mom',
  returnIndex: 0,
  returnEntries: 10
}

const abcTransactions = abcWallet.tx.getTransactions(options)

const abcTransaction = abcTransactions[0]
```

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | May be NULL which will return all transactions |

| Option Params | Type | Description |
| --- | --- | --- |
| startIndex | <code>Int</code> | Index into the full list of transactions. If unspecified, no transactions are filtered and search passes on to `startDate`. Index 0 refers to the most recent transaction. |
| startEntries | <code>Int</code> | Number of entries to return from start of index. `startIndex` must be specified |
| startDate | <code>Date</code> | Date object, in local time, when to start returning transactions. If unspecified, transactions are not filtered by date and search passes on to `searchString` |
| endDate | <code>Date</code> | Date object, in local time, when to stop returning transactions. Must be later than `startDate` and `startDate` must be specified |
| searchString | <code>String</code> | Include only transactions that have metaData that matches `searchString`. (Optional) |
| returnIndex | <code>Int</code> | Index into the filtered list of transactions. If unspecified, no transactions are filtered and current results are returned.  Index 0 refers to the most recent transaction. |
| returnEntries | <code>Int</code> | Number of entries to return from index of filtered transactions. `returnEntries` must be specified |

| Return Param | Type | Description |
| --- | --- | --- |
| transactions | <code>ABCTransaction</code> | Array of [ABCTransaction](#abctransaction) objects |

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
| void | <code>Void</code> | none |

| Return Param | Type | Description |
| --- | --- | --- |
| height | <code>Int</code> | Block height of current wallet |

Gets the current blockchain height of the wallet's cryptocurrency. Not used for subwallets

### createNewReceiveAddress

```javascript
const abcReceiveAddress = abcWallet.tx.createNewReceiveAddress(callback)

// Example
const abcReceiveAddress = abcWallet.tx.createNewReceiveAddress(function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| abcReceiveAddress | <code>ABCReceiveAddress</code> | [ABCReceiveAddress](#abcreceiveaddress) |

Returns an ABCReceiveAddress object with an unused public address.

### getReceiveAddress

```javascript
const abcReceiveAddress = abcWallet.tx.getReceiveAddress(callback)

// Example
const abcReceiveAddress = abcWallet.tx.getReceiveAddress('1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE', function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| abcReceiveAddress | <code>ABCReceiveAddress</code> | [ABCReceiveAddress](#abcreceiveaddress) |

Returns an ABCReceiveAddress object with the specific public address.

### createNewSpend

```javascript
abcWallet.tx.createNewSpend()

// Example
const abcSpend = abcWallet.tx.createNewSpend()
```

| Param | Type | Description |
| --- | --- | --- |
| void | <code>Void</code> | none |

| Return Param | Type | Description |
| --- | --- | --- |
| abcSpend | <code>[ABCSpend](#abcspend)</code> | [ABCSpend](#abcspend) object |

Creates an [ABCSpend](#abcspend) object which can be used to add spend outputs for a transaction and eventually send the transaction to the network.

### parseUri

```javascript
const abcParsedUri = abcWallet.tx.parseUri(uri)
```

| Param | Type | Description |
| --- | --- | --- |
| uri | <code>String</code> | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | <code>[ABCParsedUri](#abcparseduri)</code> | Object with parsed parameters |

Parses a URI extracting various elements into an [ABCParsedUri](#abcparseduri) object

### sweepPrivateKey

coming soon...

### exportTransactionsToCSV

coming soon...

### exportTransactionsToQBO

coming soon...

## ABCSpend

```javascript
// Example

const abcSpend = abcWallet.tx.createNewSpend()
abcSpend.addAddress('1PfLSCgMZdzHRKsQDSya6Pin3ugqLKri3n', 150000000)
abcSpend.signBroadcastAndSave(function(error, abcTransaction) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

ABCSpend is used to build a Spend from the [ABCWallet](#abcwallet) that generated this ABCSpend object. Caller can add multiple spend targets by calling [addAddress](#addaddress) repeated times. Use [signBroadcastAndSave](#signbroadcastandsave) to send the transaction to the blockchain. This spend may also be signed without broadcast by calling [signTx](#signtx).

### addAddress

```javascript
const abcError = abcSpend.addAddress(publicAddress, amountSatoshi)

// Example 
abcSpend.addAddress('1PfLSCgMZdzHRKsQDSya6Pin3ugqLKri3n', 150000000)
```

| Param | Type | Description |
| --- | --- | --- |
| publicAddress | <code>String</code> | Destination address for this spend target |
| amountSatoshi | <code>Int</code> | Amount to send denominated in smallest unit of currency |

| Return Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Add an address to the output of an ABCSpend. `addAddress` may be called multiple times to send to multiple addresses.

### addTransfer

```javascript
abcSpend.addTransfer(destAbcWallet, amountSatoshi, callback)

// Example 
abcSpend.addTransfer(destAbcWallet, 150000000, function(error) {
  if (!error) {
    // Success, transfer 
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| destAbcWallet | <code>[ABCWallet](#abcwallet)</code> | Destination [ABCWallet](#abcwallet) for this spend target |
| amountSatoshi | <code>Int</code> | Amount to send denominated in smallest unit of currency |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

This routine requests a transfer of funds between two different wallets in the same [ABCAccount](#abcaccount). The ABCWallet.tx.cryptoCurrencyCode of both the source and destination wallet must match. Future implementation will allow cross currency transfers utilizing services such as ShapeShift. `addTransfer` may only be called once on an `ABCSpend` object.

### addPaymentRequest

```javascript
abcSpend.addPaymentRequest(abcPaymentRequest)

// Example 
const abcParsedUri = abcWallet.tx.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcParsedUri.getPaymentRequest(function(error, abcPaymentRequest) {
  const abcError = abcSpend.addPaymentRequest(abcPaymentRequest)
  if (!abcError) {
    abcSpend.signBroadcastAndSave(function(error, abcTransaction) {
      if (!error) {
        // Success, transaction sent
        console.log("Sent transaction with ID = " + abcTransaction.txid)
      }
    })
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcPaymentRequest | <code>[ABCPaymentRequest](#abcpaymentrequest)</code> | [ABCPaymentRequest](#abcpaymentrequest) object from a call to [ABCWallet.tx.parseUri](#parseuri) |

| Return Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Adds a BIP70 payment request to this `ABCSpend` transaction. No amount parameter is provided as the payment request always has the amount included. Generate an ABCPaymentRequest object by calling [ABCWallet.tx.parseURI](#parseuri) then [ABCParsedUri.getPaymentRequest](#getpaymentrequest). `addPaymentRequest` may only be called once on an `ABCSpend` object. This routine may not be supported on all cryptocurrency types.

### signBroadcastAndSave

```javascript
abcSpend.signBroadcastAndSave(callback)

// Example 
abcSpend.signBroadcastAndSave(function(error, abcTransaction) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |
| abcTransaction | <code>[ABCTransaction](#abctransaction)</code> | [ABCTransaction](#abctransaction) object of sent transaction |

Signs this spend request and broadcasts it to the blockchain.

### signTx

```javascript
abcSpend.signTx(callback)

// Example 
abcSpend.signTx(function(error, abcTransaction) {
  if (!error) {
    // Success, transaction signed
    console.log("Signed transaction with txId = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |
| abcTransaction | <code>[ABCTransaction](#abctransaction)</code> | [ABCTransaction](#abctransaction) object of signed transaction |

Signs this spend request and returns an [ABCTransaction](#abctransaction) object. Does not broadcast this to the blockchain or save it in the local transaction cache.

Call [ABCTransaction.broadcastTx](#broadcasttx) followed by [ABCTransaction.saveTx](#savetx) to broadcast and save the transaction.

### getFees

coming soon...

### getMaxSpendable

coming soon...

## ABCParsedUri

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | <code>String</code> | Public address from URI |
| privateKey | <code>String</code> | Private key |
| bitIDURI | <code>String</code> | Full BitID URI |
| bitIDDomain | <code>String</code> | Domain name portion of BitID URI |
| bitIDCallbackURI | <code>String</code> | BitID Callback URI |
| paymentRequestURL | <code>String</code> | BIP70 Payment Request URL |
| amountSatoshi | <code>Int</code> | Amount in the currency's smallest denomination (satoshis)
| metadata | <code>[ABCMetaData](#abcmetadata)</code> | [ABCMetaData](#abcmetadata) object with info extracted from URI |
| returnURI | <code>String</code> | URI to send user after URI/payment has been processed |
| bitidPaymentAddress | <code>Bool</code> | True if BitID URI is requesting a payment address (experimental)|
| bitidKYCProvider | <code>Bool</code> | True if BitID URI would like to provide KYC token (experimental) |
| bitidKYCRequest | <code>Bool</code> | True if BitID URI is requesting a KYC token (experimental) |


Object contains various parts of a parsed URI depending on the source URI. Any of the properties may be NULL.

### getPaymentRequest

```javascript

// Example 
const abcParsedUri = abcWallet.tx.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcParsedUri.getPaymentRequest(function(error, abcPaymentRequest) {
  const abcError = abcSpend.addPaymentRequest(abcPaymentRequest)
  if (!abcError) {
    abcSpend.signBroadcastAndSave(function(error, abcTransaction) {
      if (!error) {
        // Success, transaction sent
        console.log("Sent transaction with ID = " + abcTransaction.txid)
      }
    })
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |
| abcPaymentRequest | <code>[ABCPaymentRequest](#abcpaymentrequest)</code> | [ABCPaymentRequest](#abcpaymentrequest) BIP70 payment request |

Communicates over network with BIP70 payment request URL to get exact payment parameters. Add this payment to an [ABCSpend](#abcspend) object using [ABCSpend.addPaymentRequest](#addpaymentrequest).

## ABCPaymentRequest

Object provides basic UI displayable info about a BIP70 payment request.

| Property | Type | Description |
| --- | --- | --- |
| domain | <code>String</code> | DNS name of originator of request |
| amountSatoshi | <code>Int</code> | Amount of request |
| memo | <code>String</code> | Memo field returned by merchant | 
| merchant | <code>String</code> | Name of merchat (may be blank) |


## ABCGetTransactionsOptions

## ABCReceiveAddress

### Class Properties

```javascript
// Example

abcWallet.tx.createNewReceiveAddress(function(error, abcReceiveAddress) {
  if (!error) {
    console.log("My bitcoin address: " + abcReceiveAddress.publicAddress)
    abcReceiveAddress.amountSatoshi = 150000000 // 1.5 BTC 
    abcReceiveAddress.metaData.payeeName = "Johnny Be Good"
    abcReceiveAddress.metaData.category = "Income:Rent"
    abcReceiveAddress.updateReceiveAddres(function(error) {
      if (!error) {
        console.log("Send funds to this URI: " + abcReceiveAddress.addressUri)
      }
    })
  }
})
```

The following properties may be set to modify the `uri` and update the metadata associated with this address.

| Property | Type | Description |
| --- | --- | --- |
| amountSatoshi | <code>Int</code> | Amount of request denominated in the smallest unit of this wallet's currency (ie. bitcoin satoshis) |
| metaData | <code>ABCMetaData</code> | [ABCMetaData](#abcmetadata) object corresponding to this address. Any transactions receiving funds into this address will automaticall have this metadata in the [ABCTransaction](#abctransaction) object.

Any modification to `amountSatoshi` or `metaData` above requires a call to [ABCReceiveAddress. updateReceiveAddress()](#updatereceiveaddress) before reading back `addressUri` or `qrCode`. `publicAddress` is static and can always be read.

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | <code>String</code> | Raw public address is native format of wallet currency type. (ie. base58 for bitcoin, base16 for ethereum) |
| addressUri | <code>String</code> | BIP21 or equivalent URI that encodes public address and optionally requested amount, name of requestor, and category of requested funds |
| qrCode | <code>String</code> | Base64 encoded image of addressUri |

### updateReceiveAddress

```javascript
abcReceiveAddress.updateReceiveAddress(callback)

// Example
abcReceiveAddress.updateReceiveAddress(function (error) {
  if (!error) {
    // Success
  }
})
```

Updates the internal database of `metaData` corresponding to this `receiveAddress`. Any transactions that are received in this address are automatically tagged with the `metaData` from this `receiveAddress`.

Also updates `ABCReceiveAddress.addressUri` and `ABCReceiveAddress.qrCode` to reflect any changes to `amountSatoshi`, `metaData.label`, `metaData.payeeName`, `metaData.category`, `metaData.notes`.

### finalizeReceiveAddress

```javascript
abcReceiveAddress.finalizeReceiveAddress(callback)

// Example
abcReceiveAddress.finalizeReceiveAddress(function (error) {
  if (!error) {
    // Success
  }
})
```

Finalizes this `receiveAddress` so that any future calls to [ABCWalletTx.createNewReceiveAddress](#createnewreceiveaddress) will no longer return this address.

## ABCMetaData

Non-blockchain transaction meta data associated to an [ABCTransaction](#abctransaction) or to an [ABCReceiveAddress](#abcreceiveaddress)

| Property | Type | Description |
| --- | --- | --- |
| payeeName | <code>String</code> | Name of external recipient or sender of funds |
| category | <code>String</code> | Transaction category of format "Expense:Food & Dining". Category must be of the form [Category]:[Sub Category] where category is one of "Income", "Expense", "Transfer", or "Exchange". Income refers to incoming funds such as payroll or business sales. Expense is the purchase of goods or services. Transfer is a transfer of funds to/from another wallet or exchange account owned by the user. Exchange is the change of funds from one type of currency to another. If ABCWallet.tx is of type bitcoin, an incoming transaction from the purchase of bitcoin with USD should be categorized as "Exchange:Buy Bitcoin". |
| notes | <code>String</code> | Misc notes field |
| amountFiat | <code>Float</code> | Amount of transaction in the wallet's fiat currency at the time of the transaction |
| bizId | <code>Int</code> | Unique bizId associated to a business listing in the Airbitz Business Directory |
| miscJson | <code>String</code> | Generic JSON string that can be used for additional meta data |



## ABCTransaction

Object represents a signed transaction that may or may not be broadcast to the blockchain

| Property | Type | Description |
| --- | --- | --- |
| abcWalletTx | <code>[ABCWalletTx](#abcwallettx)</code> | [ABCWalletTx](#abcwallettx) this transaction is from |
| metaData | <code>[ABCMetaData](#abcmetadata)</code> | [ABCMetaData](#abcmetadata) of this transaction |
| txid | <code>String</code> | Transaction ID as represented by the wallet's crypto currency. For bitcoin this is base16 |
| date | <code>Date</code> | Date that transaction was broadcast, detected, or confirmed on the blockchain. If the tx detection date is after the confirmation time, then this is the confirmation time. NULL if transaction has not been broadcast |
| blockHeight | <code>Int</code> | Block number that included this transaction |
| amountSatoshi | <code>Int</code> | Amount of fees in denomination of smallest unit of currency |
| providerFee | <code>Int</code> | Additional app provider fees in denomination of smallest unit of currency (ie. Satoshis) |
| runningBalance | <code>Int</code> | Running balance of entire wallet as of this transaction |
| cryptoReserved |<code>Object</code> | Crypto currency specific data |

`cryptoReserved` has the following parameters for bitcoin wallets

| Property | Type | Description |
| --- | --- | --- |
| isReplaceByFee | <code>Bool</code> | True if this transaction is marked as RBF (BIP125) |
| isDoubleSpend | <code>Bool</code> | True if this transaction is found to be a double spend attempt |
| inputOutputList | <code>Array</code> | Array of transaction inputs and outputs |

### broadcastTx

```javascript
abcTransaction.broadcastTx(callback)

// Example 
abcTransaction.broadcastTx(function(error) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Broadcasts transaction to the blockchain.

### saveTx

```javascript
abcTransaction.saveTx(callback)

// Example 
abcTransaction.saveTx(function(error) {
  if (!error) {
    // Success, transaction saved
    console.log("Saved transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Saves transaction to local cache. This will cause the transaction to show in calls to [ABCWallet.tx.getTransactions](#gettransactions).

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
| amount | <code>Float</code> | Amount of source currency to convert | 
| sourceCurrency | <code>String</code> | 3 character currency code of source currency | 
| destinationCurrency | <code>String</code> | 3 character currency code of destination currency | 

| Return | Type | Description |
| --- | --- | --- |
| destinationAmount | <code>Float</code> | Amount of destination currency after conversion | 

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
| name | <code>String</code> | Name of the exchange rate source |

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

Cryptocurrency functionality for AirbitzCore is provided by currency API libraries that follow the Currency Plugin API. These libraries can be easily added to Airbitz by providing the following library API for import into an [ABCWallet](#abcwallet). ABC will call into the library [abcTxLibInit](#abctxlibinit) to initialize the library with a set of callbacks. Although an ABCWallet object may get passed into [abcTxLibInit](#abctxlibinit), the [ABCWallet.dataStore](#abcdatastore) may not be initialized in the case of a non logged in wallet. In such a case, only the [ABCWallet.localDataStore](#abcdatastore) object will be available for use.

To add additional currency functionality, create a library that exposes an API that follows the ABCWalletTxLibrary template below.

The repo `airbitz-core-js-bitcoin` exposes this API for bitcoin

## ABCWalletTxLibrary

This prototype class provides all the necessary API to support a cryptocurrency in Airbitz. 

### abcTxLibGetInfo

```javascript
const details = abcTxLibGetInfo()

console.log(details)
    
"
{ currencyCode: "BTC",
  denominations: 
                  [
                    { name: "bits", multiplier: 100,       symbol: "ƀ" },
                    { name: "mBTC", multiplier: 100000,    symbol: "mɃ" },
                    { name: "BTC",  multiplier: 100000000, symbol: "Ƀ" },
                  ],
  symbolImage: "qq/2iuhfiu1/3iufhlq249r8yq34tiuhq4giuhaiwughiuaergih/rg"
  metaTokens:
  [
    { currencyCode: "XCP",
      denominations: [
                      { name: "XCP",  multiplier: 1 }
                     ],
      symbolImage: "fe/3fthfiu1/3iufhlq249r8yq34tiuhqggiuhaiwughiuaergih/ef"
    },
    { currencyCode: "TATIANACOIN",
      denominations: [
                      { name: "TATIANACOIN",  multiplier: 1 }
                     ],
      symbolImage: "qe/3fthfi2fg1/3iufhlq249r8yq34tiuhqggiuhaiwughiuaergih/ef"
    },    
}
"
```
| Return Param | Type | Description |
| --- | --- | --- |
| details | <code>Object</code> | Details of supported currency |

The `details` object includes the following params:

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | <code>String</code> | The 3 character code for the currency |
| denominations | <code>Array</code> | An array of Objects of the possible denominations for this currency | 
| symbolImage | <code>String</code> | Base64 encoded png or jpg image of the currency symbol (optional) |
| metaTokens | <code>Object</code> | Array of objects describing the supported metatokens |

The `denominations` object includes the following params:

| Param | Type | Description |
| --- | --- | --- |
| name | <code>String</code> | The human readable string to describe the denomination. |
| multiplier | <code>Int</code> | The value to multiply the smallest unit of currency to get to the denomination. |
| symbol | <code>String</code> | The human readable 1-3 character symbol of the currency. ie. "Ƀ" |
| font | <code>String</code> | (Optional) The font required to display the symbol specified above. If not given, will use the default system font. |

Get details of the crypto currency supported by this library

### abcTxLibInit

```javascript
// Example

function abcTxLibCBNewTransaction(abcTransaction)

const callbacks =
{
  abcTxLibCBNewTransaction
}

const options = 
{
  enableTokens = [ "XCP, "TATIANACOIN" ]
}

abcTxLibInit(abcWallet, options, callbacks, function(error) {
  if (error === null) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| wallet | <code>[ABCWallet](#abcwallet)</code> | Parent wallet to initialize this currency in |
| options | <code>Object</code> | Options for abcTxLibInit |
| callbacks | <code>[ABCTxLibCallbacks](#abctxlibcallbacks)</code> | Various callbacks when wallet is updated |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Initialization of the library effectively creates a cryptocurrency wallet within the [ABCWallet](#abcwallet) object. The TxLib should spin up any background tasks necessary to begin querying the blockchain and field any requests for transactions. `abcTxLibInit` will be called once for every wallet of the same or different currency. 

Any global information that the TxLib needs to keep should be kept in the [ABCWallet.abcAccount](#abcaccount) using the `dataStore` for encrypted, backed-up data, and using the `localDataStore` for unencrypted, device specific data. 

It is recommended the master public keys be keps in the [ABCWallet](#abcwallet) `localDataStore` so they can be accessed for querying the blockchain while not logged in. Local blockchain cache information can be stored in either the [ABCWallet](#abcwallet) or [ABCWallet.abcAccount](#abcaccount) `localDataStore` depending on whether the implementation chooses to hold a global blockchain cache or per wallet information.

### abcTxLibEnableTokens

(Under Construction)

```javascript
// Example

const tokens = 
{
  enableTokens = [ "XCP, "TATIANACOIN" ]
}

abcTxLibEnableTokens(abcWalletTx, tokens, function(error) {
  if (error === NULL) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcWalletTx | <code>[ABCWalletTx](#abcwalletTx)</code> | Parent ABCWalletTx to use |
| tokens | <code>Array</code> | Array of strings specifying the currency codes of tokens to enable in this wallet |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |

Enable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library should begin checking the blockchain for the specified tokens and triggering the callbacks specified in abcTxLibInit.

### abcTxLibGetBalance

Get the current balance of this wallet in the currency's smallest denomination (ie. satoshis)

### abcTxLibGetNumTransactions

Get the number of transactions in the wallet

### abcTxLibGetTransactions

```javascript
const options = { startIndex: 5,
                  numEnteries: 50 }
                  
abcTxLibGetTransactions(options, function(error, transactions) {
  if (error === null) {
    console.log(transactions[0].txid) // => "1209befa09ab3efc039abf09490ac34fe09abc938"
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | Options for abcTxLibGetTransactions. If NULL, return all transactions |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | <code>[ABCError](#abcerror)</code> | [ABCError](#abcerror) object |
| transactions | <code>Array</code> | Array of [ABCTransaction](#abctransaction) objects |

Returns an array of transactions matching the options specified. The [ABCTransaction](#abctransaction) must have the following fields filled out by the TxLib:
`abcWallet`, `txid`, `date`, `blockHeight`, and `amountSatoshi`. The remaining fields are updated by Airbitz Core. 

The `options` parameter may include the following:

| Param | Type | Description |
| --- | --- | --- |
| startIndex | <code>Int</code> | The starting index into the list of transactions. 0 specifies the newest transaction |
| numEntries | <code>Int</code> |  The number of entries to return. If there aren't enough transactions to return `numEntries`, then the TxLib should return the maximum possible |

## ABCTxLibCallbacks

### abcTxLibCBNewTransaction

```javascript
abcTxLibCBNewTransaction(abcTransaction)
```
| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | <code>[ABCTransaction](#abctransaction)</code> | Transaction object |

Callback fires when the TxLib detects a new transaction from the blockchain network. The [ABCTransaction](#abctransaction) must have the following fields filled out by the TxLib:
`abcWalletTx`, `txid`, `date`, `blockHeight`, and `amountSatoshi`. The remaining fields are updated by Airbitz Core. 


# Airbitz Account Management UI

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
| apiKey | <code>String</code> | API Key from https://developer.airbitz.co |
| accountType | <code>String</code> | App account type of form 'account:repo:com.mydomain.myapp' |
| bundlePath | <code>String</code> | Website path to the airbitz-core-js-ui repo |
| vendorName | <code>String</code> | Name of your app |
| vendorImageUrl | <code>String</code> | URL to image of your app's logo |

| Return Param | Type | Description |
| --- | --- | --- |
| abcUiContext | <code>[ABCUIContext](#abcuicontext)</code> | Airbitz account object |


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
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |
| account | <code>[ABCAccount](#abcaccount)</code> | Airbitz account object |

![Login UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.04-PM.png)

### openManageWindow

```javascript
abcUiContext.openManageWindow(_account, function(error) {
    
})
```

Launch an account management window for changing password, PIN, and recovery questions

| Param | Type | Description |
| --- | --- | --- |
| account | <code>[ABCAccount](#abcaccount)</code> | Airbitz account object to modify |
| callback | <code>Callback</code> | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#abcerror)</code> | (Javascript) Error object. Null if no error |



![Manage UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.26-PM.png)




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

#### Update 2016-08-07: For compatibility with the next Airbitz release, use the `develop` branch of each of the above repos.

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

The next important part of your plugin is the javascript. As we can see from the `index.html`, we are using `abc.js`, `jquery-2.1.3.min.js`, `qrcode.min.js` and finally `script.js`. `script.js` is the plugin's code that implements the core business logic and application functionality

The `script.js` calls into the Airbitz core in a few ways. First it calls `Airbitz.ui.title` to change the page title. Next is sets up a wallet listener using `Airbitz.core.setupWalletChangeListener`, so when the user changes their selected wallet, our code knows about it. Lastly, it requests the current wallet using `Airbitz.ui.getSelectedWallet`. You can view the sample code here.

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


// Example

s

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


// Example

s

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

