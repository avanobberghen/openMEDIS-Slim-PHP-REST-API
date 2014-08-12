#openMEDIS REST API Specification v1


===
## <a name="whatis"></a>openMEDIS

* openMEDIS is a tool to collect, do inventory and process information on health technology infrastructure, e.g. medical devices. More information: [http://www.swisstph.ch/?id=1245](http://www.swisstph.ch/?id=1245)  
* As an addition to the browser-based openMEDIS, the mobile client for Android phones lets you manage assets, requests an interventions with your openMEDIS account:
[**Google Play Store**](https://play.google.com/store/apps/details?id=ch.swisstph.openmedis)
* All responses are *gzip encoded* to save bandwidth.
* All responses **and** requests should have the HTTP-Header: `Content-Type: application/json;charset=utf-8`

## Links
* [#1 **Authentication**](#1)
* [#2 **User/login**](#2)
* [#3-1 Create new **asset**](#3-1)
* [#3-2 Update **asset**](#3-2)
* [#3-3 Delete **asset**](#3-3)
* [#4-1 Create new **request**](#4-1)
* [#4-2 Update **request**](#4-2)
* [#4-3 Delete **request**](#4-3)
* [#5-1 Create new **intervention**](#5-1)
* [#5-2 Update **intervention**](#5-2)
* [#5-3 Delete **intervention**](#5-3)
* [#5-4 Create new **intervention material**](#5-4)
* [#5-4 Create new **intervention work**](#5-5)
* [#6-1 Export whole database **content**](#6-1)
* [#6-2 Export whole database **metainformation**](#6-2)


*You can link to subtitles of this document with anchors, e.g. with [example.com/#2-1](#2-1)*



## <a name="1"></a>1. Authentication
* The REST API uses a HMAC authentication.  
* Currently, only the `/login` route and this specification document at `/` can be accessed without authentication.

Every request **needs two HTTP header fields**, e.g. for the demo/demo user on a GET request:

```
X-PublicKey: 7286a32e66c2e8d38f6f51e91fa3d9ec4f33e09880a8c8835a2689e92236a1d4
X-Hash: e651e0f6450f89d82ab0a34c1d421097a635897f5e719179e49263ff145e6ed9
```

* **X-PublicKey:** This is the Public Key associated with every user with a login. It is generated only once on the server (and can not be repeated to get a different Public Key) on a `/login` POST request if valid credentials are provided (see [2.1 POST Login](#2-1) for details).

* **X-Hash:** This is the hash of the content in the HTTP request body **and** the private key stored internally on the client.
  1. The *secret private key* associated with every user: a SHA256 hash of the concatenated username and password, e.g. in PHP: `$private_key = hash('sha256', $username . $password);`
  2. The *HTTP request body* (on a GET request it is empty String). JSON request must be minified (whitespaces count for hash generation)
  3. So the whole X-Hash creation looks like this in PHP (similar "HMAC"-functions are available on many client platforms.)
`$hash = hash_hmac('sha256', $content, $private_key);`


If no valid headers are provided in the request, a `401 Unauthorized` status code will throw with no response HTTP body.




```
{"username":"demo","password":"demo"}
```

##### Response example

Success `200 OK`

```
{
    "error": false,
    "login_id": 123,
    "username": "demo",
    "group_id": 3,
    "locale": "en"
    "access_level": " AND (facilities.FacilityID = 11 OR facilities.FacilityID = 14 OR facilities.FacilityID = 13 OR facilities.FacilityID = 17 OR facilities.FacilityID = 16 OR facilities.FacilityID = 15 OR facilities.FacilityID = 12)",
    "public_key": "7286a32e66c2e8d38f6f51e91fa3d9ec4f34e09881a8c8835a2689e92236a1d4"
}
```

Authorization required `401 Unauthorized`

```
{
    "error": true,
    "message": "Login failed. Incorrect credentials"
}
```

> **Important**: A user has only access to assets / requests / interventions based on his access level which is specified in the employee table. A request to [/database/export](#6-1) returns only these based on the access level.

		"AssetFullName": "Aparat ultrasonograf",
		"GenericAssetID": 14278,
		"UMDNS": 14278,
		"ManufacturerID": "4b0bebf192bfb",
		"Model": "Aloca SSD-1100",
		"SerialNumber": "Mo2701",
		"InternalIventoryNumber": "1381018",
		"LocationID": "4b8fb11102c59",
		"ResponsiblePers": "asistenta superioar?",
		"AssetStatusID": 1,
		"AssetUtilizationId": 1,
		"PurchaseDate": "2002-01-10",
		"InstallationDate": null,
		"Lifetime": 7,
		"PurchasePrice": 264295,
		"CurrentValue": 0,
		"WarrantyContractID": 2,
		"AgentID": "4c90677ca7db7",
		"WarrantyContractExp": null,
		"WarrantyContractNotes": "nr.24 din 24.01.2010",
		"EmployeeID": "4c91bf1be32f6",
		"SupplierID": "4c497f7c7ef04",
		"DonorID": "4cc6a5791a3bd",
		"ServiceManual": "",
		"Notes": null,
		"Picture": "gen_images/14278.jpg",
		"lastmodified": "2010-03-31 13:20:21",
		"by_user": "anastasia",
		"URL_Manual": null,
		"MetrologyDocument": null,
		"MetrologyDate": null,
		"Metrology": null
	}



```
GenericAssetID, AssetFullName, ManufacturerID, Model, WarrantyContractID, SerialNumber, LocationID, DonorID, AssetStatusID, AssetUtilizationID, AgentID, SupplierID
```
        
        

{
	"error": true,
	"message": "Required field Metrology missing, empty or null"
}

{
	"error": true,
	"message": "Could not create asset"
}

Success – `201 (Created)`

```
{
	"error": false,
	"message": "Asset created"
}
```
		"AssetFullName": "Aparat ultrasonograf",
		"GenericAssetID": 14278,
		"UMDNS": 14278,
		"ManufacturerID": "4b0bebf192bfb",
		"Model": "Aloca SSD-1100",
		"SerialNumber": "Mo2701",
		"InternalIventoryNumber": "1381018",
		"LocationID": "4b8fb11102c59",
		"ResponsiblePers": "asistenta superioar?",
		"AssetStatusID": 1,
		"AssetUtilizationId": 1,
		"PurchaseDate": "2002-01-10",
		"InstallationDate": null,
		"Lifetime": 7,
		"PurchasePrice": 264295,
		"CurrentValue": 0,
		"WarrantyContractID": 2,
		"AgentID": "4c90677ca7db7",
		"WarrantyContractExp": null,
		"WarrantyContractNotes": "nr.24 din 24.01.2010",
		"EmployeeID": "4c91bf1be32f6",
		"SupplierID": "4c497f7c7ef04",
		"DonorID": "4cc6a5791a3bd",
		"ServiceManual": "",
		"Notes": null,
		"Picture": "gen_images/14278.jpg",
		"lastmodified": "2010-03-31 13:20:21",
		"by_user": "anastasia",
		"URL_Manual": null,
		"MetrologyDocument": null,
		"MetrologyDate": null,
		"Metrology": null
	}



```
GenericAssetID, AssetFullName, ManufacturerID, Model, WarrantyContractID, SerialNumber, LocationID, DonorID, AssetStatusID, AssetUtilizationID, AgentID, SupplierID
```
        
        

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not update asset"
}

Success – `200 (OK)`

```
{
	"error": false,
	"message": "Asset updated"
}
```

### <a name="3-3"></a>3.3 DELETE `/asset/{AssetID}`
   
        

{
	"error": true,
	"message": "Asset does not exist"
}

{
	"error": true,
	"message": "Could not delete asset"
}

Success – `200 (OK)`

```
{
	"error": false,
	"message": "Asset deleted"
}
```

## <a name="4"></a>4. request: Operations about asset requests



{
	"Request_date":"2014-02-14",
	"Request_desc":"needsrepair",
	"Request_contact_name":"Nicola Keller",
	"Request_note":"somenotes",
	"Request_st_id":1,
	"VisiTpID":4
}




Success - `201 (Created)`
```
{
    "error": false,
    "message": "Request added",
    "request_id": "53ce4f80b5db6",
    "asset_id": "4b0beb98ce160"
}
```

Missing fields `400 (Bad Request)`

```
{
    "error": true,
    "message": "Required field xy missing, empty or null"
}
```

Server error`500 (Internal server error)`

```
{
    "error": true,
    "message": "Could not add request"
}
```

### <a name="4-2"></a>4.2 PUT `/asset/{AssetID}/{RequestID}`

> Updates existing request of an specific asset

##### HTTP-Request example (should be minified)

```
{
	"Request_date":"2014-02-14",
	"Request_desc":"blub",
	"Request_contact_name":"democontact",
	"Request_note":"somenotes",
	"Request_st_id":"1",
	"VisiTpID":"4"
}
```

Required params:


##### HTTP-Response example

Success - `200 (OK)`

```
{
    "error": false,
    "message": "Request updated"
}
```

Missing field(s) – `400 (Bad Request)`

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not update request"
}

### <a name="4-3"></a>4.3 DELETE `/request/{RequestID}`

> Deletes existing request and associated interventions and its material or work.


##### HTTP-Response example

Success - `200 (OK)`

```
{
    "error": false,
    "message": "Request deleted"
}
```

Not found – `404 (Not found)`

{
	"error": true,
	"message": "Request not found"
}

{
	"error": true,
	"message": "Could not delete Request"
}

## <a name="5"></a>5. interventions: Operations about interventions on requests

### <a name="5-1"></a>5.1 POST `/asset/{AssetID}/{RequestID}/intervention`
> Creates new intervention on a specified request

##### HTTP-Request example (should be minified)

```
{
	"Date":"2014-01-01",
	"EmployeeID":"4d64045c4a525",
	"AssetStatusID":1,
	"FaildPart":"Cable",
	"FailurCategID":2,
	"FailureCauseID":1,
	"Interv_desc":"blub",
	"Comments":"yes",
	"RespEng":"4c8fcac9f06fa",
	"TotalWork":1,
	"TotalCosts":2
}
```
Required fields:

```
Date, EmployeeID, AssetStatusID, 'AssetID_Visit', FaildPart, FailurCategID, FailureCauseID, Interv_desc, Comments, RespEng, TotalWork, TotalCosts
```

##### HTTP-Response example

Success - `200 (OK)`

```
{
    "message": "Intervention added",
    "intervention_id": "53cf9860dd600"
}
```

Missing field(s) – `400 (Bad Request)`

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not add intervention"
}

{
	"error": true,
	"message": "Could not select intervention type of request"
}

> Updates existing intervention on a specified request

##### HTTP-Request example (should be minified)

```
{
	"Date":"2014-01-01",
	"EmployeeID":"4d64045c4a525",
	"AssetStatusID":1,
	"FaildPart":"Cable Red",
	"FailurCategID":2,
	"FailureCauseID":1,
	"Interv_desc":"blub",
	"Comments":"yes",
	"RespEng":"4c8fcac9f06fa",
	"TotalWork":1,
	"TotalCosts":2
}
```
Required fields:

```
Date, EmployeeID, AssetStatusID, 'AssetID_Visit', FaildPart, FailurCategID, FailureCauseID, Interv_desc, Comments, RespEng, TotalWork, TotalCosts
```

##### HTTP-Response example

Success - `200 (OK)`

```
{
    "error": false,
    "message": "Intervention updated",
    "request_id": "53ce48a405cd9",
    "intervention_id": "53cf9860dd600"
}
```

Missing field(s) – `400 (Bad Request)`

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not update intervention"
}

{
	"error": true,
	"message": "Could not select intervention type of request"
}


> Deletes existing intervention and associated intervention material or work.


##### HTTP-Response example

Success - `200 (OK)`

```
{
    "error": false,
    "message": "Intervention deleted"
}
```

Not found – `404 (Not found)`

{
	"error": true,
	"message": "Intervention does not exist"
}

{
	"error": true,
	"message": "Could not delete intervention"
}

> Creates new intervention material

##### HTTP-Request example (should be minified)

```
{
	"Description":"test",
	"Amount":10,
	"PartNumber":"abc",
	"UnitPrice":5.0,
	"IntervID":"0edc3bfd4e"
}
```
Required fields:

```
Description, Amount, PartNumber, UnitPrice, IntervID
```

##### HTTP-Response example

Success - `200 (OK)`

```
{
	"error": false,
	"message": "Intervention material created"
}
```

Missing field(s) – `400 (Bad Request)`

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not create intervention material"
}

> Creates new intervention work

##### HTTP-Request example (should be minified)

```
{
	"IntervID":"53d0cc77124de",
	"Action":"yes",
	"Date_action":"2014-01-01",
	"Time":20
}
```
Required fields:

```
IntervID, Action, Date_action, Time
```

##### HTTP-Response example

Success - `200 (OK)`

```
{
	"error": false,
	"message": "Intervention work created",
	"work_id": "53d0cc77124de"
}
```

Missing field(s) – `400 (Bad Request)`

{
	"error": true,
	"message": "Required field xy missing, empty or null"
}

{
	"error": true,
	"message": "Could not create intervention work"
}





{
	"assets":[
		{
		"AssetID":"4b67517f4462a",
		"GenericAssetID":"12636",
		"UMDNS":"12636",
		"AssetFullName":"Dash",
		"ManufacturerID":"4c44276c3c2c0",
		"Model":"4000 ",
		"SerialNumber":"SD008484463GA",
		...
		},
	],
	"location":[
		{
		"LocationID":"4b6b4f5120321",
		"FacilityID":"11",
		"DeptID":"2",
		...
		},
	...


> Returns database scheme for specified tables (in config.php editable). For each field the meta information of all specified tables (in config.php editable):   
`Fieldname, type, Null, Primary Key, Default, Extra`

##### HTTP-Response example
```
{
"assets":[
	{
		"Field":"AssetID",
		"Type":"char(13)",
		"Null":"NO",
		"Key":"PRI",
		"Default":null,
		"Extra":""
	},
	...
```

===