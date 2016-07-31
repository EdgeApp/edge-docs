## ABCConditionCode

```javascript
abc = require('abc.js')
abcc = abc.ABCConditionCode

// Use condition code
if (error.code === abcc.ABCConditionCodeOk)
  console.log("Yay!")
```

```objective_c
// Automatically included as enum in AirbitzCore.h

ABCConditionCode code = nserror.code;
if (code === abcc.ABCConditionCodeOk)
  printf("Yay!");

```

The AirbitzCore API uses the following error codes:


Error Number | Code | Meaning
---------- | ------- | ------------
0 | ABCConditionCodeOk | The function completed without an error
1 | ABCConditionCodeError | An error occured
2 | ABCConditionCodeNULLPtr | Unexpected NULL pointer
3 | ABCConditionCodeNoAvailAccountSpace | Max number of accounts have been created
4 | ABCConditionCodeDirReadError | Could not read directory
5 | ABCConditionCodeFileOpenError | Could not open file
6 | ABCConditionCodeFileReadError | Could not read from file
7 | ABCConditionCodeFileWriteError | Could not write to file
8 | ABCConditionCodeFileDoesNotExist | No such file
9 | ABCConditionCodeUnknownCryptoType | Unknown crypto type
10 | ABCConditionCodeInvalidCryptoType | Invalid crypto type
11 | ABCConditionCodeDecryptError | Decryption error
12 | ABCConditionCodeDecryptFailure | Decryption failure due to incorrect key
13 | ABCConditionCodeEncryptError | Encryption error
14 | ABCConditionCodeScryptError | Scrypt error
15 | ABCConditionCodeAccountAlreadyExists | Account already exists
16 | ABCConditionCodeAccountDoesNotExist | Account does not exist
17 | ABCConditionCodeJSONError | JSON parsing error
18 | ABCConditionCodeBadPassword | Incorrect password
19 | ABCConditionCodeWalletAlreadyExists | Wallet already exists
20 | ABCConditionCodeURLError | URL call failure
21 | ABCConditionCodeSysError | An call to an external API failed
22 | ABCConditionCodeNotInitialized | No required initialization made
23 | ABCConditionCodeReinitialization | Initialization after already initializing
24 | ABCConditionCodeServerError | Server error
25 | ABCConditionCodeNoRecoveryQuestions | The user has not set recovery questions
26 | ABCConditionCodeNotSupported | Functionality not supported
27 | ABCConditionCodeMutexError | Mutex error if some type
28 | ABCConditionCodeNoTransaction | Transaction not found
28 | ABCConditionCodeEmpty_Wallet | Wallet is Empty
29 | ABCConditionCodeParseError | Failed to parse input text
30 | ABCConditionCodeInvalidWalletID | Invalid wallet ID
31 | ABCConditionCodeNoRequest | Request (address) not found
32 | ABCConditionCodeInsufficientFunds | Not enough money to send transaction
33 | ABCConditionCodeSynchronizing | We are still sync-ing
34 | ABCConditionCodeNonNumericPin | Problem with the PIN
35 | ABCConditionCodeNoAvailableAddress | Unable to find an address
36 | ABCConditionCodeInvalidPinWait | The user has entered a bad PIN, and must wait.
36 | ABCConditionCodePinExpired | Server expired PIN. (Deprecated)
37 | ABCConditionCodeInvalidOTP | Two Factor required
38 | ABCConditionCodeSpendDust | Trying to send too little money.
1000 | ABCConditionCodeObsolete | The server says app is obsolete and needs to be upgraded.