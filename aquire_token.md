# Aquire Access Token

An access token can be aquired through the use of libraries, such as [Microsoft Authentication Library (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) for any .net project. Other languages/frameworks have similar libraries. But it is also very easy to aquire the token manually using just simple HTTP request(s).

The code is essentially like this: 

```csharp

private readonly HttpClient _client = new HttpClient();
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

internal class TokenResponse
{
	[JsonPropertyName("access_token")]
	public string AccessToken { get; set; }
}

```

Inputs are from the [app registration](app_reg.md).

The resource value is the identifier for our API, it is not the URL.