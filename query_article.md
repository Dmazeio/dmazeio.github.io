# Query for Articles <!-- omit in toc -->

- [Free-text query](#free-text-query)
- [Filtered query](#filtered-query)

## Free-text query

```csharp
var freetext = "first";
var queryUrl = $"{_baseUrl}/search/articles/assessment?query={freetext}";
var searchResults = await _client.GetFromJsonAsync<ApiResult<Article>>(queryUrl, _opt);
```

Very simple free-text query, will return list of all assessment articles where the word `first` is present. By default the API uses fuzzy matching for free-text queries, so even a typo will be "forgiven". Thie beahavior can be disabled by setting `fuzzyQueryEnabled` to false in the querystring.

## Filtered query

```csharp
	var draftStatusId = "208aada5-8efd-4f16-970a-c497c4398fb6";
	var filteredQueryUrl = $"{_baseUrl}/search/articles/assessment?assstat={draftStatusId}";
	var filteredSearchResults = await _client.GetFromJsonAsync<ApiResult<Article>>(filteredQueryUrl, _opt);
```

This example uses one filter (`assstat`) to only list assessments in draft state. Where does the `draftStatusId` value come from? And where does `assstat`  comes from? The draft status id can be found by getting all assessment statuses, see [Basic Entity API](basic.md), just replace `location` with `assessmentstatus` and inspect the `indraft` property of the entities. 

The query parameter `assstat` is much harder to find - this is only defined in the template description and there is no easy way to discover what it is (unfortunately). The easiest way to discover what the query parameter for different filters should be is to use the web application and look at the underlaying API calls... Other alternatives means downloading the template (see [Basic Entity API](basic.md)) or opening the Admin App and inspecting the template that way - the query parameter is defined on each field for each type, the template property is called `queryname`.
In the future this information _might_ be made available in the Open API specification.

It is also of course possible to combine free-text and filters in one request.