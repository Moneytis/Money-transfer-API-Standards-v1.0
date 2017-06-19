# Level 3 - Create senders and handle KY

## Use cases

### Create and manage senders

Creating a sender requires 2 entries : 

1. Create the sender -- see Create sender [POST]
2. Eventually upload Sender’s Documents (KYC) -- Upload sender document  [POST]


## Entry points 

### Sender

#### Create Sender [POST]

Name	|	Description 
----- | ----
Path	|	/sender
Method	|	POST
Authentication	|	Necessary
Description	|	Create a new sender. 



##### 1. request

Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
*    |   *   |   * |   see Sender standardized

<aside class="notice">
The sender information will be in the body with as index the "name" of the fields given in the user documentation (see Sender standardized).
</aside>

##### 2. Response

```shell
curl -H "Content-Type: application/json" -X POST -d
'{
   "senderType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "address" : {
        "street" : "Denver Street #3, 11th Micro-District",
        "city" : "Ulaanbaatar",
        "postalCode" : "14190",
        "country" : "MN"
   },
   "phoneNumber": "+33612345678",
   "nationality": "FR",
   "birthDate": "1980-08-12T12:00:00.000Z"
}' "http://example.com/api/v1.0/sender"
```

> The above command returns JSON structured like this:

```json
{ 
   "senderId" : "121323", 
   "senderType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "address" : {
        "street" : "Denver Street #3, 11th Micro-District",
        "city" : "Ulaanbaatar",
        "postalCode" : "14190",
        "country" : "MN"
   },
   "phoneNumber": "+33612345678",
   "nationality": "FR",
   "birthDate": "1980-08-12T12:00:00.000Z"
}
```

The sender information are returned.



#### Upload Sender Document [POST]

Name  | Description 
----- | ----
Path  | /sender/:senderId/uploadDocument
Method  | POST
Authentication  | Necessary
Description | Upload a new document for the sender.



##### 1. request

Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
:senderid  |   string  |   Mandatory   |   Identification code of the sender (url parameter)
documentType   |   string   |  Mandatory |   see Dictionnary
documentSubType |   string   |  Conditional |   see Dictionnary
file |   Object   |  Mandatory | the file itself containing the document

A file is :
Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
name  |   string  |   Mandatory   |  name of the document
buffer   |   data   |  Mandatory |   the file buffer
contentType  |   Object   |  Mandatory | content type of the document as defined in RFC 7231, section 3.1.1.5


##### 2. Response

```shell
curl -H "Content-Type: application/json" -X POST -d
'{
   "documentType" : "PROOFIDENTITY",
   "documentSubType" : "PASSPORT",
   "file": {
      name : “PassportVR.jpg”
      buffer : [ ... ]
      contentType : “image/jpeg”
   },

}' "http://example.com/api/v1.0/sender/:senderId/uploadDocument"
```

> The above command returns JSON structured like this:

```json
{ 
   "status" : "OK",
}
```

Only the status is returned.





#### Update Sender [PUT]

Name  | Description 
----- | ----
Path  | /sender/:senderId
Method  | PUT
Authentication  | Necessary
Description | Update a existing  sender. 

##### 1. request

Name    |   Type    |   Requirement |   Description 
----- | ---- | ----- | ----
senderId  | string  | Mandatory | Identification code of the sender
*    |   *   |   * |   see Sender standardized

<aside class="notice">
The sender information will be in the body with as index the "name" of the fields given in the user documentation (see Sender standardized).
</aside>

##### 2. Response

```shell
curl -H "Content-Type: application/json" -X PUT -d
'{
   "senderType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "address" : {
        "street" : "Denver Street #3, 11th Micro-District",
        "city" : "Ulaanbaatar",
        "postalCode" : "14190",
        "country" : "MN"
   },
   "phoneNumber": "+33612345678",
   "nationality": "FR",
   "birthDate": "1980-08-12T12:00:00.000Z",
}' "http://example.com/api/v1.0/sender/121323"
```

> The above command returns JSON structured like this:

```json
{ 
   "senderId" : "121323", 
   "senderType": "INDIVIDUAL",
   "firstName" : "Jean",
   "lastName" : "Dujardin",
   "address" : {
        "street" : "Denver Street #3, 11th Micro-District",
        "city" : "Ulaanbaatar",
        "postalCode" : "14190",
        "country" : "MN"
   },
   "phoneNumber": "+33612345678",
   "nationality": "FR",
   "birthDate": "1980-08-12T12:00:00.000Z"
}
```

The sender information are returned.










### Sender Standardized

Sender fields


Name    |   Requirement | Type | Constraint | Description 
 ----- | ---- | ---- | ---- | ---- 
senderType | Mandatory | string | INDIVIDUAL, COMPANY | Is the sender an Individual or a Company 
companyName | Conditional | string | Regex : ^.{1-255} | If sender is a company
firstName | Conditional | string | Regex : ^.{1-255} | firstName of the sender or the representative of the company 
lastName | Conditional | string | Regex : ^.{1-255} | lastName of the sender or the representative of the company
address | Mandatory | address | see address | address of the sender or the representative of the company
companyAddress | conditional | address | see address | If sender is a company
companyNumber | conditional | string | Regex : ^.{1-255} | If sender is a company
phoneNumber | Mandatory | string | Regex : ^.{1-255} | 
nationality | Mandatory | country | ISO 3166 | Nationality of the sender or the representative of the company
birthDate | Mandatory | date | at least 18years before now | date of birth of the sender or the representative of the company
ssn | conditional | string | Regex : ^[\\d]{9}$ | if US resident, Social Security number 


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

