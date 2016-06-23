# Customer to Business API specification 

**Abbreviations**

**Term**   | **Definition**
-----------|------------------------------------
SP         | Service Provider
OTP        | One Time Pin
API        | Application Programming Interface
C2B        | Customer to Business
SAG        | Service Access Gateway
WSDL       | Web Services Definition Language 
URL        | Uniform Resource Locator  

# Introduction

This section describes a Web service for integrating the M-Pesa Check out API to a merchant site. The overall scope of this Web service is to provide primitives for application developers to handle checkout process in a simple way.

## How it works

The C2B API was created to address the submitting of payment requests from merchant checkout without using the IFRAME. A new improved authentication details have been added to improve on security and additional parameters like merchant transaction ID and encrypted parameters are passed. The API is multi-threaded to improve on transaction processing speeds.

The API allows merchant initiate C2B (paybill via web) transactions. The merchant submits authentication details, transaction details, callback url and callback method.

After request submission, the merchant receives instant feedback with validity status of their requests. The C2B API handles customer validation and authentication via USSD push. The customer therefore confirms the transaction. If the customer validation fails or declines the transaction, the API makes a callback to merchant. Otherwise the transaction is processed and status is made through a callback.

This process flow makes the following assumptions:

- The service is an on demand service where the Merchant always initiates the request when there is a payment to be settled.
- The Merchant already has an account created by the SAG
- The merchant has been provided with End point URL.

## Check out Operation procedure

![Screenshot] (img/operation_procedure.jpg)

 1. The Merchant captures the payment details and prepares call to the SAG's endpoint.
 2. The Merchant invokes SAG's **processCheckOut** interface.

**The SAG validates the request sent and returns a response.**

 1. Merchant receives the **processCheckoutResponse** parameters namely TRX\_ID, ENC\_PARAMS, RETURN\_CODE, DESCRIPTION and CUST\_MSG (Customer message).
 2. The merchant is supposed to display the CUST\_MSG to the customer after which the merchant should invoke SAG's **confirmPaymentRequest** interface to confirm the transaction
 3. The system will push a USSD menu to the customer and prompt the customer to enter their BONGA PIN and any other validation information.
 4. The transaction is processed on M-PESA and a callback is executed after completion of the transaction.

