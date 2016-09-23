# Level 2 - Create and follow transactions

## Use cases

### Transaction Creation

Create a transaction require 2 entries : 

1. Create the recipient -- see Create recipient [POST] 
2. Create the transaction itself -- see Create transaction [POST]

### Transaction payment

Return the information for the user to do the payment -- see  transaction payment information [GET

### Push Transaction notifications

A webhook which alerts the third party of the changes in the transaction status. -- see Get transaction [NOTIFICATION]






## Entry points 

### Recipient

#### Create Recipient [POST]

Name	|	Description 
----- | ----
Path	|	/sender/:senderid/recipient
Method	|	POST
Authentication	|	Necessary
Description	|	Create a new recipient. 
 | The sender is the client to which the recipient is attached.
 | A sender can be an individual or a company who has an account with you.


##### 1. request

Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
:senderid  |   string  |   Mandatory   |   Identification code of the sender (url parameter)
*    |   *   |   * |   see Recipient standardized

<aside class="notice">
The recipient information will be in the body with as index the "name" of the fields given in the recipient documentation (see Recipient standarized).
</aside>

##### 2. Response


```shell
curl -H "Content-Type: application/json" -X POST -d
'{
   "recipientType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "receptionType": "LOCAL_BANK",
   "recipientDetails" : {
       "iban" : "FR7630004003200001019471656",
       "bic": "ING23123XXX"
    }
}' "http://example.com/api/v1.0/recipient"
```

> The above command returns JSON structured like this:

```json
{ 
   "recipientId" : "121323", 
   "recipientType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "receptionType": "LOCAL_BANK",
   "recipientDetails" : {
       "iban" : "FR7630004003200001019471656",
       "bic": "ING23123XXX"
   }
}
```

The recipient information are returned.



### Transaction

#### Create transaction [POST]

Name	|	Description 
----- | ----
Path	|	/transaction
Method	|	POST
Authentication	|	Necessary
Description	|	Create a new transaction


##### 1. request


Name	|	Type	|	Requirement	|	Description 
----- | ---- | ----- | ----
optionId	|	string	|	Mandatory	|	Identification code of the quote
senderId	|	string	|	Mandatory	|	Identification code of the sender
recipientId	|	string	|	Mandatory	|	Identification code of the recipient

##### 2. Response

```shell
curl -H "Content-Type: application/json" -X POST -d
'{
 optionId: "1298212189821",
 senderId: "233239082389023",
 recipientId: "233208932889",
}' "http://example.com/api/v1.0/transaction"
```

> The above command returns JSON structured like this:

```json
{    
  "transactionId" : "433239082389023",
  "optionId": "1298212189821",
  "senderId": "233239082389023",
  "recipientId": "233208932889"
}
```

The transaction information are returned or an error code and message. (see level 1)



#### Get transaction [NOTIFICATION]

When one event is triggered on a transaction, you should send an HTTPS POST call to the Webhook’s url. The Webhook allows to track the state changes of the transaction in real time.

Name    |   Description 
----- | ----
Path    |   EXTERNAL_SERVICE_URL
Method  |   Webhook
Authentication  |   Necessary
Description |   Send to the external service the event about the transaction


##### 1. request


Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
transactionId    |   string  |   Mandatory   |   Identification code of the transaction
type    |   string  |   Mandatory   |   Notification type - see below
data |   Object  |   Optional   |   Notification data

##### 2. Response

```shell
curl -H "Content-Type: application/json" -X POST -d
'{
    type : "PAYMENT_ACCEPTED"
}' "http://webhook.handler.com/notification"
```


The acknowledgment of the notification is returned by the third party: empty body with http Code 200.

If the third party do not acknowledge. Your server should try to send the notification again every 10 minutes, up to 5 times. If there is still no acknowledge, an email should be sent to the third party technical contact.


##### 3. Dictionnary

Notification type values

Name    |   Type   
----- | ---- 
PAYMENT_ACCEPTED | The money transfer operator received the money for the transaction (or consider that the money will be paid due to special mention in contract)
PAYMENT_REJECTED | An error occur during the payment. The reason has to be add in data, ie :
    | { type: PAYMENT_REJECTED, data: { code: 1001} }
    | Code 1001 : transaction mispaid - amount inferior
    | Code 1002 : transaction mispaid - amount superior
CANCELED | Transaction has been canceled. The reason has to be add in data, ie :
     | { type: ‘canceled’, data: { code: 1103 } }
     | Code 1103: Payment not received in time
COMPLETE | Recipient has received the money. The transaction is done.


It will happen that the Money Transfer Operator and the Client or Third party have a special contract mentioning that the settlement of several transactions can arrive after the money delivery by the Money Transfer Operator. In these cases, a Push notification "PAYMENT_RECEIVED" can be sent directly



#### transaction payment information [GET]

Name    |   Description 
----- | ----
Path    |   /transaction/:transactionId/paymentInformation
Method  |   GET
Authentication  |   Necessary
Description |   Get the transaction payment information


##### 1. request


Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
transactionId    |   string  |   Mandatory   |   Identification code of the transaction

##### 2. Response

```shell
curl "http://example.com/api/v1.0/transaction/120932743321/paymentInformation"
```

> The above command returns JSON structured like this:

```json
{
  "paymentInformation":
    {
      "amount": "10000",
      "currency": "EUR",
      "paymentType": "LOCAL_BANK",
      "paymentDetails": {
           "bankName": "BANKOGAN",
           "bankCountry": "FR",
           "bankDetails": {
                  "iban": "FR230532923502343932",
                  "bic": "ING2412XXX"
           }
      }
    }
}
```

Name | Type | Requirement | Description 
 ----- | ---- | ---- | ----
paymentInformation | Object | Mandatory | Information needed to do the payment

* paymentInformation

Name | Type | Requirement | Constraint | Description 
 ----- | ---- | ---- | ---- | ---- 
amount    |   string      | Mandatory | Positive and not zero | Amount to pay
currency   |   currency      | Mandatory | See currency | Currency of the amount to pay
paymentType |   string      | Mandatory |   Must be ‘LOCAL_BANK’ | Payment option of the transaction
paymentDetails |   Object      | Mandatory |   See below | The information necessary for the payer to pay the transaction

* paymentDetails

Name | Type | Requirement | Constraint | Description 
 ----- | ---- | ---- | ---- | ---- 
bankName    |   string      | Mandatory | Regex : ^.{1,100} | Bank name of the account
bankDetails   |   Object      | Mandatory | See Local bank details | Details of the bank account
bankCountry |   country      | Mandatory |   see Country | Country of the bank account
reference |   string      | Optional | Regex : ^.{1,100} | Reference of the payment

### Recipient Standardized

#### Local bank recipient

Recipient fields for bank

> Example FRANCE recipient local bank :

```json
{
   "recipientType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "receptionType": "LOCAL_BANK",
   "recipientDetails" : {
       "iban" : "FR7630004003200001019471656",
       "bic": "ING23123XXX"
   }
}
```

> Example United-Kingdom recipient local bank :

```json
{
   "recipientType": "COMPANY",
   "companyName": "Virgin",
   "receptionType": "LOCAL_BANK",
   "recipientDetails" : {
       "sortCode" : "16-50-07",
       "accountNumber" : "1019471656"
   }
}
```

> Example Mongolia recipient local bank :

```json
{
   "recipientType": "INDIVIDUAL",
   "firstName" : "Enkhtuyaa",
   "lastName" : "Enkhjargal",
   "receptionType" : "LOCAL_BANK",
   "receptionDetails" : {
      "bic": "AGMOMNUB",
      "accountNumber" : "3212345678"
   },
    "address" : {
        "street" : "Denver Street #3, 11th Micro-District",
        "city" : "Ulaanbaatar",
        "postalCode" : "14190",
        "country" : "MN"
    }
   
}

```

Name    |   Requirement | Type | Constraint | Description 
 ----- | ---- | ---- | ---- | ---- 
recipientType | Mandatory | string | INDIVIDUAL, COMPANY | Is the recipient an Individual or a Company 
companyName | Conditional | string | Regex : ^.{1-255} | If recipient is a company
firstName | Conditional | string | Regex : ^.{1-255} | If recipient is a individual
lastName | Conditional | string | Regex : ^.{1-255} | If recipient is a individual
receptionType | Mandatory | string | ‘LOCAL_BANK’ or ‘SWIFT’ | Define the format of the information supplied in receptionDetails.
| | | | Must be the same as chosen in the option
receptionDetails | Mandatory | Object | see Reception Details | Data related to the money reception itself
address | Conditional | address | see address | Required for Any SWIFT recipient or LOCAL_BANK transfer to CA, MX, US and SG


### Field type

#### Country
A country is a two characters string as describe in the ISO 3166.

#### Currency
A country is a three characters string as describe in the ISO 5217.

#### Address
An address is an object with the following fields :

Name    |   Requirement | Type | Constraint | Description 
----- | ---- | ---- | ---- | ----
street | Mandatory | string | Regex : ^.{1,255} | Street and number
city | Mandatory | string | Regex : ^.{1,255} | city’s address
country | Mandatory | string | see country | Country’s address
postalCode | Mandatory | string | Regex : ^.{1,255} | Postal code of the address
state | Conditional | string | Regex : ^.{1,255} | State’s address


#### Bank details

##### LOCAL_BANK

* For iban countries

Name     |  Requirement  |  Type     |  Constraint   |  Description 
----- | ---- | ---- | ---- | ---- 
iban     |  Mandatory    |  string   |  Regex : ^[A-Z]{2}[0-9]{2}[A-Z0-9]{4}[0-9]{7}([A-Z0-9]?){0,16}    |  Iban code of the recipient’s account
bic  |  Mandatory    |  string   |  Regex : ^[0-9A-Z]{8}$|^[0-9A-Z]{11}$     |  Bank identification code
see List of iban countries


* Australia  

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
 BSB  |  Mandatory    |  string   |  Regex : ^\\d{6}$     |  Australian bank code
 accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{6,10}$  |  Account number


* Canada  

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
 routingNumber    |  Mandatory    |  string   |  Regex : ^\\d{9}$     |  Canadian bank code
 accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{5,12}$  |  Account number


* New-Zealand 

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
 bankCode     |  Mandatory    |  string   |  Regex : ^\\d{6}$     |  New Zealanders bank code
accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{1,10}$  |  Account number


* Hong kong 

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
clearingCode     |  Mandatory    |  string   |  Regex : ^\\d{3}$     |  Hong kong bank code
branchCode   |  Mandatory    |  string   |  Regex : ^\\d{3}$     |  Branch code
accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{6,10}$  |  Account number

* United Kingdom 

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
sortCode     |  Mandatory    |  string   |  Regex : ^\\d{6}$     |  British bank code
accountNumber    |  Mandatory    |  string   |  Regex : ^[0-9A-Z]{1,50}$     |  Account number

* South Africa 

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
CBC  |  Mandatory    |  string   |  Regex : ^\\d{6}$     |  South African bank code
accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{1,16}$  |  Account number

* United State of America

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
routingCode  |  Mandatory    |  string   |  Regex : ^[0-9A-Z]{9}$    |  US bank code
accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{1,17}$  |  Account number

* Mexico

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
clabe    |  Mandatory    |  string   |  Regex : ^\\d{18}$    |  Mexican code for account numbers

* Singapore   

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
bankCode     |  Mandatory    |  string   |  Regex : ^\\d{4}$     |  Singapore bank code
branchCode   |  Mandatory    |  string   |  Regex : ^\\d{3}$     |  Singapore branch code
accountNumber    |  Mandatory    |  string   |  Regex : ^\\d{7-11}$  |  Account number

* SWIFT

Name     |  Requirement  |  Type     |  Constraint   |  Description 
 ----- | ---- | ---- | ---- | ---- 
bic    |  Mandatory    |  string   |  Regex : ^[0-9A-Z]{8}$|^[0-9A-Z]{11}$ |  International bank identification code
 accountNumber | Mandatory | string | Regex : ^[0-9A-Z]{1,50}$ | Account number


#### List Iban Countries

List of Iban countries supported in this standard : 


Country name | Country iso2 | Currency
 ----- | ---- | ----
Austria     |  AT   |  EUR
Belgium  |  BE   |  EUR
Cyprus   |  CY   |  EUR
Czech Republic   |  CZ   |  CZK
Denmark   |  DK   |    DKK
Estonia  |  EE   |  EUR
Finland  |  FI   |  EUR
France   |  FR   |  EUR
France   |  FR   |  EUR
Germany  |  DE   |  EUR
Hungary  |  HU   |  HUF
Ireland  |  IE   |  EUR
Israel   |  IL   |  EUR
Israel   |  IL   |  ILS
Italy    |  IT   |  EUR
Latvia   |  LV   |  EUR
Lithuania    |  LT   |  EUR
Luxembourg   |  LU   |  EUR
Malta    |  MT   |  EUR
Monaco   |  MC   |  EUR
Netherland   |  NL   |  EUR
Norway   |  NO   |  NOK
Poland   |  PL   |  PLN
Portugal     |  PT   |  EUR
San Marino   |  SM   |  EUR
Slovakia     |  SK   |  EUR
Slovenia     |  SI   |  EUR
Spain    |  ES   |  EUR
Sweden   |  SE   |  SEK
Switzerland  |  CH   |  CHF
United Arab Emirates     |  AE   |  AED
