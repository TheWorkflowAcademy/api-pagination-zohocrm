# api-pagination
The api-pagination Deluge script enables you to handle pagination when calling an API. The Zoho CRM API is used here, but you can modify this to work with any API.

## Core Idea
Many APIs (including the Zoho CRM API) limit the number of results you may receive per API response page, but there is still a need to retrieve additional records from the API. When calling an API you *generally* have the option to specify the number of results per page (though limited) and the specific page of results. For example in the query parameters of the API URL, you may specify: 
```
?per_page=200&page=1
```

Deluge does not have native support for API pagination or `while`loops, which is problematic. This script enables the ability to paginate API requests in Deluge.

## Configuration
Before you can use this script with Zoho CRM, you must specify the following in the script:
1. Configured Zoho CRM Connection. Update the value for `connection` in the `invokeurl` method.
2. Zoho CRM module you wish to query (e.g. "Contacts", "Deals", etc). Update the `zohoCrmModule` variable.
3. Number of API page results you wish to get per request. The limit for the Zoho CRM API is 200. DO NOT exceed the per page limit, or else this script will not work. Update the `perPageLimit` variable.

### Zoho CRM Connection
In order to call the Zoho CRM API within a Deluge Script, you must first create a connection to it.

#### Connections In Zoho CRM
1. Go to the *Zoho CRM homepage* and click **setup**.
2. Under *Developer Space*, select **Connections**.
3. Click **Add Connection**, then select **Zoho OAuth**.
4. Specify a connection name (often *zohocrm*) and a scope, then click **Create and Connect**.
    * Scope sets the limits of data access to the connection. It is a security best practice to give the minimum required scope to a connection.
    * To retrieve records from modules select the **ZohoCRM.modules.ALL** scope.
    * For more information on Zoho CRM API scopes, [visit the Zoho CRM API documentation](https://www.zoho.com/crm/developer/docs/api/oauth-overview.html#scopes).

## Tutorial
### Page Iteration List
Since Deluge doesn't have a native `while` loop, we emulate its behavior. A `for each` iteration is performed over the `pageIterationList` variable. Within this block, we will rely on a boolean `iterationComplete` to determine whether we will continue to execute our API calls. The outer loop will technically continue to iterate, but will not be executing any internal code, thus allowing us to control iterations. 
```
...

pageIterationList = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25];

...

iterationComplete = false;

for each page in pageIterationList {

	if(iterationComplete == false) {
		...
	}
}
```
**Note:** comments removed for brevity

**Important**: This iteration list contains values from 1 to a number significantly greater than the *expected* count of result pages. If you don't want to type this list by hand, see the [Pinetools Number List Generator](https://pinetools.com/generate-list-numbers).

### Making API Request and `iterationComplete`
The API request will then be executed and manipulated into the `records` variable which will only contain the current requested records. The `records` will be subsequently added to the `allRecords` list which will contain the accumulation of all API requests.

Since the `for each` process is emulating a `while` loop we need set our `iterationComplete` variable. We will set `iterationComplete` to `true` only when there are no more records as indicated by the `more_records` parameter in the response.

```
...

allRecords = List();

...

for each page in pageIterationList {
	if(iterationComplete == false) {
		
		response = invokeurl [
			url: "https://www.zohoapis.com/crm/v2/" + zohoCrmModule + "?page=" + page + "&per_page=" + perPage
			type: GET
			connection: zohoCrmConnection
		];
		
		records = response.get("data");
		allRecords.addAll(records);
		
		if(response.get("info").get("more_records") == false) {
			iterationComplete = true;
		}
	}
}
```





