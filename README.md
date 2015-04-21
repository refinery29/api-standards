# Refinery29 API Standards

----

Hi. This API standards document is a work in progress. To contribute, please make a Pull Request recommending changes. We'll discuss PR's as a community and decide together which to incorporate.

Please limit PRs to one "idea." For example, if you want to update the "Error Handling" section with a suggestion for error codes, and the "HTTP Verbs" section to advocate for a new verb usage, please make two separate PRs.

If your idea isn't well-formed enough for a PR, feel free to open an Issue on this Github repo. We'll discuss it and figure out what changes to make.

**NOTE:** There's a good chance we'll make this repo public. Please don't discuss any proproietary info in Pull Requests or Issues!

----

* [Introduction](#introduction)
* [RESTful URLs](#restful-urls)
* [Content Type](#content-type)
* [URL Structure and Versioning](#url-structure-and-versioning)
* [HTTP Verbs](#http-verbs)
* [Responses](#responses)
* [Error Handling](#error-handling)
* [Pagination](#pagination)
* [Request & Response Examples](#request--response-examples)
* [Mock Responses](#mock-responses)
* [JSONP](#jsonp)
* [Automated Testing](#automated-testing)
* [Authentication](#authentication)
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

 * We don't think all of these guidelines should be applied retroactively to existing APIs and endpoints. Please apply the guidelines as you can as you're writing new code. If you starting a new API version, you should follow the guidelines strictly. But we're not advocating a bunch of breaking changes to content API v1 at the expense of business objectives!
 * You may need to deviate from these guidelines, even in a pristine, new codebase. That's OK, provided that 
   1. You have very good reasons for doing so. You should understand why the guideline you're breaking was put in place, and be able to articulate why that scenario doesn't apply to your situation. If your "break" is an improvement that could apply to all APIs, consider proposing a change to the standards.
   2. Your team has reached a consensus regarding the "break". Deviating from the guidelines isn't something a one or two engineers (even leads or architects!) should do alone.

## RESTful URLs

### General guidelines for RESTful URLs
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Use **plural nouns** only (no singular nouns).
	* Plural-only makes URLs consistent.
	* Plural english nouns can be hurdle for developers who's first language isn't english. `/people` vs `/person/1234`
* Use HTTP verbs (GET, POST, PUT, DELETE, PATCH) to operate on the collections and elements.
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
 * `https://dash.refinery29.com/api/0/...`
    OK to not have namespace with the subdomain.

In the future, we may pursue putting APIs on a subdomain.

### Good URL examples

* List of entries:
    * `GET http://www.refinery29.com/api/content/1/entries`
* Filtering is a query:
    * `GET http://www.refinery29.com/api/content/1/entries?year=2011&sort=desc`
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
    * http://www.refinery29.com/api/content/1/entry
    * http://www.refinery29.com/api/content/1/entry/1234
    * http://www.refinery29.com/api/content/1/publisher/magazine/1234
* Verb in URL:
    * http://www.refinery29.com/api/content/1/magazine/1234/create
* Filter outside of query string
    * http://www.refinery29.com/api/content/1/magazines/2011/desc

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.

The action taken on the representation will be contextual to the media type being worked on and its current state. Here's are some examples of how HTTP verbs map in a particular context:

Remember: REST isn't the same model as CRUD. We've included some equivalents below, but if you're thinking in terms of Relational Database CRUD, you're probably going to end up with a poorly-designed RESTful API.

| METHOD | ENDPOINT | CRUD Equivalent | Notes | 
| ------ | -------- | --------------- | ----- |
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


## Error handling

**TODO:** Please submit PR's to improve this section.

```
Draft:
#### Obama Says

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined ID), links where developers can find more info. For example:

    {
      "status" : 400,
      "developerMessage" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "userMessage" : "This is a message that can be passed along to end-users, if needed.",
      "errorCode" : "444444",
      "moreInfo" : "http://www.example.gov/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }

Use three simple, common response codes indicating (1) success, (2) failure due to client-side problem, (3) failure due to server-side problem:

* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error

#### Phil Says

* Use HTTP Error Codes
* Return multiple errors when things go wrong.
* Phil recommends following http://jsonapi.org/format/#errors
Including a very specific code and a URL to your docs for that error is very helpful
* R29 Takeaways: We have a lot to learn here, there's not much consistency in error messages across our APIs. These are good tips.

**TODO:** Work on this!
```

## Pagination

**TODO:** Please submit PR's to improve this section.

```
Draft:

#### Obama Says

* If no limit is specified, return results with a default limit.
* To get records 51 through 75 do this:
    * http://example.gov/magazines?limit=25&offset=50
    * offset=50 means, ‘skip the first 50 records’
    * limit=25 means, ‘return a maximum of 25 records’

Information about record limits and total available count should also be included in the response. Example:

    {
        "metadata": {
            "resultset": {
                "count": 227,
                "offset": 25,
                "limit": 25
            }
        },
        "results": []
    }

#### Phil Says

Phil recommends cursors over pages, limits, offsets, etc.

R29 Takeaways: We think this is brilliant too. Cursors could be stored in redis or Aerospike depending on the platform.
```

## Request & Response Examples

**TODO:** Write some of these once we nail the other details down.

```
200 OK

{
   'result': [..., ]
		// metadata .. can include pagination, etc.
}
```
 
```
401 Bad Request
 
{
   'errors`: [...]
}
```


## Mock Responses

**TODO:** Please submit PR's to improve this section.

```
Draft:

#### Obama Says

**TODO:** Is this a good idea? Chassis could be written to do this with the same metadata we use to build the docs?

It is suggested that each resource accept a 'mock' parameter on the testing server. Passing this parameter should return a mock data response (bypassing the backend).

Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

**Note:** If the mock parameter is included in a request to the production environment, an error should be raised.
```

## JSONP

Consider carefully whether your endpoint should support JSONP, there are security implications.) 

If you do, support both `?callback=` and `?jsonp=` to enable JSONP wrappers in the response. 

## Authentication

**TODO:** Please submit PR's to improve this section.

```
Draft:

* Anything public-facing should be OAuth2 for sure.
* Monorail could become an OAuth2 provider pretty easily.
* Dash could use OAuth2, against either monorail or Google accounts. (Matt M: talk with IT about SSO)
* We're not sure about internal service<->service API calls where we're currently using API Keys.

```

## Automated Testing

 > "If you don't automate your testing, you don't have testing."  
 > -- Phil Sturgeon

**TODO:** Please submit PR's to improve this section.

```
Draft:

 * Dredd for testing documentation
 * Integration/Acceptance testing that hits the endpoints
 * Thorough unit testing
```

## Documentation

**TODO:** Please submit PR's to improve this section.

```
Draft:

There's nothing here.
```

## IDs

**TODO:** Please submit PR's to improve this section.

```
Draft:

* **Phil Says:** UUID/GUIDs are better than auto-incrementing values
* R29 Takeaway: We agree, sometimes. Monorail already exposes UUIDs for users. We're not sure it has the same advantages for entries, etc. but it's worth talking about.
```

## Limiting Returned Fields

By default, every API request should respond with all the fields on the speciefied resouce. 

However, sometimes you may want to allow clients to filter the response to only include the fields that they're going to use to reduce payload size. 

To accomplish this, us a `?fields=___` parameter. When the fields parameter is specified as a comma-separated list, your API should **only** return the fields requested by the client.

**Note:** Some APIs use a `exclude` parameter that acts inversely to `fields`. In order to avoid confusion, this standard encourages only implementing the `fields` paramete, allowing clients to specify what information they want to receive instead of what they don't want.


## Embedding Resources

Every API response is a complete RESTful resource by default. Sometimes, you may want to provide clients with additionaly linked resources without making extra API calls.

To accomplish this, use an `?include=____` parameter to embed additional data for linked resources.

The value is a comma-separated list of fields to expand into full objects, and can use periods to indicated nested objects.

For example, with no includes, an API might return this response for an appointment:

```json
{
  "date": "2016-01-20 12:43:54",
  "customer": "6ed82c31-1b5e-4a11-987b-37c96ccd6e91"
}
```

Called with `?include=customer.company` the same API would return:

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
