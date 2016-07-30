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
var abcContext = null

abc.ABCContext.makeABCContext('your-api-key-here', null, function (error, context) {
  if (error) {
  } else {
    abcContext = context
  }
})
```

```objective_c
ABCContext abcContext = [ABCContext makeABCContext:abcAPIKey hbits:hbitsKey];
```

Initialize and create an ABCContext object. Required for functionality of ABC SDK.

| Param | Type | Description |
| --- | --- | --- |
| apiKey | <code>string</code> | Get an API Key from https://developer.airbitz.co |
| hbitsKey | <code>string</code> | (Optional) |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| context | <code>[ABCContext](#ABCContext)</code> | Initialized context |


### createAccount

```javascript
```

```objective_c

ABCAccount *abcAccount;

[abc createAccount:@"myUsername"
          password:@"myNot5oGoodPassw0rd"
               pin:@"2946"
          delegate:self
          callback:^(NSError *error, ABCAccount *account)
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

Initialize and create an ABCContext object. Required for functionality of ABC SDK.

| Param | Type | Description |
| --- | --- | --- |
| username | <code>string</code> | Account username |
| password | <code>string</code> | Account password |
| pin | <code>string</code> | Account PIN for fast re-login |

| Return Param | Type | Description |
| --- | --- | --- |
| error | <code>[ABCError](#ABCError)</code> | Error object. Null if no error |
| account | <code>[ABCAccount](#ABCAccount)</code> | Initialized account |



<a name="ABCAccount"></a>
## ABCAccount


