# Level 1 - Real time rates

## Use cases

### List of countries provided

To allow external services to integrate your solution you need to allow the service to retrieve the countries you operate in.

This API endpoint will be called in the best case only once a day and stored by the external service.

<img src="images/usecase-country.png" />

### Retrieve a quote

The external service will ask a quote from a specific country to another one and for a specific amount in order to receive all the possible combinations to transfer the money  (ways of paying, ways of receiving and how much you would receive for each combination)

<img src="images/usecase-quote.png" />








## Entry points 

### Countries

#### Get countries [GET]

Name	|	Description
----- | ----
Path	|	/countries
Method	|	GET
Authentication	|	Optional
Description	|	Get all countries provided


##### 1. request

No parameters given

##### 2. Response


```shell
curl "http://example.com/api/v1.0/countries"
```

> The above command returns JSON structured like this:

```json
[
     {
         "countryIso":"FR",
         "isSending":true,
         "isReceiving":true,
         "currenciesFrom":["EUR","USD"],
         "currenciesTo":["EUR"]
      },
     {
         "countryIso":"GB",
         "isSending":true,
         "isReceiving":false,
         "currenciesFrom":["GBP"]
     }
]
```

The response is an array :

Name	|	Type	|	Requirement	|	Description
----- | ---- | ----- | ----
countryIso	|	string	|	Mandatory	|	Country
isSending	|	boolean	|	Mandatory	|	true if the country is available to send money from
isReceiving	|	boolean	|	Mandatory	|	true if the country is available to send money to
currenciesFrom	|	array	|	Conditional	|	List of Currencies available to pay the transaction
currenciesTo	|	array	|	Conditional	|	List of Currencies available to receive money









### Quotes

#### Get quote [GET]

Name	|	Description
----- | ----
Path	|	/quote
Method	|	GET
Authentication	|	Optional
Description	|	Get options for a corridor and an amount


##### 1. request

Name	|	Type	|	Requirement	|	Description
----- | ---- | ----- | ----
countryFrom	|	string	|	Mandatory	|	Sending country
currencyFrom	|	string	|	Mandatory	|	Sending currency
countryTo	|	string	|	Mandatory	|	Reception country
currencyTo	|	string	|	Mandatory	|	Reception currency
paymentAmount	|	decimal	|	Conditional	|	Sending amount requested (expressed in currencyFrom)
receptionAmount	|	decimal	|	Conditional	|	Receiving amount requested (expressed in currencyTo)

<aside class="notice">
Either fill paymentAmount or receptionAmount.
</aside>


##### 2. Response

<aside class="warning">
N.B. : PaymentAmount has to be all fee included. This is the exact amount the user will pay.
</aside>


```shell
curl "http://example.com/api/v1.0/quote?
countryFrom=FR&currencyFrom=EUR&countryTo=MX&currencyTo=MXN
&paymentAmount=100"
```

> The above command returns JSON structured like this:

```json
{
"options": [
   {
            "optionId": "666f6f2d6261722d71757578",
            "paymentMethod":"LOCALBANK",
            "receptionMethod":"LOCALBANK",
            "isEstimatedAmount":false,
            "min":10,
            "max":1000000000,
            "optionDetails" :
                {
                    "paymentAmount":100,
                    "receptionAmount":1784.45,  
                    "rate":18.02299,
                    "quoteExpiration":12569537329,  
                    "fee":1,
                    "time":{"b":true,"d":3}
                }
    },
   {
            "optionId": "666f6f3d2261722d73456229",
            "paymentMethod":"CREDITCARD",
            "receptionMethod":"CASH",
            "isEstimatedAmount":false,
            "min":1,
            "max":1000,
            "optionDetails" :
                {
                    "paymentAmount":100,
                    "receptionAmount":1622.07,  
                    "rate":18.02299,
                    "quoteExpiration":12569537329,  
                    "fee":10,
                    "time":{"b":false,"d":0}
                }
    }
 ]
}
```

Name | Type | Requirement | Description
----- | ---- | ----- | ----
options | Array | Mandatory | Array of options provided

* Options

Name | Type | Requirement | Description
----- | ---- | ----- | ----
optionId	|	string		| Mandatory	|	Unique option id
paymentMethod	|	string		| Mandatory	|	Payment Option code(credit card, wire transfer…) -- see dictionnary
receptionMethod	|	string		| Mandatory	|	Reception Option code (Cash, wire transfer…) -- see dictionnary
isEstimatedAmount	|	boolean		| Mandatory	|	true, if the receptionAmount is estimated
min	|	decimal		| Mandatory	|	Minimum paymentAmount for a transaction (expressed in currencyFrom)
max	|	decimal		| Mandatory	|	Maximum paymentAmount for a transaction (expressed in currencyFrom)
urgent	|	boolean		| Optional	|	True if the option is a “urgent” feature
who	|	integer		| Optional	|	1. if this solution is only available to send money to the user itself
 | | | 2. if this solution is only available to send to someone who is not the sender (ex : sending to an email)
 | | | 0. Available for both (by default)"
optionDetails	|	Object		| Optional	|	All the main information for the option (fees, amounts). This object can be null if the amount asked is outside the limits (min - max)

* optionDetails						

Name | Type | Requirement | Description
----- | ---- | ----- | ----						
paymentAmount	|	decimal		| Mandatory	|	Amount to pay (expressed in currencyFrom)
receptionAmount	|	decimal		| Mandatory	|	Reception amount (expressed in currencyTo)
rate	|	decimal		| Mandatory	|	Exchange rate used by the mto
quoteExpiration	|	dateTime		| Mandatory	|	The time at which the quote will be considered expired
fee	|	decimal		| Mandatory	|	Fixed fees applied to the transaction
 | | | This fee is calculated with the formula : (paymentAmount - fee)*rate = receptionAmount
 | | | (expressed in currencyFrom)
time	|	object		| Mandatory	|	An estimation of the time required to receive the money-- see below


* Time

Name | Type | Requirement | Description
----- | ---- | ----- | ----
d	|	integer	|	Mandatory	|	Number of days estimated between the transaction creation and the delivery
b	|	boolean	|	Mandatory	|	true if the days are business days

