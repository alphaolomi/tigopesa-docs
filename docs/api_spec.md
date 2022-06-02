# API Specification 

## Introduction

This section covers the API specifications. Each service is divided in a Request and Response section containing the overview of the parameters and example requests and responses. The list of error codes is included in Annex A.6.4.

The following third-level domains are available for the Tigo Secure services:

`https://securesandbox.tigo.com/test` environment

`https://secure.tigo.com/production` environment

In the sections below the service URLs are relative to these two Tigo Secure domains. For example for a service called 'service1' the following URL is specified in the API specification:

<_domain>/v1/service1_

To use service1 on the test environment use the URL: `https://securesandbox.tigo.com/v1/service1`

and on the production environment: `https://secure.tigo.com/v1/service1`

1. **Generate Access Token Service**

The Generate Access Token service is used to get a valid access token. The Millicom partner can only use the assigned API services via this access token. The access token has to be specified in each request header as described in the sub paragraphs below.

The validity of the access token is time limited and after each session the token is invalidated irrespective whether the service call resulted in a positive result or a failure. The expiry time is specified in the response.

**4.3.1. Request**

Generate Access Token Request

|  |  |  |
|--|--|--|
| **URL** | `/v1/oauth/generate/accesstoken?grant\_type=client\_credentials`  
| **Method** | POST | 
| **Headers** |Content-Type:|  `application/x-www-form-urlencoded`
| **Body** | |client\_id=_<client\_id>_&client\_secret=_<client\_secret>_ 

within the body:

- <client\_id> is the unique client identifier as assigned during the registration process with Millicom

- <client\_secret> is the secret/password as provided during the registration process with Millicom

###  Response 

Average response time:< 1 second

Maximum response time: 5 seconds

In case as valid `client\_id` and `client\_secret` are submitted the following response is returned:

HTTP response code: 200 OK JSON response body:

| **Parameter** | **Type** | **Description** |
| --- | --- | --- |
| accessToken | String | unique access token |
| issuedAt | Integer | Access Token issue Date and time as Unix time |
| expiresIn | Integer | Expiry time in seconds |


**Generate Access Token Response parameters**

Example response:

Response code: 200 OK

Response body:
```json
{
  "accessToken": " ABcdef123456ABcdef123456ABcd", 
  "issuedAt": "1410268728383",
  "expiresIn": "599",
}
``` 

In case incorrect `client\_id` or `client\_secret` are provided the following error is returned:

HTTP response code: 401 Unauthorized

JSON response body:

| **Parameter** | **Type** | **Description** |
| --- | --- | --- | 
| ErrorCode | String | Error code |
| Error | String | Error description |

Example response:

Response code: 401 Unauthorized

Response body:
```json
{
  "ErrorCode" : "invalid\_client",
  "Error" :"Client credentials are invalid"
}
``` 

<br>

## System Status Service 

The System Status Service is provided to monitor the health of the service periodically (heartbeat signal). The service returns the status of both the network connectivity and the application status to the Tigo Secure server and from the Tigo Secure Server to the Tigo Operations.

### Request 

**System Status Request**
|  |   | |
| --- | --- | --- |
| **URL** | `/v1/tigo/systemstatus` |
| **Method** | GET |
| **Header** | accessToken | `<valid access token>`|


###  Response 
| | |
 | --- | --- |
 | Average response time: | < 1 second |
 | Maximum response time: | 5 seconds |
 | HTTP response code: | 200 OK |

### Response body
| **Parameter** | **#** | **Type** | **Description**|
| ------------- |:-------------:| -----:|-------------------|
| tigoSecureStatusCode      | 1 | Integer | Tigo Secure Server status code <br>0 = OK. <br>Any other number than 0 indicates a <br>problem occured|
| statusDescription | 1| String |   Description  |
| TigoOperationStatus | 0..n |  |   Description  |
| country | 1 | String  |   Three letter country code <br>(ISO 3166-1)  |
| code| 1 | Integer  |   Tigo Secure Server status code <br> 0 = OK<br> Any other number than 0 indicates a problem occured    |
| description | 1| String |   Description  |

**Example response:**
```json 
{
  "tigoSecureStatusCode" : 0, 
  "statusDescription" : "OK", 
  "TigoOperationStatus" :{
    {"country":"TZA", "code":0, 
      "description":"OK"
    } 
    {"country":"SEN", 
      "code":0, "description":"OK"
    }
  }
}
``` 

# Payment Authorization service 

## Request

For the Tigo Secure Online Payment Authorization a redirect to the following URL has to be done including a JSON request with the required payment details:

Payment Authorization Request
||||
|--|--|--|
| **URL** | `/v1/tigo/payment-auth/autorize` |
| **Method** | POST |
| **Header** | Content-Type| application/json |
|| accessToken | `<valid access token>` |

JSON Request body:

