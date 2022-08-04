```csharp

private const string _tokenUrl = "https://login.microsoftonline.com/{0}/oauth2/token";
private const string _baseUrl = "https://api.arcadiacloud.com";
private readonly HttpClient _client = new HttpClient();

public async Task<string> GetAccessToken(string tenantId, string clientId, string clientSecret)
{
	var tenantTokenUrl = string.Format(_tokenUrl, tenantId);
	var keyValues = new Dictionary<string, string>
	{
		["grant_type"] = "client_credentials",
		["client_id"] = clientId,
		["client_secret"] = clientSecret,
		["resource"] = "https://uxrisk.com/api"
	};
	var content = new FormUrlEncodedContent(keyValues);

	var response = await _client.PostAsync(tenantTokenUrl, content);
	var tokenResponse = await response.Content.ReadFromJsonAsync<TokenResponse>();
	return tokenResponse.AccessToken;
}

public async Task AdvancedAPI(string accessToken)
{
	_client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

	// get some articles of type assessment
	var someArticles = await _client.GetFromJsonAsync<ApiResult<Article>>($"{_baseUrl}/article/assessment", _opt);

	// get one article 
	var oneArticleUrl = $"{_baseUrl}/article/assessment/{someArticles.Results.Skip(1).First().Id}";
	var oneArticleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
	var oneArticle = oneArticleResults.Results.First();

	// add a property to the object and store it
	oneArticle.AdditionalProperties["newproperty"] = "some value";	
	var serialized = JsonSerializer.Serialize(oneArticle, _opt);
	var content = new StringContent(serialized, Encoding.UTF8, "application/json");
	var oneArticleStored = await _client.PutAsync(oneArticleUrl, content);

	// get the article again and verify that it has the new property
	oneArticleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
	oneArticle = oneArticleResults.Results.First();
	Console.WriteLine(oneArticle.AdditionalProperties["newproperty"]);

	// add new location (entity) reference to an article

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
}

private static JsonSerializerOptions _opt = new JsonSerializerOptions
{
	PropertyNamingPolicy = new LowerCaseNamingPolicy(),
	Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};

public class LowerCaseNamingPolicy : JsonNamingPolicy
{
	public override string ConvertName(string name) => name.ToLowerInvariant();
}

internal class Article
{
	public string Id { get; set; }
	public string Title { get; set; }
	public string ParentId { get; set; }
	public string ParentType { get; set; }
	public string Etag { get; set; }

	[JsonExtensionData]
	public Dictionary<string, object> AdditionalProperties { get; set; }
}

internal class Entity
{
	public string Id { get; set; }
	public string Name { get; set; }
	public string ParentId { get; set; }
	public string Fullpath { get; set; }
	public string Etag { get; set; }
}

internal class EntityReference
{
	public string Type { get; set; }
	public string[] Values { get; set; }
}

internal class EntityType
{
	public string Name { get; set; }
}

internal class ApiResult<T>
{
	public IEnumerable<T> Results { get; set; }
}

internal class TemplateDescription
{
	public string Id { get; set; }
	public string Name { get; set; }
}

internal class Template
{
	public Metadata Metadata { get; set; }
	public Layout Layout { get; set; }
}

internal class Metadata
{
	public string Id { get; set; }
	public string Name { get; set; }
	public string Description { get; set; }
	public string Category { get; set; }
}

internal class Layout
{
	public string Id { get; set; }
	public string Name { get; set; }
	public IEnumerable<Navigator> Navigators { get; set; }
}

internal class Navigator
{
	public string Id { get; set; }
	public string Label { get; set; }
	public Dictionary<string, object> Properties { get; set; }
}

internal class TokenResponse
{
	[JsonPropertyName("access_token")]
	public string AccessToken { get; set; }
}
```