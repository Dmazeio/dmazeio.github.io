# Create Article <!-- omit in toc -->

- [Get UXRisk IDs](#get-uxrisk-ids)
- [Get templates](#get-templates)
- [Create new article](#create-new-article)
- [Retrieve created article](#retrieve-created-article)
  - [System Properties](#system-properties)
  - [Entity References](#entity-references)
  - [Other properties](#other-properties)

## Get UXRisk IDs

```csharp
var idResult = await _client.GetFromJsonAsync<ApiResult<IdResult>>($"{_baseUrl}/id", _opt);
var ids = new Queue<string>(idResult.Results.First().Ids);
```

First we get valid UXRisk IDs - these look like GUIDs, but should be treated as simple strings, the format can change without any further notice. By default the API will serve 20 Ids in a simple request.

## Get templates
```csharp
var allTemplates = await _client.GetFromJsonAsync<ApiResult<TemplateDescription>>($"{_baseUrl}/template");
```
We then need to get the template descriptions from the API - usually you would know the template ID to use for a specific purpose, but for the example we get all the template descriptions.

## Create new article
```csharp
var oneArticle = new Article
{
    Id = ids.Dequeue(),
    Etag = string.Empty,
    ParentId = "0",
    ParentType = string.Empty,
    Template = allTemplates.Results.First(r => r.Name.Equals("Risk Assessment")).Id,
    Title = "My first assessment"
};

var oneArticleUrl = $"{_baseUrl}/article/assessment/{oneArticle.Id}";
var serialized = JsonSerializer.Serialize(oneArticle, _opt);
var content = new StringContent(serialized, Encoding.UTF8, "application/json");
var oneArticleStored = await _client.PutAsync(oneArticleUrl, content);
```

We create a new article simply by populating our article object. The bare minimum is set here, specific templates might have other extended requirements. This can be understood by playing with the API, reading the Swagger docs or inspecting the template (source of truth).

## Retrieve created article
```csharp
var articleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
var retrievedArticle = articleResults.Results.First();
```

We can then retrieve the article from the API. Below is an example of what the article *might* look like - all depending on the template configuration.

```json
{
  "id": "43567394-5c57-461a-bb95-8cf3ab78fe20",
  "title": "My first assessment",
  "parentid": "0",
  "parenttype": null,
  "etag": "W/\"datetime'2022-08-04T12%3A37%3A01.1024158Z'\"",
  "sys_template": "34e673de-cd6b-438d-ae66-5d50d67c62c4",
  "timestamp": "2022-08-04T12:37:01.1024158+00:00",
  "e_phase_ids": {
    "type": "phase",
    "values": [
      "1f9313d7-005c-43a1-9bb1-c82dfb735b00",
      "6f8eed28-ab72-41cb-825f-68a97c96bd2a"
    ]
  },
  "e_projectowner_ids": {
    "type": "user",
    "values": [
      "a8a5f6b08788f68987..."
    ]
  },
  "e_status_ids": {
    "type": "assessmentstatus",
    "values": [
      "208aada5-8efd-4f16-970a-c497c4398fb6"
    ]
  },
  "isconfidential": false,
  "sys_createdby": "a8a5f6b08788f68987...",
  "sys_createdbydate": "2022-08-04T12:37:00.1532933Z",
  "sys_deleted": false,
  "sys_identifier": 10224,
  "sys_modifiedby": "a8a5f6b08788f68987...",
  "sys_modifieddate": "2022-08-04T12:37:00.1532933Z",
  "sys_sequence": 2710822201532933
}
```
Woha! A lot of properties have _magically_ appeared in the article? How did that happen? Let us go through them in groups:

### System Properties

All properties prefixed with `sys_` are considered system properties and are controlled by the API. Even if you attempt to set these the changes will not be persisted by the API. `timestamp` is also considered a system property and changes will not be persisted.

 - sys_createdby, sys_createdbydate - denotes who created the article and when
 - sys_modifiedby, sys_modifieddate - denotes who last modified the article and when
 - sys_sequence - is used for sorting purposes
 - sys_deleted - false by default, true if article is marked as deleted (soft-delete)
 - sys_identifier - a human readable stable identifier, only organization-wide unique

### Entity References

All properties prefixed with `e_` and postfixed with `_ids` are considered entity reference properties. These can be assigned by the client or by default values specified in the template. In this example three references has been set; phase, projectowner and status. This will vary by template, but `e_status_ids` is frequently set to make sure the article has a valid status - useful in search/query scenarios.

Entity references can refer to a single entity, like status, or multiple, like phase. What is allowed is defined in the template (SingleValue vs. MultiValue) - attempting to add multiple to a property defined to only have one will result in an error (`HTTP 400`).

### Other properties
`isconfidential` can be set - requires more if set to true - to mark an article as confidential. This is not covered in our API documentation.