| **Parameter** | **#** | **Type** |  **Description** |
| --- | --- | --- | --- |
| MasterMerchant | 1 |
 | account | 1 | String |  MFS Account number in the destination country <br> (account to credit) |
 | pin | 1 | String | MFS Account PIN code |
 | id | 1 | String | Identifier of master merchant (i.e. company name) <br>as provided by Millicom |
 | Merchant | 0..1 | _[optional]_ |
 | reference | 1 | String | Reference of the originating merchant (company name) <br>in case the payment was made <br>on behalf of  another company |
 | fee | 0..1 | Decimal |Merchant fee for the transaction in the origin currency.This fee is charged from the merchant Information about this fee will not be communicated to the subscriber. This information is confidential and is to be used for reconciliation |
 | currencyCode | 0..1 | String | Currency code of the Merchant fee  |
 | Subscriber | 1 |
 | account | 1 |  String | MFS Account number (msisdn) of the paying <br>subscriber (account to debit) |
 | countryCode | 1  | String  | Country code dialing prefix (annex A.4) |
 | country | 1 | String  | Three letter country code <br>(ISO 3166-1 Annex A.2) |
 | firstName | 0..1 | String | First name of the subscriber |
 | lastName | 0..1 | String  | Last name of the subscriber |
 | emailId | 0..1  | String  | _[optional]_ Email address |
 | redirectUri | 1 | String  | Redirection URI to redirect after completing the payment| 
 | callbackUri | 0..1  | String  | _[optional]_ Result callback URI |
 | language | 1 | String  | Three letter code for the language  |
 | terminalId | 0..1  | String  | _[optional]_ Terminal ID |
 | originPayment | 1 |
 | amount |  1  | Decimal  | Total amount in the currency of the original  merchant payment |
 | currencyCode | 1  | String  | Currency code of the payment (see Annex A.1) |
 | tax | 1  | Decimal  | Tax for the transaction in the origin currency |
 | fee | 1  | Decimal  | Fee applied by the Master Merchant for the transaction in the origin currency. This fee is charged from the subscriber and will be shown to the subscriber. If no fee has been applied the field can be set to 0 |
 | exchangeRate | 0..1  | Decimal  | _[optional]_ Exchange rate between the origin currency (currency of the sending country) and local currency (currency of the receiving country) |
 | LocalPayment | 1 |
 | amount  | 1  | Decimal  | Total amount in the local currency of the paying subscriber |
 | currencyCode | 1  | String  | Currency code of the MFS account of the paying subscriber (local currency) |
 | transactionRefId | 1  | String  | Reference Identifier in order to uniquely identify the transaction. |

** Sample Request:** 
```json 
{
  "MasterMerchant":{
    "account":"255321321321",
    "pin":"1234",
    "id":"CompanyName"
  },
  "Merchant":{
    "reference":"Amazon",
    "fee":"23.45",
    "currencyCode":"EUR"
  },
  "Subscriber":{
    "account":"255111111111", 
    "countryCode": "255",
    "country":"tza",
    "firstName":"John",
    "lastName":"Doe",
    "emailId" : "johndoe@mail.com"
  },
  "redirectUri":"https://someapp.com/payment/redirecturi",
  "callbackUri":"https://someapp.com/payment/statuscallback",
  "language":"eng",
  "terminalId":"",
  "originPayment":{
    "amount":"75.00",
    "currencyCode":"USD",
    "tax":"0.00",
    "fee":"25.00"
    }
  "exchangeRate":"2182.23",
  "LocalPayment":{
    "amount":"218223.00",
    "currencyCode":"TZS"
    },
  "transactionRefId":"0a1e39ab"
}
``` 
Make sure to **use the Access Token only**** once **to initiate a Payment**
**Authorization**. For each Payment Authorization request a new accesstoken has to be generated. This is because the access token is invalidated after the transaction completed. Any other additional transaction initiated with the same access token will therefore fail.

Per Payment Authorization make sure to **use a unique transaction**** reference identifier** (transactionRefId) to identify the transaction. This will guarantee that the transaction is logged and traced correctly.

## Response 

Average response time:< 1 second
Maximum response time: 5 seconds
HTTP response code: 200 OK

JSON Response body:

| **Parameter** | **#** | **Type** | **Description** |
| --- | --- | --- | --- |
| transactionRefId | 1 | String | Unique reference Identifier of the transaction |
| redirectUrl | 1 | String | Tigo Secure redirect URL which has to be used to <br>redirect the Customer to the correct Tigo Secure Payment Authorization webpage |
| authCode | 1 | String | Unique code to authenticate the transaction for the customer when redirecting |
| creationDateTime | 1 | String | Transaction Creation Date and Time |

## Payment Authorization Response parameters

** Example response:**
```json
{
"transactionRefId":"0a1e39ab",
"redirectUrl":" https://securesandbox.tigo.com/v1/payment\_auth/transactions?… auth\_code=123123123&transaction\_ref\_id=0a1e39ab&lang=eng",
"authCode" : "123123123",
"creationDateTime":"Fri, 10 Oct 2014 13:58:25 UTC"
}
``` 

## Payment status callback 
After the customer completes the payment via Tigo Secure the status is reported back. This is done via the optional callback URI or – in case this callback URI has not been specified – in the redirect URI as specified in the Payment Authorization Request. The optional callback URI will be called reporting back the transaction status with the following parameters:

| **Method** | POST |
| --- | --- | 
| **Headers** | `Content-Type: application/x-www-form-urlencoded` |
| **Body** | trans\_status=`<transaction status success/fail>`&transaction\_ref\_Id=`<transaction_refID>`&<br>external\_ref\_id=`<external\_ref\_id>`&<br>mfs\_id=`<mfs\_id>`& <br>verification\_code=`<Access Token>`&<br>error\_code=`<error\_code>` |

 | **Parameter** | **Description** |
|-----------------|----------------|
 | **trans\_status** | Transaction status: success for a successful transaction <br>fail in case of a failed transaction |
 | **transaction\_ref\_id** | Transaction Reference Identifier as specified in the request |
 | **external\_ref\_id** | _[optional]_| Tigo transaction Id of the request and <br>responses between the internal servers <br>This will only be sent back in case of a successful payment |
| **mfs\_id** | _[optional]_ MFS Platform transaction id of the payment. This will only be sent back in case of a successful payment |
| **verification\_code** | _[optional]_ The verification code is the invalidated <br>Access Token as generated at the start of the payment <br>authorization flow. This code has to be used to <br>uniquely identify that payment status is reported back by Tigo Secure. <br>Note that this access token is invalided after the <br>transaction failed/succeeded/expired so it can't be reused. <br>The verification code (Access Token) will be omitted <br>in case the transaction failed. This is to prevent that a malicious callback can be done with a modified <br>transaction status. |
| **error\_code** | _[optional]_ The error code in case the transaction <br>failed. The error codes are defined in Annex A.6.4.1.


