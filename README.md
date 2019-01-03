# Gateway #

Gateway provides a simplified, easy-to-use interface to the EXC Platform via the REST API. This page explains how to use the API to transfer money (e.g., EXC token) on the EXC Platform.


## API ##

#### Accounts ####

* [Generate Account - `GET /v1/accounts/new`](#generate-account)
* [Get Account Balance - `GET /v1/accounts/{:account_id}/balance`](#get-account-balance)

#### Payments ####

* [Submit Payment - `POST /v1/accounts/{:account_id}/payments`](#submit-payment)
* [Confirm Payment - `GET /v1/accounts/{:account_id}/payments/{:transaction_id}`](#confirm-payment)
* [Get Payment History - `GET /v1/accounts/{:account_id}/payments`](#get-payment-history)


## Excor Overview ##

### Excor Concepts ###

EXC Platform is a settlement system and is able to digitally transfer monetary value from A to B. One can transfer various forms of digital currency instantly via EXC Platform.

In the EXC Platform, each owner is identified by and assigned by its `Account ID`. `Account ID` is identified with uniquely assigned numbers and alphabets. For example: `0xDE092383A1B3CC71`

Transfer of money is arranged by specifying (1) the source or sender ID, (2) destination or receiver ID and (3) the monetary value to be remitted. 


### About EXC Account ###

There are two kinds of EXC Accounts: a business account and a user account.

The business account is created and managed by Excor. Please contact Excor Online Support(support@excor.org) for information on how to create a business account.


#### What you can do with your business account ####
* Create user accounts of the business as sub-accounts of the business account.
* Check the balance of the business account as well as its user accounts.
* Arrange money transfers from the business account to its user accounts – as sub-accounts of the business account.
* Acquire the history of the business account and all of its user accounts which is sub-accounts of the business account.

#### What you can do with your user account ####
* Check the balance of the user account.
* Arrange money transfer from one user account to another user account.
* Check historical statements of its user accounts.


### About Gateway ###

Gateway is a protocol converter that converts existing APIs to EXC platform protocols. REST API is prepared as it is now.

The encryption key of the EXC Account is stored in `HSM`, and signature and signature verification of the transaction are performed according to a request from the Gateway.


### About HSM ###

HSM is a dedicated hardware secured module specifically designed to protect the encryption key of EXC Account. (This is currently being developed and will be ready by XX.) HSM is used for authorization by the sender and signature verification. HSM made it possible to manage and store cryptographic keys safely inside of a tamper resistant module.

HSM is required to use Gateway.

HSM will be provided by Excor to the operator who created the business account.


# Running Gateway #


## Using Gateway ##

To use Gateway, you need an activated EXC account. 

Developers also need an HTTP client to allow secure HTTP GET and POST requests. For testing there are the following options.

 * The [`curl`](http://curl.haxx.se/) commandline utility
 * The [Advanced REST client Chrome extension](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=en)



# Formatting Conventions #

The current Gateway provides an API conforming to the behavior of the REST API.

* You make HTTP requests to the API endpoint, indicating the desired resources within the URL itself.
* The HTTP method identifies what you are trying to do. Generally, HTTP `GET` requests are used to retrieve information, while HTTP `POST` requests are used to make changes or submit information.
* If more complicated information needs to be sent, it will be included as JSON-formatted data within the body of the HTTP POST request.
    * This means that you must set `Content-Type: application/json` in the headers when sending POST requests with a body.
* Upon successful completion, the server returns an HTTP status code of `200 OK`, and a `Content-Type: application/json`. The body of the response will be a JSON-formatted object containing the information returned by the endpoint.

As an additional convention, all responses from Gateway contain a `"success"` field with a boolean value indicating whether or not the success


## Errors ##

When errors occur, the server returns an HTTP status code in the 400-599 range, depending on the type of error. The body of the response contains more detailed information on the cause of the problem.

In general, the HTTP status code is indicative of where the problem occurred:

* Codes in the 200-299 range indicate success. 
    * Unless otherwise specified, methods are expected to return `200 OK` on success.
    
* Codes in the 400-499 range indicate that the request was invalid or incorrect somehow.
    * `400 Bad Request`occurs if the JSON body is malformed. This includes syntax errors as well as when invalid or mutually-exclusive options are selected.
    * `404 Not Found`occurs if the path specified does not exist, or does not support that method (for example, trying to POST to a URL that only serves GET requests)
    
* Codes in the 500-599 range indicate that the server experienced a problem. This could be due to a network outage or a bug in the software somewhere.
    * `500 Internal Server Error`occurs when the server does not catch an error. This is always a bug.
    * `502 Bad Gateway`occurs if Gateway could not contact its Excor Network at all.
    * `504 Gateway Timeout`occurs if the Excor Network took too long to respond to the Gateway.
    
When possible, the server provides a JSON response body with more information about the error. These responses contain the following fields:

| Field | Type | Description |
|-------|------|-------------|
| success | Boolean | `false` indicates that an error occurred. |
| error_code | String | A short code identifying a general category for the error that occurred. |
| error | String | A human-readable summary of the error that occurred. |
| message | String | (Can be omitted) A longer human-readable explanation for the error. |

## Quoted Numbers ##

If Gateway needs to specify a large number, use JSON string instead of JSON numeric type. Applications need to parse these numbers to numeric types with reasonable precision.

If you do not know how much accuracy you need, we recommend using arbitrary-precision data types.

## Currency Amounts ##

On the Excor network, you do not use the decimal point to specify the amount. All amounts are specified as integers. Example: `1 EXC` is specified as` `1000000000` `in the EXC Platform.

The application must divide the amount by the appropriate number and use it. Applications need to determine exactly how much precision is needed and use the required data types.


# API #

## Generate Account ##

Create a user account linked to the business account set in Gateway.

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/new
```

<!-- </div> -->

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 34890239,
    "timestamp" : 1546196333,
    "account_id": "0xAABBCCDD11223344"
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Boolean | `true` if transaction succeeded. |
| transaction_id | Number | `Transaction ID` of this transaction. |
| timestamp | Number | `Timestamp (UNIX TIME)` of this transaction. |
| account_id | String | `Account ID` of generated account. |


## Get Account Balance ##

Gets the current balance of the given account.

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/balance
```

<!-- </div> -->

The following URL parameters are required by this API endpoint:

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | `Account ID` of the account whose balances to retrieve. |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 92348910,
    "timestamp" 1546197573: 
    "balance": "104629877312"
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Boolean | `true` if transaction succeeded. |
| transaction_id | Number | `Transaction ID` of this transaction. |
| timestamp | Number | `Timestamp (UNIX TIME)` of this transaction. |
| balance | String | Account balance. |


## Submit Payment ##

Transfer money from account to another account.

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
POST /v1/accounts/{:account_id}/payments
```

<!-- </div> -->

The following URL parameters are required by this API endpoint:

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | `Account ID` of the account from which money is transferred. |

#### Request Body ####

```js
{
    "payment": [
        {
            "destination_account_id": "0x99887766FFEEDDCC",
            "value" : "100001"
        },
        {
            "destination_account_id": "0xFF00EE11DD22CC33",
            "value": "123456"
        }
        ...
    ]
}
```

The request body must be a JSON object with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| destination_account_id | String | `Account ID` of the destination account. |
| value | String | Transfered value. |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 3982012,
    "timestamp" 1546198264: 
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Boolean | `true` if transaction succeeded. |
| transaction_id | Number | `Transaction ID` of this transaction. |
| timestamp | Number | `Timestamp (UNIX TIME)` of this transaction. |


## Confirm Payment ##

Get transfer history.

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/payments/{:transaction_id}
```

<!-- </div> -->

The following URL parameters are required by this API endpoint:

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | `Account ID` of account involved in transfer. |
| transaction_id | Number | `Transaction ID` of the transaction that transfer money。 |

#### Response ####

```js
{
    "success": true,
    "transaction_id" : 82948234,
    "timestamp" : 1546198964,
    "payment": [
        {
            "source_account_id": "0xAABBCCDDEE11223344",
            "destination_account_id": "0x99887766FFEEDDCC",
            "value": "129831"
        },
        {
            "source_account_id": "0xAABBCCDDEE11223344",
            "destination_account_id": "0xFF00EE11DD22CC33",
            "value": "349834"
        }
        ...
    ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | `true` if transaction succeeded. |
| transaction_id | Number | `Transaction ID` of the acquired transaction. |
| timestamp | Number | `Timestamp(UNIX TIME)` of the acquired transaction. |
| source_account_id | String | `Account ID` of the source account. |
| destination_account_id | String | `Account ID` of the destination account. |
| value | String | Transfered value. |


## Get Payment History ##

Get transfer history that affected the specified account.

<!-- <div class='multicode'> -->

<!-- *REST* -->

```
GET /v1/accounts/{:account_id}/payments
```

<!-- </div> -->

The following URL parameters are required by this API endpoint:

| Field   | Value | Description |
|---------|-------|-------------|
| account_id | String | `Account ID` of account involved in transfer. |

Optionally, you can also include the following query parameters:

| Field | Type | Description |
|-------|------|-------------|
| source_account_id | String | If provided, only include transfers sent by a given account. |
| destination_account_id | String | If provided, only include transfers received by a given account. |
| order | String | If true, sort results with the oldest transfer first. Defaults to false (sort with the most recent transfer first). |
| start_transaction_id | Number | If provided, exclude history older than the given `Transaction ID`. |
| end_transaction_id | Number | If provided, exclude history newer than the given `Transaction ID`. |
| limit | Number | The maximum number of transfers to be returned at once. Defaults to 10. Maximam is 100. |

#### Response ####

```js
{
    "success": true,
    "payments" : [
        {
            "transaction_id" : 948302234,
            "timestamp" : 1546199715,
            "payment": [
                {
                    "source_account_id": "0xAABBCCDDEE11223344",
                    "destination_account_id": "0x99887766FFEEDDCC",
                    "value": "129384"
                },
                {
                    "source_account_id": "0xAABBCCDDEE11223344",
                    "destination_account_id": "0xFF00EE11DD22CC33",
                    "value": "123912"
                }
                ...
            ]
        }
        ...
    ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| success | Bool | `true` if transaction succeeded. |
| transaction_id | Number | `Transaction ID` of the acquired transaction. |
| timestamp | Number | `Timestamp(UNIX TIME)` of the acquired transaction. |
| source_account_id | String | `Account ID` of the source account. |
| destination_account_id | String | `Account ID` of the destination account. |
| value | String | Transfered value. |
