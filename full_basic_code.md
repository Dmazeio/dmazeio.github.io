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

private static JsonSerializerOptions _opt = new JsonSerializerOptions
{
	PropertyNamingPolicy = new LowerCaseNamingPolicy(),
	Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};

public class LowerCaseNamingPolicy : JsonNamingPolicy
{
	public override string ConvertName(string name) => name.ToLowerInvariant();
}

internal class Entity
{
	public string Id { get; set; }
	public string Name { get; set; }
	public string ParentId { get; set; }
	public string Fullpath { get; set; }
	public string Etag { get; set; }
}

internal class EntityType
{
	public string Name { get; set; }
}

internal class ApiResult<T>
{
	public IEnumerable<T> Results { get; set; }
	public int Total  { get; set; }
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