## Payment status callback parameters 

**Successful payment callback example:**
```html
POST HTTP/1.1

Host: <callback URI>

Content-Type: application/x-www-form-urlencoded

Cache-Control: no-cache

trans\_status= **success** &transaction\_ref\_id=0a1e39ab &external\_ref\_id=38c1069f-2497-4f9c-9&mfs\_id=CO140924.1414.A00113& **verification\_code** =pfCIHgyWWg6qsUIOVFS u2HR3F4jy&lang=eng
``` 

The verification code in the status callback is the invalidated

Access Token as generated at the start of the payment transaction.

**In order to confirm that successful payment status has been reported back by Tigo Secure the following steps have to be performed:**

1. Lookup the payment transaction using the transaction\_ref\_id

1. Compare the verification\_code against the original access token as used during the transaction

1. Only when the verification\_code is equal to the original access token can the payment be treated as successful.

In case of a **mismatch** between the verification code and the access token the transaction should be **treated as failed** and

reported back to Millicom. The external\_ref\_id and transaction\_id can be used for traceability of the transaction within the Millicom Tigo Operation.

**Failed payment example:**

```html
POST HTTP/1.1

Host: <callback URI>

Content-Type: application/x-www-form-urlencoded

Cache-Control: no-cache

trans\_status= **fail** &transaction\_ref\_id=0a1e39ab-d0ec -4f8b-9746-b2c4122220b2123&error\_code= purchase-3008-30434-E>
``` 

For a failed transaction the verification code (access token) is not reported back. This is to prevent a malicious callback with a modified transaction status.

After the payment status callback a HTTP redirect will be done to the URI as specified in the redirectUri parameter in the Payment Authorization Request without any extra parameters.

Note that in case no callbackUri was specified in the original request the payment status is reported back in the redirectUri in the manner as for the callback URI explained above.

**Validate MFS Account Service**

The Validate MFS Account Service can be used to check whether the subscriber has a valid MFS account in the designated country. The request requires the subscriber MSISDN, first name and last name and country code as shown below.

** Request**

Validate MFS Account Request
| | | |
| --- | --- | --- |
 | **URL** | _<domain>_/v1/tigo/mfs/validateMFSAccount |
 | **Method** | POST |
 | **Header** | Content-Type | application/json |
 || accessToken | `<valid access token>` |

JSON Request body:

 | **Parameter** | **Cardinality** | **Type**  | **Description** |
| --- | --- | --- | --- | 
 | transactionRefId  | 1  | String | Reference Identifier in order to uniquely identify the transaction |
 | ReceivingSubscriber | 1 |
 | account | 1  | String | MFS Account to validate of the receiving subscriber 
 countryCallingCode | 1 | Integer | County Calling code |
 | countryCode | 1 | String | Three letter country code (ISO 3166-1) |
 | firstName | 0..1  | String | _[optional]_ First name of the subscriber |
 | lastName  | 0..1 | String | _[optional]_ Last name of the subscriber |



## Table 6: Validate MFS Account Request parameters 

** Example request:**
```json
{
  "transactionRefId" : "1300074238", 
  "ReceivingSubscriber" :{
    "account" : "255658123964", 
    "countryCallingCode" : "255", 
    "countryCode" : "TZA", 
    "firstName" : "John", 
    "lastName" : "Doe"
  }
}
``` 

**Response**

Average response time:3 seconds

Maximum response time: 5 seconds

HTTP response code: 200 OK

JSON response body:

| **Parameter** | **Type**  | **Description** |
| --- | --- | --- |
| ValidateMFSAccountResponse |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ResponseHeader |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GeneralResponse |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;correlationID | String  | The is the transaction id as sent in the request |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;status | String | Status of executing the account validation request (OK, ERROR) |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;code  String | Status code of the account validation |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description | String | Technical and brief description of the result |
 | ResponseBody |
 | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;validMFSAccount | String | MFS account validation status true = account valid for provided details |


**Table 7: Validate MFS Account Reponse parameters**

The following response is returned for a valid MFS account:

HTTP response code: 200 OK

** JSON response body:**
```json
{
"ValidateMFSAccountResponse":
  {
  "ResponseHeader":
    {
    "GeneralResponse":
      {
      "correlationID":1234,
      "status":"OK", 
      "code":"Validatemfsaccount-3018-0000-S", 
      "description":"Provided MSISDN is a valid MFSAccount."
      }
    },
  "ResponseBody":
    {
    "validMFSAccount":"true"
    }
  }
}
``` 

In case of an invalid (non-existent) MFS account the follow response is returned:

HTTP response code: 500 Internal Server Error

JSON response body:
```json
{
  "Fault":{
    "faultcode": "env:Server",
    "faultstrn": "Subscriber not found",
    "detail":{
     "ValidateMFSAccountFault":{
      "ResponseHeader":{
        "GeneralResponse":{
          "correlationID": 1300074238,
          "status": "ERROR",
          "code": "Validatemfsaccount-3018-3001-E", 
          "description": "Subscriber not found"
        }
      }
     }
    }
  }
}
``` 

<br>

## Deposit Remittance Service 

With the Deposit Remittance Service the money for the international remittance in deposited in the subscriber's wallet. The partner wallet is debited and the subscriber wallet is credited for the amount in the local currency as specified in the request. The response will confirm success of failure of the money deposit which includes a unique Transaction ID from the MFS platform.

### **Request**

Deposit Remittance Request

| | ||
| --- | --- | --- |
| **URL** | _<domain>_/v1/tigo/mfs/depositRemittance |
| **Method** | POST |
 | **Header** | Content-Type | application/json |
