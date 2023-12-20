# Frequently asked questions <!-- omit in toc -->


- [Are IDs in Dmaze GUIDs?](#are-ids-in-Dmaze-guids)
- [Why do some users have short GUID-like IDs and others have longer hash-like IDs?](#why-do-some-users-have-short-guid-like-ids-and-others-have-longer-hash-like-ids)
- [What is the correlation between a Users id and their Azure Active Directory Object ID?](#what-is-the-correlation-between-a-users-id-and-their-azure-active-directory-object-id)
- [How do I get a valid Dmaze ID?](#how-do-i-get-a-valid-Dmaze-id)
- [How can I get the OpenAPI (Swagger) specification for your API?](#how-can-i-get-the-openapi-swagger-specification-for-your-api)
- [What are the difference between an entity and an article?](#what-are-the-difference-between-an-entity-and-an-article)
- [What are the base required fields for an entity?](#what-are-the-base-required-fields-for-an-entity)
- [What are the base required fields for an article?](#what-are-the-base-required-fields-for-an-article)
- [What are the limits for articles?](#what-are-the-limits-for-articles)
- [What is sys_sequence vs. sequence?](#what-is-sys_sequence-vs-sequence)
- [How do I generate a link to the web app?](#how-do-i-generate-a-link-to-the-web-app)
- [What should the value for etag be?](#what-should-the-value-for-etag-be)
- [The API responds with 400 - Bad Request, how do I resolve the errors?](#the-api-responds-with-400---bad-request-how-do-i-resolve-the-errors)
- [The Open API specification says that an e_ or a_ field is required, do I need an ID in the 'values' array or can it be empty?](#the-open-api-specification-says-that-an-e_-or-a_-field-is-required-do-i-need-an-id-in-the-values-array-or-can-it-be-empty)

## Are IDs in Dmaze GUIDs?
Yes and now. Most of the IDs looks like Guids, but Dmaze and any other 3^rd^ party system should not treat IDs as Guids, but as strings of any length. The format and length can (and will) change at any time.

However, all IDs (except on user entity) are globally unique. User entity is the only exception - a user in Organization A has the same ID in Organization B (assuming the user has access to both). All other IDs should be considered globally unique. This means that it is safe to cache *almost* all objects using their ID only. 

## Why do some users have short GUID-like IDs and others have longer hash-like IDs?
As a general rule "long hash-like IDs" are from "proper" users syncronized from Azure AD or Auth0 logins. Whilst short "GUID-like IDs" are from "soft users" added manually by other users.

## What is the correlation between a Users id and their Azure Active Directory Object ID?
For Azure AD users we take their `Object ID` (`oid` claim) and perform a SHA-256 hash on it. For Auth0 users we take their `sub` claim and do the same SHA-256 hash. In .net the code to produce the hash looks like this:

```csharp
var objectId = "bfc744e5-e5c6-4072-9405-1521d29ddc93";
var bytes = Encoding.Unicode.GetBytes(objectId);
var hashstring = SHA256.Create();
var hash = hashstring.ComputeHash(bytes);
var hashedId = hash.Aggregate(string.Empty, (current, x) => current + $"{x:x2}");
```

## How do I get a valid Dmaze ID?
Simple, make a request to our ID endpoint

```
Request:
GET https://api.dmaze.com/id

Response:
{
  "results": [
    {
      "ids": [
        "4bc8cd90-ebdd-4d91-ba34-1a8bd9ed21d8",
        "55391f67-eef5-4b17-a137-4b4c0073fc00",
        "..."
      ]
    }
  ],
  "total": 0
}
```

By default you get 20 IDs in one call, these can be cached and consumed by your client.

## How can I get the OpenAPI (Swagger) specification for your API? 
In the Admin Application you can download the API specification (Open API v2) for any template. This document can be loaded into [Swagger Editor](https://swagger.io/tools/swagger-editor/) to try out the API endpoints. See the chapter on [aquire token](aquire_token.md) to learn how to get a token for the API. Prefix the token with `Bearer` in Swagger Editor to properly authenticate.

## What are the difference between an entity and an article?

In short an Entity is a code table - consider the canonical example of normalizing a database; given an order system where each order would have order lines and each order line would be refering to a product - in Dmaze this would be setup with `order` and `order line` as articles and `product` as entity (other properties omitted). The key here is that product will be used in many order lines, but does not (necessarily) refer to any other pieces of data (in real world it probably would!). 

From Dmaze APIs point of view the rules for articles and entities are quite different, see below for more details. Rule of thumb: If you are looking to syncronize 'static' data, such as locations, departments, product names etc. those are *usually* entities. But dynamic data, such as incidents, accidents, actions etc. are *usually* articles. 


## What are the base required fields for an entity?
All entities (locations, units, statuses etc.) requires:
<!-- no toc -->
- id
- name
- parentid ("0" for root)

Any other (non-nested) property is allowed, such as booleans and numbers. Nested properties are generally not supported for entities (some exceptions, un-documented atm.).

Example:
```json
{
    "id": "754be35a-dd34-4e17-9927-1215be41cbce",
    "name": "Name of the entity",
    "parentid": "a45ee5bc-f4b2-4cf6-93e5-cada4966b60a",
    "sequence": 100,
    "isdefault": false
}
```

## What are the base required fields for an article?
All articles (incidents, accidents etc.) requires:
<!-- no toc -->
- id
- title
- parentid ("0" for root)
- parenttype ("" for root)
- etag ("0" for new)
- sys_template 

Any other (non-nested) property is allowed, such as booleans and numbers. Nested properties are used to link to entities and children. Entity references starts with `e_` and child (article) references starts with `a_`.

Example:
```json
{
    "id": "5aa2837a-4a0d-48e5-9541-b044a10b9515",
    "title": "Title for the article",
    "parentid": "b24fd878-a0e6-41bc-95b9-ddeb2949c1ef",
    "parenttype": "incident",
    "numberofaffected": 20,
    "issevere": false,
    "e_wound_location_ids": {
        "type": "wound",
        "values": 
        [ 
            "0c2887c4-1a48-4d0e-9cab-38ee1491ed7e" 
        ]
    },
    "a_measures_ids": {
        "type": "measure",
        "values": 
        [ 
            "69800f7f-107e-4700-9f01-bc64d683b2a9", 
            "b66c09a0-57bd-4d60-be3e-a12b4660e365"
        ]
    },
    "a_causes": {
        "type": "cause",
        "values": 
        [ 
            "8a51233f-c2e6-472a-926a-38744c68fc63",
            "0d4d64b6-2ec0-4dfd-bf93-d4d4800e73ab"
        ]
    }
}
```

Any number of properties can be added, even properties not described in a template.

## What are the limits for articles?

Guidelines:

1. No more than 200 properties
2. No more than 2000 chars in a text field
3. No more than 100 Entity/Article references

## What is sys_sequence vs. sequence?
bla bla

## How do I generate a link to the web app?
The basic link to a root type and id looks like this: 

`https://app.Dmaze.com/article/[Root ID]?type=[Root Type]`

To link to a child article the link should look something like this:

`https://app.Dmaze.com/article/[Root ID]?child=[Child ID]&childType=[Child Type]&tab=[Tab Name]&type=[Root Type]`

Root Id and Type, Child Id and Type should be self explainatory, but "Tab name" requires some explanation. This is taken from the template definition and can/will be different from one template to the next. The value to be used in the URL can be found by either browsing data and simply copying it or by inspecting the template. In the layouts section there are a number of navigators, each with an id. It is this ID that should be used in the URL.

## What should the value for etag be?
When creating a new article (incident, accident, action etc.) you should use either empty string or string zero:

```json
{
    "etag": "",
    "etag": "0"
}
```

Either one will work just as well. For updates you must get the article from the API and then use the current etag. If you provide an old etag the API will respond with 409 - Conflict

## The API responds with 400 - Bad Request, how do I resolve the errors?
The response should include more information on what is missing or incorrect. The most common issue is with missing fields (such as title) or entity references. You might have to study the template to understand why an article has very specific requirements.

## The Open API specification says that an e_ or a_ field is required, do I need an ID in the 'values' array or can it be empty?

If the field is required you have to supply a valid Id for the given type. To remove the value (for non required field) - you have to set the array as empty, not remove the field all together.