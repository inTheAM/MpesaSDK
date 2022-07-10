# MpesaSDK
An unofficial Swift library for interacting with the Safaricom Mpesa API in iOS Apps.

<br>

## Contents:
- [Usage](#usage)
    - [Configuration](#configuration)
    - [Mpesa Express](#mpesa-express)
      - [Transactions](#transactions)
      - [Query](#query)
    - [C2B](#c2b)
      - [URL Registration](#urlregistration)
      - [Transaction confirmation](#transactionconfirmation)
    
<br>
<br>

## Usage:
Currently supported parts of the Mpesa API are:
  - Mpesa Express.
  - C2B.

Ensure you have the required data to authorize your requests i.e. your app Consumer Key, Secret and Passkey.
This is done by registering your application on the [Safaricom Developer Portal](https://developer.safaricom.co.ke/).
Once your app is created, copy your `Consumer Key` and `Consumer Secret` and head over to your application on Xcode or your preferred IDE.

The whole API is wrapped under one `Mpesa` type, and each part of the API can be accessed through the shared instance of the corresponding type:
```
Mpesa.Express.shared....
Mpesa.C2B.shared....
```

Since the Mpesa API uses different endpoints for testing and live applications, you will need to set the environment on the main Mpesa instance:
```
// For testing
Mpesa.setEnvironment(to: .development)

// For deployment
Mpesa.setEnvironment(to: .production)
```

### Configuration:
Mpesa contains a `Manager` module that will hold the configuration for your app and handle authorization for all the requests made.
First, create an instance of `MpesaConfiguration` as follows:
```
let configuration = MpesaConfiguration(consumerKey: "xxxxxx", consumerSecret: "xxxxx", passKey: "xxxxx")
```

Then pass the configuration to the `setConfiguration(_:)` method of the shared Manager instance as follows. 
The Manager will attempt to authorize the application and return a result through the completion handler.
```
Mpesa.Manager.shared.setConfiguration(configuration) { result in
  // Handle success/failure result.
}
```

From here on out, the Manager instance will auto-authorize all the requests made.


### Mpesa Express:
The Express API is used to perform LipaNaMpesa transactions or query the status of a transaction.

#### Transactions:
To perform a transaction, set up a `BuyGoodsRequest` or `PayBillRequest`. They both have the same structure:
```
let request = PayBillRequest(payBillNumber: "174379",
                             accountName: "account",
                             senderPhoneNumber: "254712345678",
                             amount: 1,
                             callbackURL: "https://yourcallbackurl",
                             description: "description")
```
Then pass the request to the `buyGoods(_:)` or `payBill(_:)` method of the shared Express instance and handle the result in the completion closure:
```
Mpesa.Express.shared.payBill(request) { result in
  // Handle success/failure
}
```
A successful result will contain an instance of either `BuyGoodsResponse` or `PayBillResponse` from Mpesa, which contains details you can use to query the status of the transaction.


#### Query:
To perform a query on the status of a transaction, set up a `TransactionQueryRequest`:
```
let request = TransactionQueryRequest(businessCode: "174379", timestamp: "20220710115624", checkoutRequestID: "ws_CO_20052022164830071714338813")
```
Then pass the request to the `checkStatus(of:_)` method of the shared Express instance and handle the result in the completion closure:
```
Mpesa.Express.shared.checkStatus(of: request) { result in
  // Handle success/failure
}
```
A successful result will contain an instance of `TransactionQueryResponse` which contains the details of the status of the transaction.


### Mpesa C2B:
The C2B API is used to confirm the status of transactions through the validation and confirmation URLs you set up for your business.

#### URL Registration:
To register your callback urls, set up a `CallbackURLRegistrationRequest`, supplying a fallback action that Mpesa will perform if your url is unreachable for some reason. The request should contain an instance of `CallbackURL`, which is the details of the validation and confirmation paths:
```
// eg validatioin path: https://mydomain.com/validation

let url = Mpesa.C2B.CallbackURL(baseURL: "mydomain.com", validationPath: "validation", confirmationPath: "confirmation")

let request = CallbackURLRegistrationRequest(businessCode: 600984, fallbackAction: .cancel, callbackURL: url)
```
Then pass the request to the `registerURLs(_:)`  method of the shared C2B instance:
```
Mpesa.C2B.shared.registerURLs(request) { result in
  // Handle success/failure
}
```

#### Transaction confirmation:
To confirm a transaction through the urls you have set up, create a `C2BTransactionConfirmationRequest`:
```
let request = C2BTransactionConfirmationRequest(businessCode: 600989, transactionType: .buyGoods, amount: 1, senderPhoneNumber: 254712345678, account: "account")
```
Then pass the request to the `confirmTransaction(_:)` method of the shared C2B instance:
```
Mpesa.C2B.shared.confirmTransaction(request) { result in
  // Handle success/failure
}
```

A successful request will return an instance of `C2BTransactionConfirmationResponse` containing the response from Mpesa.

