## End Points
 1. Production Endpoint:
 2. [https://safaricom.co.ke/mpesa\_online/lnmo\_checkout\_server.php?wsdl](https://safaricom.co.ke/mpesa_online/lnmo_checkout_server.php?wsdl)
3. WSDL:  [https://safaricom.co.ke/mpesa\_online/lnmo\_checkout\_server.php?wsdl](https://safaricom.co.ke/mpesa_online/lnmo_checkout_server.php?wsdl)

## Operation: processCheckOut

The Merchant makes a SOAP call to the SAG from which the SAG will call the C2B process to complete the transaction on M-Pesa. The request has two parts namely; process CheckOutRequest and CheckOutHeader. CheckOutHeader carries authentication and validation parameters while processCheckOutRequest carry the transaction parameters.

### Request parameters: Request Header

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **MERCHANT\_ID** | xsd:string | No | Merchant ID:When SAG registers a merchant, SAG allocates an ID to the Merchant |
| **PASSWORD** | xsd:string | No | SAG issues the passkey to the Merchant for use in authentication which is an encrypted form of Merchant password issued on creation of the merchant account. Password consists of passkey concatenated with MERCHANT\_ID and TIMESTAMP using a defined method of encryption preferably base64 encoding. |
| **REFERENCE\_ID** | xsd:string | No | Reference to the product/service offered |
| **TIMESTAMP** | xsd:string | No | This is the date time as timestamp of the Merchant request processing page. This is used to generate a password. |

### Password encryption

The password should have parameters (MERCHANT­\_ID, passkey, TIMESTAMP) encrypted using base64\_encode and hashed using sha256.

Example below shows the order of parameters and how it should be done

`$MERCHANT\_ID = MERCHANT\_ID`

`$passkey = passkey provided by SAG`

`$TIMESTAMP = TIMESTAMP at merchants checkout page`

**NOTE:** The same should be passed to the SAG as part of CheckOutHeader input.

`$PASSWORD = base64\_encode(hash("sha256", $MERCHANT\_ID $passkey $TIMESTAMP));`

The result of the `SHA254` Hash should be in capital letters.

### Requestparameters: Request body

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **MERCHANT\_TRANSACTION\_ID** | xsd:string | No | This is the unique merchant transaction identifier. |
| **MSISDN** | xsd:string | No | This is Merchant customer / shopper phone number. It uniquely identifies shopper/ merchant customer. |
| **REFERENCE\_ID** | xsd:string | No | Reference to the product/service offered |
| **ENC\_PARAMS** | xsd:string | Yes | These are merchant encrypted figures, values or parameters that belong to them. It may be anything they want to be passed to help in transaction processing, control or pass. |
| **AMOUNT** | xsd:double | No | Merchant send the value of the transaction.|
| **CALL\_BACK\_URL** | xsd:string | No | This is the Merchant's payment notification endpoint URL. This is important to merchant and their customers to notify on transaction status and the request result. |
| **CALL\_BACK\_METHOD** | xsd:string | No | This is the call back method used to send parameters to the Merchant's payment notification endpoint |

### Sample request
```xml 
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="tns:ns">

<soapenv:Header>

<tns:CheckOutHeader>

<MERCHANT\_ID>898945</MERCHANT\_ID>

<PASSWORD>MmRmNTliMjIzNjJhNmI5ODVhZGU5OTAxYWQ4NDJkZmI2MWE4ODg1ODFhMTQ3ZmZmNTFjMjg4M2UyYWQ5NTU3Yw==</PASSWORD>

<TIMESTAMP>20141128174717</TIMESTAMP>

</tns:CheckOutHeader>

</soapenv:Header>

<soapenv:Body>

<tns:processCheckOutRequest>

<MERCHANT\_TRANSACTION\_ID>911-000</MERCHANT\_TRANSACTION\_ID>

<REFERENCE\_ID>1112254500</REFERENCE\_ID>

<AMOUNT>54</AMOUNT>

<MSISDN>2547204871865</MSISDN>

<!--Optional:-->

<ENC\_PARAMS></ENC\_PARAMS>

<CALL\_BACK\_URL>http://172.21.20.215:8080/test</CALL\_BACK\_URL>

<CALL\_BACK\_METHOD>xml</CALL\_BACK\_METHOD>

<TIMESTAMP>20141128174717</TIMESTAMP>

</tns:processCheckOutRequest>

</soapenv:Body>

</soapenv:Envelope>
```

### Output Message: processCheckOutResponse

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **ENC\_PARAMS** | xsd:string | No | These are encrypted parameters that merchant passes and are passed back to the merchant page during callback. |
| **RETURN\_CODE** | xsd:string | No | Return/Response Code. "00" for Success and "01" for Failed. |
| **DESCRIPTION** | xsd:string | No | Transaction Status description |
| **TRX\_ID** | xsd:string | No | Transaction ID as generated by the system. |
| **CUST\_MSG** | xsd:string | No | This is a customer message that informs the customer on how to complete the online checkout transaction validation. |

### Sample response message
```xml 

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="tns:ns">

<SOAP-ENV:Body>

<ns1:processCheckOutResponse>

<RETURN\_CODE>00</RETURN\_CODE>

<DESCRIPTION>Success</DESCRIPTION>

<TRX\_ID>cce3d32e0159c1e62a9ec45b67676200</TRX\_ID>

<ENC\_PARAMS/>

<CUST\_MSG>To complete this transaction, enter your Bonga PIN on your handset. if you don't have one dial \*126\*5# for instructions</CUST\_MSG>

</ns1:processCheckOutResponse>

</SOAP-ENV:Body>

</SOAP-ENV:Envelope>
```

## Operation: confirmTransaction

The Merchant makes a SOAP call to the SAG to confirm an online checkout transaction. The request has two parts namely; transactionConfirmRequest and CheckOutHeader. CheckOutHeader carries authentication and validation parameters while transactionConfirmRequest carry the transaction parameters.

### Request parameters: Request Header

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **MERCHANT\_ID** | xsd:string | No | Merchant ID:When SAG registers a merchant, SAG allocates an ID to the Merchant |
| **PASSWORD** | xsd:string | No | SAG issues the passkey to the Merchant for use in authentication which is an encrypted form of Merchant password issued on creation of the merchant account. Password consists of passkey concatenated with MERCHANT\_ID and TIMESTAMP using a defined method of encryption preferably base64 encoding. |
| **REFERENCE\_ID** | xsd:string | No | Reference to the product/service offered |
| **TIMESTAMP** | xsd:string | No | This is the date time as timestamp of the Merchant request processing page. This is used to generate a password. |

### Password encryption

The password should have parameters (MERCHANT­\_ID, passkey, TIMESTAMP) encrypted using base64\_encode and hashed using sha256.

Example below shows the order of parameters and how it should be done

$MERCHANT\_ID = MERCHANT\_ID

$passkey = passkey provided by SAG

$TIMESTAMP = TIMESTAMP at merchants checkout page.

**NOTE:** The same should be passed to the SAG as part of CheckOutHeader input.

$PASSWORD = base64\_encode(hash("sha256", $MERCHANT\_ID . $passkey . $TIMESTAMP));

### Requestparameters: Request body

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **MERCHANT\_TRANSACTION\_ID** | xsd:string | Yes | MERCHANT\_TRANSACTION\_ID:This is the unique merchant transaction identifier.
NB: If not provided, TRX\_ID must be present. |
| **TRX\_ID** | xsd:string | Yes | This is Merchant customer / shopper phone number. It uniquely identifies shopper/ merchant customer.
NB: If not provided, MERCHANT\_TRANSACTION\_ID must be present. |

### Sample request

```xml 
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="tns:ns">

<soapenv:Header>

<tns:CheckOutHeader>

<MERCHANT\_ID>898945</MERCHANT\_ID>

<PASSWORD>MmRmNTliMjIzNjJhNmI5ODVhZGU5OTAxYWQ4NDJkZmI2MWE4ODg1ODFhMTQ3ZmZmNTFjMjg4M2UyYWQ5NTU3Yw==</PASSWORD>

<TIMESTAMP>20141128174717</TIMESTAMP>

</tns:CheckOutHeader>

</soapenv:Header>

<soapenv:Body>

<tns:transactionConfirmRequest>

<!--Optional:-->

<TRX\_ID>?</TRX\_ID>

<!--Optional:-->

<MERCHANT\_TRANSACTION\_ID>911-000</MERCHANT\_TRANSACTION\_ID>

</tns:transactionConfirmRequest>

</soapenv:Body>

</soapenv:Envelope>
``` 

### Output Message: transactionConfirmResponse

| **Parameter** | **Type** | **Optional** | **Description** |
| --- | --- | --- | --- |
| **RETURN\_CODE** | xsd:string | No | Return/Response Code. "00" for Success and "01" for Failed. |
| **DESCRIPTION** | xsd:string | No | Transaction Status description |
| **MERCHANT\_TRANSACTION\_ID** | xsd:string | No | This is the unique merchant transaction identifier. |
| **TRX\_ID** | xsd:string | No | Transaction ID as generated by the system. |

### Sample response message
```xml 

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="tns:ns">

<SOAP-ENV:Body>

<ns1:transactionConfirmResponse>

<RETURN\_CODE>00</RETURN\_CODE>

<DESCRIPTION>Success</DESCRIPTION>

<MERCHANT\_TRANSACTION\_ID/>

<TRX\_ID>5f6af12be0800c4ffabb4cf2608f0808</TRX\_ID>

</ns1:transactionConfirmResponse>

</SOAP-ENV:Body>

</SOAP-ENV:Envelope>
```

## Operation: Payment Notification (Merchant Callback)

Upon completion of transaction on M-PESA, the SAG calls back the merchant's call back URL via HTTP POST, HTTP GET, XML Result Message depending on merchant callback method defined to pass the details and status of the complete transaction. A callback URL, callback method, username and password are defined on merchant registration. This acts as default parameters if the merchant does not specify the callback method and callback URL during request submission

The following parameters are passed back via the callback HTTP POST, HTTP GET, XML Result Message;

| **Parameter** | **Description** |
| --- | --- |
| **USERNAME** | (Optional) Username provided by merchant to authenticate the callback request. |
| **PASSWORD** | (Optional) Password provided by merchant to authenticate the callback request. |
| **MERCHANT \_TRANSACTION\_ID** | Merchant Transaction ID as generated by merchant system. This is passed back as passed during the request by merchant. |
| **M-PESA\_TRX\_DATE** | M-PESA transaction timestamp in the format YYYY-MM-DD HH24:MI:SS |
| **M-PESA\_TRX\_ID** | M-PESA receipt number |
| **AMOUNT** | Amount effected on M-PESA |
| **TRX\_STATUS** | Transaction Status. Pending, Success, Failed, Error |
| **RETURN\_CODE** | Return/Response Code. "00" for Success and "01" for Failed. |
| **DESCRIPTION** | Transaction Status description |
| **TRX\_ID** | Transaction ID as generated by the system |
| **ENC\_PARAMS** | These are encrypted parameters that merchant passes and are passed back to the merchant page during callback. |
| **MSISDN** | This is the customer's number who initiated the payment. |

After the callback, the merchant can optionally redirect to either a success or fail page depending on the callback response. The merchant is expected to return the callback response to acknowledge receipt of callback through echoing or printing the word ' **ok'** or ' **success'**.

### Reference Faults

Below are the response codes on the Lipa na M-Pesa online solution.

| **RETURN CODE** | **CODE TYPE** | **DESCRIPTION** |
| --- | --- | --- |
| **00** | Success | Success. The Request has been successfully received or the transaction has successfully completed. |
| --- | --- | --- |
| **01** | Failure | Insufficient Funds on MSISDN account |
| --- | --- | --- |
| **03** | Failure | Amount less than the minimum single transfer allowed on the system. |
| --- | --- | --- |
| **04** | Failure | Amount more than the maximum single transfer amount allowed. |
| --- | --- | --- |
| **05** | Failure | Transaction expired in the instance where it wasn't picked in time for processing. |
| --- | --- | --- |
| **06** | Failure | Transaction could not be confirmed possibly due to confirm operation failure. |
| --- | --- | --- |
| **08** | Failure | Balance would rise above the allowed maximum amount. This happens if the MSISDN has reached its maximum transaction limit for the day. |
| --- | --- | --- |
| **09** | Failure | The store number specified in the transaction could not be found. This happens if the Merchant Pay bill number was incorrectly captured during registration. |
| --- | --- | --- |
| **10** | Failure | This occurs when the system is unable to resolve the credit account i.e the MSISDN provided isn't registered on M-PESA |
| --- | --- | --- |
| **11** | Failure | This message is returned when the system is unable to complete the transaction. |
| --- | --- | --- |
| **12** | Failure | Message returned when if the transaction details are different from original captured request details. |
| --- | --- | --- |
| **29** | Failure | System Downtime message when the system is inaccessible. |
| --- | --- | --- |
| **30** | Failure | Returned when the request is missing reference ID |
| --- | --- | --- |
| **31** | Failure | Returned when the request amount is Invalid or blank |
| --- | --- | --- |
| **32** | Failure | Returned when the account in the request hasn't been activated. |
| --- | --- | --- |
| **33** | Failure | Returned when the account hasn't been approved to transact. |
| --- | --- | --- |
| **34** | Failure | Returned when there is a request processing delay. |
| --- | --- | --- |
| **35** | Failure | Response when a duplicate request is detected. |
| --- | --- | --- |
| **36** | Failure | Response given if incorrect credentials are provided in the request |
| --- | --- | --- |
| **40** | Failure | Missing parameters |
| --- | --- | --- |
| **41** | Failure | MSISDN is in incorrect format |

### Sample HTTP GET Callback

```http 
http://10.0.0.1?MSISDN=254700000000&MERCHANT\_TRANSACTION\_ID= FG232FT0&USERNAME=&PASSWORD=&AMOUNT=100&TRX\_STATUS=Success &RETURN\_CODE=00&DESCRIPTION=Transaction successful&M-PESA\_TRX\_DATE=2014-08-01 15:30:00&M-PESA\_TRX\_ID=FG232FT0&TRX\_ID=1448&ENC\_PARAMS=XXXXXXXXXXX
```

### Sample HTTP POST (Plain)
```html

MSISDN:254700000000

AMOUNT:100

M-PESA\_TRX\_DATE:2014-08-01 15:30:00

M-PESA\_TRX\_ID:FG232FT0

TRX\_STATUS:Success

RETURN\_CODE:00

DESCRIPTION:Transaction successful

MERCHANT\_TRANSACTION\_ID:134562

ENC\_PARAMS:XXXXXXXXXXX

TRX\_ID:1448
```

### Sample XML Result Message
```xml 

<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="tns:ns">

<SOAP-ENV:Body>

<ns1:ResultMsg>

<MSISDN ns1:type="xsd:string">254720471865</MSISDN>

<AMOUNT ns1:type="xsd:string">54.0</AMOUNT>

<M-PESA\_TRX\_DATE ns1:type="xsd:string">2014-12-01 16:24:06</M-PESA\_TRX\_DATE>

<M-PESA\_TRX\_ID ns1:type="xsd:string">null</M-PESA\_TRX\_ID>

<TRX\_STATUS ns1:type="xsd:string">Success</TRX\_STATUS>

<RETURN\_CODE ns1:type="xsd:string">00</RETURN\_CODE>

<DESCRIPTION ns1:type="xsd:string">Success</DESCRIPTION>

<MERCHANT\_TRANSACTION\_ID ns1:type="xsd:string">911-000</MERCHANT\_TRANSACTION\_ID>

<ENC\_PARAMS ns1:type="xsd:string"></ENC\_PARAMS>

<TRX\_ID ns1:type="xsd:string">cce3d32e0159c1e62a9ec45b67676200</TRX\_ID>

</ns1:ResultMsg>

</SOAP-ENV:Body>

</SOAP-ENV:Envelope>
```

## Operation: Transaction Status Query

Transaction Query request helps to get the status of the transaction. It requires MERCHANT\_TRANSACTION\_ID and TRX\_ID. Other mandatory parameters are MERCHANT\_ID, PASSWORD generated and encrypted as instructed during a normal request and finally timestamp.

NOTE: MERCHANT\_TRANSACTION\_ID is very different from MERCHANT\_ID. MERCHANT\_TRANSACTION\_ID is the Id of each transaction. A figure generated by merchant to specify a certain transaction.

During transaction query the merchant must provide the TRX\_ID and optionally provide MERCHANT\_TRANSACTION\_ID.

### Sample request

Below is the sample transaction query request.
```xml 

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tns="tns:ns">

<soapenv:Header>

<tns:CheckOutHeader>

<MERCHANT\_ID>898945</MERCHANT\_ID>

<PASSWORD>MmRmNTliMjIzNjJhNmI5ODVhZGU5OTAxYWQ4NDJkZmI2MWE4ODg1ODFhMTQ3ZmZmNTFjMjg4M2UyYWQ5NTU3Yw==</PASSWORD>

<TIMESTAMP>20141128174717</TIMESTAMP>

</tns:CheckOutHeader>

</soapenv:Header>

<soapenv:Body>

<tns:transactionStatusRequest>

<!--Optional:-->

<TRX\_ID>?</TRX\_ID>

<!--Optional:-->

<MERCHANT\_TRANSACTION\_ID>911-100</MERCHANT\_TRANSACTION\_ID>

</tns:transactionStatusRequest>

</soapenv:Body>

</soapenv:Envelope>
``` 

### Transaction Status Query Request Response

```xml 
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="tns:ns">

<SOAP-ENV:Body>

<ns1:transactionStatusResponse>

<MSISDN>254720471865</MSISDN>

<AMOUNT>54000</AMOUNT>

<M-PESA\_TRX\_DATE>2014-12-01 16:59:07</M-PESA\_TRX\_DATE>

<M-PESA\_TRX\_ID>N/A</M-PESA\_TRX\_ID>

<TRX\_STATUS>Failed</TRX\_STATUS>

<RETURN\_CODE>01</RETURN\_CODE>

<DESCRIPTION>InsufficientFunds</DESCRIPTION>

<MERCHANT\_TRANSACTION\_ID/>

<ENC\_PARAMS/>

<TRX\_ID>ddd396509b168297141a747cd2dc1748</TRX\_ID>

</ns1:transactionStatusResponse>

</SOAP-ENV:Body>

</SOAP-ENV:Envelope>
``` 
