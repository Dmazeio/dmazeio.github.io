## Welcome to UXRisk Dev Docs

UXRisk provides an extensive API to allow third party developers full access to all operations and data in UXRisk. For now, only 'Enterprise' customers can access the API, using Azure AD client id and secrets to obtain a JSON Web Token (jwt). 

To interact with our API you need to create an app registration in your Azure Active Directory. [Follow our documentation](app_reg.md) to complete the app registration.

Once the app is registered properly you can [aquire an access token](aquire_token.md) to call our API.

API Examples
 - [Basic Entity API calls](basic.md)
 - [Create an article](create_article.md)
 - [Query for articles](query_article.md)
 - [Other Article API calls](advanced.md)

[Full source code](full_source.md) is available, but please note that the code is meant for demonstration purpose only.

For troubleshooting we have an [Frequently Asked Questions](faq.md). If your question is not answered here, please reach out to us via [Support](https://www.uxrisk.com/contact) and we will get back to you as soon as possible.
