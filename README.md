# DynamicsWebApi for Microsoft Dynamics CRM Web API

[![Travis branch](https://travis-ci.org/AleksandrRogov/DynamicsWebApi.svg?branch=master&style=flat-square)](https://travis-ci.org/AleksandrRogov/DynamicsWebApi)
[![Coveralls](https://coveralls.io/repos/github/AleksandrRogov/DynamicsWebApi/badge.svg?branch=master&style=flat-square)](https://coveralls.io/github/AleksandrRogov/DynamicsWebApi)

DynamicsWebApi is a Microsoft Dynamics CRM Web API helper library written in JavaScript.
It is compatible with: Dynamics 365 (online), Dynamics 365 (on-premises), Dynamics CRM 2016, Dynamics CRM Online.

Libraries for browsers can be found in the "[dist](/dist/)" folder.

Any suggestions are welcome!

## Table of Contents

* [Quick Start](#quick-start)
  * [Configuration](#configuration)
    * [Configuration Object Properties](#configuration-object-properties)
  * [Intellisense](#intellisense)
* [Request Examples](#request-examples)
  * [Create a record](#create-a-record)
  * [Update a record](#update-a-record)
  * [Update a single property value](#update-a-single-property-value)
  * [Upsert a record](#upsert-a-record)
  * [Delete a record](#delete-a-record)
	* [Delete a single property value](#delete-a-single-property-value)
  * [Retrieve a record](#retrieve-a-record)
  * [Retrieve multiple records](#retrieve-multiple-records)
  * [Count](#count)
  * [Associate](#associate)
  * [Associate for a single-valued navigation property](#associate-for-a-single-valued-navigation-property)
  * [Disassociate](#disassociate)
  * [Disassociate for a single-valued navigation property](#disassociate-for-a-single-valued-navigation-property)
  * [Fetch XML Request](#fetch-xml-request)
  * [Execute Web API functions](#execute-web-api-functions)
  * [Execute Web API actions](#execute-web-api-actions)
* [JavaScript Promises](#javascript-promises)
* [JavaScript Callbacks](#javascript-callbacks)

## Quick Start
In order to use a library DynamicsWebApi.js needs to be added as a Web Resource in CRM.

### Configuration
To initialize a new instance of DynamicsWebApi helper with a different configuration, please use the following code:

```js
var dynamicsWebApi = new DynamicsWebApi({ webApiVersion: "8.1" });
```

To set a configuration dynamically (if needed):

```js
dynamicsWebApi.setConfig({ webApiVersion: "8.2" });
```

#### Configuration Object Properties
Property Name | Type | Description
------------ | ------------- | -------------
__impersonate__ | String | A String representing the GUID value for the Dynamics 365 system user id. Impersonates the user.
__webApiUrl__ | String | A complete URL string to Web API. Example of the URL: "https:/myorg.crm.dynamics.com/api/data/v8.2/". If it is specified then webApiVersion property will not be used even if it is not empty. 
__webApiVersion__ | String | Version of the Web API. By default version "8.0" used.

At this moment the library only works inside CRM.

### Intellisense

The work on a rich Intellisense experience is still on going but developers can already use some of the features.

#### DWA Object

DWA is a JavaScript object located inside DynamicsWebApi library file. This object enables Intellisense for Visual Studio and therefore improves an overall productivity of a software developer.
The object contains several child objects which are described below.

##### DWA.Types
`DWA.Types` contains various objects (or so called Types) that are used in DynamicsWebApi and can be used by developers to enable Intellisense for some of the functions.
At this moment only Reponse objects are described there.

The following example shows how to use `DWA.Types` in the code:

Let's assume that we needed to write a retrieveMultipleRequest.

```js
dynamicsWebApi.retrieveMultipleRequest(request).then(function (response) {
	//properties of the response object are not known
})
```

This function returns an object which properties are not known at the runtime. This problem
can be easily resolved by using XML Documentation Comments for JavaScript.

```js
dynamicsWebApi.retrieveMultipleRequest(request).then(function (response) {
    /// <param name="response" type="DWA.Types.MultipleResponse">Request response</param>

	//now Intellisense is working
	var count = response.oDataCount;
    var nextPageLink = response.oDataNextLink;
    var records = response.value;
})
```

The following types are described at this moment: `DWA.Types.ReferenceResponse`, `DWA.Types.FetchXmlResponse` and `DWA.Types.MultipleResponse`.

##### DWA.Prefer
`DWA.Prefer` contains various values that can header "Prefer" to be set with. Most of the existing operations support such header. The following list describes which
header values can be used for which operations.

* `DWA.Prefer.ReturnRepresentation` - `create`, `update`, `updateSingleProperty` - allows developers to retrieve just created or updated object in a single request.
* `DWA.Prefer.Annotations` - `retrieveRequest`, `retrieveMultipleRequest`, `executeFetchXml` - allows to retrieve additional information about lookups, option sets and etc.

Examples:

```js

//DWA.Prefer.ReturnRepresentation ("return=representation")
dynamicsWebApi.update(recordId, "leads", lead, DWA.Prefer.ReturnRepresentation); // then... catch...

//DWA.Prefer.Annotations.FormattedValue ("OData.Community.Display.V1.FormattedValue")
var request = {
    id: recordId,
    collection: "leads",
    includeAnnotations: DWA.Prefer.Annotations.FormattedValue
};

dynamicsWebApi.retrieveRequest(request); //then... catch...
```

## Request Examples

DynamicsWebApi supports __Basic__ and __Advanced__ calls to Web API. 

Basic calls can be made by using functions with most commonly used input parameters. They are most convenient for simple operations as they do 
not provide all possible ways of interaction with CRM Web API (for example, [conditional retrievals](https://msdn.microsoft.com/en-us/library/mt607711.aspx#bkmk_DetectIfChanged)
are not supported in basic functions).

Basic functions are: `create`, `update`, `upsert`, `deleteRecord`, `retrieve`, `retrieveMultiple`, `count`, `executeFetchXml`, 
`associate`, `disassociate`, `associateSingleValued`, `disassociateSingleValued`, `executeBoundFunction`, `executeUnboundFunction`, 
`executeBoundAction`, `executeUnboundAction`.

Advanced functions have a suffix `Request` added to the end of the applicable operation. 
Most of the functions have a single input parameter which is a `request` object.

The following table describes all properties that are accepted in this object. __Important!__ Not all operaions accept all properties and if you by mistake specified
an invalid property you will receive either an error saying that the request is invalid or the response will not have expected results.

Property Name | Type | Operation(s) Supported | Description
------------ | ------------- | ------------- | -------------
collection | String | All | The name of the Entity Collection, for example, for `account` use `accounts`, `opportunity` - `opportunities` and etc.
count | Boolean | `retrieveMultipleRequest` | Boolean that sets the $count system query option with a value of true to include a count of entities that match the filter criteria up to 5000 (per page). Do not use $top with $count!
entity | Object | `updateRequest`, `upsertRequest` | A JavaScript object with properties corresponding to the logical name of entity attributes (exceptions are lookups and single-valued navigation properties).
expand | Array | `retrieveRequest`, `updateRequest`, `upsertRequest` | An array of Expand Objects (described below the table) representing the $expand OData System Query Option value to control which related records are also returned.
filter | String | `retrieveRequest`, `retrieveMultipleRequest` | Use the $filter system query option to set criteria for which entities will be returned.
id | String | `retrieveRequest`, `updateRequest`, `upsertRequest`, `deleteRequest` | A String representing the GUID value for the record.
ifmatch | String | `retrieveRequest`, `updateRequest`, `upsertRequest`, `deleteRequest` | Sets If-Match header value that enables to use conditional retrieval or optimistic concurrency in applicable requests. [More info](https://msdn.microsoft.com/en-us/library/mt607711.aspx).
ifnonematch | String | `retrieveRequest`, `upsertRequest` | Sets If-None-Match header value that enables to use conditional retrieval in applicable requests. [More info](https://msdn.microsoft.com/en-us/library/mt607711.aspx).
impersonate | String | All | A String representing the GUID value for the Dynamics 365 system user id. Impersonates the user.
includeAnnotations | String | `retrieveRequest`, `retrieveMultipleRequest` | Sets Prefer header with value "odata.include-annotations=" and the specified annotation. Annotations provide additional information about lookups, options sets and other complex attribute types.
maxPageSize | Number | `retrieveMultipleRequest` | Sets the odata.maxpagesize preference value to request the number of entities returned in the response.
navigationProperty | String | `retrieveRequest` | A String representing the name of a single-valued navigation property. Useful when needed to retrieve information about a related record in a single request.
orderBy | Array | `retrieveMultipleRequest` | An Array (of Strings) representing the order in which items are returned using the $orderby system query option. Use the asc or desc suffix to specify ascending or descending order respectively. The default is ascending if the suffix isn't applied.
returnRepresentation | Boolean | `updateRequest`, `upsertRequest` | Sets Prefer header request with value "return=representation". Use this property to return just created or updated entity in a single request.
savedQuery | String | `retrieveRequest` | A String representing the GUID value of the saved query.
select | Array | `retrieveRequest`, `retrieveMultipleRequest`, `updateRequest`, `upsertRequest` | An Array (of Strings) representing the $select OData System Query Option to control which attributes will be returned.
top | Number | `retrieveMultipleRequest` | Limit the number of results returned by using the $top system query option. Do not use $top with $count!
userQuery | String | `retrieveRequest` | A String representing the GUID value of the user query.

Basic and Advanced functions are also have differences in `expand` parameters. For Basic ones this parameter is a type of String 
while request.expand property is an Array of Expand Objects for Advanced operations. The following table describes Expand Object properties:

Property Name | Type | Description
------------ | ------------- | -------------
filter | String | Use the $filter system query option to set criteria for which related entities will be returned.
orderBy | Array | An Array (of Strings) representing the order in which related items are returned using the $orderby system query option. Use the asc or desc suffix to specify ascending or descending order respectively. The default is ascending if the suffix isn't applied.
property | String | A name of a single-valued navigation property which needs to be expanded.
select | Array | An Array (of Strings) representing the $select OData System Query Option to control which attributes will be returned.
top | Number | Limit the number of results returned by using the $top system query option.

According to CRM developers ([here](http://stackoverflow.com/a/34742977/2042071) and [here](https://community.dynamics.com/crm/b/joegilldynamicscrm/archive/2016/03/23/web-api-querying-with-expand) 
$expand does not work for retrieveMultiple requests which is claimed as a bug of CRM Web API.
As well as multi-level expands are not implemented yet. This situation may be changed with the future updates in the platform. Please look for the news!

For complex requests to Web API with multi-level expands use `executeFetchXml` function.

### Create a record

```js
//initialize a CRM entity record object
var lead = {
    subject: "Test WebAPI",
    firstname: "Test",
    lastname: "WebAPI",
    jobtitle: "Title"
};
//call dynamicsWebApi.create function
dynamicsWebApi.create(lead, "leads").then(function (id) {
    //do something with id here
}).catch(function (error) {
    //catch error here
})
```

### Update a record

#### Basic

```js
//lead id is needed for an update operation
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//initialize a CRM entity record object
//and specify fields with values that need to be updated
var lead = {
    subject: "Test update",
	jobtitle: "Developer"
}
//perform an update operation
dynamicsWebApi.update(leadId, "leads", lead).then(function () {
    //do something after a succesful operation
})
.catch(function (error) {
    //catch an error
});
```

#### Advanced using Request Object

```js
var request = {
    id: '7d577253-3ef0-4a0a-bb7f-8335c2596e70',
    collection: "leads",
	entity: {
        subject: "Test update",
		jobtitle: "Developer"
    },
    returnRepresentation: true,
    select: ["fullname"]
};

dynamicsWebApi.updateRequest(request).then(function (response) {
    var fullname = response.fullname;
	//do something with a fullname of a recently updated entity record
})
.catch(function (error) {
    //catch an error
});
```

### Update a single property value

```js
//lead id is needed for an update single property operation
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//initialize key value pair object
var keyValuePair = { subject: "Update Single" };

//perform an update single property operation
dynamicsWebApi.updateSingleProperty(leadId, "leads", keyValuePair).then(function () {
    //do something after a succesful operation
})
.catch(function (error) {
    //catch an error
});
```

### Upsert a record

#### Basic

```js
//lead id is needed for an upsert operation
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

var lead = {
    subject: "Test Upsert"
};

//initialize a CRM entity record object
//and specify fields with values that need to be upserted
dynamicsWebApi.upsert(leadId, "leads", lead).then(function (id) {
    //do something with id
})
.catch(function (error) {
    //catch an error
});
```

#### Advanced using Request Object

```js
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

var request = {
    id: leadId,
    collection: "leads",
    returnRepresentation: true,
    select: ["fullname"],
    entity: {
        subject: "Test upsert"
    },
    ifnonematch: "*" //to prevent update
};

dynamicsWebApi.upsertRequest(request).then(function (record) {
    if (record != null) {
        //record created
    }
    else {
        //update prevented
    }
})
.catch(function (error) {
    //catch an error
});
```

### Delete a record

#### Basic

```js
//record id is needed to perform a delete operation
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//perform a delete
dynamicsWebApi.deleteRecord(leadId, "leads").then(function () {
    //do something after a succesful operation
})
.catch(function (error) {
    //catch an error
});
```

#### Advanced using Request Object

```js
//delete with optimistic concurrency
var request = {
    id: recordId,
    collection: "leads",
    ifmatch: 'W/"470867"'
}

dynamicsWebApi.deleteRequest(request).then(function (isDeleted) {
    if (isDeleted){
		//the record has been deleted
	}
	else{
		//the record has not been deleted
	}
})
.catch(function (error) {
	//catch an error
});
```

#### Delete a single property value

```js
//record id is needed to perform a delete of a single property value operation
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//perform a delete of a single property value
dynamicsWebApi.deleteRecord(leadId, "leads", "subject").then(function () {
    //do something after a succesful operation
})
.catch(function (error) {
    //catch an error
});
```

### Retrieve a record

#### Basic

```js
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//perform a retrieve operaion
dynamicsWebApi.retrieve(leadid, "leads", ["fullname", "subject"]).then(function (record) {
    //do something with a record here
})
.catch(function (error) {
    //catch an error
});
```

#### Advanced using Request Object

```js
var request = {
    id: '7d577253-3ef0-4a0a-bb7f-8335c2596e70',
    collection: "leads",
    select: ["fullname", "subject"],

    //ETag value with the If-None-Match header to request data to be retrieved only 
	//if it has changed since the last time it was retrieved.
    ifnonematch: 'W/"468026"',

	//DWA object can be found at the top of the library file. 
	//It is helpful when used inside Visual Studio for better Intellisense experience.
	//Retrieved record will contain formatted values
	includeAnnotations: DWA.Prefer.Annotations.FormattedValue
};

dynamicsWebApi.retrieveRequest(request).then(function (record) {
    //do something with a record
})
.catch(function (error) {
    //if the record has not been found the error will be thrown
});
```

#### Retrieve a reference to related record using a single-valued navigation property

It is possible to retrieve a reference to the related entity (it works both in Basic and Advanced requests): `select: ["ownerid/$ref"]`. The parameter
must be the only one, it must be the name of a [single-valued navigation property](https://msdn.microsoft.com/en-us/library/mt607990.aspx#Anchor_5) 
and it must have a suffix `/$ref` attached to it. The returned object will be `DWA.Types.ReferenceResponse`. Example:

```js
var leadId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//perform a retrieve operaion
dynamicsWebApi.retrieve(leadid, "leads", ["ownerid/$ref"]).then(function (reference) {
    /// <param name="reference" type="DWA.Types.ReferenceResponse">Response</param>
    var ownerId = reference.id;
	var collectionName = reference.collection; // systemusers or teams
}) //.catch ...
```

#### Retrieve a related record data using a single-valued navigation property

In order to retrieve a related record by a signle-valued navigation property you need to add a prefix "/" to the __first__ element in a `select` array: 
`select: ["/ownerid", "fullname"]`. The first element must be the name of a [single-valued navigation property](https://msdn.microsoft.com/en-us/library/mt607990.aspx#Anchor_5) 
and it must contain a prefix "/"; all other elements in a `select` array will represent attributes of __the related entity__. Examples:

```js
var recordId = '7d577253-3ef0-4a0a-bb7f-8335c2596e70';

//perform a retrieve operaion
dynamicsWebApi.retrieve(recordId, "new_tests", ["/new_ParentLead", "fullname", "subject"])
	.then(function (leadRecord) {
		var fullname = leadRecord.fullname;
		//and etc...
	}) //.catch ...
```

In advanced request you have a choice to specify a `request.navigationProperty` or use it in the same way as for the Basic function.

```js
var request = {
    id: recordId,
    collection: "new_tests",
    navigationProperty: "new_ParentLead", //use request.navigationProperty
    select: ["fullname", "subject"]
}

//or

request = {
    id: recordId,
    collection: "new_tests",
    select: ["/new_ParentLead", "fullname", "subject"]    //inline with prefix "/"
}

dynamicsWebApi.retrieveRequest(request).then(function (leadRecord) {
    var fullname = leadRecord.fullname;
	//and etc...
}) // .catch...
```

### Retrieve multiple records

#### Basic

```js
dynamicsWebApi.retrieveMultiple("leads", ["fullname", "subject"], "statecode eq 0").then(function (records) {
    //do something with retrieved records here
})
.catch(function (error) {
    //catch an error
});
```

#### Advanced using Request Object

```js
//set the request parameters
var request = {
    collection: "leads",
    select: ["fullname", "subject"],
    filter: "statecode eq 0",
    maxPageSize: 5,
    count: true
};

//perform a multiple records retrieve operation
dynamicsWebApi.retrieveMultipleRequest(request).then(function (response) {
    /// <param name="response" type="DWA.Types.MultipleResponse">Response object</param>

    var count = response.oDataCount;
    var nextLink = response.oDataNextLink;
    var records = response.value;
    //do something else with a records array. Access a record: response.value[0].subject;
})
.catch(function (error){
    //catch an error
});
```

### Count

It is possible to count records separately from RetrieveMultiple call. In order to do that use the following snippet:

IMPORTANT! The count value does not represent the total number of entities in the system. It is limited by the maximum number of entities that can be returned.

For now please use dynamicsWebApi.retrieveMultipleRequest function and loop through all pages to rollup all records. dynamicsWebApi.countAll will be available soon.

```js
dynamicsWebApi.count("leads", "statecode eq 0").then(function (count) {
    //do something with count here
})
.catch(function (error) {
    //catch an error
});
```

### Associate

```js
var accountId = '00000000-0000-0000-0000-000000000001';
var leadId = '00000000-0000-0000-0000-000000000002';
dynamicsWebApi.associate("accounts", accountId, "lead_parent_account", "leads", leadId).then(function () {
    //success
}).catch(function (error) {
    //catch an error
});
```

### Associate for a single-valued navigation property

The name of a single-valued navigation property can be retrieved by using a `GET` request with a header `Prefer:odata.include-annotations=Microsoft.Dynamics.CRM.associatednavigationproperty`, 
then individual records in the response will contain the property `@Microsoft.Dynamics.CRM.associatednavigationproperty` which is the name of the needed navigation property. 
Usually it will be equal to a schema name of the entity attribute.

For example, there is an entity with a logical name `new_test`, it has a lookup attribute to `lead` entity called `new_parentlead` and schema name `new_ParentLead` which is needed single-valued navigation property.

```js
var new_testid = '00000000-0000-0000-0000-000000000001';
var leadId = '00000000-0000-0000-0000-000000000002';
dynamicsWebApi.associateSingleValued("new_tests", new_testid, "new_ParentLead", "leads", leadId)
	.then(function () {
		//success
	}).catch(function (error) {
		//catch an error
	});
```

### Disassociate

```js
var accountId = '00000000-0000-0000-0000-000000000001';
var leadId = '00000000-0000-0000-0000-000000000002';
dynamicsWebApi.disassociate("accounts", accountId, "lead_parent_account", leadId).then(function () {
    //success
}).catch(function (error) {
    //catch an error
});
```

### Disassociate for a single-valued navigation property
Current request removes a reference to an entity for a single-valued navigation property. The following code snippet uses an example shown in [Associate for a single-valued navigation property](#associate-for-a-single-valued-navigation-property).

```js
var new_testid = '00000000-0000-0000-0000-000000000001';
dynamicsWebApi.disassociateSingleValued("new_tests", new_testid, "new_ParentLead").then(function () {
    //success
}).catch(function (error) {
    //catch an error
});
```

### Fetch XML Request

```js
//build a fetch xml
var fetchXml = "<fetch mapping='logical'>" +
					"<entity name='account'>" +
						"<attribute name='accountid'/>" +
						"<attribute name='name'/>" +
					"</entity>" +
				"</fetch>";

dynamicsWebApi.executeFetchXml("accounts", fetchXml).then(function (response) {
    /// <param name="response" type="DWA.Types.FetchXmlResponse">Request response</param>

	//do something with results here; access records response.value[0].accountid 
})
.catch(function (error) {
    //catch an error
});
```

#### Paging

```js
//build a fetch xml
var fetchXml = "<fetch mapping='logical' count='5'>" +
					"<entity name='account'>" +
						"<attribute name='accountid'/>" +
						"<attribute name='name'/>" +
					"</entity>" +
				"</fetch>";

dynamicsWebApi.executeFetchXml("accounts", fetchXml).then(function (response) {
    /// <param name="response" type="DWA.Types.FetchXmlResponse">Request response</param>
	
	//do something with results here; access records response.value[0].accountid

	return dynamicsWebApi
        .executeFetchXml("accounts", fetchXml, null, response.PagingInfo.nextPage, response.PagingInfo.cookie);
}).then(function (response) {
    /// <param name="response" type="DWA.Types.FetchXmlResponse">Request response</param>
    
	//page 2
	//do something with results here; access records response.value[0].accountid

    return dynamicsWebApi
        .executeFetchXml("accounts", fetchXml, null, response.PagingInfo.nextPage, response.PagingInfo.cookie);
}).then(function (response) {
    /// <param name="response" type="DWA.Types.FetchXmlResponse">Request response</param>
	//page 3
	//and so on... or use a loop.
})
//catch...
```

### Execute Web API functions

#### Bound functions

```js
var teamId = "00000000-0000-0000-0000-000000000001";
dynamicsWebApi.executeBoundFunction(teamId, "teams", "Microsoft.Dynamics.CRM.RetrieveTeamPrivileges")
	.then(function (response) {
		//do something with a response
	}).catch(function (error) {
		//catch an error
	});
```

#### Unbound functions

```js
var parameters = {
    LocalizedStandardName: 'Pacific Standard Time',
    LocaleId: 1033
};
dynamicsWebApi.executeUnboundFunction("GetTimeZoneCodeByLocalizedName", parameters).then(function (result) {
    var timeZoneCode = result.TimeZoneCode;
}).catch(function (error) {
    //catch an error
});
```

### Execute Web API actions

#### Bound actions

```js
var queueId = "00000000-0000-0000-0000-000000000001";
var letterActivityId = "00000000-0000-0000-0000-000000000002";
var actionRequest = {
    Target: {
        activityid: letterActivityId,
        "@odata.type": "Microsoft.Dynamics.CRM.letter"
    }
};
dynamicsWebApi.executeBoundAction(queueId, "queues", "Microsoft.Dynamics.CRM.AddToQueue", actionRequest)
	.then(function (result) {
		var queueItemId = result.QueueItemId;
	})
	.catch(function (error) {
		//catch an error
	});
```

#### Unbound actions

```js
var opportunityId = "b3828ac8-917a-e511-80d2-00155d2a68d2";
var actionRequest = {
    Status: 3,
    OpportunityClose: {
        subject: "Won Opportunity",

		//DynamicsWebApi will add full url if the property contains @odata.bind suffix
		//but it is also possible to specify a full url to the entity record
        "opportunityid@odata.bind": "opportunities(" + opportunityId + ")"
    }
};
dynamicsWebApi.executeUnboundAction("WinOpportunity", actionRequest).then(function () {
    //success
}).catch(function (error) {
    //catch an error
});
```

### In Progress

- [ ] overloaded functions with rich request options for all Web API operations.
- [ ] get all pages requests, such as: countAll, retrieveMultipleAll, fetchXmlAll and etc.
- [ ] "formatted" values in responses. For instance: Web API splits information about lookup fields into separate properties, the config option "formatted" will enable developers to retrieve all information about such fields in a single requests and access it through DynamicsWebApi custom response objects.
- [ ] Intellisense for request objects.

Many more features to come!

Thank you for your patience!

## JavaScript Promises
Please use the following library that implements [ES6 Promises](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise): [DynamicsWebApi with Promises](/scripts/dynamics-web-api.js).

### Recommended
* [Yaku](https://github.com/ysmood/yaku) - ES6-Promise polyfill.

## JavaScript Callbacks
Please use the following library that implements Callbacks : [DynamicsWebApi with Callbacks](/scripts/dynamics-web-api-callbacks.js).
