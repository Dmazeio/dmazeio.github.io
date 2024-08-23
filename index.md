## Welcome to Dmaze Dev Docs

Dmaze provides an extensive API to allow third party developers full access to all operations and data in Dmaze. There are two ways of authorizing as an integration:

### Microsoft Entra ID integration
Customers that have connection Microsoft Entra ID can access the API using a Microsoft Entra ID application to obtain a JSON Web Token (jwt). 

To interact with our API you either need to create an app registration in your Entra ID. [Follow our documentation](app_reg.md) to complete the app registration.

Once the app is registered properly you can [aquire an access token](aquire_token.md) to call our API.

### API Key
Through our admin interface you can create API keys to call our API with.

API Examples
 - [Basic Entity API calls](basic.md)
 - [Create an article](create_article.md)
 - [Query for articles](query_article.md)
 - [Other Article API calls](advanced.md)

[Full source code](full_source.md) is available, but please note that the code is meant for demonstration purpose only.

For troubleshooting we have an [Frequently Asked Questions](faq.md). If your question is not answered here, please reach out to us via [Support](https://www.Dmaze.com/contact) and we will get back to you as soon as possible.
