# Platform specific walkthroughs

Try out a sample REST call in our [API Explorer](https://graphexplorer2.azurewebsites.net/).

Once you're done exploring the API, come back here and select your favorite platform on the left. We'll guide you through the steps to write a simple application to retrieve emails using the Microsoft Graph API.

If your preferred platform isn't listed yet, continue reading this page. We'll go through the same set of steps using raw HTTP requests.

## Getting started with the Microsoft Graph API and REST

The purpose of this guide is to walk through the process of calling the Microsoft Graph API to retrieve and send messages in Office 365 and Outlook.com. Unlike the platform-specific walkthroughs, this guide focuses on the OAuth and REST requests and responses. It will cover the sequence of requests and responses that an app uses to authenticate and retrieve messages.

## Using OAuth 2.0 to authenticate

In order to call the Microsoft Graph, your app needs an access token from Azure Active Directory (Azure AD). The app implements the [Authorization Code Grant flow](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx) to get the access tokens from Azure AD. The app must be registered to use authentication and authorization flows according to [OAuth 2.0 protocols](http://tools.ietf.org/html/rfc6749) for the Authorization Code Grant flow to work.

### Registering an app

There are currently two options to register your app.

  1. Register using the generally available model for an app that will interact with Office 365 commercial users, work, or school accounts.
 
  In order for your app to access a user's work or school account through the Microsoft Graph API, you first need to register it with Azure AD. Once you've registered your app, you can manage it through the [Azure Management Portal](https://manage.windowsazure.com).

  2. Experiment with the latest functionality and register app that works for both consumer and commercial services.
 
  A single authentication service for both work or school and personal accounts is now available in preview. This model provides a single authentication service for both work and school (Azure AD) as well as personal (Microsoft) identities. Now, you only need to implement one authentication flow in your app to enable users to use either work or school accounts, like Office 365 or OneDrive for Business, or personal accounts, like Outlook.com or OneDrive.
   
Use the  [Application Registration Portal](https://apps.dev.microsoft.com/) to register your app to implement converged auth (preview).

Please note that because converged auth is currently in preview and not ready for use in production apps. For more information on features of converged auth that are not yet supported in the public preview period, see [App model v2.0 preview: Limitations & restrictions](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

You will get a client ID and secret once you register the app. These values are used in the Authorization Code Grant flow.

The rest of this document assumes a registration on the generally available model.

### Getting an authorization code

The first step in the Authorization Code Grant flow is to get an authorization code. That code is returned to the app by Azure AD when the user logs in and consents to the level of access the app requires.

First, the app constructs a logon URL for the user. This URL must be opened in a browser so that the user can log in and provide consent.

The base URL for logon looks like `https://login.microsoftonline.com/common/oauth2/authorize`.

The app appends query parameters to this base URL to let Azure AD know what app is requesting the logon and what permissions it is requesting.

- `client_id` - The client ID generated by registering the app. This lets Azure AD know which app is requesting the logon.
- `redirect_uri` - The location that Azure will redirect to once the user has granted consent to the app. This value must correspond to the value of **Redirect URI** used when registering the app.
- `response_type` - The type of response the app is expecting. This value is `code` for the Authorization Code Grant flow.
- `resource` - The resource ID for the endpoint that the app will access. For Microsoft Graph, this is https://graph.microsoft.com.

For example, the request URL for our application that requires read access to mail would look like the following.

```http
GET https://login.microsoftonline.com/common/oauth2/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&resource=http%3A%2F%2Fgraph.microsoft.com&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

Next, redirect the user to the logon URL. The user will be presented with a sign in screen that displays the name of the app. After they sign in, the user will be presented with a list of the permissions the app requires and asked to allow or deny. Assuming they allow the required access, the browser will be redirected to the redirect URI specified in the initial request with the authorization code in the query string.

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&session_state=7B29111D-C220-4263-99AB-6F6E135D75EF&state=D79E5777-702E-4260-9A62-37F75FF22CCE
```

The next step is to exchange the authorization code returned by Azure AD for an access token.

### Getting an access token

To get an access token, the app posts form-encoded parameters to the token request URL (`https://login.microsoftonline.com/common/oauth2/token`) with the following parameters.

- `client_id`: The client ID generated by registering the app.
- `client_secret`: The client secret generated by registering the app.
- `code`: The authorization code obtained in the prior step.
- `redirect_uri`: This value must be the same as the value used in the authorization code request.
- `grant_type`: The type of grant the app is using. This value is `code` for the Authorization Code Grant flow.
- `resource`: The resource ID for the endpoint that the app will access. For Microsoft Graph, this is https://graph.microsoft.com.

The request URL for our application, using the code from the previous step, would look like the following.

```http
POST https://login.microsoftonline.com/common/oauth2/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

Azure AD responds with a JSON payload which includes the access token.

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

The access token is found in the `access_token` field of the JSON payload. The app uses this value to set the Authorization header when making REST calls to the API.

## Calling the Microsoft Graph

Once the app has an access token, it's ready to call the Microsoft Graph. Since this sample app is retrieving messages, it will use an HTTP GET request to the `https://graph.microsoft.com/v1.0/me/messages` endpoint.

### Refining the request

Apps can control the behavior of GET requests by using OData query parameters.  It is recommended that apps use these parameters to limit the number of results that are returned and to limit the fields that are returned for each item. 

Our sample app will display messages in a table that shows the subject, sender, and the date and time the message was received. The table displays a maximum of 25 rows and is sorted so that the most recently received message is at the top. The app uses the following query parameters to get these results.

- The `$select` parameter is used to specify only the `subject`, `sender`, and `dateTimeReceived` fields.
- The `$top` parameter is used to specify a maximum of 25 items.
- The `$orderby` parameter is used to sort the results by the `dateTimeReceived` field.

This results in the following request.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Now that you've seen how to make calls to the Microsoft Graph, you can use the API reference to construct any other kinds of calls your app needs to make. However, bear in mind that your app needs to have the appropriate permissions configured on the app registration for the calls it makes.

