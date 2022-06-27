# Frequently asked questions <!-- omit in toc -->


- [Why do some users have short GUID-like IDs and others have longer hash-like IDs?](#why-do-some-users-have-short-guid-like-ids-and-others-have-longer-hash-like-ids)
- [What is the correlation between a Users id and their Azure Active Directory Object ID?](#what-is-the-correlation-between-a-users-id-and-their-azure-active-directory-object-id)
- [What are the difference between an entity and an article?](#what-are-the-difference-between-an-entity-and-an-article)
- [What are the base required fields for an entity?](#what-are-the-base-required-fields-for-an-entity)
- [What is sys_sequence vs. sequence?](#what-is-sys_sequence-vs-sequence)
- [How do I generate a link to the web app?](#how-do-i-generate-a-link-to-the-web-app)
- [What should the value for etag be?](#what-should-the-value-for-etag-be)
- [The API responds with 400 - Bad Request, how do I resolve the errors?](#the-api-responds-with-400---bad-request-how-do-i-resolve-the-errors)
- [The Open API specification says that an e_ or a_ field is required, do I need an ID in the 'values' array or can it be empty?](#the-open-api-specification-says-that-an-e_-or-a_-field-is-required-do-i-need-an-id-in-the-values-array-or-can-it-be-empty)

## Why do some users have short GUID-like IDs and others have longer hash-like IDs?
Because...

## What is the correlation between a Users id and their Azure Active Directory Object ID?
Basically...

## What are the difference between an entity and an article?
bla bla...

## What are the base required fields for an entity?
All entities (locations, units, statuses etc.) requires:
<!-- no toc -->
- id
- name
- parentid ("0" for root)

Any other (non-nested) property is allowed, such as booleans and numbers.

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

## What is sys_sequence vs. sequence?
bla bla

## How do I generate a link to the web app?
https://.... 

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