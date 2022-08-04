# Advanced API <!-- omit in toc -->

Once you [have a valid access token](aquire_token.md) you can start calling our API. Below are some advanced examples of how to do so. Please read through the [Basic API](basic.md) part first, to get an understanding of how the API works.

[Complete source code](full_advanced_code.md) is available - please keep in mind that this is not production ready code. No logging, error handling, it makes a lot of assumptions etc. Please use is with caution.

## Get some articles

```csharp
var someArticles = await _client.GetFromJsonAsync<ApiResult<Article>>($"{_baseUrl}/article/assessment", _opt);
```

This will simply give you (upto) 25 articles of type `assessment`.

## Get one article

```csharp
var oneArticleUrl = $"{_baseUrl}/article/assessment/{someArticles.Results.First().Id}";
var oneArticleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
var oneArticle = oneArticleResults.Results.First();
```

This will give you the first article of type `assessment`.

## Update an article with new property

```csharp
oneArticle.AdditionalProperties["newproperty"] = "some value";	
var serialized = JsonSerializer.Serialize(oneArticle, _opt);
var content = new StringContent(serialized, Encoding.UTF8, "application/json");
var oneArticleStored = await _client.PutAsync(oneArticleUrl, content);
```

Any simple value (boolean, string, number, date) can be stored in an article. Please note that you should not exceed limits for property name length (40 chars) or number of properties (100) on a single object.

## Get article and new property value
```csharp
oneArticleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
oneArticle = oneArticleResults.Results.First();
Console.WriteLine(oneArticle.AdditionalProperties["newproperty"]);
```

## Add new entity (location) reference to article

```csharp
var locations = await _client.GetFromJsonAsync<ApiResult<Entity>>($"{_baseUrl}/entity/location", _opt);
var locationId = locations.Results.First().Id;
var locationReference = new EntityReference
{
    Type = "location",
    Values = new string[] { locationId}
};
oneArticle.AdditionalProperties["e_location_ids"] = locationReference;

serialized = JsonSerializer.Serialize(oneArticle, _opt);
content = new StringContent(serialized, Encoding.UTF8, "application/json");
oneArticleStored = await _client.PutAsync(oneArticleUrl, content);
```

Entity references refer to entity type and the id(s) of the entity. The example only adds a single reference but, depending on configuration, you are allowed to add more references.