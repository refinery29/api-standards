# Refinery29 API Standards

----

Hi. This API standards document is a work in progress. To contribute, please make a Pull Request recommending changes. We'll discuss PR's as a community and decide together which to incorporate.

Please limit PRs to one "idea." For example, if you want to update the "Error Handling" section with a suggestion for error codes, and the "HTTP Verbs" section to advocate for a new verb usage, please make two separate PRs.

If your idea isn't well-formed enough for a PR, feel free to open an Issue on this Github repo. We'll discuss it and figure out what changes to make.

----

* [Introduction](#introduction)
* [RESTful URLs](#restful-urls)
* [Content Type](#content-type)
* [URL Structure and Versioning](#url-structure-and-versioning)
* [HTTP Verbs](#http-verbs)
* [Searching](#searching)
* [Responses](#responses)
* [Error Handling](#error-handling)
* [Pagination](#pagination)
* [Sorting](#sorting)
* [Request & Response Examples](#request--response-examples)
* [Mock Responses](#mock-responses)
* [JSONP](#jsonp)
* [Automated Testing](#automated-testing)
* [Documentation](#documentation)
* [IDs](#ids)
* [Limiting Returned Fields](#limiting-returned-fields)
* [Embedding Resources](#embedding-resources)

## Introduction

This document provides guidelines and examples for Refinery 29 APIs, encouraging consistency, maintainability, and best practices across applications. 

This document borrows heavily from:
* [White House API Standards](https://github.com/WhiteHouse/api-standards)
* [Phil Sturgeon](https://speakerdeck.com/philsturgeon/api-pain-points-confoo-2015)

These are *pragmatic* guidelines. We think this is the best way for Refinery29 to build quality, consistent APIs. However:

 * We don't think all of these guidelines should be applied retroactively to existing APIs and endpoints. Please apply the guidelines as you can as you're writing new code. If you starting a new API version, you should follow the guidelines strictly. But we're not advocating a bunch of breaking changes to existing APIs at the expense of business objectives!
 * You may need to deviate from these guidelines, even in a pristine, new codebase. That's OK, provided that 
   1. You have very good reasons for doing so. You should understand why the guideline you're breaking was put in place, and be able to articulate why that scenario doesn't apply to your situation. If your "break" is an improvement that could apply to all APIs, consider proposing a change to the standards.
   2. Your team has reached a consensus regarding the "break". Deviating from the guidelines isn't something a one or two engineers (even leads or architects!) should do alone.

## General Coding Conventions

* A function that performs validation should **return early** if validation fails. A function that does not perform validation should **return once**.


## RESTful URLs

### General guidelines for RESTful URLs
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Use **plural nouns** only (no singular nouns).
	* Plural-only makes URLs consistent.
	* Plural english nouns can be hurdle for developers who's first language isn't english. `/people` vs `/person/1234`
* Use HTTP verbs (GET, HEAD, POST, PUT, DELETE, PATCH) to operate on the collections and elements.
   * Yes, you should support PATCH. More on that below. 
* URL v. header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * If it doesn’t change the logic for each response, like authorization info, put it in the header.

## Content Type

* Always support JSON and return JSON by default. 
* Use `Content-Type` headers on the request to specify different response types, such as XML.

## URL Structure and Versioning

* Never release an API without a version number.
* Versions should be integers, not decimal numbers, with no prefix. For example:
    * Good: `1`, `2`, `3`
    * Bad: `v1`, `1.1`, `v1.2`, `v-1.3`, `foo`, `XVII`
* The version number goes between the api namespace and the resource location.
* Manage past API versions. Know what's being consumed and deprecate responsibly.
* You shouldn’t need to go deeper than resources/<#identifier>/resources.

Going forward, API urls should be in the format:

 * `http://www.refinery29.com/api/feed/2/...`
 * `http://www.refinery29.com/api/content/3/...` # Content API v3
 * `http://www.refinery29.com/api/shops/1/...`
 * `http://www.refinery29.com/api/login/1/...`
 * `https://admin.refinery29.com/api/0/...`
    OK to not have namespace with the subdomain.

In the future, we may pursue putting APIs on a subdomain.

### Good URL examples

* List of entries:
    * `GET http://www.refinery29.com/api/content/1/entries`
* Filtering is a query:
    * `GET http://www.refinery29.com/api/content/1/entries?year=2011&sort=-year`
    * `GET http://www.refinery29.com/api/content/1/entries?topic=economy&year=2011`
* A single entry in JSON format:
    * `GET http://www.refinery29.com/api/content/1/entries/1234`
* All assets in (or belonging to) this entry:
    * `GET http://www.refinery29.com/api/content/1/entries/1234/assets`
* Specify optional fields in a comma separated list:
    * `GET http://www.refinery29.com/api/1/content/1/entries/1234?fields=title,subtitle,date`
* Add a new article to a particular entry:
    * `POST http://www.refinery29.com/api/content/1/entries/1234/articles`

### Bad URL examples
* Non-plural noun:
    * `http://www.refinery29.com/api/content/1/entry`
    * `http://www.refinery29.com/api/content/1/entry/1234`
    * `http://www.refinery29.com/api/content/1/publisher/magazine/1234`
* Verb in URL:
    * `http://www.refinery29.com/api/content/1/magazine/1234/create`
* Filter outside of query string
    * `http://www.refinery29.com/api/content/1/magazines/2011/desc`

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.

The action taken on the representation will be contextual to the media type being worked on and its current state. Here's are some examples of how HTTP verbs map in a particular context:

Remember: REST isn't the same model as CRUD. We've included some equivalents below, but if you're thinking in terms of Relational Database CRUD, you're probably going to end up with a poorly-designed RESTful API.

| METHOD | ENDPOINT | CRUD Equivalent | Notes | 
| ------ | -------- | --------------- | ----- |
| HEAD | /users/1234 | --- | Determine if user record exists without generating response body. |
| GET | /users | -- | List Users. |
| GET | /users/1234 | READ | Retrieve one user record. |
| PUT | /users/1234 | UPDATE | Update one user record. Use this when your payload includes all the fields. |
| PATCH* | /users/1234 | UPDATE | Update one user record. Use this when your payload only includes fields to change. |
| POST | /users | CREATE | Create a new user record. |
| DELETE | /users/1234 | DELETE | Delete a user record. |

\* PATCH isn't widely supported in browsers. You can accept a `?method=PATCH` param on a POST request to emulate it for JS clients.

Phil Sturgeon recommends these methods for uploading images:

| METHOD | ENDPOINT | CRUD Equivalent | Notes | 
| ------ | -------- | --------------- | ----- |
| PUT | /users/12/image* | -- | Upload an image for the user (when the user can only have one image) |
| POST | /users/12/images | CREATE | Upload an image for the user (when the user can have multiple images) |

\* Note the use of a singular noun when we're providing direct access to an attribute that has a 1:1 relationship with a resource.

For images, pay attention to the content type on the request. Allow both: 
 * `image/png`, `image/jpeg`: Image data to upload
 * `application/json`: JSON payload with a URL of the image to upload

## Searching
Search endpoints should be GET requests that contain a `query` parameter.    

| METHOD | ENDPOINT | 
| ------ | -------- | 
| GET | /users/search?query=something |

## Responses

 * Don't repeat HTTP response codes (normal or error cases!) in the body.
 * Don't include a 'status', 'message', etc. at the top level.
 * DO use a top-level key "result" to wrap the returned data.
 * No values in keys

   * **Good example:** No values in keys:

	```
    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],
    ``` 

   * **Bad example:** Values in keys:

    ```
    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],
    ```

 
 * Metadata should only contain direct properties of the response set, not properties of the members of the response set

## DateTime

DateTime strings should be given in [ISO-8601 Format](https://tools.ietf.org/html/rfc3339)

```
{
	"id": 5,
	"created_at": "2016-02-15T16:31:01Z",
	"title": "Water Sample"
}
```

## Error handling

This section borrows heavily from (jsonapi.org)[http://jsonapi.org/format/#errors]

 * Don't repeat HTTP response codes (normal or error cases!) in the body.
 * Don't include a 'status', 'message', etc. at the top level.
 * DO use a top-level key "errors", that contains an array of all errors that were detected.
 * DO use an appropriate HTTP Response Code in the 400 range (for client errors) or the 500 range (for server errors)
 * Don't include any messages intended for the end-user. The client should be responsibe for displaying a user-friendly, localized error message to the user.
 * DO include the following keys for each error:
   * `title` A short description of the error. It SHOULD NOT change from occurrence to occurrence of the problem.
   * `code` A unique identifer for this class of error (More specific than an HTTP Response Code)
 * The following keys are Optional for each error:
   * `id` A unique identifier (preferably a uuid or guid) for this instance of the error, which can also be written to log files or other sources to aid the API developer in troubleshooting.
   * `links` An array of URLs to documentation or resources that may help the client developer in troubleshooting.
   * `detail` A human-readable explanation specific to this occurrence of the problem.


## Pagination

We strongly encourage cursor-based pagination, but understand that your API may need to implement page-based or offset-based pagination, "because reasons".

### Guidelines For All Pagination Types

**DON'T** use the following query params for anything other than pagination: `page`, `per-page`, `offset`, `before`, `after`, `next`, `prev`/`previous`, or `limit`.

**DO** include a top-level object named “pagination” and define the following within it:
  * `prev` - a link to the previous page of data.
  * `next` - a link to the next page of data.

```
{
    "result": [...],
    "pagination": {
        "prev": "/api/3/content/entries?before=mRo9YXb3bhlEG52g",
        "next": "/api/3/content/entries?after=cEEHJc5Smh7NCg9m"
    }
}
```

### Implementing Cursor-based Pagination 

Use **cursor-based** pagination when…

* You are dealing with very large (> 10k rows) data sets.
* You are dealing with real time data.

Before you implement cursor-based pagination, make sure you have a column of monotonically increasing, unique values to sort on. Without that [you cannot](http://www.sitepoint.com/paginating-real-time-data-cursor-based-pagination/) use cursor-based pagination.

* The API should check for parameters `before` or `after`, `limit`, and `order`.
* The API should treat `before` and `after` as *exclusive*.
* If `after` is present, the db query should return `limit` results where `field > after`.
* If `before` is present, the db query should return `limit` results where `field < before` and order is reversed. Results should be flipped before they are returned.
* If `before` and `after` are present, API should return a 400 error.
* Its response should include `prev`, a link where `before` is pulled from the first result.
* Its response should include `next`, a link where `after` is pulled from the last result.

Cursor-based pagination is often used with timestamps. Twitter [explains this use case](https://dev.twitter.com/rest/public/timelines
) well.


### Other Pagination Strategies

Use **offset-based** or **page-based** pagination when the following requirements are present:

* The UI allows sorting by a column of non-unique values.
* The UI allows jumping to a specific page of results.

#### Implementing Offset-based Pagination

* The API should check for integer parameters `offset` and `limit`.
* The db query should skip `offset` number of results and return only `limit` results.
* Its response should include `prev`, a link where `offset` = current `offset - limit`.
* Its response should include `next`, a link where `offset` = current `offset + limit`.

#### Implementing Page-based Pagination

* The API should check for integer parameters `page` and `per-page`.
* The db query should skip `(page - 1) * per-page` results and return only `per-page` results.
* Its response should include `prev`, a link where `page` = current `page - 1`.
* Its response should include `next`, a link where `page` = current `page + 1`.

### A note on SEO

The consumer of paginated results must take special care to ensure every result (not just the first page) is indexed by Google. Here are two [strategies](http://www.sitepoint.com/pagination-seo-red-flags-best-practices/):

**Put a `rel="nofollow"` element on each page of results.** Create another page that displays ALL results. Allow Google to index that page. Advantage: every result will be indexed. Disadvantage: links will point to the View All page, which will load slowly and require a lot of scrolling.

**Or include`rel="prev"` and `rel="next"` on each page of results.** Google’s bot will follow the links to index each page separately. Advantage: links will point to specific pages, and you won’t have to deal with a monster View All page. Disadvantage: Google’s bot might quit before it gets to the last page of results, so you can’t assume they’ll all be indexed.

## Sorting
* The API should check for a comma delimited parameter `sort`. : `?sort=column`
* All sorts are ASC by default
* To indicate DESC sort, prepend the field by `-`: `?sort=-column1, column2`

## Request & Response Examples

### Retrieve a list of resources

#### Request

```
GET http://www.refinery29.com/api/content/3/entries?limit=10
```

#### Response

```
200 OK

{
   "result": [
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... },
   	{ ... JSON Representation of Entry ... }
   ]
   "pagination": {
        "prev": "/api/3/content/entries?before=mRo9YXb3bhlEG52g",
        "next": "/api/3/content/entries?after=cEEHJc5Smh7NCg9m"
    }
}
```
 

### Retrieve a single resources

#### Request

```
GET http://www.refinery29.com/api/content/3/entries/mRo9YXb3bhlEG52g
```

#### Response

```
200 OK

{
   "result": { ... JSON Representation of Entry ... }
}
```
 
 
### Attempt to retrieve a resource that doesn't exist

#### Request

```
GET http://www.refinery29.com/api/content/3/entries/{Not-A-Valid-ID}
```

#### Response

```
404 Not Found
 
{
   "errors": [
   	{
   	    "code": "1234",
   	    "title": "No entry exists for that ID",
   	    "id": "4eb7d6a6-7e5b-4856-a131-942520f052e6",
   	    "links": ["http://docs.myapi.com/entries/errors#1234"],
   	    "detail": "The ID \"notAValidID\" does not represent a valid entry."
   ]
}
```

### Create a new resource

#### Request

```
POST http://www.refinery29.com/api/content/3/entries
Content-type: application/json; charset=utf-8 

{
    "title": "Breaking News...",
    "text": "Cupcake ipsum dolor sit. Amet donut apple pie. Croissant bear claw toffee halvah sugar plum cake."
}
```

#### Response

```
201 Created
Location: http://www.refinery29.com/api/content/3/entries/fcp2520f052e6

{
    "id": "fcp2520f052e6",
    "title": "Breaking News...",
    "text": "Cupcake ipsum dolor sit. Amet donut apple pie. Croissant bear claw toffee halvah sugar plum cake."
}
```


## Mock Responses

It is suggested that each resource accept a 'mock' parameter on the testing server. Passing `?mock=true` should return a mock data response (bypassing the data store and business logic).

Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

**Note:** If the mock parameter is included in a request to the production environment, an error should be raised.


## JSONP

Consider carefully whether your endpoint should support JSONP, there are security implications.) 

If you do, support both `?callback=` and `?jsonp=` to enable JSONP wrappers in the response. 

## Automated Testing

It is almost impossible to provide consistent, reliable APIs without testing. We recommend a multi-pronged approach to testing API codebases.

### Unit Tests

Unit tests verify all the building blocks of your API. Implement unit testing using the best practices for your programming language and/or frameworks, and ensure that you're testing all of the software components that you use to build your API.

Unit tests should NOT access the data store or integrate with other components. Use mocks to isolate the component you are testing.

### Acceptance Testing

The second layer of testing for a reliable API is Acceptance testing that excercises each endpoint. These tests should actually make HTTP(S) calls to your API and inspect the response. Acceptance tests should access a data store with test data or fixtures.

### Test your Documentation

Tools like [Dredd](https://philsturgeon.uk/api/2015/01/28/dredd-api-testing-documentation/) should be used to confirm that your API conforms to your [documentation](#documentation) and vice-versa.

### Automate Your Tests

 > "If you don't automate your testing, you don't have testing."  
 > -- Phil Sturgeon

Make it easy to run your tests, and configure your CI tools (Travis, Jenkins,...) to run the tests automatically. Put bariers in your way to make sure your team never deploys without running all the tests.


## Documentation

Document your APIs thoroughly. Assume the client will **not** have source code access. 

The prefered method is to put all API documentation in a single apiary.apib file at the root of the repository. *Refinery29 has tooling for formatting and publishing these files, and some repos provide tooling to generate them automatically from documentation metadata in the source code itself.*

Regardless of the method you use to write and publish your documentation, it should cover:

 * List all endpoints and resources
 * Define and explain all parameters each endpoint expects and allows
 * Provide sample responses for each endpoint
 * Describe which error codes are returned and how the error response is formatted

## IDs

Prefer exposing  UUID/GUIDs over auto-incrementing values.

## Limiting Returned Fields

By default, every API request should respond with all the fields on the specified resouce. 

However, sometimes you may want to allow clients to filter the response to only include the fields that they're going to use to reduce payload size. 

To accomplish this, us a `?fields=___` parameter. When the fields parameter is specified as a comma-separated list, your API should **only** return the fields requested by the client.

**Note:** Some APIs use a `exclude` parameter that acts inversely to `fields`. In order to avoid confusion, this standard encourages only implementing the `fields` parameter, allowing clients to specify what information they want to receive instead of what they don't want.


## Embedding Resources

Every API response is a complete RESTful resource by default. Sometimes, you may want to provide clients with additionaly linked resources without making extra API calls.

To accomplish this, use an `?include=____` parameter to embed additional data for linked resources.

The value is a comma-separated list of fields to expand into full objects, and can use periods to indicated nested objects.

For example, with no includes, an API might return this response for an appointment:

```json
{
  "date": "2016-01-20 12:43:54",
  "customer_id": "6ed82c31-1b5e-4a11-987b-37c96ccd6e91"
}
```

Called with `?include=customer` the same API would return:

```json
{
  "date": "2016-01-20 12:43:54",
  "customer": {
    "id": "6ed82c31-1b5e-4a11-987b-37c96ccd6e91",
    "name": "John Smith",
    "company_id": "37c96ccd-2c31-1b5e-4a11-6e91987b6ed8"
   }
}
```

And called with `?include=customer.company`:


```json
{
  "date": "2016-01-20 12:43:54",
  "customer": {
    "id": "6ed82c31-1b5e-4a11-987b-37c96ccd6e91",
    "name": "John Smith",
    "company": {
      "name": "SanCorp",
     }
   }
}
```

Whether `include` is supported, which fields it supports it for, and how deep queries are allowed should be decided for each endpoint and documented clearly.

----

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).
