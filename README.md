# Getting-Started-with-OAuth-for-Adobe-IO-Solutions
*Get OAuth Credentials for Adobe I/O Solutions.*

These instructions provide the OAuth steps for obtaining credentials on the [Adobe I/O Console](https://console.adobe.io) or by using code to make API calls to Adobe Identity Management System (IMS) endpoints.

These instructions include the following sections:

1. [Basics of OAuth Authentication](#Basics)

1. [Setting Up the Environment](#Enviro)

1. [Obtaining Credentials on the Console](#Console)

1. [Obtaining Credentials with OAuth Code](#Code)

      * [Signing In](#Sign)
      * [Renewing Your Login](#Renew)
      * [Signing Out](#Out)

## <a name="Basics">Basics of OAuth Authentication</a>

OAuth allows end-users to sign in directly to the service they want to use from within the convenience of an intermediate application. The app acts between the user and the web service, but does not handle usernames and passwords. Instead, it is a web service that redirects the user to the website for login, and then provides a token to the application, allowing it to speak on behalf of the user.

## <a name="Enviro">Setting Up the Environment</a>

To test and integrate with the auth code method, you must create a secure (HTTPS) server. The auth code method redirects traffic from Adobe’s Sign in page back to your page only if it is hosted in a secure location. Also, if you prefer not to use front-end Ajax to communicate with the Adobe APIs, you will need server support to handle these requests. 

For basic testing, a simple option is to use Node.js or Python; otherwise, Apache provides a robust set of tools for web hosting. In each case, you will need to generate or purchase a public key certificate to complete the setup. 

## <a name="Console">Obtaining Credentials on the Console</a>

To access Adobe APIs, create an application key on the [Adobe I/O Console](https://console.adobe.io). Adobe I/O whitelists these keys and permits your application to access the APIs. It also issues and validates OAuth claims. 

To use the Adobe I/O Console:

1.	On the Adobe I/O Console, click **New Integration**.


1.	Select **Access an API** and then click **Continue**.

     ![access an api](https://user-images.githubusercontent.com/29133525/36869600-9ae7dcfe-1d59-11e8-8fe9-cfb79c7951f2.png)

1.	Select the Adobe product or service you want to use with Adobe I/O and then click **Continue**.
 
     ![adobe products](https://user-images.githubusercontent.com/29133525/36869786-33bde180-1d5a-11e8-841f-b8e004967fef.png)

1.	Click **New Integration** and then click **Continue**.

     ![new integration](https://user-images.githubusercontent.com/29133525/36869856-6f9eadba-1d5a-11e8-924a-afe32ce6f901.png)


1. For **Integration Details**, provide the following: 
  
      1. The **Name** of your application. This is not sent in your API requests, but it is good practice to use a web-friendly name suitable for later use in the X-product header.
  
      1. A **Description** of your integration, such as *Integration of Stock API with *MyWebsite.com.*
    
      1. **Platform**: Choose **iOS**, **Android**, or **Web**.

      1. A **Default redirect URI**: Provide the URL of the page or script (usually at the root of your web app) that Adobe will access during the authentication process. It must be hosted on a secure (HTTPS) server, even if it is only a localhost instance.

      *Note:  If you do not have this address yet, you can use any URL (e.g., `https://mysite.com/redirect.html`). Later, you will need to change it for your application to work.*
  
     5. A **Redirect URI pattern**: This is a URI path (or comma-separated list of paths) to which Adobe will attempt to redirect when the login flow is complete. It must be within your application domain, and is typically the root. You must escape periods (`.`) with `\\`, such as: `https://mysite\\.com/`.

    ![integration details](https://user-images.githubusercontent.com/29133525/36869926-9b3cc0b0-1d5a-11e8-8a47-1ca5326f92d8.png)


Once the integration is saved, the Adobe I/O Console generates several pieces of information you will need later. Copy everything in the **Client Credentials** section (including the **Client Secret**) and safeguard it like your private key in a secure location. 

## <a name="Code">Obtaining Credentials with OAuth Code</a>

### <a name="Sign">Signing In</a>

The sign in process begins when you click the login button in your client browser app. This calls an endpoint on the client server app that redirects to the IMS authorization endpoint. This notifies Adobe IMS to start the sign-in process. It is best if your app proxies the communication with IMS so that your front end does not expose any secure information.

### IMS URL Endpoint for Authorization

`https://ims-na1.adobelogin.com/ims/authorize `

### Parameters

| Parameter name | Description |
|----------------|-------------|
|`client_id`| API key obtained from Adobe I/O|
|`scope`|**openid**, **creative_sdk**|
|`redirect_uri`| Path that matches the redirect in the Adobe I/O integration|
|`response_type`| **code**|

### Sample Requests and Responses

**Request: Server script redirecting to IMS authorization endpoint**

```
GET /auth/signin HTTP/1.1 
Host: localhost:8443

HTTP/1.1 302 Found 
Location: https://ims-na1.adobelogin.com/ims/authorize 
?client_id=3a67c... 
&redirect_uri=https://localhost:8443/auth/token 
&scope=openid,creative_sdk 
&response_type=code 
```

**Request: IMS redirecting back to application**

```
GET /auth/token?code=eyJ4NXU...vkCnh9Q 
HTTP/1.1 
Host: localhost:8443
```

* Adobe IMS redirects to the Adobe Sign-in page, called “SUSI” (“Sign Up/Sign In”). Depending on the user’s email address, authentication is handled either by Adobe’s identity provider, or the Enterprise identity provider of the user’s parent organization. If the user has not granted permission, IMS first requests permission to allow the application to access the user’s information. The requesting app is the same one previously specified in the Adobe I/O Console. 

     ![sign in extra](https://user-images.githubusercontent.com/29133525/36870270-c056b9a4-1d5b-11e8-99a0-b9284a0c9edc.png)

 	 
* Upon successful sign-in, Adobe IMS redirects the browser back to the client redirect URI with an authorization code in the query string. 

* The redirect location works best if it is a server script and not a web page, since this code is part of the authentication sequence and should be kept secure. 

* Once the auth code is received, the server app sends a separate POST request to IMS, providing the API key, auth code and client secret (obtained earlier from Adobe I/O). The response includes an access token, a refresh token, and the user profile--as the workflow involves immediately getting the user profile once the user is signed in. 
 
### IMS URL Endpoint for Token Bearing

`https://ims-na1.adobelogin.com/ims/token`

### Parameters 

| Parameter name | Description |
|----------------|-------------|
| `grant_type` | **authorization_code** |
|`client_id`| API key obtained from Adobe I/O|
|`redirect_uri`| Path that matches the redirect in the Adobe I/O integration|
|`code`| Code sent by IMS to client redirect URI|



### Sample Requests and Responses

**Request: Send IMS POST access token**

```
POST /ims/token HTTP/1.1 
Host: ims-na1.adobelogin.com 
Content-Type: application/x-www-form-urlencoded 
grant_type=authorization_code 
&client_id=3a67c... 
&client_secret=12e7... 
&code=eyJ4NXU...vkCnh9Q 
```

**Response: IMS retruns access tokens, refresh tokens, and user profile**

```
HTTP/1.1 200 OK 
Content-Type: application/json;charset=UTF-8 
{ 
"access_token": "eyJ4NXU...qOk8-DA", 
"refresh_token": "eyJ4NXU...ZoQP_5A", 
"sub": "5BEB2BBC46CDB90599201549@AdobeID", 
"address": { 
"country": "US" 
}, 
"email_verified": "true", 
"name": "Adam Atomic", 
"token_type": "bearer", 
"given_name": "Adam", 
"expires_in": 86399985, 
"family_name": "Atomic", 
"email": "adam@atomcaps.com" 
} 
```
When the user is signed in, the server app notifies the front-end to show the signed-in state. From here, the app includes the access token with every API request made by the user. For example, the Member/Profile Adobe Stock request requires this access token, which is passed in the `Authorization: Bearer` header. 

**Request: Send Authenticated Member/Profile Request to Stock API**

```
GET /Rest/Libraries/1/Member/Profile?content_id=117487990& locale=en_US HTTP/1.1 
Host: stock-stage.adobe.io 
X-Product: IMSDemo 
x-api-key: 3a67c... 
**Authorization: Bearer eyJ4NXU...qOk8-DA**
```

**Response: Member/Profile Data**

```
{ 
"available_entitlement": { 
"quota": 85, 
"license_type_id": 15, 
"has_credit_model": true, 
"has_agency_model": false, 
"is_cce": true, 
"full_entitlement_quota": { 
"credits_quota": 75, 
"image_quota": 85 
} 
}, ... 
} 
```

### <a name="Renew">Renewing Your Login</a>

The auth code workflow includes a refresh token to renew your access periodically without needing to sign-in again. While the access token is designed to expire over a short amount of time (the default is 24 hours), the refresh token lasts for up to two weeks, by default. You can then request Adobe IMS to issue a new access token with a single API call automatically. 
To request a refresh token, you can use a similar process as requesting the original access token, except to change the value of the grant_type parameter and to replace code with a refresh_token parameter:

### IMS URL Endpoint for Renewing Login

`https://ims-na1.adobelogin.com/ims/token`

### Parameters

| Parameter name | Description |
|----------------|-------------|
| `grant_type` | `refresh_token`|
|`client_id`| API key obtained from Adobe I/O|
|`redirect_uri`| Path that matches the redirect in the Adobe I/O integration|
|`refresh_token`|	Original refresh token sent by IMS|

### Sample Request


**Request: Send IMS POST Refresh Token Request**

```
POST /ims/token HTTP/1.1 
Host: ims-na1.adobelogin.com 
Content-Type: application/x-www-form-urlencoded 
grant_type=refresh_token 
&client_id=3a67c... 
&client_secret=12e7... 
&refresh_token=eyJ4NXU... Ps1hKQug
```

### <a name="Out">Signing Out</a>

When a  user logs out of your app, the front-end redirects to a logout endpoint on your server app. This then calls the logout endpoint on IMS. Similar to the sign-in process, the best practice is to let the server app call IMS, since the access token must be passed as part of the request. 

### IMS URL Endpoint for Signing Out

`https://ims-na1.adobelogin.com/ims/logout`

### Parameters

| Parameter name | Description |
|----------------|-------------|
| `access_token` | Last access token obtained from login|
|`redirect_uri`| Path that matches the redirect in the Adobe I/O integration|

**Request: Redirect to IMS Logout Endpoint**

```
GET /auth/signout HTTP/1.1 
Host: localhost:8443 
HTTP/1.1 302 Found 
Location: https://ims-na1.adobelogin.com/ims/logout 
?access_token=eyJ4NXU...qOk8-DA 
&redirect_uri=https://localhost:8443/auth/token 
```

When the process is finished on the Adobe IMS side, IMS redirects the browser to the redirect URI. Your app can then notify the front end so that the UI is updated to show the signed-out state.

**Response: Logout from IMS**

```
GET /ims/logout_response 
?redirect_uri=https://localhost:8443/auth/token 
&client_id=3a67c... HTTP/1.1 
Host: ims-na1.adobelogin.com 
HTTP/1.1 302 Found 
Content-Type: text/html;charset=UTF-8 
Location: https://localhost:8443/auth/token
```

## Author
- John Wight [@johnwight](https://github.com/johnwight).


## Feedback?

Please help make this solution as useful as possible. If you find a problem in the documentation or have a suggestion, click the **Issues** tab on this GiHhub repository and then click the **New issue** button. Provide a title and description for your comment and then click the **Submit new issue** button.

   ![submit new issue](https://user-images.githubusercontent.com/29133525/32515298-f344bd5a-c3bc-11e7-9978-34516f964f9f.png)

