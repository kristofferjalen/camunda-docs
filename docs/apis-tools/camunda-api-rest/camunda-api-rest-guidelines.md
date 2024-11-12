---
id: camunda-api-rest-guidelines
title: "Guidelines"
description: "Learn about the basic guidelines, structures, and conventions of the Camunda 8 REST API."
---

Camunda follows a mix of proposed standards and best practices for RESTful design and consistent implementation across all components. This ensures customers working across all component APIs have a consistent and expected experience without having to study our API reference material or perform “validation testing” to see how our APIs respond.

## Naming conventions

Naming should be simple, intuitive, and consistent across Camunda 8 APIs. This improves the experience when working across multiple APIs by reducing friction.

The API overall applies the following naming conventions:

- **Nouns** over verbs, for example, _assignment_ over _assign_.
- For top-level resources, use plural terms, for example, _user-tasks_.
- In path parameters, use **kebab-case** for multiple words. Use a hyphen (-) where a space would exist, for example, _user-tasks_.
- In query parameters, use **camelCase** for multiple words. Always capitalize the first letter of words after the first. The first letter in the first word must be lowercase, for example, _userTaskKey_.

These conventions can be observed in the following endpoint example:

> POST /user-tasks/{userTaskKey}/**assignment**

When working with IDs or similar short 2- or 3-letter words or acronyms, don’t capitalize all letters, only capitalize the first letter. If standalone, all letters should be lowercase.

| term | usage                                      |
| ---- | ------------------------------------------ |
| ID   | _id_ (standalone) or _processDefinitionId_ |
| URL  | _url_ (standalone) or _externalUrl_        |
| UUID | _uuid_ (standalone) or _clusterUuid_       |

Identifiers follow a naming rule, in parameters and data attributes alike:

- Unique technical identifiers are suffixed with **key**, for example, _userTaskKey_, _processInstanceKey_, _userKey_. They are numeric values in most cases.
- Other identifiers, such as copied identifiers from the BPMN XML, can be arbitrarily named but are usually suffixed with **id**, for example, _processDefinitionId_.

## Versioning

Camunda uses the term “major version number” from [semantic versioning](https://semver.org/), but do not follow semantic versioning for APIs outright. Instead, Camunda provides updates to the API in place and only increment the version number when a major, breaking change happens.

Adding attributes and endpoints are not considered breaking changes. Breaking changes can potentially break an integration. See [GitHub’s REST documentation](https://docs.github.com/en/rest/about-the-rest-api/breaking-changes?apiVersion=2022-11-28#about-breaking-changes-in-the-rest-api) for a comprehensive summary of breaking changes. No migration is required unless there is a breaking change.

The API version number does not match the product version (8.x.x). An API’s version is rather defined by the API version number and the product version, for example, “_POST /v2/user-tasks/search_ in Camunda 8.7.0”.

Camunda does API versioning rather than endpoint versioning, for example, the version changes for all endpoints if there is a breaking change in at least one endpoint. Multiple versions of an API can exist in a product version to allow for a migration period, for example, “POST /v2/user-tasks/search and POST /v3/user-tasks/search in Camunda 8.7.0”.

## HTTP status codes & error handling

Handling errors must be consistent across all endpoints, using well-known HTTP status codes and clear descriptions. This includes the information about errors and the use of a problem details object.

Camunda follows the proposed standard from [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457) for problem details. The problem object contains at least the following members:

- Type
- Status
- Title
- Detail
- Instance

Camunda uses the following error codes and descriptions across our APIs:

- 200 OK
- 204 No Content
- 400 Bad Request
  - Generic error that contains further description in the problem detail.
- 401 Unauthorized
  - The client is not authenticated yet.
  - The client should try again with a modified Authorization header.
- 403 Forbidden
  - The client has incorrect or insufficient permissions for the request.
- 404 Not Found
- 409 Conflict
  - The request is trying to modify a resource that is currently not in the right state.
- 412 Precondition failed
  - The client should check the cluster status.
- 429 Rate Limited Exceeded
  - The client exceeds a defined limit of requests, for example, Zeebe signaling backpressure due to more requests than the broker can currently process
- 500 Internal Server Error
  - Generic error that contains further description in the problem detail.

Each error code should have clear guidance in the documentation and API reference for how they should be handled. The problem detail object can provide guidance as well.

## Data fetching

Most resources provide at least one endpoint to fetch related data. Most of those endpoints provide data with near-real time consistency queried from exported records, if records for the respective resource are exported by the engine. If resources are not based on exported records, for example license data or topology information, the data returned by those endpoints can reflect real time insights or static content.

For most resources, there are search endpoints to query by POST method and a given query request object where applicable. The structure of such query requests always follows the same schema and so does the response, always returning a list of items matching the query criteria.

[Search requests](#search-requests) are forwarded as queries to the datastores that hold the exported records and the query results are returned in the format described in [search responses](#search-responses).

Resources can also support querying subordinate resources. For example, for users and groups, with user search being available at _POST /v2/users/search_, a user's groups can be retrieved using _POST /v2/users/{userKey}/groups/search_. Each resource determines independently if subordinate resources can be accessed this way.

Search endpoints can also be used to directly access entity instances with a unique identifier. As an alternative, resources can also provide GET method endpoints for fetching the data of single instances. In most cases, this is done by a specific key parameter in the URL path, e.g. _GET /v2/users/{userKey}_.

### Search requests

Query requests consist of the components for **filter**, **sort**, and **page**.

The **filter** object defines which fields should match. Only items that match the given fields will be returned. The available fields vary by object and are described in the respective search endpoint. Filtering by a unique identifier is usually available in filtering options. Beyond that, the filter options don’t have to comprise all the returned items’ attributes.

The **sort** object specifies which fields of the object should be sorted and whether they are sorted in ascending (ASC) or descending (DESC) order.

The **page** object details how to slice the result set. An initial search request can omit the page object or define the _limit_. This specifies the maximum number of results to retrieve per request. Subsequent requests can use the value of the returned _firstItemSortValues_ and _lastItemSortValues_ of the [search responses](#search-responses) to page through the items by copying that array into one of the attributes _searchAfter_ or _searchBefore_.

```
{
  "filter":  {
    <entity-specific  filter  options>
  },
  "sort":  [
    {  "<field>":  "<order>"  },
    {  "<field>":  "<order>"  }
  ],
  "page":  {
    "searchAfter":  [  …  ,  …  ],
    "searchBefore":  [  …  ,  …  ],
    "limit":  <limit>
  }
}
```

#### Advanced search

To provide an easy yet expressive way for users to search for and filter resources, search requests can contain more advanced filter criteria than fields being _equal_ to a target value.

For example, this allows searching using logical (and, in) and comparison operators (greater than, less than). The list of supported advanced filter operators is described below. Examples of advanced operators are shown in the [search example](#search-example).

##### Conditional operators

| Operator | Syntax                                    | Description                                                                                                                                               |
| -------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $eq      | field: { "$eq": value }                   | Filter where field is equal to value. Abbreviated form field: value is also allowed.                                                                      |
| $neq     | field: { "$neq": value }                  | Filter where field is not equal to value.                                                                                                                 |
| $exists  | field: { "$exists": value }               | Filter where field does or does not exist. The value is a boolean and can be either true or false.                                                        |
| $gt      | field: { "$gt": value }                   | Filter where field is greater than value. Supports numeric and [Date values](#date-values).                                                               |
| $gte     | field: { "$gte": value }                  | Filter where field is greater than or equal to value. Supports numeric and [Date values](#date-values).                                                   |
| $lt      | field: { "$lt": value }                   | Filter where field is less than value. Supports numeric and [Date values](#date-values).                                                                  |
| $lte     | field: { "$lte": value }                  | Filter where field is less than or equal to value. Supports numeric and [Date values](#date-values).                                                      |
| $like    | field: { "$like": value }                 | Filter where field contains a string like value. Use \* as a placeholder for an arbitrary number of characters anywhere in value. Supports string values. |
| $in      | field: { "$in": [ value1, value2, ... ] } | Filter where field is equal to any of the values in the provided array.                                                                                   |

##### Logical operators

| Operator | Syntax                                                      | Description                                                                                                               |
| -------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| $or      | "$or": [ { condition1 }, { condition2 }, ... ]              | Filter where at least one of the conditions is true.                                                                      |
| and      | { field: { "$lt": value1 }, field: { "$gt": value2 }, ... } | All conditions outside of $or operators will be considered as combined by an and operator. There is no explicit operator. |

##### Variable filtering

Search endpoints can support filtering by variable values. This allows querying for process-related resources based on the values of specific variables that exist in their respective scope. For example, user task search supports filtering using the _variables_ object and defining filter criteria for specific variables in there.

In general, the syntax looks as follows:

```
POST /v2/user-tasks/search

{
  "filter": {
    "variables" :  {
      "orderVolume":  10000,
      "foo": {"$lt": 500},
      "bar": {"$exists": false}
    }
  }
}
```

This filters for user tasks containing at least the variables _orderVolume_ with a value of _10000_ and _foo_ with a value lower than _500_, and **not** containing variable _bar_.

For variable values, the filter criteria outlined above for fields can be used.

### Search responses

Query responses consist of the components **items** and **page**.

The **items** object is an array of instances of the respective endpoint’s resource. The attributes of those instances vary by endpoint and are described in the endpoint’s documentation.

The **page** object contains the pagination information that can be used for subsequent search requests to page through the results. The _totalItems_ attribute specifies the overall number of results for the query to be retrieved (note: for ES/OS, this is limited to 10,000, even if more results are available).

The _firstSortValues_ field lists the criteria identifying the **first** entry of this page to allow paging backward in the result set by copying them into a respective page attribute like _searchBefore_ in a [search request](#search-requests). The _lastSortValues_ field provides the same for the **last** entry of this page to allow paging forward using _searchAfter_. In detail, those arrays contain the values of the first or last item’s attributes corresponding to the attributes defined in the request’s _sort_ object, in the same order. The last element of the array is the item’s value of our internal tiebreaker attribute: usually that is the internal record’s key.

```
{
  "items": [
    { <field1>: <value1>, <field2>: ... }
    { <field1>: <value1>, <field2>: ... }
  ],
  "page": {
    "totalItems": <total  hits>,
    "firstSortValues": [
      <sort field1 value>,
      <sort field2 value>,
      ... ,
      <internal tiebreaker value>
    ],
    "lastSortValues": [
      <sort field1 value>,
      <sort field2 value>,
      ... ,
      <internal tiebreaker value>
    ]
  }
}
```

### Search example

Querying for the first 3 user tasks with certain criteria, sorted by state, could look as follows:

```
POST /v2/user-tasks/search

{
  "filter": {
    "assignee": {"$eq": "demo"},
    "candidateGroups": { "$in": ["groupA", "groupB"] },
    "variables" : {
      "orderVolume": 10000,
      "foo": {"$lt": 500},
      "bar": {"$exists": false}
    },
    "$or": [
      { "priority": {"$eq": "high"}, "dueDate": { "$lt": "<date>" } },
      { "priority": {"$eq": "low"}, "followUpDate": { "$lt": "<date>" } }
    ]
  },
  "sort": [
    { "state": "ASC" }
  ],
  "page": {
    "limit": 3
  }
}
```

This could yield the following example result:

```
200 OK

{
  "items": [
    {
      "state": "CREATED",
      "processInstanceKey": 22456786958,
      "userTaskKey": 22456786345,
      ...
    },
    {
      "state": "CREATED",
      "processInstanceKey": 22456786958,
      "userTaskKey": 22456786456,
      ...
    },
    {
      "state": "COMPLETED",
      "processInstanceKey": 22456786958,
      "userTaskKey": 22456786678,
      ...
    }
  ],
  "page": {
    "totalItems":  345,
    "firstSortValues": [ "CREATED", 22456786345 ]
    "lastSortValues": [ "COMPLETED", 22456786678 ]
  }
}
```

A follow-up request to receive the next 3 items could then look as follows:

```
POST /v2/user-tasks/search

{
  "filter": {
    "assignee": {"$eq": "demo"},
    "candidateGroups": { "$in": ["groupA", "groupB"] },
    "variables" : {
      "orderVolume": 10000,
      "foo": { "$lt": 500 },
      "bar": { "$exists": false }
    },
    "$or": [
      { "priority": { "$eq": "high" }, "dueDate": { "$lt": "<date>" } },
      { "priority": { "$eq": "low" }, "followUpDate": { "$lt": "<date>" } }
    ]
  },
  "sort": [
    { "state": "ASC" }
  ],
  "page": {
    "limit":  3,
    "searchAfter": [ "COMPLETED", 22456786678 ]
  }
}
```

This yields the next three user task items after the last one from the first search request’s result.

## Date values

All date values in the REST API endpoints follow the [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) notation. This includes all requests and responses. Endpoints need to validate requests and transform responses accordingly.

## Variables

Variables in the REST API endpoints are proper JSON objects, where the key defines the variable name and the value specifies the variable value. This includes all requests and responses. Endpoints need to validate requests and transform responses accordingly.