| | accessToken | `<valid access token>` |

JSON request body:

 | **Parameter** | **#** | **Type** | **Description** |
| --- | --- | --- | --- |
| transactionRefId | 1  | String | Unique Transaction Reference Identifier |
 | PaymentAggregator | 1 |
 | account  | 1 | String | MFS Account number in the destination  country |
 | pin | 1 | String | MFS Account PIN code |
 | id | 1 | String | Identifier of the payment aggregator (i.e.  company name as provided by Millicom |
 | Sender | 0..1 || _[optional]_ |
 | firstName | 1 | String | First name of the Sender. This field can be left blank in case the information is not available. |
 | lastName | 1 | String | Last name of the Sender. This field can be left blank in case the information is not available. |
 | msisdn | 0..1 | String | _[optional]_ MSISDN of the Sender |
 | emailAddress | 0..1 | String | _[optional]_ e-mail address of the Sender |
 | ReceivingSubscriber | 1 |
 | account | 1 | String | MFS Account of the receiving subscriber |
 | countryCallingCode  | 0..1  | Integer | _[optional]_ Country Calling code |
 | countryCode | 1  | String | Three letter country code (ISO 3166-1 Annex A.2) |
 | firstName  | 1  | String | First name of the subscriber |
 | lastName | 1 | String | Last name of the subscriber |
 | OriginPayment | 0..1 | _[optional]_ |
 | amount | 1 | Decimal | Total amount in the currency of the sending country |
 | currencyCode | 1 | String | Currency code of the sending country|
 | tax | 1 | Decimal | Tax for the transaction in the origin currency | 
 |fee | 1 | Decimal | Fee for the transaction in the origin currency |
 | exchangeRate | 0..1 | Decimal | _[optional]_ Exchange rate between the origin currency (currency of the sending country) and local currency (currency of the receiving country) |
 | verificationRequest| 0..1| Boolean | _[optional]_ Verification flag (true/false). This feature is currently not supported. Only set to false otherwise the transaction will fail |
| sendTextMessage | 0..1 | Boolean | _[optional]_ Flag to indicate whether a text message has to be sent (sendTextMessage= true) to the receiving subscriber in the following cases: Successful deposit: informing the subscriber received an international remittance with the amount, remittance partner and optionally the name of the sender. Unsuccessful deposit cause by the subscriber not signed up for a MFS account: informing the subscriber an international remittance was missed with the amount, remittance partner and optionally the name of the sender and the suggestion to open an MFS account. |
 | LocalPayment | 1 |
 | amount | 1 | Decimal | Total amount to payout in the local currency of the receiving subscriber (see Annex A.5 for formatting)|
 | currencyCode | 1 | String | Currency code of the receiving country (see Annex A.1) |


**Table Deposit Remittance Request parameters**

Example Deposit Remittance Request
```json 

{ 
"transactionRefId" : "1300074238",
"PaymentAggregator" : 
  {
  "account" : "255123123123", 
  "pin" : "1234",
  "id" : "Company Name"
  },
"Sender" : {
  "firstName" : "Jane", 
  "lastName" : "Doe", 
  "msisdn" : "2551234123423",
  "emailAddress" : "janedoe@mail.com"
  }, 
  "ReceivingSubscriber" : {
  "account" : "255111111111", 
  "countryCallingCode": "255", 
  "countryCode" : "TZA", 
  "firstName" : "John", 
  "lastName" : "Doe"
  },
"OriginPayment" : { 
  "amount" : "100.00", 
  "currencyCode" : "EUR", 
  "tax" : "10.00",
  "fee" : "25.00"
  },
  "exchangeRate" : "2182.23", 
  "verificationRequest" : "true", 
  "sendTextMessage" : "true", 
  "LocalPayment" : {
    "amount" : "200", 
    "currencyCode" : "TZS"
  }
}
``` 

**Response**

Average response time:3 seconds

Maximum response time: 5 seconds

HTTP response code: 200 OK

JSON response body:

| **Parameter** | **Type** | **Description** |
| --- | --- | --- | 
| DepositRemittanceResponse |
 | ResponseHeader |
 | GeneralResponse |
 | correlationID | String | The is the transaction id as sent in the request |
 | status | String | Status of executing the account validation request (OK, ERROR) |
 | code | String | Status code of the account validation (see codes below) |
 | description | String | Technical and brief description of the result |
 | ResponseBody |
 | transactionId | String | Transaction Identifier from the MFS Platform |


**Table: Deposit Remittance Response parameters**

Example Response

```json 
{
  "DepositRemittanceResponse":{
    "ResponseHeader":{
      "GeneralResponse":{
        "correlationID":1300074238,
        "status":"OK", 
        "code":"depositremittance-3017-0000-S",
        "description":"The Transaction is completed successfully."
      }
    },
    "ResponseBody":{
      "transactionId":"CO140912.1700.A00059"
    }
  }
}
```

<br>

## **Reverse Transaction service**


| | |
 |---------|--------|
 | **URL** | _<domain>_/v1/tigo/mfs/reverseTransaction |
 | **Method** | POST |
 | **Header** | accessToken `<valid access token>` |

JSON request body:

| **Parameter** | **#** | **Type**  | **Description** |
| --- | --- | --- | --- | 
| MasterAccount | 1 |
 | account | 1 | String  | The MFS account of the Master Merchant /Payment Aggregator as used in the original request Payment Request or Deposit Remittance API request |
 | pin | 1 | String | MFS Account PIN code for the Master Account |
 | id | 1 | String | Identifier of Master Merchant (i.e. company name) as provided by Millicom |
| transactionRefId | 1 | String | Transaction Reference Identifier as submitted in the request (transactionRefId) |
| **mfsTransactionId** | 1 | String | The MFS Transaction Identifier for the transaction this maps to the Payment Authorization status callback mfs\_id value or the DepositRemittanceResponse _[transactionId]_ |
| countryCode | 1 | String  | Three letter country code(ISO 3166-1 Annex A.2) |
| subscriberAccount | 0..1 | String | MFS Account of the subscriber (MSISDN) Authorization or |
| LocalPayment | 0..1 | _[optional]_ |
 | amount | 1 | Decimal | Total amount of the transaction in the local currency (see Annex A.5 for the formatting) |
 | currencyCode | 1 | String | Currency code of the Tigo country (see Annex |

 **Table: Reverse Transacton Request parameters**

Example request:
```json
{
  "MasterAccount" :{
    "account" : "255321321321", 
    "pin" : "1234",
    "id" : "CompanyName"
  },
  "transactionRefId" : "0a1e39ab", 
  "mfsTransactionId" : "CO140924.1414.A00113", 
  "countryCode" : "tza",
  "subscriberAccount" : "255111111111",
  "LocalPayment" :{
    "amount" : " 218223.00", 
    "currencyCode" :"TZS"
  }
}
```
**Response**

## Payment Authorization Transaction Status service

**Get Transaction Status Request**

 | **URL** | _<domain>_/v1/payment-auth/transactions/<MasterMerchantID><transactionRefId > |
 |------|-------|
 | **Method** | GET |
 | **Header** | accessToken <valid access token> |

with <MasterMerchantID><transactionRefId> the MasterMerchant Identifier and transaction reference ID as specified in the Payment Authorization Request. For example when the below values were provided in the original Payment Authorization Request (Section 4.5.1):
```json
"MasterMerchant":{
  "id":" **Company Name**",
  "transactionRefId":" **0a1e39ab**"
}
``` 

The example request to retrieve the payment authorization transaction status will be in that case:

```html
GET /v1/payment-auth/transactions/ **Company Name0a1e39ab** HTTP/1.1

Host: <host>

accessToken: <accessToken> Cache-Control: no-cache
``` 

**Response** 

Average response time:< 1 second

Maximum response time: 5 seconds

HTTP response code: 200 OK

JSON response body:

| **Parameter** | **#** | **Type** | **Description** |
| --- | --- | --- | --- | 
| Transaction | refId | 1 | String | Unique Transaction Reference Identifier as provided in the initial request |
| externalRefId | 0..1 | String | _[optional]_ Tigo transaction Id of the request and responses between the internal servers. This will only be returned for successful transactions |
 | mfsId | 0..1 | String | _[optional]_ MFS Platform transaction id of the payment. This will only be returned for successful transactions in the following format: Fri, 10 Oct 2014 13:58:25 UTC |
 | Status | 1  | String | Transaction status: Success Fail 
 |completedOn | 1 | Date| Completion date and time of the transaction in the following format:Fri, 10 Oct 2014 13:58:54 UTC |
| MasterMerchant | 1 |
 | account | 1 | String | MFS Account number in the destination country (account to credit) |
 | id | 1 | String | Identifier of master merchant (i.e. company name) |
| Merchant | 0..1 | _[optional]_ | 
|reference | 1 | String | Reference of the originating merchant (company name) in case the payment was made on behalf of another company |
 | fee | 1 | Decimal | Merchant fee for the transaction in the origin currency |
 | currencyCode | 1 |  String | Currency code of the Merchant fee (see Annex A.1) |
| Subscriber | 1 |
 | account | 1 | String | MFS Account number (msisdn) of the paying subscriber (account to debit). |
 | countryCode | 1  | String  | Country code dialing prefix |
 | country | 1 | String | Three letter country code (ISO 3166-1 Annex A.2) |
 | firstName | 1  | String | First name of the subscriber |
 | lastName | 1  | String | Last name of the subscriber |
 | emailId | 0..1  | String | _[optional]_ Email address |
| redirectUri | 1 | String | Redirection URI to redirect after completing the payment |
| callbackUri | 0..1  | String | _[optional]_ Result callback URI |
| language | 1 | String | Three letter code for the language (ISO 639-3 see Annex 0) |
| terminalId | 0..1 | String | _[optional]_ Terminal ID |
| OriginPayment | 1 |
 | Amount | 1  | Decimal | Total amount in the currency of the sending country |
 | currencyCode | 1 | String | Currency code of the payment (see Annex A.1) |
 | tax | 1 | Decimal | Tax for the transaction in the origin currency |
 | fee | 1 | Decimal | Fee applied by the Master Merchant for the transaction in the origin currency |
| exchangeRate | 0..1 | Decimal | _[optional]_ Exchange rate between the origin currency (currency of the sending country) and local currency (currency of the receiving country) |
| LocalPayment | 1 |
 | amount | 1 | Decimal | Total amount in the local currency of the paying subscriber |
 | currencyCode | 1 | String | Currency code of the sending country |

**Table: Payment Authorization Transaction Status Response parameters**

**Sample Payment Authorization Transaction Status response:**

```json
{
"Transaction" :{
    "refId":"0a1e39ab-d0ec-4f8b-9746-b2c4122220b2c120ww40", 
    "externalRefId" : "38c1069f-2497-4f9c-9",
    "mfsId" : "CO140924.1414.A00113",
    "createdOn" : "Fri, 10 Oct 2014 13:58:25 UTC",
    "status" : "success",
    "completedOn" : "Fri, 10 Oct 2014 13:58:31 UTC",
  },
"MasterMerchant":{
    "account":"255321321321",
    "id":"Skrill Ltd"
  },
"Merchant":{
  "reference":"Acme,Inc",
  "fee":"23.45",
  "currencyCode":"TZS",
  }
"Subscriber":{
  "account":"255111111111",
  "countryCode": "255",
  "country":"tza",
  "firstName":"John",
  "lastName":"Doe",
  "emailId" : "johndoe@mail.com"
  },
  "redirectUri":"https://someapp.com/payment/redirecturi",
  "callbackUri":" https://someapp.com/payment/statuscallback",
  "language":"eng",
  "terminalId":"",
  "originPayment":
  {
    "amount":"75.00",
    "currencyCode":"USD",
    "tax":"0.00",
    "fee":"25.00"
  },
  "exchangeRate":"2182.23",
  "LocalPayment":{
    "amount":"218223.00",
    "currencyCode":"TZS"
  }
}
```

**Deposit Remittance Transaction Status service**


Get Deposit Remittance Transaction Status Request
| |  |
|------|-----------|
| **URL** | _<domain>_/v1/tigo/mfs/depositRemittance/transactions/<PaymentAggregatorID><Transaction ID> |
 | **Method** | GET |
 | **Header** | accessToken <valid access token> |

with <PaymentAggregatorID><TransactionID> the Payment Aggregator Identifier and transaction reference ID as specified in the Deposit Remittance Request. For example when the below values were provided in the original Payment Authorization Request:

```json
{ 
"transactionRefId" : "**1300074**",
"PaymentAggregator" : {
  "id" : " **Company Name**"
  }
}
```

Example request:
```html
GET /v1/tigo/mfs/depositRemittance_/transactions/__ **Company Name** _ **1300074**
HTTP/1.1
Host: <host>
accessToken: _<accessToken>_
Cache-Control: no-cache
``` 

**Response**

Average response time:< 1 second

Maximum response time: 3 seconds

HTTP response code: 200 OK

JSON response body:

| **Parameter** | **#** | **Type** | **Description** |
| --- | --- | --- | --- |
| Transaction | 1 |
 | refId | |String | Unique Transaction Reference Identifier as provided in the initial request |
 | status | 1 | | Transaction status success/ fail |
 | mfsId | 1 | String | MFS Platform transaction id of the payment |
 | errorCode | 0..1 | String | Error code in case of a failed transaction |
| PaymentAggregator | 1 |
 | account  | 1  | String | MFS Account number in the destination country|
 | id | 1 | String | Identifier of the payment aggregator (i.e. |
 | Sender | 0..1 |
 | firstName | 1  | String | First name of the Sender |
 | lastName  | 1  | String  | Last name of the Sender |
 | msisdn | 0..1  | String  | _[optional]_ MSISDN of the Sender |
 | emailAddress  | 0..1  | String  | _[optional]_ e-mail address of the Sender |
| ReceivingSubscriber | 1  | 
|account  | 1  | String  | MFS Account of the receiving subscriber |
 | countryCallingCode | 0..1 | Integer  | _[optional]_ Country Calling code |
 | countryCode  | 1  | String | Three letter country code (ISO 3166-1 Annex A.2) |
 | firstName | 1 | String | First name of the subscriber |
 | lastName  | 1  | String | Last name of the subscriber |
| OriginPayment | 0..1 | | _[optional]_ |
 | amount  | 1  | Decimal  | Total amount in the currency of the sending country |
 | currencyCode | 1  | String  | Currency code of the sending country (see Annex A.1) |
 | tax  | 1  | Decimal | Tax for the transaction in the origin  currency |
 | fee | 1  | Decimal  | Fee for the transaction in the origin currency |
 | exchangeRate | 0..1  | Decimal  | _[optional]_ Exchange rate origin payment currency and local payment currency |
| verificationRequest | 0..1 | Boolean | _[optional]_ currently not used |
 | sendTextMessage  | 0..1  | Boolean  | _[optional]_ Flag to send text message after complete the transaction |
| localPayment | 1 | 
|amount | 1  | Decimal  | Total amount to payout in the local currency of the receiving subscriber |
 | currencyCode | 1 | String | Currency code of the receiving country (see |

Example Response:

```json
{
"Transaction": {
  "refId": "1300074239", 
  "status": "success",
  "mfsId": "CI141127.2125.A03951"
}, 
"PaymentAggregator" :{
  "account" : "255123123123",
  "id" : "CompanyName"
}, 
"Sender" :{
  "firstName" : "Jane",
  "lastName" : "Doe",
  "msisdn" : "441512121212",
  "emailAddress" : "janedoe@mail.com"
}, 
"ReceivingSubscriber" :{
  "account" : "255111111111",
  "countryCallingCode" : "255",
  "countryCode" : "TZA",
  "firstName" : "John",
  "lastName" : "Doe"
}, 
"OriginPayment" :{
  "amount" : "100.00",
  "currencyCode" : "EUR",
  "tax" : "10.00",
  "fee" : "25.00"
},
"exchangeRate" : "2182.23",
"verificationRequest" : "false",
"sendTextMessage" : "true",
  "localPayment" :{
    "amount" : "5555",
    "currencyCode" : "TZS"
  }
}
```

<!-- Misc -->
<br/>
# Currency Codes 

The Millicom supported currency codes are according to the ISO 4217 standard.

** Currency codes**

 | **Code** | **Currency** |
| --- | --- |
 | **BOB** | Boliviano |
 | **CDF** | Congolese franc |
 | **COP** | Colombian peso |
 | **EUR** | Euro |
 | **GHS** | Ghanaian cedi |
 | **GTQ** | Guatemalan quetzal |
 | **PYG** | Paraguayan guaraní |
 | **RWF** | Rwandan franc |
 | **TZS** | Tanzanian shilling |
 | **USD** | United States dollar |
 | **XAF** | CFA franc BEAC |
 | **XOF** | CFA Franc |

## Country Codes 

The Millicom supported country codes are according to the ISO 3166-1 alpha-3 standard.

 | **Code** | **Country** |
| --- | --- |
 | **BOL** | Bolivia, Plurinational State of |
 | **COD** | Congo, the Democratic Republic of the |
 | **COL** | Colombia |
 | **GHA** | Ghana |
 | **GTM** | Guatemala |
 | **HND** | Honduras |
 | **PRY** | Paraguay |
 | **RWA** | Rwanda |
 | **SEN** | Senegal |
 | **SLV** | El Salvador |
 | **TCD** | Chad |
 | **TZA** | Tanzania, United Republic of |

## Language Codes 

The Millicom supported language codes are according to the ISO 639-3 standard.

 | **Code** | **Language** |
| --- | --- |
 | **ara** | Arabic |
 | **aym** | Aymara |
 | **cab** | Garifuna |
 | **emk** | Eastern Maninkakan |
 | **eng** | English |
 | **fra** | French |
 | **ful** | Fulah |
 | **grn** | Guarani |
 | **jod** | Wojenaka |
 | **jud** | Worodougou |
 | **kfo** | Koro (Côte d'Ivoire) |
 | **kga** | Koyaga |
 | **kin** | Kinyarwanda |
 | **lin** | Lingala |
 | **lua** | Luba-Lulua |
 | **miq** | Mískito |
 | **mku** | Konyanka Maninka |
 | **msc** | Sankaran Maninka |
 | **mxx** | Mahou |
 | **mzj** | Manya |
 | **que** | Quechua |
 | **snk** | Soninke |
 | **spa** | Spanish |
 | **srr** | Serer |
 | **swa** | Swahili (macrolanguage) |
 | **wol** | Wolof |


## Country Calling Codes 
 | **Country** | **Calling code** |
| --- | --- |
 | **Bolivia, Plurinational State of** | 591 |
 | **Congo, the Democratic Republic of the** | 243 |
 | **Colombia**  | 57 |
 | **Ghana** | 233 |
 | **Guatemala** | 502 |
 | **Honduras** | 504 |
 | **Paraguay** | 595 |
 | **Rwanda** | 250 |
 | **Senegal** | 221 |
 | **El Salvador** | 503 |
 | **Chad**  | 235 |
 | **Tanzania, United Republic of** | 255 |

##  Amount Format Convention 

The amounts in the API requests are formatted according to the following standard:

`######.##`

Whereby . (dot) will be used as the separator character for decimals (cents). No separator character is required (allowed) for thousands.

# Result and error codes 

## HTTP status codes

The supported HTTP Status codes are shown in the table below.

 | **Code** | **Message** | **Type** | **Description** |
| --- | --- | --- | --- |
 | **200** | OK | Success | Returned for all Successful Responses |
 | **400** | Invalid Request | Error | One or more of the input mandatory fields are missing. Display an Invalid Request screen. |
 | **401** | Unauthorized | Error | API Key is not valid |
 | **403** | Forbidden | Error | API Key in not subscribed to service |
 | **405** | Method Not Allowed | Error | Served for Methods other than POST |
 | **406** | Not Acceptable | Error | Accept Header does not comply with x-www-form-urlencoded |
 | **500** | Internal Server Error | Error | Any Error that is returned from the back-end systems (see next subparagraphs) or errors that are not covered by this list |
 | **505** | HTTP Version Not Supported | Error | For Requests not on HTTP/1.1 protocol |

# General error codes 

## IP address not whitelisted 

The response below will be returned in case the IP address has not been whitelisted for the service called. Contact Millicom in order to add the correct IP address to the required services.

**Status code: HTTP/1.1 403 Forbidden**

```json
{
  "fault":{
      "faultstring":"Access Denied for client ip : _<IP address>_", 
    "detail":{
      "errorcode":"accesscontrol.IPDeniedAccess"
    }
  }
}
```

## Invalid Request

In case the header or JSON request body is incorrect the error as shown below will be returned. Please check the request is

**Status code: HTTP/1.1 400 Bad Request**
```json
{
  ErrorCode: "invalid\_request",
  Error: "Missing required parameter transactionRefId"
}
``` 
# Invalid Access Token 

The following error is returned in case an invalid Access Token is provided

**Status code: HTTP/1.1 401 Unauthorized** 

Invalid accessToken. Please enter valid token.

**Access Token Expired**

The following error is returned in case an invalid Access Token is provided

Status code: HTTP/1.1 401 Unauthorized

**Expired accessToken. Please enter valid token.**

**Transaction Reference ID already used**

The error below is returned in case Transaction Reference ID has already been used for a previous transaction.

**Status code: HTTP/1.1 400 Invalid Request**

```json
{
  "ErrorCode": "invalid\_request",
  "Error": "transactionRefId already exists"
}
``` 

**A.6.3. API Permission error**

The error below is returned in case the API has not been activated for the given Client\_id and secret.

Status code: HTTP/1.1 401 Invalid Request

You don't have permission to access … API. Please contact Millicom admin

**Service specific result and error codes**

The subsections below list the Result and Error codes per service. The nominal (successful) cases are covered in Section 4.

In case of an error a JSON response is returned with the following structure:

```json
{
  "Fault":{
    "faultcode":"env:Server", 
    "faultstring":"_<error description>_",
    "detail":{
      "ValidateMFSAccountFault":{
      "ResponseHeader":{
        "GeneralResponse":{
          "correlationID":_<Tigo correlation ID>_,
          "status":"ERROR",
          "code":"_<error code>_",
          "description":"_<error description>"_
          }
        }
      }
    }
  }
}
```

The error code provide in the "code" field has the following format:

```<API>-<APIID>-<ERROR CODE>-<TYPE OF FAULT>``` where

API – Tigo API name

APIID – unique numeric identifier of the API within Tigo

Error code – based on the range and the fault type .

**Type of fault with:**

E – business fault – 3000 range

F – fatal errors (such as network related faults) 2000 range

V – validation fault –4000 range

S – Success –0000 range

W – warning (scenarios such as partial responses) – 6000 range

**Authorization Service result codes**

|**Description**|
| --- | --- | --|
| **00-S** | Successful Payment ||
| | **Description** | **Error Category** |
| **01-F** | Backend System error | Backend Error caused the transaction to be terminated |
| **02-F** | Transaction timed out | The transaction timed out caused by timing out |
| **11-E** | Unable to complete transaction invalid amount | Unable to complete transaction due to invalid amount |
| **43-E** | Transaction not authorized | The customer did not authorize and therefore the transaction failed, this can be caused by the customer not authorizing payment, incorrect verification code, insufficient balance etc. |
| **45-E** | Cancel Transaction | The customer doesn't wish to continues with the transaction and wants to cancel it at its current state |

**FS Account Service result codes**

|**Description**|
|-----|-------|--------|
|**t-3018-0000-S** |Provided MSISDN is a valid MFS Account.||
||**Description** | ** Error Category**|
|**t-3018-4501-V**| Invalid request, please check the input  and resubmit|OSB Validation Error|
|**t-3018-3001-E**|`<Backend error description>`| Backend Error|
|**t-3018-2501-F**|One or more backends may be down. Please try again later| Connection Error|
|**t-3018-2502-F**|Service call has timed out, Please try again later| Timeout error |
**t-3018-2505-F**|Service Authentication failed|OWSM Authentication Failure|
**t-3018-2506-F**|Customer is not authorized to use this service| OWSM Authenticaton Failure|
**t-3018-3603-E**| Internal Service Error has occured | Internal Service Error|
**t-3018-3999-E**| Unknown/Uncaught Error has occured | Unknown/Uncaught Error has occured |
**t-3018-4502-V**| Invalid ISD code passed in the MSISDN | Validation Error|
**t-3018-4503-V**| Web Service Implementation is not available for this countr | Validation Error|
**t-3018-4504-V**| Required additional parameters are not passed in the request | When the required additonal parameters are not passed in the request |
**t-3018-4505-V**| Duplicate Additional parameters passed in the request | When the required additional parameters are passed in the request repeatedly|
**t-3018-4506-V**| Invalid consumerId passed in the request | When the consuming application id passed is invalid |

**Remittance Service result codes**

|**Description**|
| --- | --- | --- |
| **-3017-0000-S** | The Transaction is completed successfully |
| | **Description** | **Error Category** |
| **-3017-2501-F** | One or more back ends may be down. Please try again later | Connection Error. This is a Tigo internal error, This is treated as 'Service not available' |
| **-3017-2502-F** | Service call has timed out. Please try again later | Timeout error |
| **-3017-2505-F** | Service Authentication Failed. | OWSM Authentication Failure. This is a Tigo error and should be treated as 'Service not available |
| **-3017-2506-F** | Consumer is not authorized to use this service | OWSM Authentication Failure. This is a Tigo error and should be treated as 'Service not available'|
| **-3017-3001-E** | `<Backend error description>` | Uncaught error from the Tigo MFS Platform, this should be treated as 'Service not available' |
| **-3017-3002-E** | Authorization failed | The authorization for the transaction with this Account details failed. Check the account details, or make a new request. If the error persists contact Tigo |
| **-3017-3003-E** | Password expired | The PIN/Password for the MFS account expired, please contact Tigo to reset the PIN|
| **-3017-3004-E** | Sender account suspended | The MFS Account has been suspended. Contact Tigo to resolve  |
| **-3017-3005-E** | Sender account does not exist | The provided MFS Account does not exist on the platform in the country. Check the MFS Account and send a new transaction. If error persists resolve|
| **-3017-3006-E** | Receiver Account suspended | The receiving MFS account has been suspended therefore a remittance is not possible to this account |
| **-3017-3007-E** | Receiver Account does not exist | The receiving MFS account does not exist and remittance is not possible to this account. |
| **-3017-3008-E** | Invalid amount specified | An invalid amount has been specified, send in a new transaction with a correct amount. |
| **-3017-3009-E** | Maximum balance threshold for receiver |The balance of the receiving MFS account has been exceeded and therefore a remittance is not possible to this account |
| **-3017-3010-E** | Maximum number of transaction for receiver account has been reached | The maximum number of transactions of the account has been reached and therefore a remittance is not possible to this account |
| **-3017-3011-E** | Maximum number of transaction for sender account reached| The maximum number of transactions of the account has been reached and therefore a remittance to this account is not possible |
| **-3017-3012-E** | Transaction amount is less than the minimum transaction limit | The amount as passed in the request is less than the minimum transaction limit|
| **-3017-3013-E** | Maximum transaction limit exceeded | The amount as passed in the request is more than the maximum transaction limit. |
| **-3017-3014-E** | Sender and receiver account are the same | The sender and receiver accounts as specified in the request are the same. |
| **-3017-3015-E** | Service timeout | The request to deposit the remittance has timed out, therefore has not been completed. |
| **-3017-3016-E** | Insufficient funds | The account used for sending funds has insufficient funds for the case of the Payment aggregator account this should not occur, if it does, Millicom has to be contacted |
| **-3017-3017-E** | Insufficient account permission | The account has insufficient permission to perform the deposit remittance. In case of the Payment aggregator account this should not occur, if it does, Millicom should be contacted  |
| **-3017-3018-E** | User not found | The account specified in the request cannot be found |
| **-3017-3603-E** | Internal service error has occurred. | Internal service error. This is a Tigo internal error, and should be treated as 'Service not available' |
| **-3017-3999-E** | Unknown/Uncaught error has occurred. | Unknown/Uncaught error has occurred. This a tigo internal error and should be treated as 'Service not available' |
| **-3017-4002-V** | Invalid amount passed in the request. | When the amount is less than or equal to zero |
