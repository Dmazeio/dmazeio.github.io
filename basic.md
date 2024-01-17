# Basic API <!-- omit in toc -->

Once you [have a valid access token](aquire_token.md) you can start calling our API. Below are some simple examples on how to do so.

[Complete source code](full_source.md) is available - please keep in mind that this is not production ready code. No logging, error handling, it makes a lot of assumptions etc. Please use is with caution.

- [Common API Response object](#common-api-response-object)
- [Get all templates](#get-all-templates)
- [Get a specific template](#get-a-specific-template)
- [Get Open API Specification (Swagger) for a template](#get-open-api-specification-swagger-for-a-template)
- [Get all entity types](#get-all-entity-types)
- [Get Entities](#get-entities)
- [Update an entity](#update-an-entity)

## Common API Response object

All the examples use a common API response class `ApiResult` - it is a simple wrapper for a list of requests entity/article. The JSON response that is deserialized into `ApiResult` looks like this: 
```json
{
	"results": [
		{
			"name": "South America",
			"etag": "W/\"datetime'2016-06-30T06%3A30%3A19.1494232Z'\"",			
			"parentid": "0",			
			"id": "bd46901a-390d-44f0-b098-e7f9170ff188",			
			"fullpath": "South America",
			...
		},
		{...}
	],
	"total": 6
}
```

All (almost!) API endpoints where you GET data will have a structure like this, where the data is in a `results` property.

## Get all templates

```csharp
var allTemplatesResponse = await _client.GetAsync($"{_baseUrl}/template");
var allTemplates = await allTemplatesResponse.Content.ReadFromJsonAsync<ApiResult<TemplateDescription>>();
```

This will produce a list of template descriptions. Id and Name are perhaps the most interesting properties of a template description. You need the Id in the next example.

## Get a specific template

```csharp
var firstTemplateDescription = allTemplates.Results.First();
var firstTemplateResponse = await _client.GetAsync($"{_baseUrl}/template/{firstTemplateDescription.Id}");
var firstTemplate = await firstTemplateResponse.Content.ReadFromJsonAsync<ApiResult<Template>>();
```

The template classes are quite extensive, not recommended to dig too deep into. 

## Get Open API Specification (Swagger) for a template

Given that each template in Dmaze can be quite different you can extract the Open API Specification document for each template. If you get the specification for 2 templates and compare them you will see a lot of overlap - this is to be expected since many endpoints are shared and/or common.

```csharp
var docResponse = await _client.GetAsync($"{_baseUrl}/doc/{firstTemplateDescription.Id}");
var docString = await docResponse.Content.ReadAsStringAsync();
```

This string can be pasted into the [editor](https://editor.swagger.io/) - you can then browse the entire API. 

## Get all entity types

```csharp
var entityTypesResponse = await _client.GetAsync($"{_baseUrl}/entity");
var entityTypes = await entityTypesResponse.Content.ReadFromJsonAsync<ApiResult<EntityType>>();
```

This will produce a list of all entity types, with just a name. This endpoint is just useful if you need to list entity types.

## Get Entities

Let's assume that we have an entity type called `location` and see if we can find some entities of that type.

```csharp
var entityName = entityTypes.Results.First(e => e.Name.Equals("location")).Name;
var entityResponse = await _client.GetAsync($"{_baseUrl}/entity/{entityName}");
var entities = await entityResponse.Content.ReadFromJsonAsync<ApiResult<Entity>>();

// the entity is a hierarchy so we can then get second level entities
// these all have parentid = one of the root entities
var firstEntity = entities.Results.First();
var secondLevelEntityResponse = await _client.GetAsync($"{_baseUrl}/entity/{entityName}?parentid={firstEntity.Id}");
var secondLevelEntities = await secondLevelEntityResponse.Content.ReadFromJsonAsync<ApiResult<Entity>>();
```

## Update an entity

```csharp
firstEntity.Name = $"This is a new name - it was: {firstEntity.Name}";
var serialized = JsonSerializer.Serialize(firstEntity, _opt);
var content = new StringContent(serialized, Encoding.UTF8, "application/json");
var entityUrl = $"{_baseUrl}/entity/{entityName}/{firstEntity.Id}";
var updateEntityResponse = await _client.PutAsync(entityUrl, content);
```

Key here is that we only deserialized some properties, see [complete source code](full_source.md) for all classes, and the API will accept this. Also we use the `etag` from the response when doing a `PUT` - otherwise it would fail our optimistic locking protection.