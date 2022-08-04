```csharp
void Main()
{
	var tenantId = "***";
	var clientId = "***";
	var secret = "***";

	var token = GetAccessToken(tenantId, clientId, secret).GetAwaiter().GetResult();
	BasicAPI(token).GetAwaiter().GetResult();
	AdvancedAPI(token).GetAwaiter().GetResult();
	CreateArticle(token).GetAwaiter().GetResult();
	QueryForArticle(token).GetAwaiter().GetResult();
}

private const string _tokenUrl = "https://login.microsoftonline.com/{0}/oauth2/token";

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

private const string _baseUrl = "https://api.arcadiacloud.com";
private readonly HttpClient _client = new HttpClient();

public async Task BasicAPI(string accessToken)
{
	_client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

	// get all templates - list of template descriptions
	var allTemplatesResponse = await _client.GetAsync($"{_baseUrl}/template");
	var allTemplates = await allTemplatesResponse.Content.ReadFromJsonAsync<ApiResult<TemplateDescription>>();

	// get one of the templates - full translated template
	var firstTemplateDescription = allTemplates.Results.First();
	var firstTemplateResponse = await _client.GetAsync($"{_baseUrl}/template/{firstTemplateDescription.Id}");
	var firstTemplate = await firstTemplateResponse.Content.ReadFromJsonAsync<ApiResult<Template>>();

	var docResponse = await _client.GetAsync($"{_baseUrl}/doc/{firstTemplateDescription.Id}");
	var docString = await docResponse.Content.ReadAsStringAsync();
	docString.Dump();

	// get all entity types
	var entityTypesResponse = await _client.GetAsync($"{_baseUrl}/entity");
	var entityTypes = await entityTypesResponse.Content.ReadFromJsonAsync<ApiResult<EntityType>>();

	// get one of the types and get first page of entities of that type
	var entityName = entityTypes.Results.First(e => e.Name.Equals("location")).Name;
	var entityResponse = await _client.GetAsync($"{_baseUrl}/entity/{entityName}");
	var entities = await entityResponse.Content.ReadFromJsonAsync<ApiResult<Entity>>();

	// assuming that the entity was a hierarchy, such as location, we can then get second level entities
	// these all have parentid = one of the root entities
	var firstEntity = entities.Results.First();
	var secondLevelEntityResponse = await _client.GetAsync($"{_baseUrl}/entity/{entityName}?parentid={firstEntity.Id}");
	var secondLevelEntities = await secondLevelEntityResponse.Content.ReadFromJsonAsync<ApiResult<Entity>>();

	// lets update an entity
	firstEntity.Name = $"This is a new name - it was: {firstEntity.Name}";
	var serialized = JsonSerializer.Serialize(firstEntity, _opt);
	var content = new StringContent(serialized, Encoding.UTF8, "application/json");
	var entityUrl = $"{_baseUrl}/entity/{entityName}/{firstEntity.Id}";
	var updateEntityResponse = await _client.PutAsync(entityUrl, content);

	// lets get the entity again to verify the updated name
	var updatedEntity = await _client.GetFromJsonAsync<ApiResult<Entity>>($"{_baseUrl}/entity/{entityName}/{firstEntity.Id}");
}

public async Task CreateArticle(string accessToken)
{
	_client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

	// get some valid IDs
	var idResult = await _client.GetFromJsonAsync<ApiResult<IdResult>>($"{_baseUrl}/id", _opt);
	var ids = new Queue<string>(idResult.Results.First().Ids);

	// get all templates - list of template descriptions
	var allTemplates = await _client.GetFromJsonAsync<ApiResult<TemplateDescription>>($"{_baseUrl}/template");

	// create an article	
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

	// get one article 
	var articleResults = await _client.GetFromJsonAsync<ApiResult<Article>>(oneArticleUrl, _opt);
	var retrievedArticle = articleResults.Results.First();
	JsonSerializer.Serialize(retrievedArticle, _opt).Dump();
	
}

public async Task QueryForArticle(string accessToken) 
{
	_client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

	// free-text query
	var freetext = "first";
	var freetextQueryUrl = $"{_baseUrl}/search/articles/assessment?query={freetext}";
	var freetextSearchResults = await _client.GetFromJsonAsync<ApiResult<Article>>(freetextQueryUrl, _opt);

	// filtered query
	var draftStatusId = "208aada5-8efd-4f16-970a-c497c4398fb6";
	var filteredQueryUrl = $"{_baseUrl}/search/articles/assessment?assstat={draftStatusId}";
	var filteredSearchResults = await _client.GetFromJsonAsync<ApiResult<Article>>(filteredQueryUrl, _opt);

	freetextSearchResults.Dump();
	
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

	// get some valid IDs
	var idResult = await _client.GetFromJsonAsync<ApiResult<IdResult>>($"{_baseUrl}/id", _opt);
	var ids = new Queue<string>(idResult.Results.First().Ids);

	// get all templates - list of template descriptions
	var allTemplates = await _client.GetFromJsonAsync<ApiResult<TemplateDescription>>($"{_baseUrl}/template");
	
	// create an article	
	oneArticle = new Article
	{	
		Id = ids.Dequeue(),
		Etag = string.Empty,
		ParentId = "0",
		ParentType = string.Empty,
		Template = allTemplates.Results.First(r => r.Name.Equals("Risk Assessment")).Id,
		Title = "My first assessment"
	};

	var serialized = JsonSerializer.Serialize(oneArticle, _opt);
	var content = new StringContent(serialized, Encoding.UTF8, "application/json");
	var oneArticleStored = await _client.PutAsync(oneArticleUrl, content);

	// add a property to the object and store it
	oneArticle.AdditionalProperties["newproperty"] = "some value";	
	serialized = JsonSerializer.Serialize(oneArticle, _opt);
	content = new StringContent(serialized, Encoding.UTF8, "application/json");
	oneArticleStored = await _client.PutAsync(oneArticleUrl, content);

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
	
	[JsonPropertyName("sys_template")]
	public string Template { get; set; }

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

internal class IdResult
{
	public string[] Ids { get; set; }
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